# Implementación de un servidor de autorización OAuth 2

### Este capítulo cubre 

- Implementación de un servidor de autorización OAuth 2 de Spring Security 
- Uso de los tipos de concesión de código de autorización y credenciales del cliente 
- Configuración de tokens de acceso opacos y no opacos ¡ Uso de revocación e introspección de tokens

El capítulo 13 trató sobre OAuth 2 y OpenID Connect. Discutimos los actores que intervienen en un 
sistema donde la autenticación y autorización se basan en la especificación OAuth 2. El servidor de 
autorización fue uno de estos actores. Su función consiste en autenticar a un usuario y a la 
aplicación que utiliza (el cliente), así como emitir tokens que sirven como prueba de autenticación
para acceder a recursos protegidos por un backend. A veces, el cliente lo hace en nombre del usuario.

El ecosistema Spring ofrece una forma completamente personalizable de implementar un servidor de 
autorización OAuth 2/OpenID Connect. El servidor de autorización de Spring Security es la forma 
estándar para implementar un servidor de autorización usando Spring en la actualidad. En este 
capítulo, revisaremos las principales capacidades que ofrece este framework e implementaremos un 
servidor de autorización personalizado. La siguiente figura sirve para recordarle los actores de OAuth 2 
y el papel del servidor de autorización discutido en el capítulo 13.

![actors](images/chapter14/figure14.1.png)

Actores en la escena de OAuth 2. El servidor de autorización protege los detalles del usuario y del
cliente y emite tokens que el cliente puede usar para obtener autorización al llamar a los endpoints
del servidor de recursos.

Comenzamos implementando un ejemplo sencillo en la sección 14.1, que utiliza las configuraciones 
predeterminadas. La configuración por defecto implica que el servidor de autorización emitirá tokens
no opacos. En la sección 14.2, demostramos que nuestra implementación funciona con el tipo de 
concesión de código de autorización, y luego, en la sección 14.3, mostramos también el tipo de 
concesión de credenciales del cliente. En la sección 14.4, continuamos configurando el servidor de 
autorización para trabajar con tokens opacos e introspección. Finalizamos la discusión de este 
capítulo en la sección 14.5 con la revocación de tokens.

Antes de comenzar, me gustaría informarle que la forma de implementar un servidor de autorización 
con Spring Security es completamente diferente a la de años anteriores. En este capítulo, discutimos
el nuevo enfoque, pero también podría necesitar saber cómo implementar un servidor de autorización 
de una manera más antigua (por ejemplo, si necesita trabajar en una aplicación existente que no ha 
sido actualizada). En ese caso, recomiendo leer el capítulo 13 de la primera edición del libro.

## 14.1 Implementación de autenticación básica utilizando tokens web JSON

En esta sección, empleamos un servidor de autorización OAuth 2 básico utilizando el framework Spring
Security Authorization Server. Revisaremos todos los componentes principales que necesitas incorporar
en la configuración para que funcione y los analizaremos individualmente. Luego probaremos la 
aplicación utilizando los dos tipos de concesión OAuth 2 más esenciales: el código de autorización y
el de credenciales del cliente. Encontrarás este ejemplo implementado en el proyecto ssia-ch14-ex1.

Los componentes principales que necesitas configurar para que tu servidor de autorización funcione 
correctamente son:

1. El filtro de configuración para los endpoints del protocolo: Te ayuda a definir configuraciones 
específicas para las capacidades del servidor de autorización, incluyendo diversas personalizaciones
(que discutiremos en la sección 14.3). 
2. El filtro de configuración de autenticación: Al igual que en cualquier aplicación web protegida 
con Spring Security, utilizarás este filtro para definir las configuraciones de autenticación y 
autorización, así como configuraciones para otros mecanismos de seguridad como el intercambio de 
recursos de origen cruzado (CORS) y la falsificación de solicitudes entre sitios (CSRF) (ver 
capítulos 2 al 10).
3. Los componentes de gestión de detalles del usuario: Como en cualquier proceso de autenticación 
implementado con Spring Security, estos se establecen mediante un bean UserDetailsService y un 
PasswordEncoder. Funcionan tal como se discutió en los capítulos 3 y 4.
4. La gestión de detalles del cliente: El servidor de autorización utiliza un componente llamado 
RegisteredClientRepository para gestionar las credenciales del cliente y otros detalles.
5. La gestión de pares de claves (utilizadas para firmar y validar tokens): Al usar tokens no opacos,
el servidor de autorización utiliza una clave privada para firmar los tokens. El servidor de 
autorización también ofrece acceso a una clave pública que el servidor de recursos puede usar para 
validar los tokens. El servidor de autorización gestiona los pares de claves pública-privada a través
de un componente denominado "fuente de claves".
6. La configuración general de la aplicación: Un componente llamado AuthorizationServerSettings te 
ayuda a configurar personalizaciones genéricas, como los endpoints que expone la aplicación.

La siguiente figura ilustra los componentes que debemos integrar y configurar para que funcione una 
aplicación de servidor de autorización mínima.

![components](images/chapter14/figure14.2.png)

Los componentes que necesitamos configurar e incorporar para que funcione un servidor de autorización
implementado con Spring Security.

Para comenzar, debemos agregar las dependencias necesarias a nuestro proyecto. En el siguiente 
fragmento de código, encontrarás las dependencias que debes agregar a tu archivo pom.xml:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-oauth2-authorization-server</artifactId>
</dependency>
```
Escribimos las configuraciones en clases estándar de configuración de Spring, como en el siguiente 
fragmento de código.
```java
@Configuration
public class SecurityConfig {
}
```
Recuerda que, como en cualquier otra aplicación Spring, los beans pueden definirse en múltiples clases
de configuración o mediante anotaciones de estereotipo (según el caso). Si necesitas repasar la 
gestión del contexto de Spring, recomiendo la primera parte de Spring Start Here (Manning, 2021), 
otro libro que escribí.

Veamos el listado 14.1, que presenta el filtro de configuración para los endpoints del protocolo. El 
método applyDefaultSecurity() es un método de utilidad que usamos para definir un conjunto mínimo de
configuraciones que puedes sobrescribir más adelante si es necesario. Después de llamar a este 
método, el listado muestra cómo habilitar el protocolo OpenID Connect utilizando el método oidc() 
del objeto configurador OAuth2AuthorizationServerConfigurer. Además, el filtro del siguiente codigo
especifica la página de autenticación a la que la aplicación debe redirigir al usuario cuando se le 
solicite iniciar sesión. Necesitamos esta configuración porque esperamos habilitar el tipo de 
concesión de código de autorización en nuestro ejemplo, lo que implica que los usuarios deben 
autenticarse. La ruta predeterminada en una aplicación web Spring es /login, por lo que a menos que 
configuremos una personalizada, usaremos esta para la configuración del servidor de autorización.

Implementing the filter for configuring protocol endpoints:
```java
@Bean
@Order(1)
public SecurityFilterChain asFilterChain(HttpSecurity http)
throws Exception {
    OAuth2AuthorizationServerConfiguration
            .applyDefaultSecurity(http);
    http.getConfigurer(
                    OAuth2AuthorizationServerConfigurer.class)
            .oidc(Customizer.withDefaults());
    http.exceptionHandling((e) ->
            e.authenticationEntryPoint(
                    //Especificación de la página de autenticación para usuarios
                    new LoginUrlAuthenticationEntryPoint("/login"))
    );
    return http.build();
}    
```
El siguiente codigo configura la autenticación y autorización. Estas configuraciones funcionan de 
forma similar a cualquier aplicación web (como se discutió en los capítulos 2 al 10). Se establesen 
las configuraciones mínimas:

1. Habilitar la autenticación mediante formulario de inicio de sesión, para que la aplicación muestre 
una página de login sencilla al usuario.
2. Especificar que la aplicación solo permite acceso a usuarios autenticados en cualquier endpoint.

Otras configuraciones que podrías incluir aquí, además de autenticación y autorización, podrían ser 
mecanismos de protección específicos como CSRF (discutido en el capítulo 9) o CORS (discutido en el 
capítulo 10).

Observe también la anotación @Order que usamos. Esta anotación es necesaria
porque tenemos múltiples instancias de SecurityFilterChain configuradas en el contexto de la 
aplicación, y necesitamos especificar el orden de prioridad en la configuración.

Implementing the filter for authorization configuration:
```java
@Bean
@Order(2) //Establecemos el filtro para que sea interpretado después del de los endpoints del protocolo.
public SecurityFilterChain defaultSecurityFilterChain(HttpSecurity http)
throws Exception {
    //Habilitamos el metodo de autenticación de inicio de sesión con formulario.
    http.formLogin(Customizer.withDefaults());
    //Configuramos todos los endpoints para que requieran autenticación.
    http.authorizeHttpRequests(
            c -> c.anyRequest().authenticated()
    );
}
```
Si esperas que los clientes utilicen el servidor de autorización que construyas para tipos de 
concesión que impliquen autenticación de usuario (como el tipo de concesión de código de 
autorización), entonces tu servidor necesita gestionar detalles de usuario. Afortunadamente, para 
implementar la gestión de detalles de usuario, puedes usar el mismo enfoque que aprendiste en los 
capítulos 3 y 4. Todo lo que necesitas es una implementación de UserDetailsService y PasswordEncoder.

El siguiente codigo presenta una definición para estos dos componentes. En este ejemplo, usamos una 
implementación en memoria para el UserDetailsService, pero recuerda que aprendiste cómo escribir una
implementación personalizada en el capítulo 3. En la mayoría de los casos, al igual que haces con 
otras aplicaciones web, almacenarás estos detalles en una base de datos. Por lo tanto, debes escribir
una implementación personalizada para el contrato UserDetailsService.

Además, recuerda que en el capítulo 4 discutimos que NoOpPasswordEncoder es algo que solo deberías 
usar en ejemplos de aprendizaje. NoOpPasswordEncoder no transforma las contraseñas de ninguna manera,
dejándolas en texto claro y al alcance de cualquiera que pueda acceder a ellas, lo cual no es 
adecuado. Siempre debes usar un codificador de contraseñas con una función hash fuerte, como BCrypt.

Defining the user details management:
```java
@Bean
public UserDetailsService userDetailsService() {
    UserDetails userDetails = User.withUsername("bill")
            .password("password")
            .roles("USER")
            .build();
    return new InMemoryUserDetailsManager(userDetails);
}
@Bean
public PasswordEncoder passwordEncoder() {
    return NoOpPasswordEncoder.getInstance();
}
```
El servidor de autorización necesita un componente RegisteredClientRepository para gestionar los
detalles del cliente. La interfaz RegisteredClientRepository funciona de manera similar a 
UserDetailsService, pero está diseñada para recuperar detalles del cliente. De forma análoga, el 
framework proporciona el objeto RegisteredClient, cuyo propósito es describir una aplicación cliente
que el servidor de autorización conoce.

Para hacer una analogía con lo aprendido en los capítulos 3 y 4, RegisteredClient es para clientes 
lo que UserDetails es para usuarios. De igual forma, RegisteredClientRepository funciona para los 
detalles del cliente tal como UserDetailsService funciona para los detalles del usuario (siguiente 
figura).

En este ejemplo, usaremos una implementación en memoria para permitirte centrarte en la implementación
general del servidor de autorización. Sin embargo, en una aplicación real, probablemente necesitarías
proporcionar una implementación de esta interfaz para obtener los datos desde una base de datos. 
Para ello, implementas la interfaz RegisteredClientRepository de forma similar a como implementaste
la interfaz UserDetailsService en el capítulo 3.

![manage client](images/chapter14/figure14.3.png)

Para gestionar los detalles del cliente, utilizamos una implementación de RegisteredClientRepository.
El RegisteredClientRepository utiliza objetos RegisteredClient para representar los detalles del 
cliente.

El siguiente codigo muestra la definición del bean RegisteredClientRepository en memoria. El método 
crea una instancia de RegisteredClient con los detalles requeridos y la almacena en memoria para ser
utilizada durante la autenticación por el servidor de autorización.

Implementing client details management:
```java
@Bean
public RegisteredClientRepository registeredClientRepository() {
    //Creando una instancia de RegisteredClient
    RegisteredClient registeredClient =
            RegisteredClient
                    .withId(UUID.randomUUID().toString())
                    .clientId("client")
                    .clientSecret("secret")
                    .clientAuthenticationMethod(
                            ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
                    .authorizationGrantType(
                            AuthorizationGrantType.AUTHORIZATION_CODE)
                    .redirectUri("https://www.manning.com/authorized")
                    .scope(OidcScopes.OPENID)
                    .build();
    return new InMemoryRegisteredClientRepository
            //Agregándolo para ser gestionado por la implementación RegisteredClientRepository en memoria.
            (registeredClient);
}
```
Los detalles que especificamos al crear la instancia RegisteredClient son los siguientes:

- Un ID interno único — Valor que identifica de manera única al cliente y solo tiene un propósito en 
los procesos internos de la aplicación.
- Un ID de cliente — Un identificador externo del cliente, similar a lo que es un nombre de usuario 
para el usuario.
- Un secreto de cliente — Similar a lo que es una contraseña para un usuario.
- El método de autenticación del cliente — Indica cómo el servidor de autorización espera que el 
cliente se autentique al enviar solicitudes de tokens de acceso.
- Tipo de concesión de autorización — Un tipo de concesión permitido por el servidor de autorización
para este cliente. Un cliente puede usar múltiples tipos de concesión.
- URI de redirección — Una de las direcciones URI que el servidor de autorización permite al cliente
solicitar una redirección para proporcionar el código de autorización en el caso del tipo de concesión
de código de autorización.
- Un alcance — Define un propósito para la solicitud de un token de acceso. El alcance puede usarse 
posteriormente en las reglas de autorización.

En este ejemplo, el cliente solo usa el tipo de concesión de código de autorización. Sin embargo, 
puedes tener clientes que usen múltiples tipos de concesión. En caso de que desees que un cliente 
pueda usar múltiples tipos de concesión, debes especificarlos como se presenta en el siguiente 
fragmento de código. El cliente definido aquí puede usar cualquier tipo de concesión (código de 
autorización, credenciales de cliente o el token de actualización):

```java
RegisteredClient registeredClient =
    RegisteredClient
        .withId(UUID.randomUUID().toString())
        .clientId("client")
        .clientSecret("secret")
        .clientAuthenticationMethod(
        ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
        .authorizationGrantType(
        AuthorizationGrantType.AUTHORIZATION_CODE)
        .authorizationGrantType(
        AuthorizationGrantType.CLIENT_CREDENTIALS)
        .authorizationGrantType(
        AuthorizationGrantType.REFRESH_TOKEN)
        .redirectUri("https://www.manning.com/authorized")
        .scope(OidcScopes.OPENID)
        .build();
```
De manera similar, llamando repetidamente al método redirectUri(), puedes especificar múltiples URIs
de redirección permitidas. De igual forma, un cliente también puede tener acceso a múltiples alcances.
En una aplicación del mundo real, la aplicación mantendría todos estos detalles en una base de datos
desde donde tu implementación personalizada de RegisteredClientRepository los recuperaría.
Además de tener los detalles del usuario y del cliente, debes configurar la gestión de pares de claves
si el servidor de autorización utiliza tokens no opacos (discutido en el capítulo 13). Para los tokens
no opacos, el servidor de autorización usa claves privadas para firmar los tokens y proporciona a los
clientes claves públicas que pueden usar para validar la autenticidad de los tokens.
El JWKSource es el objeto que proporciona la gestión de claves para el servidor de autorización de 
Spring Security. El listado 14.5 muestra cómo configurar un JWKSource en el contexto de la aplicación.
Para este ejemplo, creo un par de claves de manera programática y lo agrego al conjunto de claves que
el servidor de autorización puede usar. En una aplicación del mundo real, la aplicación leería las 
claves desde una ubicación donde estén almacenadas de forma segura (como una bóveda configurada en el
entorno).
Configurar un entorno que se asemeje perfectamente a un sistema del mundo real sería demasiado 
complejo, y prefiero que te enfoques en la implementación del servidor de autorización. Sin embargo,
recuerda que en una aplicación real, no tiene sentido generar nuevas claves cada vez que la aplicación
se reinicia (como en nuestro caso). Si eso ocurre en una aplicación real, cada vez que se realice un
nuevo despliegue, los tokens que ya fueron emitidos dejarán de funcionar (porque ya no pueden ser 
validados con las claves existentes).
Por lo tanto, para nuestro ejemplo, generar las claves de manera programática funciona y nos ayudará
a demostrar cómo funciona el servidor de autorización. En una aplicación del mundo real, debes 
mantener las claves protegidas en algún lugar y leerlas desde la ubicación indicada.

Implementing the key pair set management:
```java
//Generando un par de claves pública-privada de manera programática utilizando el algoritmo 
// criptográfico RSA.
@Bean
public JWKSource<SecurityContext> jwkSource()
throws NoSuchAlgorithmException {
    KeyPairGenerator keyPairGenerator =
            KeyPairGenerator.getInstance("RSA");
    keyPairGenerator.initialize(2048);
    KeyPair keyPair = keyPairGenerator.generateKeyPair();
    RSAPublicKey publicKey =
            (RSAPublicKey) keyPair.getPublic();
    RSAPrivateKey privateKey =
            (RSAPrivateKey) keyPair.getPrivate();
    RSAKey rsaKey = new RSAKey.Builder(publicKey)
            .privateKey(privateKey)
            .keyID(UUID.randomUUID().toString())
            .build();
    //Agregando el par de claves al conjunto que el servidor de autorización utiliza para firmar 
    // los tokens emitidos.
    JWKSet jwkSet = new JWKSet(rsaKey);
    //Envolviendo el conjunto de claves en una implementación JWKSource y retornándolo para ser 
    // agregado al contexto de Spring.
    return new ImmutableJWKSet<>(jwkSet);
}
```
Finalmente, el último componente que necesitamos agregar a nuestra configuración mínima es un objeto
AuthorizationServerSettings (el siguiente codigo). Este objeto te permite personalizar todas las rutas de 
los endpoints que el servidor de autorización expone. Si creas el objeto como se muestra en el 
siguiente listado, las rutas de los endpoints obtendrán algunos valores predeterminados que 
analizaremos más adelante en esta sección.

Configuring the authorization server generic settings:
```java
@Bean
public AuthorizationServerSettings authorizationServerSettings() {
    return AuthorizationServerSettings.builder().build();
}
```
Ahora podemos iniciar la aplicación y probar si funciona. En la sección 14.2, ejecutaremos el flujo
de código de autorización. Luego, en la sección 14.3, probaremos que el flujo de credenciales de 
cliente funciona como se espera con nuestra implementación de código de autorización.

## 14.2 Ejecutando el tipo de concesión de código de autorización

En esta sección, probamos el servidor de autorización implementado en la sección 14.1. Esperamos que,
utilizando los detalles del cliente registrado, seamos capaces de seguir el flujo de código de 
autorización y obtener un token de acceso. Seguiremos estos pasos:

1. Verificar los endpoints que el servidor de autorización expone
2. Usar el endpoint de autorización para obtener un código de autorización
3. Usar el código de autorización para obtener un token de acceso

El primer paso es encontrar las rutas de los endpoints que el servidor de autorización expone. Debido
a que no configuramos unos personalizados, debemos usar los predeterminados. ¿Pero cuáles son los 
predeterminados? Puedes llamar al endpoint de configuración OpenID en el siguiente fragmento para 
descubrir estos detalles. Esta solicitud usa el método HTTP GET y no se requiere autenticación:

`http://localhost:8080/.well-known/openid-configuration`

Al llamar al endpoint de configuración OpenID, deberías obtener una respuesta que se ve como la 
presentada en el siguiente codigo.

The response of the OpenID configuration request:
```java
{
        "issuer":"http://localhost:8080",
        "authorization_endpoint":
        "http://localhost:8080/oauth2/authorize",
        "token_endpoint":"http://localhost:8080/oauth2/token",
        "token_endpoint_auth_methods_supported":[
        "client_secret_basic",
        "client_secret_post",
        "client_secret_jwt",
        "private_key_jwt"
        ],
        //El endpoint del conjunto de claves que un servidor de recursos llamará para obtener las 
        // claves públicas que puede usar para validar los tokens.
        "jwks_uri":"http://localhost:8080/oauth2/jwks",
        "userinfo_endpoint":"http://localhost:8080/userinfo",
        "response_types_supported":[
        "code"
        ],
        "grant_types_supported":[
        "authorization_code",
        "client_credentials",
        "refresh_token"
        ],
        "revocation_endpoint":"http://localhost:8080/oauth2/revoke",
        "revocation_endpoint_auth_methods_supported":[
        "client_secret_basic",
        "client_secret_post",
        "client_secret_jwt",
        "private_key_jwt"
        ],
        "introspection_endpoint":
        //El endpoint de introspección que un servidor de recursos puede llamar para validar tokens opacos.
        "http://localhost:8080/oauth2/introspect",
        "introspection_endpoint_auth_methods_supported":[
        "client_secret_basic",
        "client_secret_post",
        "client_secret_jwt",
        "private_key_jwt"
        ],
        "subject_types_supported":[
        "public"
        ],
        "id_token_signing_alg_values_supported":[
        "RS256"
        ],
        "scopes_supported":[
        "openid"
        ]
}
```
Echemos un vistazo a la siguiete figura para recordar el flujo de código de autorización discutido en
el capítulo 13. Lo usaremos ahora para demostrar que el servidor de autorización que construimos
funciona correctamente.
Debido a que no tenemos un cliente para nuestro ejemplo, necesitamos actuar como uno. Ahora que 
conoces el endpoint de autorización, puedes colocarlo en la barra de direcciones de un navegador 
para simular cómo el cliente redirigiría al usuario hacia él. El siguiente fragmento muestra la 
solicitud de autorización:

```http request
http://localhost:8080/oauth2/authorize?
➥response_type=code&
➥client_id=client&
➥scope=openid&
➥redirect_uri=https://www.manning.com/authorized&
➥code_challenge=QYPAZ5NU8yvtlQ9erXrUYR-T5AGCjCF47vN-KsaI2A8&
➥code_challenge_method=S256
```
![authorization](images/chapter14/figure14.4.png)

El tipo de concesión de código de autorización. Tras una autenticación exitosa, el cliente recibe un
código de autorización. Este código es luego utilizado por el cliente para obtener un token de acceso,
el cual facilita el acceso a los recursos protegidos por el servidor de recursos.

Para la solicitud de autorización, puedes ver que agregué algunos parámetros:

- response_type=code — Este parámetro de solicitud especifica al servidor de autorización que el 
cliente desea usar el tipo de concesión de código de autorización. Recuerda que un cliente puede 
tener configurados múltiples tipos de concesión. Necesita indicarle al servidor de autorización qué 
tipo de concesión desea usar.

- client_id=client — El identificador del cliente es como el "nombre de usuario" para el usuario. 
Identifica de manera única al cliente en el sistema.

- scope=openid — Especifica qué alcance el cliente desea que se le otorgue con este intento de 
autenticación.

- redirect_uri=https://www.manning.com/authorized — Especifica la URI a la cual el servidor de 
autorización redirigirá tras una autenticación exitosa. Esta URI debe ser una de las previamente 
configuradas para el cliente actual.

- code_challenge=QYPAZ5NU8yvtlQ… — Si se usa el código de autorización mejorado con PKCE (discutido en
el capítulo 13), debes proporcionar el desafío de código con la solicitud de autorización. Al 
solicitar el token, el cliente debe enviar el par verificador para demostrar que es la misma 
aplicación que inicialmente envió esta solicitud. El flujo PKCE está habilitado de forma 
predeterminada.

- code_challenge_method=S256 — Este parámetro de solicitud especifica qué método de hash se ha 
utilizado para crear el desafío a partir del verificador. En este caso, S256 significa que SHA-256 
fue usado como función hash.

Recomiendo usar el tipo de concesión de código de autorización con PKCE, pero si realmente necesitas
deshabilitar la mejora PKCE del flujo, puedes hacerlo como se presenta en el siguiente fragmento de 
código. Observa el método clientSettings() que recibe una instancia de ClientSettings donde puedes 
especificar que deshabilitas la clave de prueba para el intercambio de código:

```java
RegisteredClient registeredClient = RegisteredClient
    .withId(UUID.randomUUID().toString())
    .clientId("client")
    // …
    .clientSettings(ClientSettings.builder()
    .requireProofKey(false)
    .build())
    .build();
```
En este ejemplo, demostramos el código de autorización con PKCE, que es la forma predeterminada y 
recomendada. Al enviar la solicitud de autorización a través de la barra de direcciones del navegador,
simulamos el paso 2 de la siguiente figura. El servidor de autorización nos redirigirá a su página 
de inicio de sesión, y podemos autenticarnos usando el nombre de usuario y la contraseña. Este es el
paso 3, se muestra la página de inicio de sesión que el servidor de autorización presenta al usuario.

![login page](images/chapter14/figure14.5.png)

La página de inicio de sesión que el servidor de autorización presenta al usuario en respuesta a la 
solicitud de autorización.

Para nuestra implementación, solo tenemos un usuario (ver listado 14.3). Sus credenciales son el 
nombre de usuario bill y la contraseña password. Una vez que el usuario ingresa las credenciales 
correctas y selecciona el botón Sign In, el servidor de autorización redirige al usuario a la URI de
redirección solicitada y proporciona un código de autorización (como se presenta en la figura; paso 
4).

![following](images/chapter14/figure14.6.png)

Tras una autenticación exitosa, el servidor de autorización guía al usuario hacia la URI de 
redirección especificada y emite un código de autorización. El cliente luego utiliza este código para
adquirir un token de acceso.

Una vez que el cliente tiene el código de autorización, puede solicitar un token de acceso. El 
cliente puede solicitar el token de acceso utilizando el endpoint de token. El siguiente fragmento 
muestra una solicitud cURL de un token. La solicitud usa el método HTTP POST. Debido a que 
especificamos que se requiere autenticación HTTP Basic al registrar el cliente, la solicitud de 
token necesita autenticación con HTTP Basic con el ID y el secreto del cliente:
```http request
curl -X POST 'http://localhost:8080/oauth2/token?
➥client_id=client&
➥redirect_uri=https://www.manning.com/authorized&
➥grant_type=authorization_code&
➥code=ao2oz47zdM0D5gbAqtZVB…
➥code_verifier=qPsH306-… \
--header 'Authorization: Basic Y2xpZW50OnNlY3JldA=='
```
Los parámetros de solicitud que utilizamos son:

- client_id=client — Necesario para identificar al cliente.
redirect_uri=https://www.manning.com/authorized — La URI de redirección a través de la cual el 
servidor de autorización proporcionó el código de autorización tras la autenticación exitosa del 
usuario.

- grant_type=authorization_code — Indica qué flujo usa el cliente para solicitar el token de acceso.

- code=ao2oz47zdM0D5… — El valor del código de autorización que el servidor de autorización proporcionó
al cliente.

- code_verifier=qPsH306-ZDD… — El verificador en base al cual se creó el desafío que el cliente envió 
en la autorización.

`NOTA` Presta mucha atención a todos los detalles. Si alguno de los valores no coincide correctamente
con lo que la aplicación conoce o con lo que se envió en la solicitud de autorización, la solicitud 
de token no tendrá éxito.
El siguiente fragmento muestra el cuerpo de la respuesta de la solicitud de token. Ahora el cliente 
tiene un token de acceso que puede usar para enviar solicitudes al servidor de recursos:
```json
{
  "access_token": "eyJraWQiOiI4ODlhNGFmO…",
  "scope": "openid",
  "id_token": "eyJraWQiOiI4ODlhNGFmOS1…",
  "token_type": "Bearer",
  "expires_in": 299
}
```
Debido a que habilitamos el protocolo OpenID Connect, por lo que no dependemos únicamente de OAuth 2,
un token de ID también está presente en la respuesta del token. Si el cliente hubiera sido registrado 
utilizando el tipo de concesión de token de actualización, un token de actualización también habría 
sido generado y enviado a través de la respuesta.

### Generando el verificador y el desafío de código

En el ejemplo en el que trabajamos en esta sección, usé el código de autorización con PKCE. En las 
solicitudes de autorización y token, utilicé un desafío y un valor verificador que había generado 
previamente. No presté demasiada atención a estos valores porque son responsabilidad del cliente y no 
algo que el servidor de autorización o de recursos genere. En una aplicación del mundo real, tu 
aplicación JavaScript o móvil tendrá que generar ambos valores cuando los use en el flujo OAuth 2.
Pero en caso de que te lo preguntes, explicaré cómo generé estos dos valores en este recuadro. Puedes
encontrar este ejemplo en el proyecto ssia-ch14-ex2.
El verificador de código es un dato aleatorio de 32 bytes. Para facilitar su transferencia a través 
de una solicitud HTTP, estos datos deben ser codificados en Base64 utilizando un codificador de URL 
y sin relleno. El siguiente fragmento de código muestra cómo hacerlo en Java:
```java
SecureRandom secureRandom = new SecureRandom();
byte [] code = new byte[32];
secureRandom.nextBytes(code);
String codeVerifier = Base64.getUrlEncoder()
.withoutPadding()
.encodeToString(code);
```
Una vez que tienes el verificador de código, usas una función hash para generar el desafío. El 
siguiente fragmento de código muestra cómo crear el desafío utilizando la función hash SHA-256. Al 
igual que con el verificador, necesitas usar Base64 para convertir el arreglo de bytes en un valor 
String, facilitando su transferencia a través de la solicitud HTTP:
```java
MessageDigest messageDigest = MessageDigest.getInstance("SHA-256");
byte [] digested = messageDigest.digest(verifier.getBytes());
String codeChallenge = Base64.getUrlEncoder()
.withoutPadding()
.encodeToString(digested);
```
Ahora tienes un verificador y un desafío. Puedes usarlos en las solicitudes de autorización y de 
token como se discutió en esta sección.

## 14.3 Ejecutando el tipo de concesión de credenciales de cliente

En esta sección, probaremos el tipo de concesión de credenciales de cliente utilizando el servidor de
autorización que implementamos en la sección 14.1. Recuerda que el tipo de concesión de cliente es un
flujo que permite al cliente obtener un token de acceso sin la autenticación ni el consentimiento de
un usuario. Preferiblemente, no deberías tener un cliente capaz de usar tanto un tipo de concesión 
dependiente del usuario (como el código de autorización) como uno independiente del usuario (como las
credenciales de cliente).
Como aprenderás en el capítulo 15, donde discutimos el servidor de recursos, la implementación de 
autorización podría no ser capaz de distinguir entre un token de acceso obtenido a través del tipo de
concesión de código de autorización y uno que el cliente obtuvo a través del tipo de concesión de 
credenciales de cliente. Por lo tanto, es mejor usar diferentes registros para tales casos y 
preferiblemente distinguir el uso del token a través de diferentes alcances.
El siguiente codigo se muestra un cliente registrado capaz de usar el tipo de concesión de credenciales de 
cliente. Observa que también he configurado un alcance diferente. En este caso, "CUSTOM" es 
simplemente un nombre que elegí; puedes elegir cualquier nombre para los alcances. El nombre que 
elijas generalmente debería hacer que el propósito del alcance sea más fácil de entender. Por ejemplo,
si esta aplicación necesita usar el tipo de concesión de credenciales de cliente para obtener un 
token para verificar el estado de actividad del servidor de recursos, entonces quizás sea mejor 
nombrar el alcance "LIVENESS" para que las cosas sean obvias.
Puedes encontrar el ejemplo discutido en esta sección en el proyecto ssia-ch14-ex3.

Configurando un cliente registrado para el tipo de concesión de credenciales de cliente:
```java
@Bean
public RegisteredClientRepository registeredClientRepository() {
    RegisteredClient registeredClient =
            RegisteredClient.withId(UUID.randomUUID().toString())
                    .clientId("client")
                    .clientSecret("secret")
                    .clientAuthenticationMethod(
                            //Permitiendo al cliente registrado usar el tipo de concesión de 
                            // credenciales de cliente.
                            ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
                    .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
                    //Configurando un alcance para que coincida con el propósito de la solicitud 
                    // del token de acceso.
                    .scope("CUSTOM")
                    .build();
    return new InMemoryRegisteredClientRepository(registeredClient);
}
```
muestra el flujo de credenciales de cliente que discutimos en el capítulo 13. Para obtener un token 
de acceso, el cliente simplemente envía una solicitud y se autentica usando sus credenciales (el ID 
y el secreto del cliente).

![client](images/chapter14/figure14.7.png)

El tipo de concesión de credenciales de cliente. Una aplicación puede obtener un token de acceso 
autenticándose únicamente con sus credenciales de cliente.

El siguiente fragmento muestra una solicitud cURL de token. Si lo comparas con la solicitud que 
usamos en la sección 14.2 cuando ejecutamos el tipo de concesión de código de autorización,
observarás que esta es más simple. El cliente solo necesita mencionar que usa el tipo de concesión 
de credenciales de cliente y el alcance en el que solicita un token. El cliente usa sus credenciales
con HTTP Basic en la solicitud para autenticarse:
```http request
curl -X POST 'http://localhost:8080/oauth2/token?
➥grant_type=client_credentials&
➥scope=CUSTOM' \
--header 'Authorization: Basic Y2xpZW50OnNlY3JldA=='
```
El siguiente fragmento muestra el cuerpo de la respuesta HTTP que contiene el token de acceso 
solicitado:
```json
{
  "access_token": "eyJraWQiOiI4N2E3YjJiNS…",
  "scope": "CUSTOM",
  "token_type": "Bearer",
  "expires_in": 300
}
```

## 14.4 Usando tokens opacos e introspección

Hasta ahora en este capítulo, hemos demostrado el tipo de concesión de código de autorización 
(sección 14.2) y el tipo de concesión de credenciales de cliente (sección 14.3). Con ambos, logramos
configurar clientes que pueden obtener tokens de acceso no opacos. Sin embargo, también puedes 
configurar fácilmente los clientes para usar tokens opacos. En esta sección, te mostraré cómo 
configurar los clientes registrados para obtener tokens opacos y cómo el servidor de autorización 
ayuda con la validación de los tokens opacos. Puedes encontrar el ejemplo discutido en esta sección 
en el proyecto ssia-ch14-ex4.

El siguiente codigo muestra cómo configurar un cliente registrado para usar tokens opacos. Recuerda que 
los tokens opacos pueden usarse con cualquier tipo de concesión. En esta sección, usaré el tipo de 
concesión de credenciales de cliente para mantener las cosas simples y permitirte enfocarte en el 
tema discutido. También puedes generar tokens opacos con el tipo de concesión de código de 
autorización.

Configuring clients to use opaque tokens:
```java
@Bean
public RegisteredClientRepository registeredClientRepository() {
    RegisteredClient registeredClient =
            RegisteredClient.withId(UUID.randomUUID().toString())
                    .clientId("client")
                    .clientSecret("secret")
                    .clientAuthenticationMethod(
                            ClientAuthenticationMethod.CLIENT_SECRET_BASIC)
                    .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
                    .tokenSettings(TokenSettings.builder()
                            //Configurando el cliente para usar tokens de acceso opacos.
                            .accessTokenFormat(OAuth2TokenFormat.REFERENCE)
                            .build())
                    .scope("CUSTOM")
                    .build();
    return new InMemoryRegisteredClientRepository(registeredClient);
}
```
Si solicitas un token de acceso como aprendiste a hacerlo en la sección 14.3, obtendrás un token 
opaco. Este token es más corto y no contiene datos. El siguiente fragmento es una solicitud cURL para
un token de acceso:
```http request
curl -X POST 'http://localhost:8080/oauth2/token?
➥grant_type=client_credentials&
➥scope=CUSTOM' \
--header 'Authorization: Basic Y2xpZW50OnNlY3JldA=='
```
El siguiente fragmento muestra la respuesta similar a la que obtuviste cuando esperábamos tokens no 
opacos. La única diferencia es el token en sí, que ya no es un token JWT, sino uno opaco:
```json
{
  "access_token": "iED8-...",
  "scope": "CUSTOM",
  "token_type": "Bearer",
  "expires_in": 299
}
```
El siguiente fragmento muestra un ejemplo de un token opaco completo. Observa que es mucho más corto
y no tiene la misma estructura que un JWT (faltan las tres partes separadas por puntos):
```http request
iED8-aUd5QLTfihDOTGUhKgKwzhJFzY
➥WnGdpNT2UZWO3VVDqtMONNdozq1
➥r9r7RiP0aNWgJipcEu5HecAJ75V
➥yNJyNuj-kaJvjpWL5Ns7Ndb7Uh6
➥DI6M1wMuUcUDEjJP
```
Como un token opaco no contiene datos, ¿cómo puede alguien validarlo y obtener más detalles sobre el
cliente (y potencialmente el usuario) para quien el servidor de autorización lo generó? La forma más
sencilla (y más utilizada) es preguntarle directamente al servidor de autorización. El servidor de 
autorización expone un endpoint donde se puede enviar una solicitud con el token. El servidor de 
autorización responde con los detalles necesarios sobre el token. Este proceso se llama introspección
(siguiente figura).

![token](images/chapter14/figure14.8.png)

Introspección de tokens. Cuando se usan tokens opacos, el servidor de recursos necesita enviar 
solicitudes al servidor de autorización para descubrir si el token es válido y obtener más detalles 
sobre a quién fue emitido.

El siguiente fragmento muestra la llamada cURL al endpoint de introspección que expone el servidor 
de autorización. El cliente debe usar HTTP Basic para autenticarse con sus credenciales al enviar la
solicitud. El cliente envía el token como parámetro de solicitud y recibe detalles sobre el token en
respuesta:
```http request
curl -X POST 'http://localhost:8080/oauth2/introspect?token=iED8-…' \
--header 'Authorization: Basic Y2xpZW50OnNlY3JldA=='
```
El siguiente fragmento muestra un ejemplo de una respuesta a la solicitud de introspección para un 
token válido. Cuando el token es válido, su estado aparece como "active", y la respuesta proporciona
todos los detalles que el servidor de autorización tiene sobre el token:
```json
{
  "active": true,
  "sub": "client",
  "aud": [
    "client"
  ],
  "nbf": 1682941720,
  "scope": "CUSTOM",
  "iss": "http://localhost:8080",
  "exp": 1682942020,
  "iat": 1682941720,
  "jti": "ff14b844-1627-4567-8657-bba04cac0370",
  "client_id": "client",
  "token_type": "Bearer"
}
```
Si el token no existe o ha expirado, su estado activo es false, como se muestra en el siguiente 
fragmento.
```json
{
  "active": false
}
```
El tiempo activo predeterminado para un token es de 300 segundos. En los ejemplos, preferirás hacer
la vida del token más larga. De lo contrario, no tendrás suficiente tiempo para usar el token en las
pruebas, lo que puede volverse frustrante. El listado 14.10 muestra cómo cambiar el tiempo de vida 
del token. Prefiero hacerlo muy grande para propósitos de ejemplo (como 12 horas en este caso), pero
recuerda nunca configurarlo tan grande para una aplicación del mundo real. En una aplicación real, 
normalmente irías con un tiempo de vida de 10 a 30 minutos como máximo.

Changing the access token time to live:
```java
RegisteredClient registeredClient = RegisteredClient
    .withId(UUID.randomUUID().toString())
    .clientId("client")
    // …
    .authorizationGrantType(AuthorizationGrantType.CLIENT_CREDENTIALS)
    .tokenSettings(TokenSettings.builder()
        .accessTokenFormat(OAuth2TokenFormat.REFERENCE)
        .accessTokenTimeToLive(Duration.ofHours(12))
        .build())
    .scope("CUSTOM")
    .build();
```

## 14.5 Revocando tokens

Supón que descubres que un token ha sido robado. ¿Cómo podrías hacer que un token sea inválido para 
su uso? La revocación de tokens es una forma de invalidar un token que el servidor de autorización 
emitió previamente. Normalmente, la vida útil de un token de acceso es corta, por lo que robar un 
token aún hace que sea difícil usarlo. Pero a veces quieres ser especialmente cauteloso.

El siguiente fragmento muestra un comando cURL que puedes usar para enviar una solicitud al endpoint
de revocación de tokens que expone el servidor de autorización. Puedes usar cualquiera de los 
proyectos en los que trabajamos en este capítulo para la prueba. La función de revocación está activa 
de forma predeterminada en un servidor de autorización de Spring Security. La solicitud solo requiere
el token que deseas revocar y autenticación HTTP Basic con las credenciales del cliente. Una vez que
envías la solicitud, el token ya no puede ser utilizado:

curl -X POST 'http://localhost:8080/oauth2/revoke?token=N7BruErWm-44-…'

--header 'Authorization: Basic Y2xpZW50OnNlY3JldA=='

Si usas el endpoint de introspección con un token que has revocado, deberías observar que el token 
ya no está activo después de la revocación (incluso si su tiempo de vida aún no ha expirado):

curl -X POST 'http://localhost:8080/oauth2/introspect?token=N7BruErWm-44-…'

--header 'Authorization: Basic Y2xpZW50OnNlY3JldA=='

Usar la revocación de tokens tiene sentido a veces, pero no es algo que siempre desearías. Recuerda 
que si quieres usar la función de revocación, esto también implica que necesitas usar introspección 
(incluso para tokens no opacos) con cada llamada para validar que el token sigue activo. Usar la 
introspección con tanta frecuencia podría tener un gran impacto en el rendimiento. Siempre debes 
preguntarte: ¿Realmente necesito esta capa de protección adicional?

Recuerda nuestra discusión en el primer capítulo. A veces, esconder la llave debajo del tapete es 
suficiente; otras veces, necesitas sistemas de alarma avanzados, complejos y costosos. Lo que uses 
depende de lo que proteges.

### Resumen

- El framework del servidor de autorización de Spring Security te ayuda a construir un servidor de 
autorización OAuth 2/OpenID Connect personalizado desde cero.

- Dado que el servidor de autorización gestiona los detalles del usuario y del cliente, debes 
implementar componentes que definan cómo la aplicación recopila estos datos:

    - Para gestionar los detalles del usuario, el servidor de autorización necesita un componente de Spring
    Security similar al de cualquier otra aplicación web: una implementación de UserDetailsService.

    - Para gestionar los detalles del cliente, el servidor de autorización proporciona otro contrato
  que debes implementar: el RegisteredClientRepository.


- Puedes registrar clientes que usen varios flujos de autenticación (tipos de concesión). 
Preferiblemente, el mismo cliente no debería usar tanto flujos dependientes del usuario (como el 
tipo de concesión de código de autorización) como flujos independientes del usuario (como el tipo de
concesión de credenciales de cliente).

- Cuando se usan tokens no opacos (generalmente JWTs), también debes configurar un componente para 
gestionar los pares de claves que el servidor de autorización usa para firmar los tokens. Este 
componente se denomina JWKSource.

- Cuando se usan tokens opacos (tokens que no contienen datos), el servidor de recursos debe usar el 
endpoint de introspección para verificar la validez de un token y recopilar los datos necesarios 
para la autorización.

- A veces, necesitarías una forma de invalidar tokens ya emitidos. El servidor de autorización ofrece
el endpoint de revocación para esta capacidad. Cuando se usa la revocación, el servidor de recursos 
siempre debe introspeccionar los tokens (incluso los no opacos) para verificar su validez.