# Implementando un servidor de recursos OAuth 2.

### Este capítulo cubre

- Implementando un servidor de recursos OAuth 2 de Spring Security
- Usando tokens JWT con claims personalizados
- Configurando la introspección para tokens opacos o revocación
- Implementando escenarios más complejos y multitenencia

Este capítulo trata sobre cómo asegurar una aplicación backend en un sistema OAuth 2. Lo que llamamos
servidor de recursos en la terminología OAuth 2 es simplemente un servicio backend. Mientras que en 
el capítulo 14 aprendiste cómo implementar la responsabilidad del servidor de autorización usando
Spring Security, ahora es momento de discutir cómo usar el token que el servidor de autorización 
genera.
En escenarios del mundo real, puede que implementes o no un servidor de autorización personalizado 
como lo hicimos en el capítulo 14. Tu organización podría usar una implementación de terceros en 
lugar de crear software personalizado. Puedes encontrar muchas alternativas disponibles, que van 
desde soluciones de código abierto como Keycloak hasta productos empresariales como Okta, Cognito o 
Azure AD. Un ejemplo con Keycloak está disponible en el capítulo 18 de la primera edición del libro.

Aunque tienes opciones para configurar un servidor de autorización sin necesidad de implementar el 
tuyo propio, deberás implementar la autenticación y autorización en tu backend correctamente. Por esa 
razón, creo que este capítulo es esencial; las habilidades que adquieres al leerlo tienen una alta 
probabilidad de ayudarte en tu trabajo. La figura 15.1 te recuerda sobre los actores de OAuth 2 y en
qué punto estamos con nuestro plan de aprendizaje para esta parte del libro.

![oauth2](images/chapter15/figure15.1.png)

En OAuth 2, el backend de la aplicación se denomina servidor de recursos porque protege los recursos
de los usuarios y clientes (datos y acciones que se pueden realizar sobre los datos).
Comenzaremos este capítulo en la sección 15.1 discutiendo la configuración del servidor de recursos 
para tokens web JSON (JWTs). Hoy en día encontrarás JWTs usados con mayor frecuencia en el sistema 
OAuth 2; por eso también comenzamos con ellos. En la sección 15.2, discutimos la personalización de 
JWTs y el uso de los valores personalizados en los claims del cuerpo o del encabezado.
En la sección 15.3, discutimos la configuración del servidor de recursos para usar la introspección 
para la validación de tokens. El proceso de introspección es útil cuando se usan tokens opacos o 
cuando deseas que tu sistema sea capaz de revocar tokens antes de su fecha de expiración.
Terminaremos el capítulo discutiendo casos de configuración más avanzados como la multitenencia en 
la sección 15.4.

## 15.1 Configurando la validación de JWT

En esta sección, discutimos la configuración de un servidor de recursos para validar y usar JWTs, que
son tokens no opacos (contienen datos que el servidor de recursos usa para la autorización). Para 
usar JWTs, el servidor de recursos necesitará probar que son auténticos, lo que significa que el 
servidor de autorización esperado los ha emitido efectivamente como prueba de autenticación de un 
usuario y/o un cliente. En segundo lugar, el servidor de recursos necesitará leer los datos en el 
token y usarlos para implementar las reglas de autorización.

Aprenderemos a configurar un servidor de recursos poniéndolo en acción, es decir, implementando uno
y configurándolo desde cero. Comenzaremos creando un nuevo proyecto Spring Boot y agregando las 
dependencias necesarias. Luego implementaremos un endpoint de demostración (un recurso para usar con
fines de prueba) y trabajaremos en la configuración para la autenticación y autorización. Estos son 
los pasos que seguiremos:

1. Agregar las dependencias necesarias al proyecto (en el archivo pom.xml ya que usamos Maven).
2. Declarar un endpoint ficticio que usaremos para probar nuestra implementación.
3. Implementar la autenticación para JWTs configurando el servicio con la URI del conjunto de claves
públicas.
4. Implementar las reglas de autorización.
5. Probar la implementación:
    a. Generando un token con el servidor de autorización.
    b. Usando el token para llamar al endpoint ficticio que creamos en el paso 2.

El siguiente codigo presenta las dependencias necesarias. Además de las dependencias web y de 
Spring Security, también agregaremos el iniciador del servidor de recursos.

Dependencies for implementing a resource server:
```xml
    El iniciador del servidor de recursos proporciona las dependencias necesarias para implementar una
    aplicación como servidor de recursos OAuth 2.
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-resource-server</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
Una vez que tenemos las dependencias en su lugar, creamos un endpoint ficticio que usaremos para 
probar nuestra implementación al final. El siguiente codigo presenta un controlador simple que expone 
un endpoint en la ruta /demo.

Declaring a simple endpoint for test purposes:
```java
@RestController
public class DemoController {
    //Definición del endpoint ficticio que necesitamos para probar nuestras configuraciones después
    // de finalizar nuestra implementación.
    @GetMapping("/demo")
    public String demo() {
        return "Demo";
    }
}
```
Para este ejemplo, necesitas usar un servidor de autorización. Puedes usar el que creamos en el 
capítulo 14 en el proyecto ssia-ch14-ex1.
Como queremos iniciar tanto el servidor de autorización como el servidor de recursos simultáneamente
en el mismo sistema, necesitaremos configurar diferentes puertos para ellos. Debido a que el servidor
de autorización tiene el puerto predeterminado 8080, podemos cambiar el puerto del servidor de 
recursos a otro. Lo cambié a 9090, pero puedes usar cualquier puerto libre en tu sistema. El 
siguiente fragmento de código muestra la propiedad que debes agregar a tu archivo 
application.properties para cambiar el puerto:

server.port=9090

Inicia tanto el servidor de autorización en el proyecto ssia-ch14-ex1 como la aplicación actual. 
Puedes encontrar este ejemplo en el proyecto ssia-ch15-ex1 con los proyectos que proporciona el libro.
Recuerda del capítulo 14 que un servidor de autorización OpenID Connect expone una URL que puedes 
usar para obtener su configuración (incluyendo la URL para autorización, token, conjunto de claves 
públicas y otros). El siguiente fragmento presenta la denominada URL well-known:

http://localhost:8080/.well-known/openid-configuration

Necesitas este enlace para obtener información sobre la URL que el servidor de autorización expone 
para proporcionar el conjunto de claves públicas que el servidor de recursos puede usar para validar
los tokens. El servidor de recursos necesita llamar a este endpoint y obtener el conjunto de claves 
públicas. Luego el servidor de recursos usa una de estas claves para validar la firma del token de 
acceso (siguiente figura).

![resource](images/chapter15/figure15.2.png)

El listado 15.3 te recuerda sobre la respuesta que obtienes al llamar al endpoint de configuración 
well-known que expone el servidor de autorización. Como puedes observar, la URI del conjunto de 
claves públicas se encuentra entre los demás datos proporcionados. La URI del conjunto de claves
públicas es lo que necesitamos configurar en el servidor de recursos para que pueda validar los JWTs.

Respuesta de la configuración OpenID well-known, que contiene la URI del conjunto de claves:
```json
{
  "issuer": "http://localhost:8080",
  "authorization_endpoint": "http://localhost:8080/oauth2/authorize",
  "device_authorization_endpoint":
  ➥"http://localhost:8080/oauth2/device_authorization",
  "token_endpoint": "http://localhost:8080/oauth2/token",
  …
  //El endpoint del conjunto de claves proporciona las partes públicas de los pares de claves 
  //asíncronas configuradas en el lado del servidor de autorización. El servidor de autorización
  //usa las partes privadas para firmar los tokens. El servidor de recursos puede usar las partes 
  //públicas para validarlos.
  "jwks_uri": "http://localhost:8080/oauth2/jwks",
  …
  
}
```
Para configurar la URI del conjunto de claves públicas, primero la declararemos en el archivo 
application.properties del proyecto. La clase de configuración puede inyectarla en un campo de 
atributo y luego usarla para configurar la autenticación del servidor de recursos:

keySetURI=http://localhost:8080/oauth2/jwks

El siguiente codigo muestra la clase de configuración inyectando el valor de la URI del conjunto de 
claves públicas en un atributo. La clase de configuración también define un bean del tipo 
SecurityFilterChain. La aplicación usará el bean SecurityFilterChain para configurar la autenticación,
de manera similar a lo que hicimos en los capítulos anteriores del libro.

Injecting the property value in the configuration class:
```java
@Configuration
public class ProjectConfig {
    //Inyecta el valor de la URI del conjunto de claves en un atributo de la clase de configuración.
    // Lo necesitarás para la configuración de la cadena de filtros.
    @Value("${keySetURI}")
    private String keySetUri;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {
        return http.build();
    }
}    
```
Para configurar la autenticación, usaremos el método oauth2ResourceServer() del objeto HttpSecurity.
Este método es similar a httpBasic() y formLogin(), que usamos en la segunda y tercera parte de este 
libro.
Al igual que httpBasic() y formLogin(), necesitas proporcionar una implementación de la interfaz 
Customizer para configurar la autenticación. En el listado 15.5, puedes observar cómo usé el método
jwt() del objeto Customizer para configurar la autenticación JWT. Luego usé un Customizer en el 
método jwt() para configurar la URI del conjunto de claves públicas (usando el método jwkSetUri()).

Configuring the authentication with JWTs:
```java
@Configuration
public class ProjectConfig {
    @Value("${keySetURI}")
    private String keySetUri;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {
        //Configurando la aplicación como un servidor de recursos OAuth 2.
        http.oauth2ResourceServer(
                //Configurando el servidor de recursos para usar JWTs para la autenticación.
                c -> c.jwt(
                 //Configurando la URL del conjunto de claves públicas que el servidor de recursos
                 // utilizará para validar los tokens.       
                        j -> j.jwkSetUri(keySetUri)
                )
        );
        return http.build();
    }
}    
```
Recuerda hacer que los endpoints requieran autenticación. De forma predeterminada, los endpoints no
están protegidos, por lo que para probar la autenticación, primero debes asegurarte de que tu 
endpoint /demo requiera autenticación. El siguiente fragmento de código configura las reglas de
autorización de la aplicación. Para este ejemplo, podemos configurar todos los endpoints para que 
requieran autenticación:
```java
http.authorizeHttpRequests(
    c -> c.anyRequest().authenticated()
);
```
El siguiente codigo presenta el contenido completo de la clase de configuración.

The full configuration class:
```java
@Configuration
public class ProjectConfig {
    @Value("${keySetURI}")
    private String keySetUri;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {
        http.oauth2ResourceServer(
                c -> c.jwt(
                        j -> j.jwkSetUri(keySetUri)
                    )
                );
                http.authorizeHttpRequests(
                        c -> c.anyRequest().authenticated()
                );

        return http.build();
    }
}
```
Ahora debes iniciar la aplicación del servidor de recursos que acabas de crear. Asegúrate de que tu 
servidor de autorización siga en funcionamiento. Necesitarás usar las habilidades que aprendiste en 
el capítulo 14 para generar un token de acceso. Recordemos los pasos para el tipo de concesión de 
código de autorización (pero recuerda que podrías obtener el token usando cualquier otro tipo de 
concesión — es irrelevante para el servidor de recursos cómo obtuviste el token de acceso siempre y 
cuando tengas uno).
Los pasos que debes seguir con el tipo de concesión de código de autorización son los siguientes 
(siguiente figura):

1. Redirigir al usuario para iniciar sesión en el endpoint /authorize del servidor de autorización.
2. Usar las credenciales del usuario para autenticarse. El servidor de autorización te redirigirá a la 
URI de redirección y proporcionará el código de autorización.
3. Tomar el código de autorización proporcionado después de la redirección y usar el endpoint /token 
para solicitar un nuevo token de acceso.

![authorization](images/chapter15/figure15.3.png)

El tipo de concesión de código de autorización. El cliente redirige al usuario a la página de inicio
de sesión del servidor de autorización. Después de que el usuario se autentica exitosamente, el 
servidor de autorización redirige de vuelta al cliente, proporcionando un código de autorización. 
El cliente usa el código de autorización para obtener un token de acceso.
El siguiente fragmento (justo después de los puntos) muestra la URL que puedes usar en tu navegador 
para redirigir al endpoint /authorize del servidor de autorización. Recuerda que necesitas 
proporcionar algunos parámetros, y sus valores deben cumplir con lo que configuraste en el servidor 
de autorización. Los parámetros que debes enviar son:

- response_type — Usa el valor "code" si deseas usar el tipo de concesión de código de autorización.
- client_id — El ID del cliente.
- scope — El alcance al que deseas acceder. Puede ser cualquiera de los alcances configurados dentro 
del servidor de autorización.
- redirect_uri — La URI a la cual el servidor de autorización redirige al cliente después de una 
autenticación exitosa. La URI de redirección debe ser una de las configuradas en el servidor de
autorización.
- code_challenge — Si se usa PKCE (prueba de clave para el intercambio de código), necesitas 
proporcionar el desafío de código del par de desafío y verificador de código.
- code_challenge_method — Si se usa PKCE, debes especificar la función hash que usaste para cifrar 
el verificador de código (por ejemplo, SHA-256):
```bash
http://localhost:8080/oauth2/authorize?response_type=code&client_
id=client&scope=openid&redirect_uri=https://www.manning.com/authorized&code_
challenge=QYPAZ5NU8yvtlQ9erXrUYR-T5AGCjCF47vN-KsaI2A8&code_challenge_
method=S256
```
`NOTA` Recuerda que debes pegar la URL de autorización en la barra de direcciones de un navegador 
para enviar la solicitud.
Inicia sesión usando las credenciales de usuario válidas configuradas en el servidor de autorización,
y luego espera a ser redirigido a la URI de redirección solicitada. El servidor de autorización 
proporciona el código de autorización que debes usar en la solicitud al endpoint /token.
El siguiente fragmento muestra un ejemplo de comando cURL que envía una solicitud al endpoint /token 
para obtener un token de acceso. Ten en cuenta que he truncado el valor del código de autorización 
para que quepa en la página:
```bash
curl -X POST 'http://localhost:8080/oauth2/token? \
client_id=client& \
redirect_uri=https://www.manning.com/authorized& \
grant_type=authorization_code& \
code=IhKRpq7GJ7P5VQI_...& \
code_verifier=qPsH306-ZDDaOE8DFzVn05TkN3ZZoVmI_6x4LsVglQI' \
--header 'Authorization: Basic Y2xpZW50OnNlY3JldA=='
```
El siguiente fragmento presenta un cuerpo de respuesta para la solicitud /token. También he truncado
los valores del token en el fragmento:
```json
{
  "access_token": "eyJraWQiOiI2Zjk5ZmE3MC…",
  "scope": "openid",
  "id_token": "eyJraWQiOiI2Zjk5ZmE3MC0xNTQ2LTRkMjM…",
  "token_type": "Bearer",
  "expires_in": 299
}
```
Ahora puedes usar el token de acceso al llamar a cualquier endpoint que requiera autenticación. El 
siguiente fragmento muestra un comando cURL para enviar una solicitud al endpoint /demo. Observa que
el token de acceso debe enviarse en el encabezado Authorization usando el prefijo Bearer 
(siguiente figura). El prefijo Bearer implica que quien tenga el valor del token de acceso puede 
usarlo de la misma manera que cualquier otra parte que lo tenga.

![presioso](images/chapter15/figure15.4.png)

Analogía a la novela El Señor de los Anillos de J.R.R. Tolkien. El token de acceso es un recurso 
preciado. Le otorga acceso a varios recursos a quien lo posea.

El siguiente fragmento muestra el comando cURL que puedes usar para enviar una solicitud al endpoint
/demo usando el token de acceso del servidor de autorización:
```bash
curl 'http://localhost:9090/demo' \
--header 'Authorization: Bearer eyJraW…'
```

## 15.2 Usando JWTs personalizados

Las necesidades de los sistemas son diferentes entre sí, incluso en lo que respecta a la autenticación
y autorización. A menudo sucede que necesitas transferir valores personalizados entre el servidor de
autorización y el servidor de recursos a través del token de acceso. El servidor de recursos puede 
usar dichos valores para aplicar diversas reglas de autorización.
En esta sección, implementaremos un ejemplo donde los servidores de autorización y de recursos usan 
claims personalizados en el token de acceso. El servidor de autorización personaliza el JWT agregando
un claim llamado "priority" en el JWT. El servidor de recursos lee el claim "priority" y agrega su 
valor a la instancia de autenticación en el contexto de seguridad. Desde allí, el servidor de recursos
puede usarlo al implementar cualquier regla de autorización.
Seguiremos estos pasos:

1. Cambiar el servidor de autorización para agregar el claim personalizado al token de acceso.
2. Cambiar el servidor de recursos para leer el claim personalizado y almacenarlo en el contexto de 
seguridad.
3. Implementar una regla de autorización que use el claim personalizado.

¡Pero primero lo primero! Necesitamos agregar un valor personalizado en el cuerpo del token de acceso
a la clase SecurityConfig. En el servidor de autorización, esto se hace agregando un bean de tipo 
OAuth2TokenCustomizer. El siguiente fragmento de código demuestra la definición de dicho bean. Para 
simplificar las cosas y permitirte enfocarte en el ejemplo, agregué un valor ficticio en un campo que
nombré "priority". En aplicaciones del mundo real, dichos campos personalizados tendrían un propósito,
y potencialmente tendrías que escribir cierta lógica para establecer su valor:
```java
@Bean
public OAuth2TokenCustomizer<JwtEncodingContext> jwtCustomizer() {
    return context -> {
        JwtClaimsSet.Builder claims = context.getClaims();
        claims.claim("priority", "HIGH");
    };
}
```
Con este mínimo cambio, los tokens de acceso ahora contienen un campo personalizado "priority". El 
siguiente fragmento muestra un token de acceso JWT que generé en su formato codificado en Base64, y
el siguiente codigo muestra el cuerpo decodificado donde puedes observar el campo "priority":

`eyJraWQiOiI5ZTBjOTQ5Ny0zYmMyLTQ4Y2YtODU5MC04N2JmZjE2ZjczOTAiLCJhbGciOiJSUzI
1NiJ9.eyJzdWIiOiJiaWxsIiwiYXVkIjoiY2xpZW50IiwibmJmIjoxNjg3MjYzMzI5LCJzY29wZ
SI6WyJvcGVuaWQiXSwiaXNzIjoiaHR0cDovL2xvY2FsaG9zdDo4MDgwIiwiZXhwIjoxNjg3MjYz
NjI5LCJwcmlvcml0eSI6IkhJR0giLCJpYXQiOjE2ODcyNjMzMjl9.HrQECSO17tZD8HKXP0U7gm
dmea01vPgVypvcf3oR3uawiMdI_joQBsLY0zNWBIgktKn2w9-
rvgtjD2xmhWZgSxRsDW_GZofqOzV9T-
5llMuZlakF7SQLyI67UJZKuPTJK8hBd1OhnurGo7ikPfDWhaqyychKu_uI7SdFrQQVgVqbrmHii
syoURIrI9EwOhB036M7UPJnIWtOWc34fAoFHxqhPuGIVesHHX5qm6wx-
8_Orjz96eOujVSEuUGRNVtz35_SRjhozcLzgIo3Rt9lUfLI7HSzulfXTCpxtxja-
1E_l_dsk4VHSvLYJUZjlERp5kVJqSO_keaJt8JbDQ0new`

El codigo muestra el cuerpo decodificado del token de acceso presentado anteriormente. Recuerda que 
puedes usar fácilmente la herramienta en línea jwt.io para obtener la forma decodificada de un JWT.
Alternativamente, puedes decodificar en Base64 individualmente el encabezado o el cuerpo del token 
de acceso usando cualquier otro decodificador Base64. El siguiente listado demuestra que nuestros 
cambios en el servidor de autorización funcionan correctamente.

El cuerpo decodificado en Base64 del token de acceso JWT personalizado:
```json
{
  "sub": "bill",
  "aud": "client",
  "nbf": 1687263329,
  "scope": [
    "openid"
  ],
  "iss": "http://localhost:8080",
  "exp": 1687263629,
  "priority": "HIGH", //El claim "priority" personalizado que agregamos al token de acceso.
  "iat": 1687263329
}
```
Como segundo paso, realizamos los cambios en el servidor de recursos. Puedes continuar trabajando en
el ejemplo que usamos en la sección 15.1, pero para facilitar tu aprendizaje, creé un proyecto 
separado para este ejemplo. Puedes encontrar la implementación que discutimos más adelante en esta 
sección en el proyecto ssia-ch15-ex2.
La lista de pasos que necesitamos seguir para que nuestro servidor de recursos comprenda los claims 
personalizados en el token de acceso es la siguiente:

1. Crear un objeto de autenticación personalizado. Este objeto definirá la nueva forma, incluyendo los 
datos personalizados.
2. Crear un objeto convertidor de autenticación JWT. Este objeto definirá la lógica para traducir el 
JWT en el objeto de autenticación personalizado.
3. Configurar el convertidor de autenticación JWT que creaste en el paso 2 para ser usado por el 
mecanismo de autenticación.
4. Cambiar el endpoint /demo para retornar el objeto de autenticación del contexto de seguridad.
5. Probar el endpoint y verificar que el objeto de autenticación contiene el campo personalizado 
"priority".

El siguiente codigo presenta la definición del objeto de autenticación. El objeto de autenticación 
debe ser cualquier clase que directa o indirectamente herede la clase AbstractAuthenticationToken. 
Dado que usamos JWTs, es más cómodo extender el más específico JwtAuthenticationToken. De esta manera,
extenderás directamente la forma habitual de un objeto de autenticación diseñado para tokens de acceso
JWT.
Observa que la personalización en el listado 15.8 agrega un campo llamado "priority". Este campo 
contendrá el valor del claim personalizado en el cuerpo del token de acceso. De manera similar, 
puedes agregar cualquier otro detalle personalizado que tu aplicación pueda necesitar para propósitos
de autorización. Tener estos detalles directamente en el objeto de autenticación del contexto de 
seguridad facilita la escritura de configuraciones independientemente de si elegimos aplicarlas a 
nivel de endpoint (capítulos 7 y 8) o a nivel de método (capítulos 11 y 12).

Defining a custom authentication object:
```java
public class CustomAuthentication
extends JwtAuthenticationToken { //Personalizando el objeto de autenticación extendiendo la clase JwtAuthenticationToken.
    //Agregando el campo personalizado "priority".
    private final String priority;

    public CustomAuthentication(
            Jwt jwt,
            Collection<? extends GrantedAuthority> authorities,
            String priority) {
        super(jwt, authorities);
        this.priority = priority;
    }

    public String getPriority() {
        return priority;
    }
}
```
Tienes una forma personalizada del objeto de autenticación, y lo siguiente que necesitas hacer es 
indicarle a tu aplicación cómo traducir el JWT en este objeto personalizado. Puedes hacer esto 
configurando un Converter específico, como se presenta en el listado 15.9. Observa los dos tipos 
genéricos que usamos: el Jwt y el CustomAuthentication. El primer tipo genérico, el Jwt, es la entrada
para el converter, mientras que el segundo tipo, el CustomAuthentication, es la salida. Por lo tanto,
este converter transforma un objeto Jwt (que es un contrato estándar en Spring Security sobre cómo se
lee el token de acceso JWT) en el tipo personalizado que implementamos en el listado 15.8 
(ver siguiente figura).

![custom converter](images/chapter15/figure15.5.png)

Un convertidor personalizado implementa la lógica para transferir información del token de acceso a 
una forma de autenticación personalizada.

Convirtiendo el token de acceso en un objeto de autenticación:
```java
@Component
public class JwtAuthenticationConverter
implements Converter<Jwt, CustomAuthentication> {
    @Override
    public CustomAuthentication convert(Jwt source) {
        List<GrantedAuthority> authorities =
                List.of(() -> "read");
        //Obteniendo los valores de prioridad del claim personalizado del token.
        String priority =
                String.valueOf(source.getClaims().get("priority"));
        return new CustomAuthentication(source,
                authorities,
                priority); //Estableciendo el valor de prioridad en el objeto de autenticación.
    }
}
```
También puedes observar en el siguiente codigo que he definido una autoridad ficticia. En un escenario
real, tomarías estas ya sea del token de acceso (considerando que son gestionadas a nivel del servidor
de autorización), o de una base de datos u otro sistema de terceros (considerando que son gestionadas
desde una perspectiva empresarial). En este caso, simplifiqué el ejemplo y agregué una autoridad
ficticia "read" para todas las solicitudes. Pero es importante recordar que este también es el lugar
donde tratarías con las autoridades (que también deberían terminar en el objeto de autenticación del
contexto de seguridad, ya que son detalles esenciales para las reglas de autorización en la mayoría 
de los casos).
El siguiente codigo muestra cómo configurar el convertidor personalizado. En este caso, usé la 
inyección de dependencias para obtener el bean del convertidor del contexto de Spring. Luego usé el 
método jwtAuthenticationConverter() del configurador de autenticación JWT.

Configurando el convertidor de autenticación personalizado:
```java
@Configuration
public class ProjectConfig {
    // omitted code
    //Inyectando el objeto convertidor en un campo de clase.
    private final JwtAuthenticationConverter converter;

    // omitted constructor
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {
        http.oauth2ResourceServer(
                c -> c.jwt(
                        j -> j.jwkSetUri(keySetUri)
                                .jwtAuthenticationConverter(converter) //Configurando el objeto convertidor dentro del mecanismo de autenticación.
                )
        );        
        http.authorizeHttpRequests(
                c -> c.anyRequest().authenticated()
        );

        return http.build();
    }
}
```
Eso es todo en cuanto a la configuración que necesitábamos implementar para usar el claim personalizado
del token de acceso. Probemos nuestra implementación y demostremos que funciona como se esperaba. El
siguiente fragmento de código muestra los cambios que hice al endpoint /demo. Hice que el endpoint 
/demo retorne la instancia de autenticación del contexto de seguridad. Debido a que Spring sabe cómo 
inyectar el valor automáticamente en un parámetro de tipo Authentication, solo necesité agregar este
parámetro y luego hacer que el método de acción del endpoint lo retorne tal cual:
```java
@GetMapping("/demo")
public Authentication demo(Authentication a) {
    return a;
}
```
Si todo funciona como se desea, al enviar una solicitud al endpoint /demo, obtendrás una respuesta 
con un cuerpo similar al presentado en el siguiente listado. Observa que el atributo personalizado 
"priority" aparece correctamente en el objeto de autenticación con el valor "HIGH".

La respuesta del endpoint /demo contiene el campo priority:
```json
{
  "authorities": [
    {
      "authority": "read"
    }
  ],
  "details": {
    "remoteAddress": "0:0:0:0:0:0:0:1",
    "sessionId": null
  },
  "authenticated": true,
  …
  "name": "bill",
  "priority": "HIGH" //El valor del claim personalizado aparece en la instancia de autenticación.
}
```

## 15.3 Configurando la validación de tokens mediante introspección

En esta sección, discutimos el uso de la introspección para la validación de tokens de acceso. Si tu
aplicación usa tokens opacos, o si deseas un sistema donde puedas revocar tokens a nivel del servidor
de autorización, entonces la introspección es el proceso que debes usar para la validación de tokens.
La siguiente figura te recordará el proceso de introspección, discutido en detalle en la sección 14.4.

![token introspection](images/chapter15/figure15.6.png)

Introspección de tokens. En situaciones donde el servidor de recursos no puede depender de la 
validación basada en firma de los tokens de acceso (como cuando es necesaria la revocación de tokens),
o cuando el token no contiene información detallada (como con los tokens opacos), el servidor de 
recursos debe realizar consultas al servidor de autorización. Esto se hace para determinar la validez
de un token y recopilar información adicional sobre él.

Implementaremos un servidor de recursos para demostrar el uso de la introspección. Para lograr nuestro
objetivo, debemos seguir estos pasos:

1. Asegurarse de que el servidor de autorización reconozca al servidor de recursos como un cliente. 
El servidor de recursos necesita credenciales de cliente registradas en el lado del servidor de 
autorización.
2. Configurar la autenticación en el lado del servidor de recursos para usar la introspección.
3. Obtener un token de acceso del servidor de autorización.
4. Usar un endpoint de demostración para probar que la configuración funciona de la manera que esperamos
con el token de acceso que obtuvimos en el paso 3.

El siguiente fragmento de código muestra un ejemplo de creación de una instancia de cliente, que 
registraremos en el lado del servidor de autorización. Este cliente representa nuestro servidor de 
recursos. Como observas en la figura anterior, el servidor de recursos envía solicitudes al servidor
de autorización (para la introspección), por lo que de esta manera también se convierte en un cliente
del servidor de autorización.
Para enviar las solicitudes de introspección, el servidor de recursos necesita credenciales de cliente
para autenticarse, de manera similar a cualquier otro cliente. Para este ejemplo, cambiaré el proyecto
ssia-ch14-ex4 que creamos al discutir los tokens opacos en el capítulo 14:
```java
RegisteredClient resourceServer =
    RegisteredClient.withId(UUID.randomUUID().toString())
        .clientId("resource_server")
        .clientSecret("resource_server_secret")
        .clientAuthenticationMethod(
            ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
        .authorizationGrantType(
            AuthorizationGrantType.CLIENT_CREDENTIALS)
        .build();
```
Recuerda que las contraseñas y los datos de configuración nunca deben estar codificados de forma fija
como lo hice en el fragmento anterior. Simplifiqué estos ejemplos tanto como fue posible para 
permitirte enfocarte en el tema que estamos discutiendo. En una aplicación del mundo real, debes 
colocar las configuraciones en archivos fuera de la implementación y persistir de forma segura los 
detalles secretos (como las credenciales) en algún lugar.
El siguiente listado muestra cómo agregar ambas instancias de detalles de cliente (la del cliente y
la del servidor de recursos) al componente RegisteredClientRepository del servidor de autorización.

The RegisteredClientRepository definition:
```java
@Bean
public RegisteredClientRepository registeredClientRepository() {
    RegisteredClient registeredClient = //Definiendo una instancia de detalles de cliente para la aplicación cliente.
            RegisteredClient.withId(UUID.randomUUID().toString())
                    .clientId("client")
                    .clientSecret("secret")
                    .clientAuthenticationMethod(
                            ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
                    .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
                    .tokenSettings(TokenSettings.builder()
                            .accessTokenFormat(OAuth2TokenFormat.REFERENCE)
                            .accessTokenTimeToLive(Duration.ofHours(12))
                            .build())
                    .scope("CUSTOM")
                    .build();
    RegisteredClient resourceServer =  //Definiendo una instancia de detalles de cliente para el servidor de recursos (que también se convierte en un cliente al llamar al endpoint de introspección).
            RegisteredClient.withId(UUID.randomUUID().toString())
                    .clientId("resource_server")
                    .clientSecret("resource_server_secret")
                    .clientAuthenticationMethod(
                            ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
                    .authorizationGrantType(
                            AuthorizationGrantType.CLIENT_CREDENTIALS)
                    .build();
    return new InMemoryRegisteredClientRepository(
            registeredClient, //Agregando ambas instancias de detalles de cliente en el repositorio del servidor de autorización.
            resourceServer);
}    
```
Con los cambios realizados en el codigo anterior, ahora tenemos un conjunto de credenciales que 
nuestro servidor de recursos puede usar para llamar al endpoint de introspección que expone el 
servidor de autorización. Podemos comenzar a implementar el servidor de recursos. Puedes encontrar 
este ejemplo en el proyecto ssia-ch15-ex3. El siguiente codigo muestra cómo, en los archivos de 
propiedades, configuré los tres valores esenciales necesarios para la introspección:

La URI de introspección que expone el servidor de autorización, que permite al servidor de recursos 
validar los tokens.
El ID de cliente del servidor de recursos que le permite identificarse a sí mismo al llamar al 
endpoint de introspección.
El secreto de cliente del servidor de recursos que este usa junto con su ID de cliente para 
autenticarse al enviar solicitudes al endpoint de introspección.

Junto con estos, también cambié el puerto del servidor a 9090, uno diferente al del servidor de 
aplicaciones (8080), permitiendo así que ambas aplicaciones se ejecuten simultáneamente.

El archivo application.properties del servidor de recursos:
```properties
# Cambiando el puerto del servidor de recursos para permitir que tanto el servidor de recursos como el servidor de autorización se ejecuten simultáneamente.
server.port=9090
# Configurando la URI de introspección como una propiedad.
introspectUri=httpg://localhost:8080/oauth/introspect
# Configurando el ID de cliente del servidor de recursos como una propiedad.
resourceserver.clientID=resource_server
# Configurando el secreto de cliente del servidor de recursos como una propiedad.
resourceserver.secret=resource_server_secret
```
Luego puedes inyectar los valores del archivo de propiedades en campos de la clase de configuración 
y usarlos para configurar la autenticación. El siguiente listado muestra la clase de configuración 
inyectando los valores del archivo de propiedades en campos.

Inyectando los valores en campos de la clase de configuración:
```java
@Configuration
public class ProjectConfig {
    //Inyectando la URI de introspección, el ID de cliente de introspección y el secreto de 
    // introspección del archivo de propiedades en campos de la clase de configuración.
    @Value("${introspectionUri}")
    private String introspectionUri;
    @Value("${resourceserver.clientID}")
    private String resourceServerClientID;
    @Value("${resourceserver.secret}")
    private String resourceServerSecret;
}
```
Usa la URI de introspección y las credenciales para configurar la autenticación. Configuras la 
autenticación de manera similar a como la configuramos para los tokens de acceso JWT, usando el método
oauth2ResourceServer() del objeto HttpSecurity. Sin embargo, llamamos a un método de configuración 
diferente del objeto customizer de oauth2ResourceServer(): opaqueToken(). Para el método opaqueToken(),
configuramos la URI de introspección y las credenciales. El siguiente listado presenta esta 
configuración.

Configurando la autenticación del servidor de recursos para tokens opacos:
```java
@Configuration
public class ProjectConfig {
    @Value("${introspectionUri}")
    private String introspectionUri;
    @Value("${resourceserver.clientID}")
    private String resourceServerClientID;
    @Value("${resourceserver.secret}")
    private String resourceServerSecret;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {
        http.oauth2ResourceServer(
                c -> c.opaqueToken( //Configurando la autenticación del servidor de recursos para tokens opacos.
                        o -> o.introspectionUri(introspectionUri) //Configurando la URI de introspección que el servidor de recursos debe usar para validar y obtener detalles sobre los tokens.
                                .introspectionClientCredentials( //Configurando las credenciales que el servidor de recursos debe usar para autenticarse al llamar a la URI de introspección del servidor de autorización.
                                        resourceServerClientID,
                                        resourceServerSecret)
                )
        );
        return http.build();
    }
}
```
Recuerda también agregar las configuraciones de autorización. El siguiente fragmento de código
muestra la forma estándar que aprendiste en los capítulos 7 y 8 para hacer que todos los endpoints 
requieran autenticación en las solicitudes:
```java
http.authorizeHttpRequests(
c -> c.anyRequest().authenticated()
);
```
El siguiente listado muestra el contenido completo de la clase de configuración.

Contenido completo de la clase de configuración:
```java
@Configuration
public class ProjectConfig {
    @Value("${introspectionUri}")
    private String introspectionUri;
    @Value("${resourceserver.clientID}")
    private String resourceServerClientID;
    @Value("${resourceserver.secret}")
    private String resourceServerSecret;

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {
        http.oauth2ResourceServer(
                c -> c.opaqueToken(
                        o -> o.introspectionUri(introspectionUri)
                                .introspectionClientCredentials(
                                        resourceServerClientID,
                                        resourceServerSecret)
                )
        );
        //Agregando la configuración de autorización del endpoint. Las solicitudes para cualquier 
        // endpoint requieren autenticación.
        http.authorizeHttpRequests(
                c -> c.anyRequest().authenticated()
        );
        return http.build();
    }
}
```
Un endpoint /demo simple como el del siguiente fragmento de código es suficiente para probar que la 
autenticación funciona correctamente:
```java
@RestController
public class DemoController {
    @GetMapping("/demo")
    public String demo() {
        return "Demo";
    }
}
```
Ahora puedes iniciar ambas aplicaciones: el servidor de autorización y el servidor de recursos. Ambas
deben ejecutarse simultáneamente. El siguiente fragmento de código incluye el comando cURL que puedes
usar para enviar una solicitud al endpoint /token. Para simplificar este ejemplo, uso el tipo de 
concesión de credenciales de cliente, pero podrías usar cualquier tipo de concesión que aprendiste 
en el capítulo 14 para obtener el token de acceso. Recuerda que la configuración del servidor de 
recursos es la misma independientemente de cómo obtengas el token de acceso:

```terminaloutput
curl -X POST 'http://localhost:8080/oauth2/token? \
client_id=client& \
grant_type=client_credentials' \
--header 'Authorization: Basic Y2xpZW50OnNlY3JldA=='
```
Si la solicitud es exitosa, obtendrás el token de acceso de vuelta en la respuesta. El cuerpo de la
respuesta se ve como el siguiente fragmento. He truncado el valor del token para que quepa mejor en 
la página:
```json
{
  "access_token": "2zLyYA8b6Q54-…",
  "token_type": "Bearer",
  "expires_in": 43199
}
```
De la misma manera que para los tokens de acceso JWT, al enviar una solicitud a un endpoint protegido,
agrega el token como el valor del encabezado "Authorization". El valor del token de acceso debe tener
como prefijo la cadena "Bearer". El siguiente fragmento muestra el comando cURL que puedes usar para
enviar una solicitud al endpoint /demo. Si todo funciona correctamente, obtendrás de vuelta la cadena
"Demo" en el cuerpo en una respuesta con estado 200 OK:
```terminaloutput
curl 'http://localhost:9090/demo' \
--header 'Authorization: Bearer 2zLyYA8b6Q54-…'
```

## 15.4 Implementando sistemas multitenencia

En las aplicaciones del mundo real, las cosas no siempre son perfectas. A veces nos encontramos en 
una situación en la que debemos adaptar nuestra implementación para que coincida con algún caso no 
estándar al integrarse con un tercero. Además, a veces necesitamos implementar backends que dependen
de múltiples servidores de autorización para la autenticación y autorización (sistemas multitenencia).
¿Cómo debemos implementar las configuraciones de nuestras aplicaciones en tales casos?
Afortunadamente, Spring Security ofrece flexibilidad para implementar cualquier escenario. En esta 
sección, discutimos la implementación de configuraciones del servidor de recursos para casos más 
complejos, como sistemas multitenencia o la interacción con aplicaciones que no siguen los estándares.
Echemos un vistazo a la siguiente figura para revisar el diseño de autenticación de Spring Security que 
discutimos en detalle en las dos primeras partes de este libro. Un filtro intercepta la solicitud 
HTTP. La responsabilidad de autenticación se delega entonces a un gestor de autenticación. El gestor
de autenticación utiliza además un proveedor de autenticación, que implementa la lógica de 
autenticación.

![authentication class](images/chapter15/figure15.7.png)

El diseño de la clase de autenticación. Durante el procedimiento de autenticación, el filtro captura
la solicitud y la pasa a un componente gestor de autenticación. Este gestor emplea entonces un 
proveedor de autenticación que ejecuta la lógica de autenticación necesaria. Tras una autenticación 
exitosa, la aplicación registra los detalles del principal autenticado en el contexto de seguridad.

¿Por qué es importante recordar este diseño? Porque para el servidor de recursos, como cualquier otro
enfoque de autenticación, necesitas cambiar el proveedor de autenticación si deseas personalizar cómo
funciona la autenticación.
En el caso de un servidor de recursos, Spring Security te permite conectar a la configuración un 
componente llamado resolvedor del gestor de autenticación (figura 15.8). Este componente permite que
la ejecución de la aplicación decida qué gestor de autenticación llamar. De esta manera, puedes 
delegar la autenticación a cualquier gestor de autenticación personalizado que pueda usar un proveedor
de autenticación personalizado.

![implementing](images/chapter15/figure15.8.png)

Al implementar un resolvedor del gestor de autenticación, le indicas a tu aplicación a qué gestor de
autenticación delegar la responsabilidad de autenticación.

Si deseas que tu aplicación use múltiples servidores de autorización que usen JWTs, Spring Security 
incluso proporciona una implementación del resolvedor del gestor de autenticación lista para usar 
(siguiente figura). Para tal caso, solo necesitas conectar la implementación personalizada 
JwtIssuerAuthenticationManagerResolver que proporciona Spring Security.

![system might](images/chapter15/figure15.9.png)

Tu sistema podría necesitar usar múltiples servidores de autorización para autenticar a los usuarios
y clientes.

El siguiente codigo muestra cómo usar el método authenticationManagerResolver() al configurar la 
autenticación. En este ejemplo, observas que solo tuve que crear una instancia de la clase 
JwtIssuerAuthenticationResolver, para la cual proporcioné todas las direcciones de emisión de los 
servidores de autorización. Puedes encontrar este ejemplo implementado en el proyecto ssia-ch15-ex4.

`NOTA` Recuerda nunca escribir URLs (o cualquier detalle configurable similar) directamente en el 
código. Usamos este enfoque solo en los ejemplos para simplificar el código y permitirte enfocarte 
en lo que es esencial para aprender. Todo lo que sea configurable siempre debe escribirse en archivos
de configuración o variables de entorno.

Trabajando con dos servidores de autorización que usan tokens de acceso JWT:
```java
@Configuration
public class ProjectConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {
        http.oauth2ResourceServer(
                j -> j.authenticationManagerResolver(
                        authenticationManagerResolver())
        );
        http.authorizeHttpRequests(
                c -> c.anyRequest().authenticated()
        );
        return http.build();
    }

    @Bean
    public AuthenticationManagerResolver<HttpServletRequest>
        authenticationManagerResolver() {
            return new JwtIssuerAuthenticationManagerResolver(
                    "http://localhost:7070",
                    "http://localhost:8080");
            
    }
}    
```
Con una configuración como la presentada en la siguiente figura, tu servidor de recursos funciona 
con dos servidores de autorización ejecutándose en los puertos 7070 y 8080.
Sin embargo, a veces las cosas son más complejas. Spring Security no puede proporcionar todas las 
personalizaciones posibles. En tal caso, donde necesitas personalizar aún más las capacidades del 
servidor de recursos, debes implementar tu propio resolvedor del gestor de autorización personalizado.
Consideremos el siguiente escenario: necesitas que tu servidor de recursos funcione tanto con tokens
JWT como con tokens opacos con dos servidores de autorización diferentes. Supongamos que tu servidor
de recursos discrimina las solicitudes en función del valor de un parámetro "type". Si el valor del 
parámetro "type" es "jwt", el servidor de recursos debe autenticar la solicitud con un servidor de 
autorización usando tokens de acceso JWT; de lo contrario, usa un servidor de autorización con tokens
de acceso opacos.

![employing](images/chapter15/figure15.10.png)

Empleando dos servidores de autorización distintos, cada uno manejando un tipo diferente de token. 
Dependiendo de un valor específico en un encabezado de solicitud HTTP utilizado por el cliente, el 
servidor de recursos determina qué servidor de autorización usar para validar los tokens de acceso.

El siguiente codigo implementa este escenario. El servidor de recursos usa un servidor de autorización
diferente basado en el valor del encabezado "type" en la solicitud HTTP. Para lograr esto, el servidor
de recursos usa un gestor de autenticación diferente basado en el valor de este encabezado.

Using both JWT and opaque tokens:
```java
@Configuration
public class ProjectConfig {
    // Omitted code
    @Bean
    public AuthenticationManagerResolver<HttpServletRequest>
        authenticationManagerResolver(
            JwtDecoder jwtDecoder,
            OpaqueTokenIntrospector opaqueTokenIntrospector
        ) {
        //Definiendo un gestor de autenticación para el servidor de autorización que gestiona tokens
        // de acceso JWT.
        AuthenticationManager jwtAuth = new ProviderManager(
                new JwtAuthenticationProvider(jwtDecoder)
        );
        //Definiendo otro gestor de autenticación para el servidor de autorización que gestiona tokens opacos.
        AuthenticationManager opaqueAuth = new ProviderManager(
                new OpaqueTokenAuthenticationProvider(opaqueTokenIntrospector)
        );
        //Definiendo la lógica del resolvedor del gestor de autenticación personalizado para 
        // seleccionar un gestor de autenticación basado en el encabezado "type" de la solicitud HTTP.
        return (request) -> {
            if ("jwt".equals(request.getHeader("type"))) {
                return jwtAuth;
            } else {
                return opaqueAuth;
            }
        };
    }
    @Bean
    public JwtDecoder jwtDecoder() {
        //Configurando la URI del conjunto de claves públicas para el gestor de autenticación que 
        // trabaja con el servidor de autorización que gestiona tokens de acceso JWT.
            return NimbusJwtDecoder
            .withJwkSetUri("http://localhost:7070/oauth2/jwks")
            .build();
    }
    @Bean
    //Configurando la URI de introspección y las credenciales para el gestor de autenticación que 
    // trabaja con el servidor de autorización que gestiona tokens opacos.
    public OpaqueTokenIntrospector opaqueTokenIntrospector() {
        return new SpringOpaqueTokenIntrospector(
                "http://localhost:6060/oauth2/introspect",
                "client", "secret");
    }
}
```
El siguiente codigo muestra el resto de la configuración que configura el resolvedor del gestor de 
autorización personalizado usando el parámetro customizer del método authenticationManagerResolver().

Configuring the AuthenticationManagerResolver:
```java
@Configuration
public class ProjectConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http)
            throws Exception {
        http.oauth2ResourceServer(
                j -> j.authenticationManagerResolver(
                        authenticationManagerResolver(
                                jwtDecoder(),
                                opaqueTokenIntrospector()
                        ))
        );
                http.authorizeHttpRequests(
                        c -> c.anyRequest().authenticated()
                );
        return http.build();
    }
// Omitted code
}
```
Incluso en este ejemplo, usamos implementaciones de proveedores de autenticación ofrecidas por Spring
Security: JwtAuthenticationProvider y OpaqueTokenAuthenticationProvider. En este caso, 
JwtAuthenticationProvider implementa la lógica de autenticación para trabajar con un servidor de 
autorización estándar que usa tokens de acceso JWT. OpaqueTokenAuthenticationProvider implementa la 
lógica de autenticación trabajando con tokens opacos. Pero podrías tener casos aún más complejos en 
aplicaciones del mundo real.
Si necesitas implementar algo muy personalizado, como la integración con un sistema que no sigue 
ningún estándar, entonces incluso puedes implementar tu propio proveedor de autenticación personalizado.

### Resumen

- Spring Security ofrece soporte para implementar servidores de recursos OAuth 2/OpenID Connect. Para
configurar la autenticación como un servidor de recursos OAuth 2/OpenID Connect, usa el método 
oauth2ResourceServer() del objeto HttpSecurity.
- Si deseas usar JWTs, necesitas aplicar la configuración usando el método jwt() del parámetro 
customizer de oauth2ResourceServer().
- También puedes usar la introspección si tu sistema utiliza tokens opacos o si deseas poder revocar
JWTs en el lado del servidor de autorización. En tal caso, debes configurar la autenticación usando 
el método opaqueToken() del parámetro customizer de oauth2ResourceServer().
- Cuando uses JWTs, debes configurar la URI del conjunto de claves públicas. La URI del conjunto de 
claves públicas es una URI expuesta por el servidor de autorización. El servidor de recursos llama a
esta URI para obtener las partes públicas de los pares de claves configurados en el lado del servidor
de autorización. El servidor de autorización usa las partes privadas para firmar los tokens de acceso,
mientras que el servidor de recursos necesita las partes públicas para validarlos.
- Cuando uses la introspección, necesitas configurar la URI de introspección. El servidor de recursos
envía solicitudes a la URI de introspección para preguntarle al servidor de autorización si un token
es válido y para obtener más detalles sobre él. Al llamar a la URI de introspección, el servidor de 
recursos actúa como un cliente del servidor de autorización, por lo que necesita sus propias 
credenciales de cliente para autenticarse.
- Spring Security ofrece la oportunidad de personalizar la lógica de autenticación usando un 
componente resolvedor del gestor de autenticación. Defines y configuras dicho componente personalizado
cuando debes implementar un caso más específico, como la multitenencia o la adaptación de tu 
aplicación a una implementación no estándar.