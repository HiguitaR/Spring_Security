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

#### Flujo: 

`UserDetailsService`(1)---->`UserDetails`(2)o-----`GrantedAuthority`

`UserDetailsManager`(3)---->`UserDetailsService`

1. El UserDetailsService utiliza el contrato de UserDetails
2. Un UserDatails tiene 1 o mas autoridades
3. El UserDetailsManager extiende el contrato de UserDetailsService

Las dependencias entre los componentes involucrados en la gestión de usuarios son las siguientes: 
El `UserDetailsService` recupera los detalles de un usuario buscándolo por su nombre. El usuario está 
caracterizado por el contrato `UserDetails`. Cada usuario posee una o más autoridades, las cuales están 
representadas por la interfaz `GrantedAuthority`. Para incorporar operaciones como crear, eliminar o 
modificar la contraseña de un usuario, se utiliza el contrato `UserDetailsManager`, que amplía el 
`UserDetailsService`, para incluir estas funcionalidades.

Comprender los vínculos entre estos objetos en la arquitectura de Spring Security y las formas de 
implementarlos te ofrece una amplia gama de opciones para elegir al trabajar en aplicaciones. 
Cualquiera de estas opciones podría ser la pieza correcta en la aplicación en la que estás trabajando,
y necesitas tomar tu decisión sabiamente. Pero para poder elegir, primero necesitas saber de qué 
opciones dispones.

### 3.2 Descripción del usuario

En esta sección, aprenderás cómo describir a los usuarios de tu aplicación para que Spring Security
los entienda. Aprender a representar a los usuarios y hacer que el marco de trabajo los reconozca es
un paso esencial para construir un flujo de autenticación. En función del usuario, la aplicación toma
una decisión: si se permite o no una llamada a una funcionalidad determinada.

Para trabajar con usuarios, primero necesitas comprender cómo definir el prototipo del usuario en tu 
aplicación. Esta sección describe con ejemplos cómo establecer una plantilla para tus usuarios en 
una aplicación Spring Security.

Para Spring Security, la definición de un usuario debe cumplir con el contrato de UserDetails. Este 
contrato representa al usuario tal como lo entiende Spring Security. La clase de tu aplicación que 
describe al usuario debe implementar esta interfaz, y de esta manera, el marco de trabajo la reconoce.

#### 3.2.1 Descripción de usuarios con el contrato UserDetails

En esta sección, aprenderás cómo implementar la interfaz UserDetails para describir a los usuarios 
en tu aplicación. Analizaremos los métodos declarados por el contrato UserDetails para comprender 
cómo y por qué implementar cada uno de ellos. Comencemos primero examinando la interfaz, como se 
muestra en el siguiente listado.

The UserDetails interface:

```java
import java.io.Serializable;
import java.util.Collection;

public interface UserDetails extends Serializable {
    //Los dos primeros metodos retornan las credenciales del usuario
    String getUsername();
    String getPassword();
    //Devuelve las acciones que la aplicación permite al usuario realizar, como una colección de 
    // instancias de GrantedAuthority.
    Collection<? extends GranteAuthorities> getAuthorities;
    //Estos 4 metodos activan o desactivan la cuenta por razones diferentes
    boolean isAccountNonExpired();
    boolean isAccountNonLocked();
    boolean isCredentialsNonExpired();
    boolean isEnabled();
}
```

Los métodos `getUsername()` y `getPassword()` devuelven, como cabría esperar, el nombre de usuario y la 
contraseña. La aplicación utiliza estos valores en el proceso de autenticación, y son los únicos 
detalles relacionados con la autenticación en este contrato. Los otros cinco métodos se relacionan
con la autorización del usuario para acceder a los recursos de la aplicación.

En general, la aplicación debería permitir que un usuario realice algunas acciones que tengan 
sentido en el contexto de la aplicación. Por ejemplo, el usuario debería poder leer, escribir o 
eliminar datos. Decimos que un usuario tiene o no tiene el privilegio de realizar una acción, y una 
autoridad representa el privilegio que tiene el usuario. Implementamos el método `getAuthorities()` 
para devolver el grupo de autoridades concedidas a un usuario. 

`NOTA` Como aprenderás en el capítulo 6, Spring Security utiliza autoridades para referirse a 
privilegios específicos o a roles, que son grupos de privilegios. Para facilitar la lectura, en este
libro me referiré a los privilegios específicos como autoridades. 

Además, como se observa en el contrato UserDetails, un usuario puede: 
- Permitir que la cuenta caduque
- Bloquear la cuenta 
- Permitir que las credenciales caduquen 
- Deshabilitar la cuenta

Suponga que decide implementar estas restricciones de usuario en la lógica de su aplicación. En ese 
caso, necesita implementar los métodos `isAccountNonExpired()`, `isAccountNonLocked()`, 
`isCredentialsNonExpired()` y `isEnabled()`, de modo que aquellos que necesiten estar habilitados 
devuelvan true. No todas las aplicaciones tienen cuentas que caducan o se bloquean bajo ciertas 
condiciones. Si no necesita implementar estas funcionalidades en su aplicación, simplemente puede 
hacer que estos cuatro métodos devuelvan true.

`NOTA` Los nombres de los últimos cuatro métodos en la interfaz `UserDetails` pueden sonar extraños.
Se podría argumentar que no están bien elegidos en términos de codificación limpia y mantenibilidad. 
Por ejemplo, el nombre `isAccountNonExpired()` parece una doble negación, y a primera vista podría 
generar confusión. Pero analicemos atentamente los nombres de los cuatro métodos. Están nombrados 
de tal manera que todos devuelven false cuando la autorización debería fallar y true en caso 
contrario. Este es el enfoque correcto porque la mente humana tiende a asociar la palabra "false" 
con escenarios negativos y la palabra "true" con escenarios positivos.

#### 3.2.2 Detallando sobre el contrato GrantedAuthority

Como observaste en la definición de la interfaz UserDetails en la sección 3.2.1, las acciones 
concedidas a un usuario se denominan autoridades. En los capítulos 7 al 12, escribiremos 
configuraciones de autorización basadas en estas autoridades del usuario. Por lo tanto, es esencial
saber cómo definirlas.

Las autoridades representan lo que el usuario puede hacer en tu aplicación. Sin ellas, todos los 
usuarios serían iguales. Aunque existen aplicaciones simples en las que los usuarios son iguales, en 
la mayoría de los escenarios prácticos, una aplicación define varios tipos de usuarios. Una 
aplicación podría tener usuarios que solo pueden leer información específica, mientras que otros 
también pueden modificar los datos. Y necesitas que tu aplicación los diferencie, según los 
requisitos funcionales de la aplicación, que determinan las autoridades que necesita un usuario.

Para describir las autoridades en Spring Security, utilizas la interfaz GrantedAuthority.

Antes de discutir la implementación de UserDetails, comprendamos la interfaz GrantedAuthority. 
Usamos esta interfaz en la definición de los detalles del usuario. Representa un privilegio concedido
al usuario. Un usuario debe tener al menos una autoridad. A continuación, se muestra la 
implementación de la definición de GrantedAuthority:

```java
import java.io.Serializable;

public interface GrantedAuthority extends Serializable{
    String getAuthority();
}
```

Para crear una autoridad, solo necesitas encontrar un nombre para ese privilegio para poder referirte
a él más adelante al escribir las reglas de autorización. Por ejemplo, un usuario puede leer los 
registros gestionados por la aplicación o eliminarlos. Escribe las reglas de autorización basándote 
en los nombres que asignas a estas acciones.

En este capítulo, implementaremos el método `getAuthority()` para devolver el nombre de la autoridad 
como un String. La interfaz `GrantedAuthority` tiene un único método abstracto, y en este libro 
encontrarás a menudo ejemplos en los que usamos una expresión lambda para su implementación. Otra 
posibilidad es utilizar la clase `SimpleGrantedAuthority` para crear instancias de autoridad. La clase
`SimpleGrantedAuthority` ofrece una forma de crear instancias inmutables del tipo `GrantedAuthority`. 
Proporcionas el nombre de la autoridad al crear la instancia. En el siguiente fragmento de código, 
encontrarás dos ejemplos de implementación de una `GrantedAuthority`. Aquí utilizamos primero una 
expresión lambda y luego la clase `SimpleGrantedAuthority`:
```java
GrantedAuthority g1 = () -> "READ";
GrantedAuthority g2 = new SimpleGrantesAuthority("READ");
```

#### 3.2.3 Escribir una implementación mínima de UserDetails

En esta sección, escribirás tu primera implementación del contrato UserDetails. Comenzamos con una 
implementación básica en la que cada método devuelve un valor estático. Luego la modificamos para 
obtener una versión más realista, que te permita tener múltiples instancias de usuarios diferentes. 
Ahora que sabes cómo implementar las interfaces UserDetails y GrantedAuthority, podemos escribir la 
definición de usuario más simple para una aplicación.

Con una clase llamada DummyUser, implementaremos una descripción mínima de un usuario, como se 
muestra en el siguiente listado. Esta clase se utiliza principalmente para demostrar la 
implementación de los métodos del contrato UserDetails. Las instancias de esta clase siempre hacen 
ereferencia a un único usuario, "bill", que tiene la contraseña "12345" y una autoridad llamada "READ".

The DummyUser class:
```java
public class DummyUser implements UserDetails {
    @Override
    public String getUsername() {
        return "bill";
    }
    @Override
    public String getPassword() {
        return "12345";
    }
    // Omitted code
}
```

La clase del listado 3.2 implementa la interfaz `UserDetails` y necesita implementar todos sus métodos.
Aquí encontrarás la implementación de `getUsername()` y `getPassword()`. En este ejemplo, estos métodos 
solo devuelven un valor fijo para cada una de las propiedades.

A continuación, agregamos una definición para la lista de autoridades. El siguiente listado muestra 
la implementación del método `getAuthorities()`. Este método devuelve una colección con una única 
implementación de la interfaz `GrantedAuthority`.

Implementation of the getAuthorities() method:
```java
public class DummyUser implements UserDetails {
    // Omitted code
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(() -> "READ");
    }
    // Omitted code
}
```
Finalmente, debes agregar una implementación para los últimos cuatro métodos de la interfaz 
`UserDetails`. Para la clase `DummyUser`, estos métodos siempre devuelven true, lo que significa que 
el usuario está permanentemente activo y disponible para su uso. Puedes encontrar ejemplos en el 
siguiente listado.

Implementation of the last four UserDetails interface methods:
```java
public class DummyUser implements UserDetails {
    // Omitted code
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
    //ommited code
}
```
Por supuesto, esta implementación mínima implica que todas las instancias de la clase representan al 
mismo usuario. Es un buen punto de partida para comprender el contrato, pero no es algo que harías 
en una aplicación real. Para una aplicación real, deberías crear una clase que te permita generar 
instancias que puedan representar diferentes usuarios. En este caso, tu definición tendría al menos 
el nombre de usuario y la contraseña como atributos en la clase, como se muestra en el siguiente 
listado.

A more practical implementation of the UserDetails interface:
```java
public class SimpleUser implements UserDetails {
    private final String username;
    private final String password;

    public SimpleUser(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public String getUsername() {
        return this.username;
    }

    @Override
    public String getPassword() {
        return this.password;
    }
// Omitted code
}
```

#### 3.2.4 Usar un constructor para crear instancias del tipo UserDetails
Algunas aplicaciones son simples y no necesitan una implementación personalizada de la interfaz 
`UserDetails`. En esta sección, analizamos el uso de una clase constructora proporcionada por Spring 
Security para crear instancias de usuario simples. En lugar de declarar una clase adicional en tu 
aplicación, obtienes rápidamente una instancia que representa a tu usuario mediante la clase 
constructora `User`.

La clase `User` del paquete org.springframework.security.core.userdetails es una forma sencilla de 
construir instancias del tipo `UserDetails`. Mediante esta clase, puedes crear instancias inmutables 
de `UserDetails`. Debes proporcionar al menos un nombre de usuario y una contraseña, y el nombre de 
usuario no debe ser una cadena vacía. El siguiente ejemplo muestra cómo usar este constructor. Al 
construir el usuario de esta manera, no necesitas tener una implementación personalizada del contrato
`UserDetails`.

Constructing a user with the User builder class:
```java
UserDetails u = User.withUsername("bill")
                .password("12345")
                .authorities("read", "write")
                .accountExpired(false)
                .disabled(true)
                .build();
```

Con la lista anterior como ejemplo, profundicemos en la anatomía de la clase constructora `User`. El 
método `User.withUsername(String username)` devuelve una instancia de la clase constructora 
`UserBuilder`, anidada dentro de la clase `User`. Otra forma de crear el constructor es a partir de 
otra instancia existente de `UserDetails`. En el listado 3.7, la primera línea construye un 
`UserBuilder` a partir del nombre de usuario dado como cadena. Posteriormente, se muestra cómo crear 
un constructor a partir de una instancia ya existente de `UserDetails`.

Creating the User.UserBuilder instance:
```java
// Construye un usuario con su username
User.UserBuilder builder1 = User.withUsername("bill");
UserDetails u1 = builder1
                    .password("12345")
                    .authorities("read", "write")
        //El codificador de contraseñas es solo una función que realiza una codificación. 
                    .passwordEncoder(p -> encode(p))
                    .accountExpired(false)
                    .disabled(true)
        //Al final del proceso de construcción, llama al metodo build().
                    .build();
        //También puedes construir un usuario a partir de una instancia existente de UserDetails.
User.UserBuilder builder2 = User.withUserDetails(u);
UserDetails u2 = builder2.build();
```
Puedes ver con cualquiera de los constructores definidos en el listado 3.7 que es posible utilizar 
el constructor para obtener un usuario representado por el contrato UserDetails. Al final del proceso
de construcción, llamas al método build(). Este método aplica la función definida para codificar la 
contraseña si proporcionas una, construye la instancia de UserDetails y la devuelve.

`NOTA` Observa que el codificador de contraseñas se proporciona aquí como una 
`Function<String, String>` y no en la forma de la interfaz `PasswordEncoder` proporcionada por 
Spring Security. La única responsabilidad de esta función es transformar una contraseña a una 
codificación determinada. En la siguiente sección, discutiremos en detalle el contrato 
`PasswordEncoder` de Spring Security que usamos en el capítulo 2. Analizaremos con más detalle el 
contrato `PasswordEncoder` en el capítulo 4.

#### 3.2.5 Combinar múltiples responsabilidades relacionadas con el usuario
En la sección anterior, aprendiste cómo implementar la interfaz UserDetails. En escenarios del mundo
real, esto suele ser más complicado. En la mayoría de los casos, encuentras múltiples 
responsabilidades con las que un usuario se relaciona. Y si almacenas usuarios en una base de datos,
entonces en la aplicación necesitarías una clase para representar la entidad de persistencia también.
O si recuperas usuarios a través de un servicio web desde otro sistema, probablemente necesitarías 
un objeto de transferencia de datos (DTO) para representar las instancias del usuario. Suponiendo el
primer caso, uno simple pero también típico, consideremos que tenemos una tabla en una base de datos
SQL donde almacenamos los usuarios. Para hacer el ejemplo más breve, le asignamos a cada usuario 
solo una autoridad. El siguiente listado muestra la clase entidad que mapea la tabla.

Definir la clase entidad JPA para el usuario:
```java
@Entity
public class User {
    @Id
    private Long id;
    private String username;
    private String password;
    private String authority;
    // Omitted getters and setters
}
```
Si haces que la misma clase también implemente el contrato de Spring Security para los detalles del
usuario, la clase se vuelve más complicada. ¿Qué opinas sobre cómo se ve el código en el siguiente 
listado? En mi opinión, es un desastre. Me perdería en él.

La clase User tiene dos responsabilidades:
```java
@Entity
public class User implements UserDetails {
    @Id
    private int id;
    private String username;
    private String password;
    private String authority;
    @Override
    public String getUsername() {
        return this.username;
    }
    @Override
    public String getPassword() {
        return this.password;
    }
    public String getAuthority() {
        return this.authority;
    }
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(() -> authority);
    }
// Omitted code
}
```
La clase contiene anotaciones JPA, getters y setters, de los cuales tanto `getUsername()` como 
`getPassword()` anulan los métodos del contrato UserDetails. Tiene un método `getAuthority()` que 
devuelve un String, así como un método `getAuthorities()` que devuelve una Collection. El método 
`getAuthority()` es simplemente un getter en la clase, mientras que `getAuthorities()` implementa el 
método de la interfaz `UserDetails`. Las cosas se complican aún más al añadir relaciones con otras 
entidades. De nuevo, ¡este código no es nada amigable!

¿Cómo podemos escribir este código de forma más limpia? La raíz del aspecto confuso del ejemplo de 
código anterior es una mezcla de dos responsabilidades. Aunque es cierto que necesitas ambas en la 
aplicación, en este caso, nadie dice que debas ponerlas en la misma clase. Intentemos separarlas 
definiendo una clase aparte llamada `SecurityUser`, que adapte la clase User. Como muestra el 
siguiente listado, la clase `SecurityUser` implementa el contrato `UserDetails` y lo utiliza para 
integrar nuestro usuario en la arquitectura de `Spring Security`. La clase User se queda únicamente 
con su responsabilidad como entidad JPA.

Implementar la clase User únicamente como una entidad JPA:
```java
@Entity
public class User {
    @Id
    private int id;
    private String username;
    private String password;
    private String authority;
    // Omitted getters and setters
}
```
La clase User en el listado 3.10 tiene únicamente su responsabilidad como entidad JPA, y por lo 
tanto, se vuelve más legible. Si lees este código, ahora puedes concentrarte exclusivamente en los 
detalles relacionados con la persistencia, que no son importantes desde la perspectiva de 
Spring Security. En el siguiente listado, implementamos la clase SecurityUser para envolver la 
entidad User.

La clase SecurityUser implementa el contrato UserDetails:
```java
public class SecurityUser implements UserDetails {
    private final User user;
    public SecurityUser(User user) {
        this.user = user;
    }
    @Override
    public String getUsername() {
        return user.getUsername();
    }
    @Override
    public String getPassword() {
        return user.getPassword();
    }
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(() -> user.getAuthority());
    }
    // Omitted code
}
```
Como puedes observar, utilizamos la clase SecurityUser únicamente para mapear los detalles del 
usuario en el sistema al contrato UserDetails comprendido por Spring Security. Para indicar el 
hecho de que SecurityUser no tiene sentido sin una entidad User, hacemos que el campo sea final. 
Debes proporcionar el usuario a través del constructor. La clase SecurityUser adapta la clase de 
entidad User y añade el código necesario relacionado con el contrato de Spring Security sin 
mezclarlo en una entidad JPA, implementando así múltiples tareas diferentes.

NOTA Puedes encontrar diferentes enfoques para separar las dos responsabilidades. No pretendo decir
que el enfoque que presento en esta sección sea el mejor o el único. Por lo general, la forma en 
que eliges implementar el diseño de clases varía mucho de un caso a otro. Pero la idea principal 
es la misma: evita mezclar responsabilidades y trata de escribir tu código de forma lo más 
desacoplada posible para aumentar la mantenibilidad de tu aplicación.

### 3.3 Instruir a Spring Security sobre cómo gestionar usuarios

En la sección anterior, implementaste el contrato UserDetails para describir a los usuarios de 
forma que Spring Security los comprenda. Pero, ¿cómo gestiona Spring Security a los usuarios? ¿De 
dónde se obtienen al comparar credenciales, y cómo añades nuevos usuarios o modificas los 
existentes? En el capítulo 2, aprendiste que el framework define un componente específico al que el
proceso de autenticación delega la gestión de usuarios: la instancia UserDetailsService. Incluso 
definimos un UserDetailsService para anular la implementación predeterminada proporcionada por 
Spring Boot.

En esta sección, experimentarás con diversas formas de implementar la clase `UserDetailsService`. 
Comprenderás cómo funciona la gestión de usuarios al implementar la responsabilidad descrita por 
el contrato `UserDetailsService` en nuestro ejemplo. Después, descubrirás cómo la interfaz 
`UserDetailsManager` añade más comportamiento al contrato definido por `UserDetailsService`. Al final 
de esta sección, usaremos las implementaciones proporcionadas de la interfaz `UserDetailsManager` 
ofrecidas por Spring Security. Escribiremos un proyecto de ejemplo donde usaremos una de las 
implementaciones más conocidas proporcionadas por Spring Security: la clase `JdbcUserDetailsManager`. 
Tras aprender esto, sabrás cómo indicarle a Spring Security dónde encontrar a los usuarios, lo cual 
es esencial en el flujo de autenticación.

#### 3.3.1 Entendiendo el contrato de UserDetailsService

En esta sección, aprenderás sobre la definición de la interfaz UserDetailsService. Antes de 
comprender cómo y por qué implementarla, primero debes entender el contrato. Es momento de 
profundizar en UserDetailsService y cómo trabajar con implementaciones de este componente. La 
interfaz UserDetailsService contiene un solo método, como se muestra a continuación:
```java
public interface UserDetailsService {
    UserDetails loadUserByUsername(String username)
        throws UsernameNotFoundException;
}
```
La implementación de autenticación llama al método `loadUserByUsername(String username)` para obtener 
los detalles de un usuario con un nombre de usuario dado. El nombre de usuario, por supuesto, se 
considera único. El usuario devuelto por este método es una implementación del contrato `UserDetails`.
Si el nombre de usuario no existe, el método lanza una excepción `UsernameNotFoundException`.

`NOTA` `UsernameNotFoundException` es una `RuntimeException`.  La cláusula `throws` en la interfaz 
`UserDetailsService` es solo con fines de documentación. `UsernameNotFoundException` hereda 
directamente del tipo `AuthenticationException`, que es la clase base de todas las excepciones 
relacionadas con el proceso de autenticación. `AuthenticationException`, a su vez, hereda de la 
clase `RuntimeException`.

#### Flujo de la Autenticacion

`AutheticationProvicer`(1) --> `UserDetailsService``loadUserByUsername(String username)`(2)

1. El AuthenticationProvider utiliza el UserDetailsService para cargar los detalles del usuario en 
la lógica de autenticación. 
2. Podríamos implementar el UserDetailsService para cargar el usuario desde una base de datos, un 
sistema externo, un almacén seguro (vault), etc.

El `AuthenticationProvider` es el elemento encargado de ejecutar el proceso de autenticación y 
utiliza el `UserDetailsService` para obtener los detalles del usuario. Invoca el método 
`loadUserByUsername(String username)` para localizar al usuario según su nombre de usuario.

#### 3.3.2 Implementando el contrato de UserDetailsService

En esta sección, trabajamos en un ejemplo práctico para demostrar la implementación de 
UserDetailsService. Tu aplicación gestiona detalles sobre credenciales y otros aspectos del usuario.
Es posible que estos datos se almacenen en una base de datos o sean manejados por otro sistema al 
que accedes mediante un servicio web o por otros medios (figura 3.3). Independientemente de cómo 
ocurra esto en tu sistema, lo único que Spring Security necesita de ti es una implementación para 
recuperar el usuario por su nombre de usuario.

En el siguiente ejemplo, escribimos un `UserDetailsService` que tiene una lista de usuarios en
memoria. En el capítulo 2, usaste una implementación proporcionada que hace exactamente lo mismo: 
`InMemoryUserDetailsManager`. Dado que ya estás familiarizado con cómo funciona esta implementación,
he elegido una funcionalidad similar, pero esta vez para implementarla por nuestra cuenta. 
Proporcionamos una lista de usuarios al crear una instancia de nuestra clase `UserDetailsService`. 
Puedes encontrar este ejemplo en el proyecto ssia-ch3-ex1. En el paquete llamado model, definimos 
el `UserDetails` como se presenta en el siguiente listado.

La implementación de la interfaz UserDetails:
```java
public class User implements UserDetails {
    /*
    La clase User no es inmutable. Implementa la interfaz CredentialsContainer para permitir que 
    la contraseña se borre después de la autenticación. Esto puede causar efectos secundarios si 
    almacenas instancias en memoria y las reutilizas. En ese caso, asegúrate de devolver una 
    copia desde tu UserDetailsService cada vez que se invoque.
    */
    private final String username;
    private final String password;
    private final String authority;
    public User(String username, String password, String authority) {
        this.username = username;
        this.password = password;
        //Para simplificar el ejemplo, un usuario tiene solo una autoridad.
        this.authority = authority;
    }
    /*
    Devuelve una lista que contiene solo el objeto GrantedAuthority con el nombre proporcionado 
    al crear la instancia.
    */
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(() -> authority);
    }
    @Override
    public String getPassword() {
        return password;
    }
    @Override
    public String getUsername() {
        return username;
    }
    //La cuenta no expira ni se bloquea
    @Override
    public boolean isAccountNonExpired() {
        return true;
    }
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }
    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }
    @Override
    public boolean isEnabled() {
        return true;
    }
}
```
En el paquete llamado services, creamos una clase llamada `InMemoryUserDetailsService`. La siguiente
lista muestra cómo implementamos esta clase. 

La implementacion de la interface UserDetailsService:
```java
public class InMemoryUserDetailsService implements UserDetailsService {
    //UserDetailsService gestiona la lista de usuarios en memoria.
    private final List<UserDetails> users;
    public InMemoryUserDetailsService(List<UserDetails> users) {
        this.users = users;
    }
    @Override
    public UserDetails loadUserByUsername(String username)
            throws UsernameNotFoundException {
        return users.stream()
                .filter(
                        u -> u.getUsername().equals(username)
                )
                .findFirst()  //Si existe dicho usuario, lo devuelve.
                .orElseThrow( //Si no existe un usuario con ese nombre de usuario, lanza una excepción. 
                        () -> new UsernameNotFoundException("User not found")
                );
    }
}
```
El método `loadUserByUsername(String username)` busca en la lista de usuarios el nombre de usuario 
especificado y devuelve la instancia `UserDetails` deseada. Si no existe una instancia con ese nombre
de usuario, lanza una excepción `UsernameNotFoundException`. Ahora podemos utilizar esta 
implementación como nuestro `UserDetailsService`. La siguiente lista muestra cómo la agregamos como 
un bean en la clase de configuración y registramos un usuario dentro de ella.

UserDetailsService registrado como un bean en la clase de configuración:
```java
@Configuration
public class ProjectConfig {
    @Bean
    public UserDetailsService userDetailsService() {
        UserDetails u = new User("john", "12345", "read");
        List<UserDetails> users = List.of(u);
        return new InMemoryUserDetailsService(users);
}
    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```
Finalmente, creamos un endpoint simple y probamos la implementación. La siguiente lista define el 
endpoint.

La definición del endpoint utilizado para probar la implementación:
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
}
```
Al llamar al endpoint usando cURL, observamos que para el usuario `john` con contraseña `12345`, 
obtenemos un código de estado `HTTP 200 OK`. Si usamos otras credenciales, la aplicación devuelve 
un error `401 Unauthorized`:

`curl -u john:12345 http://localhost:8080/hello`

El cuerpo de la respuesta es:

`Hello!`

#### 3.3.3 Implementando el contrato de UserDetailsManager

En esta sección, se analiza el uso e implementación de la interfaz `UserDetailsManager`. Esta 
interfaz extiende `UserDetailsService` y añade más métodos al contrato. Spring Security necesita el 
contrato `UserDetailsService` para realizar la autenticación. Sin embargo, en general, las 
aplicaciones también requieren gestionar usuarios. La mayoría de las veces, una aplicación debe 
poder agregar nuevos usuarios o eliminar los existentes. En este caso, se implementa una interfaz
más específica definida por Spring Security: `UserDetailsManager`. Esta extiende `UserDetailsService`
y añade operaciones adicionales que debemos implementar:
```java
public interface UserDetailsManager extends UserDetailsService {
    void createUser(UserDetails user);
    void updateUser(UserDetails user);
    void deleteUser(String username);
    void changePassword(String oldPassword, String newPassword);
    boolean userExists(String username);
}
```
El objeto `InMemoryUserDetailsManager` que usamos en el capítulo 2 es en realidad una implementación 
de `UserDetailsManager`. En ese momento, solo consideramos sus características como 
`UserDetailsService`. El proyecto ssia-ch3-ex2 acompaña al ejemplo de esta sección.

#### Usar `JdbcUserDetailsManager` para la gestión de usuarios

Además de `InMemoryUserDetailsManager`, a menudo se utiliza otra implementación: 
`JdbcUserDetailsManager`. Esta clase gestiona usuarios en una base de datos SQL, conectándose 
directamente a través de JDBC, lo que la hace independiente de otros frameworks o especificaciones 
relacionados con la conexión a bases de datos.

Para entender cómo funciona `JdbcUserDetailsManager`, lo mejor es verlo en acción con un ejemplo.
En el siguiente ejemplo, se implementa una aplicación que gestiona usuarios en una base de datos 
MySQL utilizando `JdbcUserDetailsManager`. Se muestra el lugar que ocupa esta implementación en el 
flujo de autenticación. 

1. El filtro de autenticación intercepta la solicitud enviada por el cliente.
2. La responsabilidad de autenticación se delega al administrador de autenticación.
3. El administrador de autenticación utiliza un proveedor de autenticación, que implementa la 
lógica de autenticación.
4. El proveedor de autenticación llama a un `JdbcUserDetailsManager` para obtener los detalles del 
usuario por nombre de usuario.
5. La instancia de `JdbcUserDetailsManager` busca al usuario en la base de datos y devuelve sus 
detalles.
6. Si se encuentra al usuario, un codificador de contraseñas verifica si la contraseña enviada por 
el usuario coincide con la almacenada en la base de datos.
7. Después de una autenticación exitosa, los detalles de la entidad autenticada se almacenan en el 
contexto de la sesión.
8. La solicitud se redirige al controlador.

El flujo de autenticación de Spring Security. Aquí utilizamos un `JdbcUserDetailsManager` como 
nuestro componente `UserDetailsService`. El `JdbcUserDetailsManager` utiliza una base de datos para 
gestionar los usuarios.

Comenzarás a trabajar en nuestra aplicación de demostración que utiliza `JdbcUserDetailsManager`
creando una base de datos y dos tablas. En nuestro caso, nombraremos la base de datos spring, y a 
una de las tablas la llamaremos users y a la otra authorities. Estos nombres son los nombres de 
tabla predeterminados conocidos por `JdbcUserDetailsManager`. Como aprenderás al final de esta 
sección, la implementación de `JdbcUserDetailsManager` es flexible y te permite sobrescribir estos 
nombres predeterminados si así lo deseas. El propósito de la tabla users es almacenar los registros 
de usuarios. La implementación de `JdbcUserDetailsManager` espera tres columnas en la tabla users: 
un username, un password y un campo enabled, que puedes usar para desactivar al usuario.

Puedes optar por crear la base de datos y su estructura tú mismo utilizando la herramienta de línea
de comandos de tu sistema de gestión de bases de datos (DBMS) o una aplicación cliente. Por ejemplo,
para MySQL, puedes usar MySQL Workbench. Pero lo más sencillo es dejar que Spring Boot ejecute los 
scripts por ti. Para ello, simplemente agrega dos archivos más a tu proyecto en la carpeta 
resources: schema.sql y data.sql. En el archivo schema.sql, añade las consultas relacionadas con 
la estructura de la base de datos, como crear, modificar o eliminar tablas. En el archivo data.sql,
añade las consultas que trabajan con los datos dentro de las tablas, como `INSERT, UPDATE o DELETE`. 
Spring Boot ejecuta automáticamente estos archivos cuando inicias la aplicación. Una solución más
sencilla para crear ejemplos que necesiten bases de datos es usar una base de datos en memoria H2, 
evitando así la instalación de un DBMS separado.

NOTA: Puedes usar H2 (como hago en el proyecto ssia-ch3-ex2) al desarrollar las aplicaciones de 
este libro. Sin embargo, en la mayoría de los casos, he optado por implementar los ejemplos con un 
DBMS externo para dejar claro que es un componente externo del sistema y así evitar confusiones.

Utiliza el código del siguiente listado para crear la tabla users con un servidor MySQL. Puedes 
añadir este script al archivo schema.sql en tu proyecto Spring Boot.

La consulta SQL para crear la tabla users es:
```mysql
CREATE TABLE IF NOT EXISTS `spring`.`users` (
`id` INT NOT NULL AUTO_INCREMENT,
`username` VARCHAR(45) NOT NULL,
`password` VARCHAR(45) NOT NULL,
`enabled` INT NOT NULL,
PRIMARY KEY (`id`));
```
La tabla authorities almacena las autoridades (roles o permisos) asignadas a cada usuario. Cada 
registro contiene un nombre de usuario y una autoridad otorgada al usuario con ese nombre de usuario.

La consulta SQL para crear la tabla authorities es:
```mysql
CREATE TABLE IF NOT EXISTS `spring`.`authorities` (
`id` INT NOT NULL AUTO_INCREMENT,
`username` VARCHAR(45) NOT NULL,
`authority` VARCHAR(45) NOT NULL,
PRIMARY KEY (`id`));
```
`NOTA`: Por simplicidad y para permitirte concentrarte en las configuraciones de Spring Security que
discutimos, en los ejemplos proporcionados con este libro, se omiten las definiciones de índices o 
claves foráneas.

Para asegurarte de tener un usuario para pruebas, inserta un registro en cada una de las tablas. 
Puedes agregar estas consultas en el archivo data.sql en la carpeta resources del proyecto 
Spring Boot:
```mysql
INSERT INTO `spring`.`authorities`
(username, authority)
VALUES
('john', 'write');
INSERT INTO `spring`.`users`
(username, password, enabled)
VALUES
('john', '12345', '1');
```
Para tu proyecto, necesitas agregar al menos las dependencias indicadas en la siguiente lista. 
Revisa tu archivo pom.xml para asegurarte de que hayas incluido estas dependencias.

Las dependencias necesarias para desarrollar el ejemplo:
```xml
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-security</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-web</artifactId>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
<groupId>com.h2database</groupId>
<artifactId>h2</artifactId>
</dependency>
```
`NOTA`: En tus ejemplos, puedes usar cualquier tecnología de base de datos SQL siempre que agregues
el controlador JDBC correcto a las dependencias.

Recuerda que debes agregar el controlador JDBC según la tecnología de base de datos que utilices. 
Por ejemplo, si usas MySQL, necesitas agregar la dependencia del controlador de MySQL como se 
muestra en el siguiente fragmento:
```xml
<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<scope>runtime</scope>
</dependency>
```
Puedes configurar un origen de datos (datasource) en el archivo application.properties de tu 
proyecto o como un bean independiente. Si eliges usar el archivo application.properties, necesitas 
agregar las siguientes líneas a ese archivo:
```pom
spring.datasource.url=jdbc:h2:mem:ssia
spring.datasource.username=sa
spring.datasource.password=
spring.sql.init.mode=always
```
En la clase de configuración del proyecto, defines el `UserDetailsService` y el `PasswordEncoder`. 
`JdbcUserDetailsManager` necesita el `DataSource` para conectarse a la base de datos. El origen de 
datos puede inyectarse mediante un parámetro del método (como se muestra en el siguiente listado) o 
mediante un atributo de la clase.

Registrar el `JdbcUserDetailsManager` en la clase de configuración:
```java
@Configuration
public class ProjectConfig {
@Bean
public UserDetailsService userDetailsService(DataSource dataSource) {
    return new JdbcUserDetailsManager(dataSource);
}
    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```
Para acceder a cualquier endpoint de la aplicación, ahora necesitas usar autenticación HTTP Basic 
con uno de los usuarios almacenados en la base de datos. Para demostrarlo, creamos un nuevo endpoint,
como se muestra en el siguiente listado, y luego lo llamamos con cURL.

El endpoint de prueba para verificar la implementación:
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
}
```
En el siguiente fragmento de código, se muestra el resultado al llamar al endpoint con el nombre de 
usuario y contraseña correctos:

`curl -u john:12345 http://localhost:8080/hello`

La respuesta a la llamada es:

`Hello!`

`JdbcUserDetailsManager` también permite configurar las consultas utilizadas. En el ejemplo anterior,
nos aseguramos de usar los nombres exactos para las tablas y columnas, tal como los espera la 
implementación de `JdbcUserDetailsManager`. Pero podría ser que esos nombres no sean los más 
adecuados para tu aplicación. El siguiente listado muestra cómo sobrescribir las consultas para 
`JdbcUserDetailsManager`.

Cambiar las consultas de JdbcUserDetailsManager para buscar al usuario:
```java
@Bean
public UserDetailsService userDetailsService(DataSource dataSource) {
    String usersByUsernameQuery =
        "select username, password, enabled from users where username = ?";
    String authsByUserQuery =
        "select username, authority from spring.authorities where username = ?";
    var userDetailsManager = new JdbcUserDetailsManager(dataSource);
    userDetailsManager.setUsersByUsernameQuery(usersByUsernameQuery);
    userDetailsManager.setAuthoritiesByUsernameQuery(authsByUserQuery);
    return userDetailsManager;
}
```
De la misma manera, podemos modificar todas las consultas utilizadas por la implementación de 
`JdbcUserDetailsManager`.

`EJERCICIO`: Escribe una aplicación similar en la que nombres las tablas y columnas de forma 
diferente en la base de datos. Sobrescribe las consultas para la implementación de 
`JdbcUserDetailsManager` (por ejemplo, la autenticación funcione con una nueva estructura de tablas).
El proyecto ssia-ch3-ex2 incluye una posible solución.

#### Usar un LdapUserDetailsManager para la gestión de usuarios

Spring Security ofrece una implementación de UserDetailsManager para LDAP. Aunque es menos común que
`JdbcUserDetailsManager`, puedes utilizarla si necesitas integrar el sistema con un servidor LDAP 
para gestionar usuarios. En el proyecto ssia-ch3-ex3 se incluye una demostración sencilla del uso de
`LdapUserDetailsManager`.

Dado que no se puede usar un servidor LDAP real para la demostración, se configura uno embebido en 
la aplicación Spring Boot. Para ello, se define un archivo en formato LDIF (LDAP Data Interchange 
Format) que contiene los datos iniciales del directorio. A continuación, se muestra un ejemplo del 
contenido de un archivo LDIF:

La definición del archivo LDIF:
```ldif
dn: dc=springframework,dc=org           //Defines the base entity
objectclass: top
objectclass: domain
objectclass: extensibleObject
dc: springframework
dn: ou=groups,dc=springframework,dc=org     //Defines a group entity
objectclass: top
objectclass: organizationalUnit
ou: groups
dn: uid=john,ou=groups,dc=springframework,dc=org     //Defines a user
objectclass: top
objectclass: person
objectclass: organizationalPerson
objectclass: inetOrgPerson
cn: John
sn: John
uid: john
userPassword: 12345
```
En el archivo LDIF, agrego solo un usuario para el cual necesitamos probar el comportamiento de la 
aplicación al final de este ejemplo. Podemos agregar el archivo LDIF directamente a la carpeta 
resources. De esta forma, queda automáticamente en el classpath y podemos referenciarlo fácilmente 
más adelante. He nombrado el archivo LDIF server.ldif. Para trabajar con LDAP y permitir que Spring 
Boot inicie un servidor LDAP embebido, necesitas agregar las siguientes dependencias al archivo 
pom.xml:
```xml
<dependency>
<groupId>org.springframework.security</groupId>
<artifactId>spring-security-ldap</artifactId>
</dependency>
<dependency>
<groupId>com.unboundid</groupId>
<artifactId>unboundid-ldapsdk</artifactId>
</dependency>
```
En el archivo application.properties, también necesitas agregar las configuraciones para el servidor
LDAP embebido, como se muestra en el siguiente fragmento de código. Los valores que necesita la 
aplicación para iniciar el servidor LDAP embebido incluyen la ubicación del archivo LDIF, un puerto 
para el servidor LDAP y los valores de etiqueta del componente de dominio base (DN):
```pom
spring.ldap.embedded.ldif=classpath:server.ldif
spring.ldap.embedded.base-dn=dc=springframework,dc=org
spring.ldap.embedded.port=33389
```
Una vez que tienes un servidor LDAP para autenticación, puedes configurar tu aplicación para usarlo.
La siguiente lista muestra cómo configurar LdapUserDetailsManager para permitir que tu aplicación 
autentique usuarios a través del servidor LDAP.

La definición de LdapUserDetailsManager en el archivo de configuración:
```java
@Configuration
public class ProjectConfig {
    @Bean  //--> Agrega una implementación de UserDetailsService al contexto de Spring
    Spring context
    public UserDetailsService userDetailsService() {
        var cs = new DefaultSpringSecurityContextSource(
                //Crea una fuente de contexto para especificar la dirección del servidor LDAP
                "ldap://127.0.0.1:33389/dc=springframework,dc=org");
        cs.afterPropertiesSet();
        
        //Crea la instancia de LdapUserDetailsManager
        var manager = new LdapUserDetailsManager(cs);
        manager.setUsernameMapper(
                //Establece un mapeador de nombres de usuario para indicar a LdapUserDetailsManager
                // cómo buscar usuarios
                new DefaultLdapUsernameToDnMapper("ou=groups", "uid"));
        
        //Establece la base de búsqueda de grupos que la aplicación necesita para buscar usuarios
        manager.setGroupSearchBase("ou=groups");
        return manager;
    }
    @Bean
    public PasswordEncoder passwordEncoder() {
        return NoOpPasswordEncoder.getInstance();
    }
}
```
Vamos a crear también un endpoint simple para probar la configuración de seguridad. He agregado una
clase controlador, como se muestra en el siguiente fragmento de código:
```java
@RestController
public class HelloController {
    @GetMapping("/hello")
    public String hello() {
        return "Hello!";
    }
}
```
Ahora inicia la aplicación y llama al endpoint /hello. Necesitas autenticarte como el usuario john 
para que la aplicación te permita acceder al endpoint. El siguiente fragmento de código muestra el 
resultado de llamar al endpoint con cURL:

`curl -u john:12345 http://localhost:8080/hello`

La respuesta a la llamada es:

`Hello!`

### Resumen

- La interfaz `UserDetails` es el contrato que se utiliza para describir un usuario en Spring Security.

- La interfaz UserDetailsService es el contrato que Spring Security espera que implementes en la 
arquitectura de autenticación para definir cómo la aplicación obtiene los detalles del usuario.

- La interfaz `UserDetailsManager` extiende `UserDetailsService` y añade funcionalidades relacionadas 
con la creación, modificación o eliminación de usuarios.

- Spring Security proporciona varias implementaciones del contrato `UserDetailsManager`, entre ellas 
`InMemoryUserDetailsManager`, `JdbcUserDetailsManager` y `LdapUserDetailsManager`.

- La clase `JdbcUserDetailsManager` tiene la ventaja de usar JDBC directamente, lo que evita la 
dependencia de otros frameworks.

