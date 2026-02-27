# Parte 1: Say hello to Spring Security
¿Sumergiéndonos en el mundo de Spring Security? ¡Emprendamos este viaje juntos! En esta primera
parte del libro, sentaremos las bases estableciendo una fundamentación sólida.

El capítulo 1 comienza presentándote el mundo de Spring Security. A medida que avancemos, 
profundizaremos en la esencia de la seguridad informática, tratando de responder preguntas clave 
como: «¿Qué es la seguridad del software?» y «¿Por qué es de suma importancia?». Además, 
describiremos la ruta de aprendizaje de este libro, asegurándonos de que sepas qué esperar a medida 
que avances.

El capítulo 2 será una experiencia práctica. Comenzaremos creando tu primer proyecto con Spring. 
Si alguna vez te has preguntado cuál es la arquitectura y el diseño de clases que sustentan Spring 
Security, este capítulo te ofrecerá una visión general. Pero no se trata solo de comprender los 
mecanismos predeterminados. El libro irá más allá, guiándote paso a paso para sobrescribir las 
configuraciones por defecto. Esto incluirá análisis profundos sobre la personalización de los 
detalles del usuario, la mejora de la autorización en distintos puntos de acceso, la exploración de
diversos métodos de configuración, la definición de tu propia lógica de autenticación y la 
utilización eficiente de múltiples clases de configuración.

Para cuando termines esta parte, no solo tendrás una comprensión sólida de los fundamentos teóricos 
de Spring Security, sino que también contarás con una aplicación funcional protegida mediante esta 
tecnología. Será una combinación de entender el «por qué» y dominar el «cómo», todo en un breve 
período de tiempo.

Este capítulo cubre
¡ Qué es Spring Security y qué puedes resolver utilizando esta herramienta
¡ Qué es la seguridad para una aplicación de software
¡ Por qué la seguridad del software es esencial y por qué deberías preocuparte

Los desarrolladores han tomado cada vez más conciencia sobre la necesidad de software seguro y han 
asumido responsabilidad sobre la seguridad desde el inicio del desarrollo. Generalmente, como 
desarrolladores, comenzamos aprendiendo que el propósito de una aplicación es resolver problemas de 
negocio. Este propósito implica que los datos sean procesados de alguna manera, almacenados y 
finalmente mostrados al usuario según lo especificado por los requisitos. Esta visión del desarrollo 
de software, que de alguna forma se impone desde las primeras etapas del aprendizaje, tiene la 
desventaja de ocultar prácticas que también forman parte del proceso. Aunque la aplicación funcione 
correctamente desde la perspectiva del usuario y haga lo esperado en términos de funcionalidad, 
muchos aspectos quedan ocultos en el resultado final. Cualidades no funcionales del software, como 
el rendimiento, la escalabilidad, la disponibilidad y la seguridad, entre otras, pueden tener efectos 
a corto y largo plazo. Si no se consideran desde el principio, estas cualidades pueden afectar 
drásticamente la rentabilidad de los propietarios de la aplicación (figura 1.1). Además, descuidar 
estas consideraciones puede provocar fallos en otros sistemas (por ejemplo, al participar 
involuntariamente en un ataque de denegación de servicio distribuido [DDoS]). Los aspectos ocultos 
de los requisitos no funcionales (el hecho de que sea mucho más difícil detectar si algo falta o está 
incompleto) los convierten en aún más peligrosos.

Existen múltiples aspectos no funcionales a considerar al trabajar en un sistema de software. En la 
práctica, todos ellos son importantes y deben abordarse responsablemente durante el proceso de 
desarrollo. Este libro se centra en uno de ellos: la seguridad. Aprenderás cómo proteger tu 
aplicación, paso a paso, utilizando Spring Security.

Este capítulo te mostrará la visión general de los conceptos relacionados con la seguridad. A lo 
largo del libro trabajaremos con ejemplos prácticos, y cuando sea apropiado, haré referencia a la 
descripción que doy en este capítulo. Donde corresponda, también proporcionaré detalles adicionales. 
De vez en cuando, encontrarás referencias a otros materiales (libros, artículos y documentación) 
sobre temas específicos útiles para una lectura posterior.

## 1.1 Descubriendo Spring Security

Esta sección analiza la relación entre Spring Security y Spring. Primero, es importante comprender 
el vínculo entre ambos antes de comenzar a usarlos. Si revisamos el sitio web oficial 
(https://spring.io/projects/spring-security), podemos ver que Spring Security se describe como un 
potente y altamente personalizable marco para la autenticación y el control de acceso. Simplemente 
diría que es un marco que simplifica enormemente la aplicación (o integración) de la seguridad en 
aplicaciones Spring.

Spring Security es la opción principal para implementar seguridad a nivel de aplicación en 
aplicaciones Spring. En general, su propósito es ofrecer una forma altamente personalizable de 
implementar autenticación, autorización y protección contra ataques comunes. Spring Security es un 
software de código abierto publicado bajo la licencia Apache 2.0. Puedes acceder a su código fuente 
en GitHub en http://mng.bz/vPmJ. Recomiendo encarecidamente que también contribuyas al proyecto.

NOTA: Puedes usar Spring Security tanto para servlets web estándar y aplicaciones reactivas, como 
para aplicaciones no web. En este libro, usaremos Spring Security con las últimas versiones 
compatibles a largo plazo de Java, Spring y Spring Boot (Java 21, Spring 6 y Spring Boot 3). Sin 
embargo, todos los ejemplos del libro también funcionan con Java 17, la versión anterior compatible 
a largo plazo.

Puedo suponer que si has abierto este libro, es porque trabajas con aplicaciones Spring y estás 
interesado en protegerlas. Spring Security es muy probablemente la mejor opción para ti. Es la 
solución estándar para implementar seguridad a nivel de aplicación en aplicaciones Spring. Sin 
embargo, Spring Security no protege automáticamente tu aplicación. No es una especie de panacea 
mágica que garantice una aplicación libre de vulnerabilidades. Los desarrolladores deben entender 
cómo configurar y personalizar Spring Security según las necesidades de sus aplicaciones. Cómo 
hacerlo depende de muchos factores, desde los requisitos funcionales hasta la arquitectura.

Técnicamente, aplicar seguridad con Spring Security en aplicaciones Spring es sencillo. Ya has 
implementado aplicaciones Spring, así que sabes que la filosofía del framework comienza con la 
gestión del contexto de Spring. Definimos beans en el contexto de Spring para permitir que el 
framework los gestione según las configuraciones que especifiquemos.

Utilizas anotaciones para indicarle a Spring qué debe hacer: exponer endpoints, envolver métodos en
transacciones, interceptar métodos en aspectos, etc. Lo mismo ocurre con las configuraciones de 
Spring Security, que es donde entra en juego este framework. Quieres usar anotaciones, beans y, en 
general, un estilo de configuración al modo Spring de forma cómoda al definir la seguridad a nivel 
de aplicación. En una aplicación Spring, el comportamiento que necesitas proteger está definido por 
métodos.

Para pensar en la seguridad a nivel de aplicación, puedes considerar tu hogar y la forma en que 
permites el acceso a él. ¿Colocas la llave debajo del felpudo de entrada? ¿Tienes siquiera una llave
para tu puerta principal? El mismo concepto se aplica a las aplicaciones, y Spring Security te ayuda
a desarrollar esta funcionalidad. Es un rompecabezas que ofrece muchas opciones para construir la 
imagen exacta que describe tu sistema. Puedes optar por dejar tu casa completamente sin protección, 
o decidir no permitir que todos entren en tu hogar. La forma en que configures la seguridad puede 
ser sencilla, como esconder la llave debajo del felpudo, o más compleja, como elegir varios sistemas
de alarma, cámaras de video y cerraduras. En tus aplicaciones, tienes las mismas opciones, pero, 
como en la vida real, cuanto más complejidad añadas, más costosa será. En una aplicación, este costo 
se refiere a cómo la seguridad afecta la mantenibilidad y el rendimiento.

Pero, ¿cómo se utiliza Spring Security con aplicaciones Spring? En general, a nivel de aplicación, 
uno de los casos de uso más frecuentes es decidir si se permite a alguien realizar una acción o 
acceder a ciertos datos. Según las configuraciones realizadas, se escriben componentes de Spring 
Security que interceptan las solicitudes y aseguran que quien las realiza tenga permiso para acceder 
a los recursos protegidos. El desarrollador configura estos componentes para que hagan exactamente 
lo deseado. Si instalas un sistema de alarma, eres tú quien debe asegurarse de que esté configurado 
también para las ventanas, y no solo para las puertas. Si olvidas configurarlo para las ventanas, 
no es culpa del sistema de alarma que no se active cuando alguien fuerce una ventana.

Otras responsabilidades de los componentes de Spring Security están relacionadas con el almacenamiento
y la transmisión de datos entre diferentes partes del sistema. Al interceptar llamadas a estas partes,
los componentes pueden actuar sobre los datos. Por ejemplo, al almacenar datos, estos componentes 
pueden aplicar algoritmos de cifrado o de hash. Estas codificaciones hacen que los datos solo sean 
accesibles para entidades privilegiadas. En una aplicación Spring, el desarrollador debe añadir y 
configurar un componente para realizar esta tarea donde sea necesario. Spring Security nos proporciona 
un contrato que indica lo que el framework requiere implementar, y nosotros escribimos la 
implementación según el diseño de la aplicación. Lo mismo se puede decir respecto a la transmisión 
de datos.

En implementaciones reales, encontrarás casos en los que dos componentes que se comunican no se 
confían mutuamente. ¿Cómo puede el primero saber que el segundo envió un mensaje específico y no fue 
otra persona quien lo hizo? Imagina que tienes una llamada telefónica con alguien a quien debes dar 
información privada. ¿Cómo te aseguras de que en realidad esté al otro lado una persona autorizada 
para recibir esos datos, y no alguien más? La misma situación es válida para tu aplicación. Spring 
Security proporciona componentes que te permiten resolver estos problemas de varias maneras, pero 
debes saber qué parte configurar y luego implementarla en tu sistema. De este modo, Spring Security 
intercepta los mensajes y se asegura de validar la comunicación antes de que la aplicación utilice 
cualquier tipo de dato enviado o recibido.

Como cualquier framework, uno de los propósitos principales de Spring es permitirte escribir menos 
código para implementar la funcionalidad deseada. Esto es exactamente lo que hace Spring Security: 
complementa a Spring como framework ayudándote a escribir menos código para realizar uno de los 
aspectos más críticos de una aplicación: la seguridad. Spring Security ofrece funcionalidad 
predefinida para ayudarte a evitar escribir código repetitivo o lógica que se repite de una 
aplicación a otra. Sin embargo, también te permite configurar cualquiera de sus componentes, 
ofreciendo gran flexibilidad. Para resumir brevemente esta discusión:

- Utilizas Spring Security para integrar la seguridad a nivel de aplicación en tus aplicaciones 
siguiendo el enfoque de Spring. Con esto me refiero a que usas anotaciones, beans, el Lenguaje de 
Expresiones de Spring (SpEL), etc.
- Spring Security es un framework que te permite construir seguridad a nivel de aplicación. Sin 
embargo, depende de ti, el desarrollador, entender y usar correctamente Spring Security. Por sí solo,
Spring Security no protege una aplicación ni los datos sensibles en reposo o en tránsito.
- Este libro te proporciona la información necesaria para usar Spring Security de forma efectiva.

### 1.2 Alternativas a Spring Security
Este libro trata sobre Spring Security, pero como con cualquier solución, prefiero siempre tener una
visión general amplia. Nunca olvides conocer las alternativas disponibles para cualquier opción. 
Una de las cosas que he aprendido con el tiempo es que no existe un bien o mal absoluto. ¡El dicho 
“todo es relativo” también se aplica aquí!

No encontrarás muchas alternativas a Spring Security cuando se trata de proteger una aplicación 
Spring. Una alternativa que podrías considerar es Apache Shiro (https://shiro.apache.org).  Ofrece 
flexibilidad en la configuración y es fácil de integrar con aplicaciones Spring y Spring Boot. A 
veces, Apache Shiro resulta una buena alternativa al enfoque de Spring Security.

Si ya has trabajado con Spring Security, te resultará fácil y cómodo aprender Apache Shiro. Ofrece 
sus propias anotaciones y un diseño para aplicaciones web basado en filtros HTTP, lo que simplifica 
mucho el trabajo. Además, puedes proteger más que solo aplicaciones web con Shiro, desde pequeñas 
aplicaciones de línea de comandos y móviles hasta aplicaciones empresariales a gran escala. Aunque 
es sencillo, es lo suficientemente potente para usarse en una amplia gama de funciones, desde 
autenticación y autorización hasta criptografía y gestión de sesiones.

Sin embargo, Apache Shiro podría ser demasiado ligero para las necesidades de tu aplicación. Spring 
Security no es solo un martillo, sino un conjunto completo de herramientas. Ofrece una mayor variedad 
de posibilidades y está diseñado específicamente para aplicaciones Spring. Además, se beneficia de 
una comunidad más grande de desarrolladores activos y se mejora continuamente.

La seguridad del software se refiere a la práctica de proteger aplicaciones y sistemas frente a 
accesos no autorizados, modificaciones, destrucción o interceptación de datos sensibles. Los sistemas
informáticos gestionan grandes cantidades de datos, muchos de los cuales pueden considerarse 
confidenciales, especialmente en contextos como el establecido por el Reglamento General de Protección
de Datos (RGPD) en Europa. Cualquier información que un usuario considere privada —como números de 
teléfono, correos electrónicos, identificaciones o datos bancarios— debe estar protegida para que 
solo las personas autorizadas puedan acceder, modificar o transmitirla. En esencia, la seguridad del 
software garantiza la confidencialidad, integridad y disponibilidad de la información, asegurando 
que no sea vulnerada en reposo ni en tránsito.

NOTA El RGPD generó gran impacto a nivel mundial tras su introducción en 2018. En general, representa 
un conjunto de leyes europeas relacionadas con la protección de datos que otorgan a las personas 
mayor control sobre su información privada. El RGPD se aplica a los responsables de sistemas que 
tengan usuarios en Europa. Los propietarios de tales aplicaciones arriesgan sanciones significativas 
si no cumplen con las regulaciones establecidas.

Aplicamos la seguridad en capas, y cada capa requiere un enfoque diferente. Compara estas capas con 
un castillo protegido (figura 1.2). Un pirata informático debe sortear varios obstáculos para obtener
los recursos gestionados por la aplicación. Cuanto mejor protejas cada capa, menor será la 
probabilidad de que una persona con intenciones maliciosas acceda a los datos o realice operaciones 
no autorizadas.

La seguridad es un tema complejo. En el caso de un sistema de software, la seguridad no existe 
únicamente a nivel de aplicación. Por ejemplo, en lo que respecta a la red, existen problemas que 
deben considerarse y prácticas específicas que deben aplicarse, mientras que en el almacenamiento, 
es otra discusión completamente distinta. De igual forma, hay una filosofía diferente en cuanto al 
despliegue, y así sucesivamente. Spring Security es un framework que pertenece a la seguridad a nivel 
de aplicación. En esta sección, obtendrás una visión general de este nivel de seguridad y sus 
implicaciones.

La seguridad a nivel de aplicación (figura 1.3) se refiere a todo lo que una aplicación debe hacer 
para proteger el entorno en el que se ejecuta, así como los datos que procesa y almacena. Y no se 
trata solo de los datos que la aplicación afecta o utiliza directamente. Una aplicación podría 
contener vulnerabilidades que permitan a un individuo malintencionado afectar a todo el sistema. 
Para ser más claros, analicemos algunos casos prácticos. Consideraremos una situación en la que 
desplegamos un sistema como se muestra en la figura 1.4. Esta situación es común en un sistema 
diseñado con una arquitectura de microservicios, especialmente si se despliega en múltiples zonas de 
disponibilidad en la nube.

NOTA Si estás interesado en implementar aplicaciones Spring eficientes orientadas al cloud, recomiendo
encarecidamente Cloud Native Spring in Action de Thomas Vitale (Manning, 2022). En este libro, el 
autor se centra en todos los aspectos necesarios que un profesional debe conocer para desarrollar 
aplicaciones Spring de alta calidad destinadas a despliegues en la nube.

Con este tipo de servicios y arquitecturas de microservicios, podemos encontrarnos con diversas 
vulnerabilidades, por lo que debes tener precaución. Como se mencionó anteriormente, la seguridad es 
un aspecto transversal que se diseña en múltiples capas. Al abordar las preocupaciones de seguridad 
de una de las capas, la mejor práctica es asumir que la capa superior no existe. Piensa en la analogía
con el castillo de la figura 1.2. Si gestionas la capa con 30 soldados, quieres prepararlos para ser 
lo más fuertes posible, y lo haces a pesar de saber que, para llegar a ellos, alguien primero tendría
que cruzar el puente llameante.

Con esto en mente, consideremos que un individuo con intenciones maliciosas podría iniciar sesión en 
la máquina virtual (VM) que aloja la primera aplicación. Supongamos también que la segunda aplicación 
no valida las solicitudes enviadas por la primera. El atacante podría entonces explotar esta 
vulnerabilidad y controlar la segunda aplicación haciéndose pasar por la primera.

Además, considera que desplegamos los dos servicios en ubicaciones diferentes. En ese caso, el 
atacante no necesita acceder directamente a ninguna de las VM, ya que puede actuar directamente en 
medio de las comunicaciones entre ambas aplicaciones.

NOTA Una zona de disponibilidad (AZ en la figura 1.4) en términos de despliegue en la nube es un 
centro de datos independiente. Este centro de datos está situado lo suficientemente lejos 
geográficamente (y tiene infraestructuras independientes) de otros centros de datos de la misma 
región, de modo que si una zona de disponibilidad falla, la probabilidad de que las demás también 
fallen es mínima. En cuanto a la seguridad, un aspecto importante es que el tráfico entre dos centros 
de datos diferentes generalmente requiere una atención especial porque a menudo transita a través de
una red pública.

He mencionado anteriormente la autenticación y la autorización. Estos conceptos están presentes en 
la mayoría de las aplicaciones. A través de la autenticación, una aplicación identifica a un usuario 
(una persona o otra aplicación). El propósito de identificar a los usuarios es poder decidir 
posteriormente qué se les permite hacer: eso es la autorización. Ofreceré muchos detalles sobre 
autenticación y autorización, comenzando en el capítulo 3 y continuando a lo largo del libro. En una 
aplicación, a menudo surge la necesidad de implementar autorización en diferentes escenarios. 
Considera otra situación: la mayoría de las aplicaciones tienen restricciones sobre el acceso del 
usuario a ciertas funcionalidades. Lograr esto implica primero la necesidad de identificar quién 
realiza la solicitud a una función específica: eso es autenticación. También necesitamos conocer sus 
privilegios para permitirle usar esa parte del sistema. A medida que el sistema se vuelve más 
complejo, encontrarás diferentes situaciones que requieren una implementación específica relacionada 
con la autenticación y la autorización.

Por ejemplo, ¿qué sucede si deseas autorizar un componente específico del sistema para acceder a un 
subconjunto de datos u operaciones en nombre del usuario? Supongamos que una impresora necesita acceso
para leer los documentos del usuario. ¿Deberías compartir simplemente las credenciales del usuario 
con la impresora? Eso le otorgaría a la impresora más permisos de los necesarios y, además, expondría
las credenciales del usuario. ¿Existe una forma adecuada de hacerlo sin suplantar al usuario? Estas 
son preguntas fundamentales que surgen al desarrollar aplicaciones: cuestiones que no solo queremos 
responder, sino para las cuales verás soluciones con Spring Security en este libro.

Dependiendo de la arquitectura del sistema, encontrarás mecanismos de autenticación y autorización 
tanto a nivel del sistema completo como en cada uno de sus componentes. Como verás más adelante, con 
Spring Security a veces preferirás usar autorización incluso para diferentes niveles dentro del mismo
componente. En el capítulo 11 se profundizará en la seguridad a nivel de métodos, que aborda 
precisamente este aspecto. El diseño se vuelve aún más complejo cuando se trabaja con un conjunto 
predefinido de roles y autoridades.

También quiero llamar tu atención sobre el almacenamiento de datos. Los datos en reposo aumentan la 
responsabilidad de la aplicación. Tu aplicación no debería almacenar todos sus datos en un formato 
legible. A veces es necesario mantener los datos cifrados con una clave privada o aplicar funciones 
de hash. Secretos como credenciales y claves privadas también se consideran datos en reposo y deben 
almacenarse cuidadosamente, generalmente en un almacén de secretos (secrets vault).

NOTA Clasificamos los datos como “en reposo” o “en tránsito”. En este contexto, los datos en reposo 
se refieren a datos almacenados en un dispositivo de almacenamiento, es decir, datos persistentes. 
Los datos en tránsito se refieren a toda la información que se intercambia de un punto a otro. Por 
lo tanto, deben aplicarse diferentes medidas de seguridad según el tipo de dato.

Finalmente, una aplicación en ejecución también debe gestionar su memoria interna. Puede parecer 
extraño, pero los datos almacenados en el heap de la aplicación también pueden representar 
vulnerabilidades. A veces, el diseño de clases permite que la aplicación almacene datos sensibles, 
como credenciales o claves privadas, durante mucho tiempo. En tales casos, alguien con privilegios 
para realizar un volcado de memoria (heap dump) podría obtener estos datos y usarlos con fines 
maliciosos.

Con esta breve descripción de diferentes escenarios, espero haber proporcionado una visión general 
de lo que entendemos por seguridad a nivel de aplicación, así como haber ilustrado la complejidad 
del tema. La seguridad del software es un asunto complejo. Una persona que desee convertirse en 
experta en este campo necesitaría comprender (y aplicar) y luego probar soluciones para todas las 
capas que colaboran en un sistema. Sin embargo, en este libro nos centraremos únicamente en presentar
todos los detalles que necesitas entender específicamente sobre Spring Security. Descubrirás dónde 
se aplica este framework y dónde no, cómo ayuda y por qué deberías usarlo. Por supuesto, lo haremos 
con ejemplos prácticos que podrás adaptar a tus propios casos de uso.

## 1.3 ¿Por qué es importante la seguridad?
La mejor forma de entender la importancia de la seguridad es desde tu perspectiva como usuario. Al 
igual que cualquier persona, usas aplicaciones que tienen acceso a tus datos, pueden modificarlos, 
usarlos o exponerlos. Piensa en todas las aplicaciones que utilizas, desde tu correo electrónico 
hasta tus cuentas de servicios bancarios en línea. ¿Cómo valorarías la sensibilidad de los datos que 
gestionan todos estos sistemas? ¿Y las acciones que puedes realizar a través de ellos? Al igual que 
con los datos, algunas acciones son más importantes que otras. Quizás no te importe mucho si alguien 
logra leer algunos de tus correos, pero seguramente sí te preocuparía si otra persona pudiera vaciar 
tu cuenta bancaria.

Una vez que hayas reflexionado sobre la seguridad desde tu punto de vista, intenta obtener una imagen
más objetiva. Los mismos datos o acciones pueden tener un grado diferente de sensibilidad para otras
personas. Algunas podrían preocuparse mucho más que tú si acceden a su correo y pueden leer sus 
mensajes. Tu aplicación debe asegurarse de proteger todo con el grado adecuado de acceso. Cualquier 
fuga que permita el uso de datos, funcionalidades o la propia aplicación para afectar a otros 
sistemas se considera una vulnerabilidad, y debes solucionarla.

No considerar suficientemente la seguridad tiene un costo que seguramente no estás dispuesto a pagar.
En general, se trata del dinero. Pero el costo puede variar, y hay múltiples formas en las que puedes
perder rentabilidad. No se trata solo de perder dinero de una cuenta bancaria o usar un servicio sin
pagar. Estas cosas implican un costo, sí. La imagen de una marca o empresa también es valiosa, y 
perder una buena reputación puede ser caro, a veces incluso más que los gastos directos resultantes 
del aprovechamiento de una vulnerabilidad en el sistema. La confianza que los usuarios tienen en tu 
aplicación es uno de sus activos más valiosos y puede marcar la diferencia entre el éxito o el fracaso.

Aquí tienes algunos ejemplos ficticios. Piensa en cómo los verías como usuario. ¿Cómo podrían afectar
a la organización responsable del software?

- Una aplicación de back-office debería gestionar los datos internos de una organización, pero, de 
alguna manera, se filtra información.
- Los usuarios de una aplicación de transporte compartido observan que se les descuenta dinero de 
sus cuentas por viajes que no han realizado.
- Tras una actualización, los usuarios de una aplicación bancaria móvil ven transacciones que 
pertenecen a otros usuarios.

En la primera situación, la organización que utiliza el software, así como sus empleados, pueden 
verse afectados. En algunos casos, la empresa podría ser responsable y perder una cantidad 
significativa de dinero. En este escenario, los usuarios no tienen opción de cambiar la aplicación,
pero la organización sí puede decidir cambiar al proveedor del software.

En el segundo caso, los usuarios probablemente optarán por cambiar de proveedor del servicio. La 
imagen de la empresa desarrolladora de la aplicación se vería gravemente afectada. La pérdida 
económica en este caso es mucho menor que el daño a la reputación. Incluso si se devuelven los pagos 
a los usuarios afectados, la aplicación perderá algunos clientes. Esto impacta en la rentabilidad y 
puede incluso llevar a la quiebra. En el tercer caso, el banco podría enfrentar consecuencias graves
en términos de confianza, así como repercusiones legales.

En la mayoría de estos escenarios, invertir en seguridad es más seguro que asumir las consecuencias 
si alguien explota una vulnerabilidad en tu sistema. Para todos los ejemplos, una pequeña debilidad 
podría causar cada uno de estos resultados. En el primer ejemplo, podría ser una autenticación 
defectuosa o una falsificación de solicitudes entre sitios (CSRF). En los segundos y terceros 
ejemplos, podría ser la falta de control de acceso a métodos. Y en todos los casos, podría ser una 
combinación de vulnerabilidades.

Por supuesto, podemos ir aún más allá y hablar de la seguridad en sistemas relacionados con la 
defensa. Si consideras importante el dinero, ¡añade vidas humanas al costo! ¿Puedes imaginar qué 
podría ocurrir si se viera afectado un sistema de salud? ¿Qué pasaría con los sistemas que controlan
plantas nucleares? Puedes reducir cualquier riesgo invirtiendo desde el principio en la seguridad de
tu aplicación y asignando suficiente tiempo para que profesionales de seguridad desarrollen y prueben
tus mecanismos de protección.

NOTA La lección aprendida de quienes fallaron antes que tú es que el costo de un ataque suele ser 
mayor que la inversión necesaria para evitar la vulnerabilidad. En el resto de este libro, verás 
ejemplos de cómo aplicar Spring Security para evitar situaciones como las descritas. Supongo que 
nunca se podrán decir suficientes palabras sobre la importancia de la seguridad. Cuando tengas que 
tomar decisiones que comprometan la seguridad de tu sistema, intenta estimar tus riesgos correctamente.

Este libro ofrece un enfoque práctico para aprender Spring Security. A lo largo de sus capítulos, 
profundizarás en este framework mediante ejemplos que van de lo simple a lo complejo. Aprenderás a 
implementar y personalizar mecanismos de autenticación y autorización, configurar endpoints seguros 
y proteger tu aplicación contra ataques comunes como la falsificación de solicitudes entre sitios 
(CSRF) y ataques de scripting cruzado (XSS). También explorarás la configuración de seguridad a nivel
de método, el uso de CORS, y la implementación de sistemas OAuth2/OpenID Connect, incluyendo 
servidores de autorización y recursos. Además, se cubre la integración con bases de datos, el manejo
seguro de contraseñas, la gestión de sesiones y la creación de filtros de seguridad personalizados. 
Finalmente, aprenderás a escribir pruebas para tus configuraciones de seguridad, todo actualizado 
para Spring Boot 3 y la última versión de Spring Security.

- La arquitectura y los componentes básicos de Spring Security, y cómo utilizarlos para proteger tu 
aplicación
- Autenticación y autorización con Spring Security, incluyendo los flujos OAuth 2 y OpenID Connect, 
y cómo aplicarlos en una aplicación lista para producción
- Cómo implementar seguridad con Spring Security en diferentes capas de tu aplicación
- Diferentes estilos de configuración y buenas prácticas para usarlos en tu proyecto
- Uso de Spring Security en aplicaciones reactivas
- Pruebas de las implementaciones de seguridad

Para facilitar el aprendizaje de cada concepto descrito, trabajaremos con múltiples ejemplos sencillos.
Al finalizar, sabrás cómo aplicar Spring Security en los escenarios más prácticos, y entenderás 
cuándo hacerlo y cuáles son las mejores prácticas. También recomiendo encarecidamente que trabajes 
con todos los ejemplos que acompañan a las explicaciones.

- Spring Security es la opción principal para proteger aplicaciones Spring. Ofrece una gran cantidad
de alternativas aplicables a diferentes estilos y arquitecturas.
- Debes aplicar la seguridad en capas en tu sistema, y para cada capa es necesario utilizar prácticas
diferentes.
- La seguridad es un aspecto transversal que debes considerar desde el inicio de un proyecto de 
software.
- Por lo general, el costo de un ataque es mayor que la inversión necesaria para evitar 
vulnerabilidades desde el principio.
- A veces, los errores más pequeños pueden causar daños significativos. Por ejemplo, exponer datos 
sensibles a través de registros o mensajes de error es una forma común de introducir vulnerabilidades
en tu aplicación. 