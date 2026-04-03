# Parte 4

## Implementación de OAuth 2 y OpenID Connect

En una era donde los métodos de autenticación seguros y fluidos son fundamentales, protocolos como 
`OAuth 2 y OpenID` Connect se han consolidado como estándares del sector. Esta parte del libro 
desentraña las complejidades de estos protocolos, arrojando luz sobre sus mecanismos, beneficios y 
posibles riesgos.

El capítulo 13 sienta las bases con una visión general de ambos protocolos, describiendo en detalle 
los distintos tipos de concesión de `tokens` y destacando las vulnerabilidades potenciales dentro de 
`OAuth 2`.

El capítulo 14 profundiza en la configuración de un servidor de autorización robusto con Spring 
Security, incluyendo la definición de detalles del cliente y la gestión de claves criptográficas.

El capítulo 15 ofrece orientación para crear un servidor de recursos resistente, enfatizando la 
introspección de `tokens` y la protección de recursos.

El capítulo 16 concluye esta sección, demostrando cómo obtener `tokens` del servidor de autorización 
y acceder a recursos bajo la protección del servidor de recursos.

Al finalizar esta parte, estarás capacitado para integrar `OAuth 2 y OpenID Connect` en tus 
aplicaciones, fortaleciéndolas contra accesos no autorizados y garantizando experiencias de usuario 
fluidas. Habrás adquirido la experiencia necesaria para diseñar e implementar estratégicamente 
políticas de autorización integrales, asegurando que tu aplicación sea funcional y segura.

# ¿Qué son OAuth 2 y OpenID Connect?

### Este capítulo abarca

- El propósito de los tokens de acceso
- Cómo se emiten y validan los tokens en un sistema OAuth 2
- Los roles involucrados en un sistema OAuth 2/OpenID Connect

Supongamos que trabajas para una organización grande y utilizas varias herramientas en tu trabajo 
diario. Usas aplicaciones para rastrear errores, aplicaciones para documentar tu trabajo, 
aplicaciones para registrar tu tiempo, y así sucesivamente. En cada una de ellas, necesitas 
autenticarte para poder usarlas. ¿Usarías un conjunto diferente de credenciales para cada aplicación?
Claro, podrías hacerlo, pero este enfoque sería incómodo para el usuario (es decir, para ti), y 
además complicaría el propósito de las aplicaciones que utilizas.

Para ti, la complejidad radica en que tendrías que recordar las credenciales e iniciar sesión varias
veces en cada una de las aplicaciones que usas. Para las aplicaciones, la complejidad adicional 
proviene del hecho de que también tendrían que implementar la capacidad de almacenar y proteger las 
credenciales, además del proceso de autenticación real.

¿Qué tal si gestionamos la responsabilidad de almacenar las credenciales y realizar la autenticación 
en una aplicación aparte? En este caso, los usuarios solo tendrían que iniciar sesión una vez y 
podrían usar todas sus aplicaciones sin necesidad de autenticarse repetidamente. ¿Existe tal 
solución? Sí. Puedes implementar la autenticación mediante la especificación OAuth 2.

La segunda consideración es que podemos pensar aún más grande. Una aplicación que pueda ser usada 
por usuarios públicos (usuarios fuera de una organización, una aplicación que creas para todo el mundo)
también podría necesitar capacidades de autenticación. La aplicación puede implementar esas 
funcionalidades, pero:

- Implementar la autenticación en esta aplicación requiere más trabajo y esfuerzo.
- Los usuarios necesitan un conjunto específico de credenciales para esta aplicación.
- A veces, los usuarios no confían en crear credenciales específicas para cualquier pequeña aplicación que usan.

¿Podrías permitir que los usuarios de esta aplicación inicien sesión con credenciales que ya poseen?
¿Podrían los usuarios de tu aplicación iniciar sesión con sus cuentas de Facebook, GitHub, Twitter 
o Google, por ejemplo? Es posible que ya hayas visto esto en muchos lugares. Aplicaciones en la web 
permiten a los usuarios registrarse o iniciar sesión usando diversas plataformas de redes sociales. 
De esta manera, una aplicación permite a sus usuarios autenticarse con un conjunto de credenciales 
que ya tienen, sin que tú tengas que implementar algo específico en tu aplicación. Este enfoque:

- Reduce los costos (por ejemplo, de implementar y mantener la autenticación en tu aplicación)
- Evita problemas de confianza por parte del usuario (por ejemplo, tener que registrarse y crear otro
conjunto de credenciales que tu aplicación tendrá que gestionar)
- Ayuda a los usuarios a minimizar la cantidad de credenciales que utilizan

OAuth 2 es una especificación que indica cómo separar las responsabilidades de autenticación en un 
sistema. De esta forma, múltiples aplicaciones pueden usar otra aplicación que implemente la 
autenticación, ayudando a los usuarios a autenticarse más rápido, manteniendo sus datos más seguros
y reduciendo los costos de implementación en las aplicaciones.

Comenzaremos con la sección 13.1, donde presentaré los actores principales involucrados en un sistema
donde la autenticación y autorización se construyen sobre la especificación OAuth 2. En la sección 
13.1, aprenderás todas las responsabilidades de las partes de un sistema OAuth 2, como el usuario, 
el cliente, el servidor de autorización y el servidor de recursos. En la sección 13.2, discutiremos 
los tokens. Los tokens son como llaves de acceso para una aplicación. Aprenderás que puedes usar 
múltiples tipos de tokens y cuándo es mejor usar cada tipo. La sección 13.3 revisa las formas más 
importantes mediante las cuales se pueden emitir los tokens (que implementaremos y probaremos en el 
capítulo 14). Finalizamos este capítulo con la sección 13.4, donde repasaremos los posibles problemas
que debes considerar al implementar OAuth 2.

Antes de comenzar, me gustaría mencionar que en este capítulo ofrezco una perspectiva sencilla de 
todo lo que necesitas saber para comprender mejor nuestra discusión en los capítulos 14 al 16. No 
pretendo convertirte en un experto en OAuth 2 y OpenID Connect en un solo capítulo. No creo que eso 
sea posible, ya que ambos son lo suficientemente complejos como para que otras personas hayan escrito
libros enteros sobre ellos. Si deseas ampliar tus conocimientos sobre el tema, recomiendo OAuth 2 in 
Action de Justin Richer y Antonio Sanso (Manning, 2017) y OpenID Connect in Action de Prabath 
Siriwardena (Manning, 2023).

## 13.1 La gran imagen de OAuth 2 y OpenID Connect

Supongamos que necesitas asistir a una entrevista con una gran corporación. Te han llamado a su sede
principal para una reunión presencial. Pero no cualquiera puede entrar a las oficinas de la empresa.
Existen protocolos específicos implementados para los visitantes.

Para entrar al edificio y asistir a la reunión, primero debes visitar la recepción y demostrar quién
eres mediante una identificación. Después de ser identificado, recibirás una tarjeta de acceso en la
recepción, que te permitirá abrir ciertas puertas. Es posible que ni siquiera puedas usar todos los 
ascensores, sino solo algunos específicos.

![Oauth](images/chapter13/figure13.1.png)

La especificación OAuth 2 es muy similar a acceder a un edificio de oficinas.
El proceso de entrar al edificio para la reunión es muy parecido a cómo funcionan la autenticación y
la autorización en una implementación de OAuth 2. Tú eres el usuario que necesita ejecutar un caso 
de uso específico (ir a una sala determinada para la reunión). Para ello, usas tus credenciales 
(el documento de identidad) para autenticarte en la recepción (el servidor de autorización). Una vez
que demuestras quién eres, recibes una tarjeta de acceso (el token). Pero solo puedes usar este token
para acceder a recursos específicos (como ascensores y puertas determinadas). Además, solo puedes 
usar la tarjeta de acceso por un período corto. Después de tu reunión, debes devolverla a la recepción.

En esta sección, analizamos las responsabilidades que interactúan entre sí en un sistema OAuth 2, y
verás cómo se asemeja a visitar la sede de una organización para una entrevista. También explicamos
qué es OAuth 2 como especificación y cuál es la diferencia entre OpenID Connect (un protocolo) y 
OAuth 2 (la especificación sobre la que se basa). Considero esencial comprender bien los conceptos 
detrás de este enfoque de autenticación y autorización antes de profundizar en su implementación en 
los capítulos 14 al 16.

Primero, descubramos quiénes desempeñan roles en un sistema OAuth 2. La figura 13.2 presenta los 
actores principales de un sistema OAuth 2. Por actor me refiero a cualquier entidad que tenga un rol
en la funcionalidad del sistema. En un sistema OAuth 2, encontrarás los siguientes actores:

- El usuario: la persona que usa la aplicación. Los usuarios suelen interactuar con una aplicación 
frontal, a la que llamamos cliente. Los usuarios no siempre están presentes en un sistema OAuth 2, 
como se analiza en la sección 13.3.3, donde conocerás el tipo de concesión de credenciales del cliente.
- El cliente: la aplicación que llama a un backend y necesita autenticación y autorización. Puede 
ser una aplicación web, una app móvil, una aplicación de escritorio o incluso un servicio backend 
independiente. Normalmente, el sistema no tiene un usuario cuando el cliente es un servicio backend.
- El servidor de recursos: una aplicación backend que autoriza y atiende llamadas enviadas por una o
más aplicaciones cliente.
- El servidor de autorización: una aplicación que implementa la autenticación y el almacenamiento 
seguro de credenciales.

![participants](images/chapter13/figure13.2.png)

Los participantes en un marco OAuth 2. Los usuarios interactúan a través de un cliente que requiere 
autorización para ciertas operaciones en el servicio backend, conocido como servidor de recursos. 
Para la autorización del backend, el primer paso del cliente es la autenticación por parte del 
servidor de autorización.

Veamos ahora cómo ocurre realmente la autenticación y la autorización. Los pasos son sencillos:

1. El usuario ejecuta un caso de uso determinado con la aplicación cliente.
2. La aplicación cliente obtiene autorización para llamar al servidor de recursos y atender la 
solicitud del usuario.
3. Para obtener la autorización, el cliente solicita primero un token (llamado token de acceso) al 
servidor de autorización. Este token es simplemente una información específica que ayuda al cliente a demostrar que el servidor de autorización lo ha identificado correctamente.
4. El cliente utiliza el token emitido por el servidor de autorización para obtener autorización al 
enviar solicitudes a su backend (el servidor de recursos).

La siguiente figura detalla visualmente el flujo. Los pasos numerados en la figura representan lo 
siguiente:

1. El usuario intenta usar la aplicación cliente para ejecutar un caso de uso determinado.
2. La aplicación cliente sabe que no puede llamar a su backend sin antes tener un token que le 
permita obtener autorización. El cliente solicita ese token de acceso al servidor de autorización.
3. Tras la solicitud de la aplicación cliente, el servidor de autorización emite un token y lo envía
a la aplicación cliente. 
4. El cliente utiliza el token para enviar solicitudes a su backend (el servidor de recursos).
5. El servidor de recursos autoriza la solicitud del cliente. Si la autorización es exitosa, el 
servidor de recursos ejecuta la solicitud y responde.
6. El cliente muestra el resultado al usuario.

![simples explanations](images/chapter13/figure13.3.png)

La explicación más sencilla del procedimiento de autenticación OAuth 2 implica que el cliente obtenga
un token del servidor de autorización. Este token luego se utiliza para obtener autorización para 
las solicitudes enviadas a la aplicación backend, o servidor de recursos.

Pero ¿qué es exactamente el token que emite el servidor de autorización? Un token puede ser cualquier
pieza de datos (normalmente una cadena de caracteres) que permite al cliente demostrar que él (y/o 
el usuario) ha sido identificado por el servidor de autorización. El token también puede servir para
obtener más detalles sobre el usuario y el cliente si es necesario. Dado que el servidor de 
autorización gestiona ahora todos los datos del usuario y del cliente, el backend a veces necesita 
obtener parte de esta información desde dicho servidor y utilizarla. El backend obtendrá esos 
detalles a través del token. En ocasiones, el propio token contiene la información necesaria (como 
se explica en la sección 13.2, estos tokens se denominan tokens no opacos); en caso contrario, el 
backend debe llamar al servidor de autorización para obtener los datos sobre el cliente y el usuario
(es decir, tokens opacos). Además, a diferencia de una llave física, un token de acceso no tiene una
vida útil larga. Expira tras un breve período (en la mayoría de los casos, minutos), tras lo cual el
cliente debe solicitar otro token al servidor de autorización. De esta manera, un token perdido 
(como una llave perdida) no puede ser mal utilizado.

OAuth 2 describe múltiples flujos mediante los cuales un cliente puede obtener un token. Estos 
flujos se conocen como tipos de concesión, y en la sección 13.3 se analizan los tipos de concesión
más comunes.

## 13.2 El uso de diversas implementaciones de tokens
Los tokens son las tarjetas de acceso (figura 13.4) que el cliente utiliza para obtener autorización
al enviar solicitudes al backend (el servidor de recursos). Los tokens son una parte esencial del 
proceso de autenticación y autorización de OAuth 2, ya que se usan para demostrar la autenticidad de 
la autenticación del cliente y del usuario, pero también son el medio mediante el cual el backend 
obtiene más detalles sobre el cliente y el usuario.

En esta sección, se analiza cómo se clasifican los tokens y, según el tipo de token, cómo se 
utilizan en el proceso de autorización.

![zglorb](images/chapter13/figure13.4.png)

Zglorb (el usuario) necesita acceder a la Nave Madre (servidor de recursos). Para hacerlo, primero 
se autentica (mediante el servidor de autorización) y luego se le entrega una tarjeta de acceso 
(el token). Zglorb solo puede acceder a áreas específicas (recursos) de la Nave Madre utilizando su
tarjeta de acceso.

Clasificamos los tokens según la forma en que proporcionan al servidor de recursos los datos para la
autorización:

- Opacos: tokens que no almacenan datos. Para implementar la autorización, el servidor de recursos 
normalmente necesita llamar al servidor de autorización, proporcionar el token opaco y obtener los 
detalles. Esta llamada se conoce como llamada de introspección.
- No opacos: tokens que almacenan datos, lo que permite al backend implementar inmediatamente la 
autorización. El JSON Web Token (JWT) es la implementación de token no opaco más utilizada.

### 13.2.1 Usando tokens opacos

Los tokens opacos no contienen datos que el backend pueda usar para identificar al usuario o al 
cliente, ni para implementar las reglas de autorización. Los tokens opacos son simplemente una prueba
de un intento de autenticación. Cuando un servidor de recursos recibe un token opaco, necesita llamar
al servidor de autorización para verificar si el token es válido y obtener más información que le
permita aplicar restricciones de autorización.

Un token opaco es literalmente como una llave de una caja fuerte. No proporciona ninguna información
por adelantado; solo sabes que funciona cuando intentas abrir la caja con ella. Una vez que descubres
que es válida, también te da acceso a lo que hay dentro de la caja (en este caso, los detalles del 
usuario y del cliente). La siguiente figura ilustra esta analogía.

![analogy](images/chapter13/figure13.5.png)

Una analogía con los tokens opacos. Un token opaco es igual que una llave. No sabes si funciona hasta
que lo pruebas. Si la llave funciona, también obtienes acceso a lo que hay dentro.

The resource server calls an endpoint provided by the authorization server to find
out whether the opaque token is valid and obtain the needed details about the client
and the user to whom the token was issued. This call is named a token introspection
(figure 13.6). Once the resource server has these details, it can apply the authorization
constraints.

![token](images/chapter13/figure13.6.png)

La llamada de introspección de token. El servidor de recursos envía una solicitud al servidor de
autorización para verificar si el token opaco es válido y obtener los detalles sobre a quién fue 
emitido.

### 13.2.2 Usar tokens no opacos

A diferencia de los tokens opacos que analizamos en la sección 13.2.1, los tokens no opacos 
contienen información sobre el cliente y el usuario a quienes el servidor de autorización emitió los
tokens durante el proceso de autenticación. Puedes comparar los tokens no opacos con documentos 
firmados (siguiete figura).

![non-opaque](images/chapter13/figure13.7.png)

El token no opaco es como un documento firmado. Contiene los detalles necesarios para que el servidor
de recursos aplique las restricciones de autorización, así como una firma para validar su autenticidad.

La implementación más común de un token no opaco es el JWT. Un JWT se compone de tres partes 
(siguiente figura):

- El encabezado: Suele contener datos sobre el token, como el algoritmo criptográfico utilizado para 
firmarlo o el ID de la clave que el servidor de autorización usó para firmarlo.

- El cuerpo: Suele contener datos sobre la entidad a la que se emitió el token, como detalles del 
cliente y del usuario.

- La firma: Un valor generado criptográficamente que permite comprobar que el servidor de autorización
emitió efectivamente el token y que nadie ha alterado su contenido (en el encabezado o el cuerpo) 
después de su creación.

Los datos del encabezado y el cuerpo están en formato JSON, luego se codifican en Base64 para 
hacerlos más pequeños y fáciles de transferir. Las tres partes van separadas por puntos.

![jwt token](images/chapter13/figure13.8.png)

La anatomía de un token JWT. El encabezado y el cuerpo contienen los detalles necesarios para que el
servidor de recursos valide la autenticidad del token y aplique las restricciones de autorización.

El siguiente fragmento muestra un ejemplo de un JWT en el que las tres partes están codificadas en 
Base64 y separadas por puntos:
```jsunicoderegexp
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFmZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```
Quizás te estés preguntando ahora: “¿Cuándo debo usar tokens opacos y cuándo debo usar no opacos?”. 
Como mencioné anteriormente en esta sección, los tokens no opacos son los más utilizados actualmente
porque no requieren introspección para validarse. Sin embargo, los tokens no opacos contienen datos,
y el cliente envía estos datos a través de la red a su backend. Cualquier persona que obtenga el 
token también puede ver la información que transporta. En la mayoría de los casos, esto no representa
un problema. Y recomiendo evitar incluir demasiada información dentro del token.

Pero, ¿qué debes hacer si necesitas transportar una gran cantidad de datos o información sensible que
no es seguro enviar a través de la red dentro de un token? En ese caso, los tokens opacos podrían ser
una buena alternativa. Recomiendo considerar primero los tokens no opacos y recurrir a los opacos 
solo si la cantidad de datos es demasiado grande o si necesitas enviar información más sensible y 
deseas evitar intercambiarla a través del token.

## 13.3 Obtención de tokens mediante diversos tipos de concesión

Esta sección trata sobre los tipos de concesión. Un tipo de concesión es un proceso mediante el cual
un cliente obtiene un token. En las aplicaciones, se pueden encontrar distintos enfoques para que un
cliente obtenga un token del servidor de autorización. Analizaremos los tres tipos de concesión más 
utilizados. Al final de esta sección, exploraremos cómo un cliente puede regenerar un token tras su
expiración.

`NOTA` Es posible que aún encuentres aplicaciones que implementen otros dos tipos de concesión: la 
concesión implícita y la concesión de contraseña. Estos dos tipos han quedado obsoletos porque no 
ofrecen suficiente seguridad. No los trataremos en este libro; no recomiendo su uso en aplicaciones.
Siempre puedes reemplazarlos por alguno de los otros tipos de concesión discutidos en esta sección. 
Si deseas obtener más información sobre la concesión de contraseña, hay una buena explicación al 
respecto en el capítulo 12 de la primera edición de este libro. Además, comentaremos brevemente la 
concesión implícita y por qué ha quedado obsoleta al tratar la concesión de código de autorización 
en esta sección.

La sección 13.3.1 analiza el tipo de concesión de código de autorización, que es el más utilizado 
cuando el sistema necesita permitir la autenticación de un usuario. En la sección 13.3.2, discutimos
una extensión de este tipo de concesión: la clave de prueba para el intercambio de códigos (PKCE). 
En la sección 13.3.3, continuamos con el escenario en el que una aplicación necesita obtener un token
sin que intervenga la autenticación de un usuario, y concluimos con la forma de regenerar tokens en 
la sección 13.3.4.

### 13.3.1 Obtención de un token mediante el tipo de concesión de código de autorización

El tipo de concesión de código de autorización es el más utilizado actualmente. Se emplea cuando 
nuestra aplicación necesita autenticar a un usuario (para entender fácilmente este tipo de concesión,
véase en la siguiente figura, que ilustra los pasos en un diagrama de secuencia):

1. El usuario desea realizar una acción dentro de la aplicación que está usando. Por ejemplo, 
supongamos que la persona del lado izquierdo del diagrama es María, una contadora que desea ver todas
las facturas que debe pagar la empresa para la que trabaja.
2. La aplicación que usa María es el cliente. En este caso, María está frente a su computadora, por 
lo que su aplicación cliente es una aplicación web. Pero también podría haber usado la versión móvil
de la aplicación. En ambos casos, el tipo de concesión sería el mismo. Como María no ha iniciado 
sesión, la aplicación la redirige a una página de inicio de sesión alojada por el servidor de 
autorización.
3. Ahora María ve la página de inicio de sesión en su navegador. Esta página no forma parte de la 
aplicación a la que accedió, sino que está alojada por un sistema diferente. María reconoce la página
a la que fue redirigida como la aplicación central de autenticación que utiliza para cualquier 
aplicación de su empresa. Sabe que, tras enviar sus credenciales, el navegador la devolverá a la 
aplicación de facturas y podrá verlas y operar con los datos que necesita. Ingresa las credenciales 
correctas y selecciona el botón de inicio de sesión.
4. Como las credenciales proporcionadas por María son correctas, el servidor de autorización la 
redirige de vuelta a la aplicación de facturas. Además, proporciona a la aplicación inicial 
(el cliente) un código único denominado "código de autorización". El cliente utilizará este código 
para obtener un token de acceso.
5. El cliente solicita un token de acceso. Necesita este token para enviar solicitudes a su backend 
(el servidor de recursos).
6. Como el código de autorización es correcto (el mismo que el servidor proporcionó en el paso 4),
el servidor de autorización responde con un token de acceso.
7. La aplicación cliente utiliza el token de acceso para enviar una solicitud a su backend y obtener
autorización.

![authorization code](images/chapter13/figure13.9.png)

El tipo de concesión de código de autorización. Se requiere que el usuario inicie sesión. Tras el 
inicio de sesión, el servidor de autorización emite un código de autorización al cliente, el cual 
utiliza posteriormente para obtener un token de acceso. Este token de acceso permite que las 
solicitudes del cliente sean autorizadas por el servidor de recursos.

Algunas observaciones para ayudarte a entender mejor este flujo:

- Presta atención a las flechas punteadas. Es esencial recordar que representan redirecciones en el 
navegador, no solicitudes ni respuestas. En el paso 2, la aplicación cliente redirige al usuario a 
la página de inicio de sesión del servidor de autorización (una redirección en el navegador hacia 
una página web de otra aplicación). En el paso 4, el servidor de autorización redirige de vuelta a 
la aplicación cliente, proporcionando el código de autorización (normalmente como parámetro de 
consulta).

- Mary (la usuaria) no es consciente de los pasos del 4 al 7. Tras iniciar sesión, al final verá las 
facturas mostradas por la aplicación cliente, que las obtendrá como respuesta al paso 7 tras haber 
sido autorizada.

- Recuerda no confundir el código de autorización con el token de acceso. El token de acceso es lo que
la aplicación necesita finalmente para ser autorizada por su backend (paso 7). Pero para obtenerlo, 
primero debe obtener un código de autorización (pasos 4 y 5).

Además, muchos desarrolladores nuevos en autorización y autenticación tienen dudas sobre el paso 4.
La pregunta frecuente es: “¿Por qué el servidor de autorización no devuelve directamente el token de
acceso aquí?”. Parece extraño que el cliente deba dar un paso adicional para obtener el token de 
acceso cuando podría haberlo recibido directamente en el paso 4. Pero tiene sentido. De hecho, en la
primera versión de OAuth, el servidor de autorización entregaba el token de acceso en lugar del 
código de autorización en el paso 4. Esto es lo que ahora conocemos como “tipo de concesión implícita”,
que ha quedado obsoleto y ya no se recomienda. La razón es que una redirección puede interceptarse 
fácilmente, y un atacante podría obtener sin problemas el token de acceso. Al devolver un código de 
autorización, el servidor de autorización obliga al cliente a realizar una nueva solicitud en la que
debe autenticarse nuevamente con sus credenciales. De este modo, si alguien intercepta la redirección
y obtiene el código de autorización, no es suficiente para obtener el token de acceso; también 
necesitaría conocer las credenciales del cliente para enviar la solicitud y obtener el token.

La siguiente figura muestra visualmente los dos pasos en los que el código de autorización añade una
protección adicional para evitar que alguien obtenga el token de acceso.

![logging](images/chapter13/figure13.10.png)

Tras iniciar sesión y recibir un código de autorización, el cliente debe realizar una solicitud 
adicional para obtener el token de acceso. Durante esta solicitud, el cliente debe verificar su 
identidad utilizando sus credenciales. Este método incrementa la dificultad para cualquier persona 
que intente obtener ilegítimamente el token de acceso, ya que necesitaría interceptar el código de 
autorización y, además, conocer las credenciales del cliente.

### 13.3.2 Aplicar protección PKCE al tipo de concesión de código de autorización.

¿Qué ocurre si una persona con malas intenciones logra obtener también las credenciales del cliente?
En ese caso, podría obtener un token de acceso y enviar solicitudes al servidor de recursos. ¿Existe
alguna forma de prevenir este escenario? Sí, la clave de prueba para el intercambio de códigos 
(PKCE, generalmente pronunciado "pixy") es una mejora añadida al flujo de código de autorización para 
hacerlo más seguro. En esta sección, explicamos cómo PKCE protege contra el robo de credenciales del
cliente.

El uso de PKCE solo afecta a dos pasos del tipo de concesión de código de autorización, discutidos 
en la sección 13.3.1. En la siguiente figura, las flechas que representan los pasos 3 y 5 están más 
gruesas, ya que son los pasos donde se aplica PKCE:

- Primero, el cliente debe generar un valor aleatorio. Este valor puede ser una cadena aleatoria de 
bytes, conocido como verificador (verifier).

- Segundo, el cliente aplica una función hash al valor generado aleatoriamente en el paso 1. Una función
hash es un proceso criptográfico en el que el resultado no puede revertirse al valor original. El 
resultado de aplicar la función hash al verificador se denomina desafío (challenge).

![client](images/chapter13/figure13.11.png)

El cliente enviará el desafío en el paso 3 y el verificador en el paso 5 para demostrar que es el 
mismo cliente que inicialmente solicitó al usuario que iniciara sesión.

El cliente envía el desafío en el paso 3 junto con el inicio de sesión del usuario. El servidor de 
autorización guarda el desafío y espera el verificador en la solicitud del paso 5 para obtener el 
token de acceso. Si el verificador que el cliente envía al solicitar el token en el paso 5 coincide 
con el desafío enviado en el paso 3, el servidor de autorización sabe que la aplicación cliente que 
solicita el token es la misma que solicitó inicialmente la autenticación del usuario. Ahora, alguien
no puede obtener un token de acceso aunque logre interceptar el código de autorización en el paso 4,
porque también necesitaría conocer el valor del verificador. No pueden conocer el verificador, ya 
que el cliente aún no lo ha enviado a través de la red. Tampoco pueden obtenerlo interceptando el 
desafío (en el paso 3), porque el desafío se crea mediante una función hash, cuyo resultado no puede
revertirse al valor original.

### 13.3.3 Obtención de un token con el tipo de concesión de credenciales de cliente

A veces, una aplicación necesita obtener autorización sin intervención del usuario. Si no hay un 
usuario presente, la aplicación deberá usar el tipo de concesión de credenciales de cliente para 
obtener un token de acceso. Esta situación es común cuando un servicio necesita invocar a otro 
servicio tras un evento objetivo, como la activación de un temporizador en un proceso programado.

Mediante este tipo de concesión, la aplicación solo necesita autenticarse con sus credenciales de 
cliente. En la siguiente figura se ilustra este flujo:

1. La aplicación solicita un token de acceso al servidor de autorización, utilizando sus credenciales 
para autenticarse.

2. Si las credenciales son válidas, el servidor de autorización emite un token de acceso.

3. La aplicación utiliza el token de acceso para autorizar sus solicitudes al servidor de recursos.

![grant type](images/chapter13/figure13.12.png)

El tipo de concesión de cliente. Una aplicación obtiene un token de acceso sin necesidad de que un 
usuario inicie sesión.

## 13.4 Lo que OpenID Connect aporta a OAuth 2

Ciertamente, todavía existe mucha confusión sobre OpenID Connect (a veces llamado OIDC) y OAuth 2, 
así como las diferencias entre ambos. Suelo decirles a mis alumnos que no se estresen demasiado con
este tema: "Si entiendes OAuth 2, también sabes cómo usar OpenID Connect".

De hecho, OIDC es un protocolo construido sobre la especificación OAuth 2. Por esta razón, entender 
OAuth 2 te ayuda a comprender fácilmente OIDC. Permíteme darte una analogía sobre especificaciones 
frente a protocolos.

Todos usamos enchufes eléctricos todos los días. Los enchufes se ven diferentes en todo el mundo. A
veces, esto es un verdadero dolor de cabeza cuando viajas. Es posible que necesites adaptadores para
asegurarte de que puedes cargar tus dispositivos, especialmente si viajas entre diferentes regiones 
geográficas.

Pero, tras bambalinas, todos los enchufes funcionan de la misma manera. Hay algunos cables que emiten
el voltaje eléctrico. Puedes definir un marco de trabajo (framework) sobre el cual funcionan todos 
los enchufes eléctricos del mundo en solo un par de puntos clave:

- Un enchufe eléctrico tiene tres cables que permiten que fluya la corriente eléctrica: la fase, el 
neutro y una toma de tierra. La toma de tierra es opcional.

- El enchufe eléctrico proporciona un voltaje que puede rondar los 120 voltios o los 230 voltios.

Ahora, no te preocupes si no eres una persona técnica; no necesitas entender estos dos puntos. Al 
menos no para aprender Spring Security. Solo confía en mi palabra.

El problema es que, aunque todos los enchufes del mundo cumplan con estas especificaciones, seguimos
encontrando situaciones en las que necesitamos adaptadores para viajar. La razón es que no tienen un 
protocolo común. Los adaptadores son necesarios para adaptar el enchufe de un protocolo a otro (por 
ejemplo, de Norteamérica a Europa).

Lo mismo ocurre con las aplicaciones y la autenticación y autorización. Si dos aplicaciones cumplen 
con la especificación OAuth 2, aún pueden darse situaciones en las que no sean totalmente compatibles
y necesiten ser adaptadas porque no ejecutan el mismo protocolo. OpenID Connect es un protocolo que
restringe un poco la libertad de la especificación OAuth 2, introduciendo algunos cambios. Los 
cambios principales son:

- Valores específicos para los scopes o alcances (como profile u openid).

- El uso de un token adicional llamado ID token, utilizado para almacenar los detalles sobre la 
identidad del usuario y del cliente al que se le emitió el token.

- Normalmente, el término grant type (tipo de concesión) también se denomina flujo (flow) cuando se 
discute en términos de OIDC, mientras que el servidor de autorización se suele llamar proveedor de 
identidad o IdP.

Los pecados de OAuth 2
Esta sección analiza las posibles vulnerabilidades de las aplicaciones que utilizan la autenticación
y autorización de OAuth 2. Es fundamental comprender qué puede salir mal al usar OAuth 2 para evitar
estos escenarios durante el desarrollo de aplicaciones. Por supuesto, como cualquier otro aspecto del
desarrollo de software, OAuth 2 no es infalible. Tiene sus propias vulnerabilidades, de las cuales 
debemos estar conscientes al construir nuestras aplicaciones. A continuación, enumero algunas de las
más comunes:

- Ataques de falsificación de solicitudes entre sitios (CSRF) en el cliente: cuando un usuario ha 
iniciado sesión, es posible un ataque CSRF si la aplicación no implementa ningún mecanismo de 
protección. Tuvimos una excelente discusión sobre la protección CSRF implementada por Spring 
Security en el capítulo 9.

- Robo de credenciales del cliente: almacenar o transferir las credenciales sin protección puede 
generar brechas que permitan a los atacantes robarlas y utilizarlas.

- Reutilización de tokens: como se analizó en la sección 13.2, los tokens son las "llaves" que usamos
en una arquitectura de autenticación y autorización OAuth 2 para acceder a los recursos. Se envían a
través de la red y, a veces, pueden ser interceptados. Si esto ocurre, son robados y pueden 
reutilizarse. Imagina que pierdes la llave de la puerta principal de tu casa. ¿Qué podría pasar? 
Alguien más podría usarla para abrir la puerta tantas veces como desee (repetición).

- Secuestro de tokens: interfiere en el proceso de autenticación y roba tokens que puede usar para 
acceder a recursos. Esta también es una vulnerabilidad potencial del uso de tokens de actualización 
(refresh tokens), ya que también pueden ser interceptados y usados para obtener nuevos tokens de 
acceso. Recomiendo este artículo útil: http://mng.bz/am5z.

Recuerda, OAuth 2 es un marco de trabajo. Las vulnerabilidades son el resultado de una implementación
incorrecta sobre dicho marco. Usar Spring Security ya nos ayuda a mitigar la mayoría de estas 
vulnerabilidades en nuestras aplicaciones. Al implementar una aplicación con Spring Security, como 
viste en este capítulo, debemos configurar ciertos parámetros, pero confiamos en el flujo ya 
implementado por Spring Security.

Para obtener más detalles sobre las vulnerabilidades relacionadas con el marco OAuth 2 y cómo una 
persona con malas intenciones podría explotarlas, consulta la parte 3 de OAuth 2 In Action de Justin
Richer y Antonio Sanso (Manning, 2017), disponible en http://mng.bz/g7Ql.

### Resumen

- El marco OAuth 2 describe formas seguras en las que un backend puede autenticar a sus clientes. 
OpenID Connect es un protocolo que implementa el cliente OAuth 2 aplicando ciertas restricciones a 
las posibles implementaciones.

- Los cuatro actores principales en un sistema OAuth 2 son:

  - El usuario: una persona que desea ejecutar un caso de uso.
  - El cliente: una aplicación que debe estar autorizada para acceder a un recurso o caso de uso en un
  backend determinado.
  - El servidor de recursos: un backend que necesita autorizar a un cliente para ejecutar un caso de
  uso específico o acceder a un recurso.
  - El servidor de autorización: una aplicación que gestiona los detalles del usuario y del cliente,
  permitiéndoles autenticarse, y que proporciona un token que puede utilizarse para fines de 
  autorización.

- El token es una tarjeta de acceso (o una llave) que el cliente obtiene del servidor de autorización
y utiliza para obtener autorización y ejecutar un caso de uso o acceder a un recurso específico en un
backend seguro (el servidor de recursos).

- Clasificamos los tokens en dos categorías:

  - Opacos: tokens que no contienen detalles sobre el usuario y el cliente para los que fueron emitidos. Para este tipo de tokens, el servidor de recursos siempre necesita contactar al servidor de autorización para validar el token y obtener los datos necesarios para autorizar la solicitud. Esta verificación del token se denomina introspección.
  - No opacos: tokens que contienen información sobre el usuario y el cliente a quienes fueron emitidos. La implementación más común de tokens no opacos es el token web JSON (JWT).

- Existen múltiples flujos mediante los cuales una aplicación cliente puede solicitar un token al 
servidor de autorización. Estos flujos de emisión de tokens se denominan tipos de concesión 
(grant types). Los tipos de concesión más comunes son:

    - El tipo de concesión por código de autorización (authorization code grant type).
    - El tipo de concesión por credenciales del cliente (client credentials grant type).

- A veces se añade seguridad adicional al tipo de concesión por código de autorización mediante un 
método conocido como clave de prueba para el intercambio de códigos (PKCE). En este caso, el cliente
utiliza valores adicionales para evitar que alguien pueda obtener tokens de acceso robando las 
credenciales del cliente y el código de autorización.

- En casos específicos, una aplicación puede necesitar obtener nuevos tokens de acceso sin que el
usuario tenga que volver a autenticarse. Para estos casos, la aplicación puede usar tokens de 
actualización (refresh tokens). Los tokens de actualización son tokens especiales que solo pueden 
utilizarse para obtener nuevos tokens de acceso.