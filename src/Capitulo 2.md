# Parte 2: Hello, Spring Security

`Este capítulo cubre`
- Crear tu primer proyecto con Spring Security
- Diseñar funcionalidades simples utilizando los componentes básicos de autenticación y autorización
- El concepto subyacente y cómo usarlo en un proyecto determinado
- Aplicar los contratos básicos y comprender cómo están relacionados
- Escribir implementaciones personalizadas para responsabilidades principales
- Anular las configuraciones predeterminadas de Spring Boot

Spring Boot apareció como una etapa evolutiva para el desarrollo de aplicaciones con el Framework 
Spring. Al eliminar la necesidad de escribir todas las configuraciones, Spring Boot incluye 
configuraciones predefinidas, de modo que solo puedes anular aquellas que no coinciden con tus 
implementaciones. También llamamos a este enfoque convención sobre configuración. Spring Boot ya no 
es un concepto nuevo, y hoy disfrutamos escribiendo aplicaciones utilizando su tercera versión.

Antes de Spring Boot, los desarrolladores solían escribir docenas de líneas de código repetidamente 
para todas las aplicaciones que tenían que crear. Esta situación era menos evidente en el pasado, 
cuando la mayoría de las arquitecturas se desarrollaban de forma monolítica. Con una arquitectura 
monolítica, solo era necesario escribir esas configuraciones una vez al principio, y rara vez había 
que modificarlas después. Con la evolución de las arquitecturas de software orientadas a servicios, 
comenzamos a notar el problema del código repetitivo que había que escribir al configurar cada 
servicio. Si te parece interesante, puedes consultar el capítulo 3 de Spring in Practice de Willie 
Wheeler con Joshua White (Manning, 2013). Este capítulo describe la creación de una aplicación web 
con Spring 3, lo que te permitirá entender cuántas configuraciones había que escribir para una 
pequeña aplicación web de una sola página. El capítulo está disponible en http://mng.bz/46la.

Por esta razón, con el desarrollo de aplicaciones recientes, especialmente aquellas para 
microservicios, Spring Boot se volvió cada vez más popular. Spring Boot proporciona autoconfiguración 
para tu proyecto y reduce el tiempo necesario para la configuración. Podemos decir que viene con la 
filosofía adecuada para el desarrollo de software actual.

En este capítulo, comenzaremos con nuestra primera aplicación que utiliza Spring Security. Para las 
aplicaciones que desarrolles con el Framework Spring, Spring Security es una excelente opción para 
implementar seguridad a nivel de aplicación. Usaremos Spring Boot y analizaremos las configuraciones
predeterminadas establecidas por convención, con una breve introducción sobre cómo anular estos 
valores predeterminados. Considerar las configuraciones por defecto ofrece una excelente introducción 
a Spring Security, que también ilustra el concepto de autenticación.

Una vez que comencemos con el primer proyecto, discutiremos en mayor detalle las diversas opciones 
de autenticación. En los capítulos 3 al 6, continuaremos con configuraciones más específicas para 
cada una de las responsabilidades que verás en este primer ejemplo. También verás diferentes formas 
de aplicar esas configuraciones, dependiendo de los estilos arquitectónicos. Los pasos que 
discutiremos en el capítulo actual son los siguientes:

1. Crea un proyecto con solo las dependencias de Spring Security y web para observar su 
comportamiento cuando no se agrega ninguna configuración. De esta forma, comprenderás qué esperar de
la configuración predeterminada para autenticación y autorización.
2. Modifica el proyecto para añadir funcionalidad de gestión de usuarios, anulando las configuraciones
predeterminadas para definir usuarios y contraseñas personalizados.
3. Después de observar que la aplicación autentica todos los endpoints por defecto, aprende que esto
también se puede personalizar.
4. Aplica diferentes estilos para las mismas configuraciones con el fin de entender las mejores 
prácticas.

### 2.1 Comenzando tu primer proyecto
Creemos el primer proyecto para tener un ejemplo inicial. Este proyecto es una pequeña aplicación 
web que expone un endpoint REST. Verás cómo, sin hacer mucho, Spring Security protege este endpoint 
utilizando autenticación HTTP Basic. HTTP Basic es un mecanismo mediante el cual una aplicación web 
autentica a un usuario a través de un conjunto de credenciales (nombre de usuario y contraseña) que 
la aplicación obtiene del encabezado de la solicitud HTTP.

`NOTA` Con la configuración predeterminada, la aplicación tiene dos mecanismos de autenticación 
diferentes implementados: HTTP Basic y Form Login. Sin embargo, decidí presentar el ejemplo paso a 
paso y tratar el inicio de sesión mediante formulario (Form Login) en capítulos posteriores. Pero si
intentas acceder a la URL desde un navegador, verás que tu aplicación implementa un formulario 
elegante para la autenticación del usuario y no muestra un cuadro HTTP Basic poco atractivo por esta
razón. No quiero que te confundas en caso de que decidas hacer pruebas con un navegador, pero nos 
centraremos en esto en la sección sobre HTTP Basic. 

Tan solo con crear el proyecto y agregar las dependencias correctas, Spring Boot aplica 
configuraciones predeterminadas, incluyendo un nombre de usuario y una contraseña, cuando se inicia 
la aplicación.

`NOTA` Tienes varias alternativas para crear proyectos Spring Boot. Algunos entornos de desarrollo 
ofrecen la posibilidad de crear el proyecto directamente. Para más detalles, recomiendo Spring Boot:
Up and Running de Mark Heckler (O’Reilly Media, 2021) y Spring Boot in Practice (Manning, 2022) de 
Somnath Musib, o incluso Spring Start Here (Manning, 2021), otro libro que escribí.

Los ejemplos en este libro hacen referencia al código fuente complementario del libro. Con cada 
ejemplo, también se especifican las dependencias que debes agregar al archivo pom.xml. Puedes, y se 
recomienda que lo hagas, descargar los proyectos proporcionados con el libro y el código fuente 
disponible en https://www.manning.com/downloads/2105. Los proyectos te serán de ayuda si te quedas 
atascado en algo. También puedes usarlos para validar tus soluciones finales.

NOTA Los ejemplos en este libro no dependen de la herramienta de construcción que elijas. Puedes usar
tanto `Maven` como `Gradle`. Para mantener la coherencia, todos los ejemplos fueron construidos con 
`Maven.`

El primer proyecto también es el más pequeño. Es una aplicación sencilla que expone un endpoint REST
al que puedes acceder y recibir una respuesta, como se describe en la figura 2.1. Este proyecto es 
suficiente para que aprendas los primeros pasos al desarrollar una aplicación con Spring Security y 
Spring Boot. Presenta los conceptos básicos de la arquitectura de Spring Security para autenticación 
y autorización.

Aprendemos Spring Security creando un proyecto vacío y nombrándolo ssia-ch2-ex1. (Este ejemplo 
también se encuentra con el mismo nombre en otros proyectos proporcionados.) Las únicas dependencias
que necesitas incluir para nuestro primer proyecto son spring-boot-starter-web y 
spring-boot-starter-security, como se muestra en el listado 2.1. Después de crear el proyecto, 
asegúrate de agregar estas dependencias a tu archivo pom.xml. El objetivo principal de trabajar en 
este proyecto es observar el comportamiento de una aplicación configurada por defecto con Spring 
Security. También queremos comprender qué componentes forman parte de esta configuración 
predeterminada, así como su propósito.

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

Podríamos iniciar directamente la aplicación ahora. Spring Boot aplica la configuración 
predeterminada del contexto de Spring en función de las dependencias que agregamos al proyecto. 
Sin embargo, no aprenderíamos mucho sobre seguridad si no tuviéramos al menos un endpoint protegido. 
Creemos un endpoint sencillo y llamémoslo para ver qué sucede. Para ello, agregamos una clase al 
proyecto vacío y la llamamos HelloController. Esta clase se añade a un paquete llamado controllers 
dentro del espacio de nombres principal del proyecto Spring Boot.

`NOTA` Spring Boot solo escanea componentes en el paquete (y sus subpaquetes) que contiene la clase 
anotada con @SpringBootApplication. Si anotas clases con cualquiera de los componentes estereotipo 
de Spring fuera del paquete principal, debes declarar explícitamente la ubicación usando la 
anotación @ComponentScan.

En el siguiente listado, la clase HelloController define un controlador REST y un endpoint REST para
nuestro ejemplo. 

```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
}
```

La anotación @RestController registra el bean en el contexto y le indica a Spring que la aplicación 
use esta instancia como un controlador web. Además, la anotación especifica que la aplicación debe 
establecer el cuerpo de la respuesta HTTP a partir del valor devuelto por el método. La anotación 
@GetMapping mapea la ruta /hello al método implementado mediante una solicitud GET. Una vez que 
ejecutas la aplicación, además de las demás líneas en la consola, deberías ver un mensaje similar a:
`Using generated security password: 93a01cf0-794b-4b98-86ef-54860f36f7f3` 

Cada vez que ejecutas la aplicación, se genera una nueva contraseña y se imprime en la consola, como
se muestra en el fragmento de código anterior. Debes usar esta contraseña para acceder a cualquiera 
de los endpoints de la aplicación mediante autenticación HTTP Basic. Primero, intentemos llamar al 
endpoint sin usar el encabezado de autorización:
`curl http://localhost:8080/hello`

`NOTA` En este libro, usamos cURL para llamar a los endpoints en todos los ejemplos. Considero que 
cURL es la solución más clara y legible. Pero si lo prefieres, puedes usar la herramienta de tu 
elección. Por ejemplo, quizás desees una interfaz gráfica más cómoda. En ese caso, Postman, Insomnia
o Bruno son excelentes opciones. Si el sistema operativo que usas no tiene instalada ninguna de estas
herramientas, probablemente necesites instalarlas tú mismo.

Y la respuesta a la llamada es:
```json
{
"status":401,
"error":"Unauthorized",
"message":"Unauthorized",
"path":"/hello"
}
```
El estado de la respuesta es HTTP 401 Unauthorized. Esperábamos este resultado, ya que no usamos las
credenciales adecuadas para la autenticación. Por defecto, Spring Security espera el nombre de 
usuario predeterminado (user) junto con la contraseña generada (en mi caso, la que comienza con 93a01).
Intentémoslo nuevamente, pero ahora con las credenciales correctas:
`curl -u user:93a01cf0-794b-4b98-86ef-54860f36f7f3 http://localhost:8080/hello`

La respuesta a la llamada es:
`Hello!`

`NOTA` El código de estado HTTP 401 Unauthorized es un poco ambiguo. Por lo general, se utiliza para
representar un error de autenticación, no de autorización. Los desarrolladores lo usan en el diseño 
de la aplicación para casos como la ausencia o credenciales incorrectas. Para un error de 
autorización, probablemente usaríamos el estado 403 Forbidden. En general, un HTTP 403 significa que
el servidor identificó al remitente de la solicitud, pero este no tiene los privilegios necesarios 
para realizar la acción que intenta hacer.

Una vez que enviamos las credenciales correctas, puedes observar en el cuerpo de la respuesta 
exactamente lo que devuelve el método HelloController que definimos anteriormente.

Llamando al endpoint con autenticación HTTP Basic
Con cURL, puedes establecer el nombre de usuario y la contraseña para autenticación HTTP mediante la
opción -u. En segundo plano, cURL codifica la cadena <nombre de usuario>:<contraseña> en Base64 y la 
envía como valor del encabezado Authorization, precedida por la palabra Basic.

Aunque es más sencillo usar la opción -u con cURL, también es importante conocer cómo se ve la 
solicitud real. Por ello, probemos crear manualmente el encabezado Authorization.

En el primer paso, toma la cadena <nombre de usuario>:<contraseña> y codifícala en Base64. Cuando 
tu aplicación realice la llamada, necesitas saber cómo formar el valor correcto para el encabezado 
Authorization. Puedes hacerlo usando la herramienta base64 en una consola Linux. También puedes usar
una página web que codifique cadenas en Base64, como https://www.base64encode.org. Este fragmento 
muestra el comando en una consola Linux o Git Bash (el parámetro -n indica que no se agregue un 
salto de línea al final):

`echo -n user:93a01cf0-794b-4b98-86ef-54860f36f7f3 | base64`

La ejecución de este comando devuelve la siguiente cadena codificada en Base64:

`dXNlcjo5M2EwMWNmMC03OTRiLTRiOTgtODZlZi01NDg2MGYzNmY3ZjM=`

El resultado de la llamada es:
`Hello!`

No hay configuraciones de seguridad significativas que analizar en un proyecto por defecto. 
Principalmente usamos las configuraciones predeterminadas para verificar que las dependencias 
correctas están en su lugar. Hacen poco en cuanto a autenticación y autorización, y esta 
implementación no es adecuada para una aplicación lista para producción. Sin embargo, el proyecto 
por defecto es un excelente punto de partida.

Con este primer ejemplo funcionando, al menos sabemos que Spring Security está activo. El siguiente 
paso es modificar las configuraciones para adaptarlas a los requisitos del proyecto. Primero 
profundizaremos en lo que Spring Boot configura por defecto en términos de Spring Security y luego 
veremos cómo anular estas configuraciones.

### 2.2 La imagen general del diseño de clases en Spring Security
En esta sección, analizamos los principales componentes de la arquitectura que participan en los 
procesos de autenticación y autorización. Es esencial conocer estos elementos, ya que deberás anular
estos componentes preconfigurados para adaptarlos a las necesidades de tu aplicación. Comenzaré 
describiendo cómo funciona la arquitectura de Spring Security para autenticación y autorización, y 
luego aplicaremos este conocimiento a los proyectos de este capítulo. Sería demasiado extenso tratar
todos los aspectos a la vez, por lo que, para reducir el esfuerzo de aprendizaje en este capítulo, 
me centraré en la visión general de cada componente. Detalles específicos sobre cada uno se tratarán
en capítulos posteriores.

En la sección 2.1, viste cierta lógica ejecutándose para autenticación y autorización. Teníamos un 
usuario por defecto y obteníamos una contraseña aleatoria cada vez que iniciábamos la aplicación. 
Pudimos usar este usuario y contraseña predeterminados para llamar a un endpoint. Pero ¿dónde se 
implementa toda esta lógica? Como probablemente ya sepas, Spring Boot configura algunos componentes 
automáticamente según las dependencias que uses (es decir, la convención sobre configuración que 
discutimos al inicio del capítulo). La figura 2.2 muestra una visión general de los principales 
actores (componentes) en la arquitectura de Spring Security y las relaciones entre ellos. Estos 
componentes tienen una implementación preconfigurada en el primer proyecto. En este capítulo, 
demostraré qué configura Spring Boot en tu aplicación en términos de Spring Security y también 
analizaremos las relaciones entre las entidades que forman parte del flujo de autenticación.

1. El authentication filter captura la peticion `"request"`
2. El authentication manager asume la responsabilidad del authentication
3. El authentication manager emplea a authentication provider para ejecutar la lógica de authentication.
4. El authentication provider encuentra al usuario mediante un servicio de detalles de usuario y 
valida la contraseña utilizando un codificador de contraseñas.
5. El resultado del authentication es devuelto al filter.
6. Los detalles sobre la entidad autenticada son guardados en un contexto de seguridad.

Los elementos centrales involucrados en el proceso de autenticación de Spring Security y sus 
interconexiones son el enfoque aquí. Este marco forma la estructura esencial para ejecutar la 
autenticación usando Spring Security. A lo largo del libro, haremos referencia frecuente a esta 
arquitectura al examinar diversas estrategias de autenticación y autorización.

En sintesis:

1. El filtro de autenticación delega la solicitud de autenticación al gestor de autenticación y, 
según la respuesta, configura el contexto de seguridad.
2. El gestor de autenticación utiliza el proveedor de autenticación para procesar la autenticación.
3. El proveedor de autenticación implementa la lógica de autenticación.
4. El servicio de detalles de usuario implementa la responsabilidad de gestión de usuarios, que el 
proveedor de autenticación utiliza en la lógica de autenticación.
5. El codificador de contraseñas implementa la gestión de contraseñas, que el proveedor de 
autenticación utiliza en la lógica de autenticación.
6. El contexto de seguridad mantiene los datos de autenticación tras el proceso de autenticación. 
El contexto de seguridad conservará los datos hasta que finalice la acción. Por lo general, en una 
aplicación con un hilo por solicitud, esto significa hasta que la aplicación envíe la respuesta al 
cliente.