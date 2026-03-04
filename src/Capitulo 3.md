# ***PART 2***
# Configuración de autenticación

La autenticación se sitúa en primera línea de cualquier aplicación segura, determinando quién 
interactúa con ella. En la segunda parte de este libro, nos adentramos directamente en el corazón de
este mecanismo.

El capítulo 3 te familiariza con la gestión de usuarios en Spring Security, incluyendo los contratos
esenciales UserDetails y GrantedAuthority, y los matices para guiar a Spring Security en el manejo 
de usuarios.

El capítulo 4 profundiza en la seguridad de contraseñas, explorando el contrato PasswordEncoder, cómo
crear tu propio codificador y el uso del módulo Crypto de Spring Security para cifrado y generación 
de claves.

El capítulo 5 introduce el papel fundamental de los filtros en Spring Security. Aprenderás a 
integrarlos, ordenarlos y emplear una variedad de filtros para mejorar la seguridad de tu aplicación.

El capítulo 6 lo une todo. Aquí descubrirás la esencia del AuthenticationProvider, profundizarás en 
la lógica de autenticación personalizada y te familiarizarás con diferentes métodos de autenticación
de inicio de sesión, incluyendo los enfoques HTTP Basic y basados en formularios.

Al final de esta sección, tendrás una comprensión sólida de las capas y mecánicas intrincadas de la
autenticación en aplicaciones Spring.

# CAPITULO 3
## Gestión de usuarios

Este capítulo abarca
- Describir un usuario con la interfaz UserDetails
- Usar el UserDetailsService en el flujo de autenticación
- Crear una implementación personalizada del UserDetailsService
- Crear una implementación personalizada de UserDetailsManager
- Usar el JdbcUserDetailsManager en el flujo de autenticación

Uno de mis colegas de la universidad cocina bastante bien. No es un chef en un restaurante elegante,
pero le apasiona mucho cocinar. Un día, mientras compartíamos opiniones en una conversación, le 
pregunté cómo logra recordar tantas recetas. Me dijo que eso es fácil. “No tienes que recordar toda
la receta, sino la forma en que los ingredientes básicos combinan entre sí. Es como algunos contratos
del mundo real que te indican qué puedes mezclar o qué no debes mezclar. Luego, para cada receta, 
solo recuerdas algunos trucos”. 

Esta analogía es similar a la forma en que funcionan las arquitecturas. Con cualquier framework 
robusto, usamos contratos para desacoplar las implementaciones del framework de la aplicación 
construida sobre él. En Java, usamos interfaces para definir esos contratos. Un programador es 
similar a un chef, que sabe cómo los ingredientes funcionan juntos para elegir la implementación 
adecuada. El programador conoce las abstracciones del framework y las utiliza para integrarse con él.

Este capítulo trata sobre entender en detalle uno de los roles fundamentales que encontraste en el 
primer ejemplo que trabajamos en el capítulo 2: el `UserDetailsService`.  Junto con el 
`UserDetailsService`, discutiremos las siguientes interfaces (contratos):

- UserDetails, que describe al usuario para Spring Security.
- GrantedAuthority, que nos permite definir las acciones que el usuario puede ejecutar.
- UserDetailsManager, que extiende el contrato de UserDetailsService.
Además del comportamiento heredado, también describe acciones como crear un usuario, y modificar o 
eliminar la contraseña de un usuario.

Del capítulo 2, ya tienes una idea de los roles que desempeñan UserDetailsService y PasswordEncoder 
en el proceso de autenticación. Pero solo discutimos cómo integrar una instancia definida por ti en 
lugar de usar la configurada por defecto en Spring Boot. Tenemos más detalles por abordar, como
- Las implementaciones proporcionadas por Spring Security y cómo usarlas
- Cómo definir una implementación personalizada de los contratos y cuándo hacerlo
- Formas de implementar interfaces que encuentras en aplicaciones del mundo real
- Buenas prácticas para usar estas interfaces

El plan es comenzar con cómo Spring Security entiende la definición de un usuario. Para ello, 
analizaremos los contratos `UserDetails` y `GrantedAuthority`. Luego, detallaremos el 
`UserDetailsService` y cómo UserDetailsManager extiende este contrato. Aplicarás implementaciones 
para estas interfaces (por ejemplo, `InMemoryUserDetailsManager`, `JdbcUserDetailsManager` y 
`LdapUserDetailsManager`). Cuando estas implementaciones no sean adecuadas para tu sistema, escribirás
una implementación personalizada.

### 3.1 Implementación de la autenticación en Spring Security

En el capítulo anterior, comenzamos con Spring Security. En el primer ejemplo, discutimos cómo 
Spring Boot especifica algunos valores predeterminados que definen cómo funciona inicialmente una 
nueva aplicación. También aprendiste cómo anular estos valores predeterminados utilizando varias 
alternativas que a menudo encontramos en aplicaciones. Sin embargo, solo consideramos la superficie 
de estos aspectos, para que tengas una idea general de lo que trataremos. En este capítulo, así como
en los capítulos 4 y 5, analizaremos estas interfaces con mayor detalle, junto con sus diferentes 
implementaciones y dónde puedes encontrarlas en aplicaciones del mundo real.

Se presenta el flujo de autenticación en Spring Security. Esta arquitectura es la base del proceso 
de autenticación tal como lo implementa Spring Security. Es importante comprenderla porque dependerás
de ella en cualquier implementación con Spring Security. Observarás que discutimos partes de esta 
arquitectura en casi todos los capítulos de este libro. La verás con tanta frecuencia que 
probablemente la aprenderás de memoria, lo cual es positivo. Si conoces esta arquitectura, eres como
un chef que conoce sus ingredientes y puede preparar cualquier receta.

A continuacion, `UserDetailsService` y `PasswordEncoder`. Estos dos componentes se centran en la parte 
del flujo que suelo denominar "la parte de gestión de usuarios". En este capítulo, 
`UserDetailsService` y `PasswordEncoder` son los componentes que interactúan directamente con los 
detalles del usuario y sus credenciales. Analizaremos `PasswordEncoder` en detalle en el capítulo 4.

1. El `Authentication filter` captura la peticion entrante
2. La responsabilidad del autenticador es pasarlo al `Authentication Manager`
3. El `Authentication Manager` involucra al `Authetication provider` para llevar a cabo los 
procedimientos de autenticación.
4. El `Authentication provider` encuentra al usuario mediante el `User Details Service` y valida la 
contraseña utilizando un `password encoder`.
5. El resultado de la autenticacion es retornado al Authentication filter -> (Security context)
6. La información sobre la entidad autenticada se almacena dentro del contexto de seguridad.

El flujo de autenticación de Spring Security. El `AuthenticationFilter` captura la solicitud entrante 
y transfiere la tarea de autenticación al `AuthenticationManager`. El `AuthenticationManager`, a su vez,
utiliza un `Authetication provider` para llevar a cabo el proceso de autenticación. Para verificar
el nombre de usuario y la contraseña, el `AuthenticationProvider` depende de un `UserDetailsService`
y un `PasswordEncoder`.

Como parte de la gestión de usuarios, utilizamos las interfaces `UserDetailsService` y 
`UserDetailsManager`. El `UserDetailsService` solo es responsable de recuperar al usuario por su nombre
de usuario. Esta acción es la única necesaria para que el framework complete la autenticación. 
`UserDetailsManager` añade funcionalidades para crear, modificar o eliminar usuarios, lo cual es una 
funcionalidad requerida en la mayoría de las aplicaciones. La separación entre estos dos contratos 
es un excelente ejemplo del principio de segregación de interfaces. Separar las interfaces permite 
una mejor flexibilidad, ya que el framework no te obliga a implementar comportamientos que tu 
aplicación no necesita. Si la aplicación solo necesita autenticar usuarios, entonces implementar el 
contrato `UserDetailsService` es suficiente para cubrir la funcionalidad deseada.

Para gestionar a los usuarios, los componentes `UserDetailsService` y `UserDetailsManager` necesitan una
forma de representarlos. Spring Security ofrece el contrato UserDetails, que debes implementar para 
describir un usuario de la manera que el framework entiende. Como aprenderás en este capítulo, en 
Spring Security un usuario tiene un conjunto de privilegios, que son las acciones que se le permiten 
realizar. Trabajaremos extensamente con estos privilegios en los capítulos 7 al 12 al tratar sobre 
autorización. Pero por ahora, Spring Security representa las acciones que un usuario puede realizar 
mediante la interfaz GrantedAuthority. A menudo llamamos a estos privilegios authorities, y un 
usuario tiene uno o más de ellos. En la figura 3.2 se muestra una representación de la relación 
entre los componentes de la parte de gestión de usuarios del flujo de autenticación.


