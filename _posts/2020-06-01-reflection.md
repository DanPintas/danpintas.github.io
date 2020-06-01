---
layout: default
title: Metaprogramación - reflexión y anotaciones
categories: java
order: 2
---

# Reflexión

La reflexión es la capacidad del código de inspeccionarse a sí mismo. Algunos casos de uso comunes son:
* Inyección de dependencias (por ejemplo Spring) 
* Habilitar o deshabilitar plugins
* Análisis estático de código

La herramienta básica para usar reflexión en Java es el paquete `java.lang.reflect`.

En las explicaciones usaremos como ejemplo la siguiente clase:
```java
package es.danpintas.reflexion;

public class Persona {

    private final String nombre;
    private final String apellido;
    private int edad;

    public Persona() {
        this("Jonh", "Doe", 0);
    }

    public Persona(String nombre, String apellido) {
        this(nombre, apellido, 0);
    }

    private Persona(String nombre, String apellido, int edad) {
        this.nombre = nombre;
        this.apellido = apellido;
        this.edad = edad;
    }

    public String getNombre() {
        return nombre;
    }

    public String getApellido() {
        return apellido;
    }

    public int getEdad() {
        return edad;
    }

    public void setEdad(int edad) {
        this.edad = edad;
    }

    @Override
    public String toString() {
        return "Persona{" +
                "nombre='" + nombre + '\'' +
                ", apellido='" + apellido + '\'' +
                ", edad=" + edad +
                '}';
    }
}
```

## Cómo acceder

El punto de entrada para el uso de reflexión es la clase `Class`.
Para acceder a la información de una clase por reflexión hay varias maneras de hacerlo:

```java
// La clase estática nos garantiza la clase exacta.
Class<Persona> claseEstatica = Persona.class;

Persona persona = new Persona();
// A partir de una instancia, la reflexión no está segura de que no trata con una subclase.
Class<? extends Persona> clasePorInstancia = persona.getClass();

// Por nombre no tenemos errores de compilación si no existe pero sí una excepción.
// Con este se tratan dependencias opcionales(como plugins o drivers específicos).
Class<?> clasePorNombre = Class.forName("es.danpintas.reflexion.Persona");

// La clase obtenida es realmente la misma, podemos castear si lo necesitamos.
System.out.println(claseEstatica.equals(clasePorInstancia));  // true
System.out.println(claseEstatica.equals(clasePorNombre));     // true
```

Para explorar la jerarquía de clases tenemos los métodos `getSuperclass` y `getInterfaces`.

Podemos saber si una clase (o cualquiera de sus miembros) es pública, privada, etc... usando el método `getModifiers`, cuyo resultado se pasa a métodos estáticos como `Modifier.isPublic`.

## Constructores

```java
Class<Persona> clase = Persona.class;

// Si la clase tiene un constructor público sin argumentos podemos usarlo directamente.
clase.newInstance();

// Se puede recuperar el listado de constructores públicos
Constructor<?>[] constructoresPublicos = clase.getConstructors();

// Para el listado completo, debemos usar este método
Constructor<?>[] constructores = clase.getDeclaredConstructors();

// También podemos recuperar un constructor específico;
// lanzará excepción si no existe un constructor así
Constructor<Persona> constructor = clase.getDeclaredConstructor(
        String.class, String.class, int.class);

// Para usar un miembro que no sea público tenemos que "abrirlo"
constructor.setAccessible(true);

// El constructor no comprueba en compilación los tipos recibidos,
// es nuestra responsabilidad que sean correctos
Persona persona = constructor.newInstance(
        "Arturo", "Plateado", 0);

// Persona{nombre='Arturo', apellido='Plateado', edad=0}
System.out.println(persona);
```

## Campos

```java
Class<Persona> clase = Persona.class;
Persona persona = new Persona("Arturo", "Plateado");

// Igual que con constructores, usaríamos getFields para tener sólo los públicos.
Field[] campos = clase.getDeclaredFields();

// Recuperar un campo por nombre lanza excepción si no existe.
Field apellido = clase.getDeclaredField("apellido");

// También tenemos que abrir el campo para poder usarlo si no es público.
apellido.setAccessible(true);

// Para ver el valor se usa get(objeto).
// Estos métodos no comprueban que se pase un objeto de una clase válida.
System.out.println(apellido.get(persona));

// Para cambiar el valor se usa set(objeto, valor).
// Al usar setAccessible, podemos incluso cambiar campos finales.
apellido.set(persona, "Dorado");

// Persona{nombre='Arturo', apellido='Dorado', edad=0}
System.out.println(persona);
```

## Métodos

```java
Class<Persona> clase = Persona.class;
Persona persona = new Persona("Arturo", "Plateado");

// Usaríamos getDeclaredMethods para la lista completa.
Method[] metodos = clase.getMethods();

// Aquí también tendremos excepción si el método no existe,
// y tendríamos que usar setAccessible si no fuera público.
Method setEdad = clase.getMethod("setEdad", int.class);

// Se llama al método con invoke(objeto, argumentos...).
// Somos responsables del número y tipo de argumentos, como en los constructores.
setEdad.invoke(persona, 18);

// Persona{nombre='Arturo', apellido='Dorado', edad=18}
System.out.println(persona);
```

## ¿Qué pasa con los genéricos?

Trabajar con genéricos por reflexión es complicado, ya que los genéricos Java usan borrado de tipos; esto significa los tipos genéricos como tal sólo se usan para compilar y no en tiempo de ejecución, siendo reemplazados por `Object` o por el límite inferior (por ejemplo, en una clase `Registro<T extends Persona>` se el compilador usaría `Persona`).

Si queremos usar genéricos en nuestro código con reflexión, tendremos que usar algún mecanismo que recoja información de un tipo genérico, a ser posible sin crear un objeto de este tipo. El escenario más común en el que enfrentamos este problema es la inyección de dependencias que usen genéricos, por lo que es habitual encontrar soluciones refinadas en librerías relacionadas, por ejemplo `ParameterizedTypeReference` en Spring o `TypeLiteral` en Guava.

# Anotaciones

Las anotaciones son un mecanismo para **añadir metadatos a nuestro código**. Los usos principales de las anotaciones son:
* Dar instrucciones a librerías que usan reflexión (por ejemplo Spring) para que sepan cómo procesar el código.
* Dar instrucciones a los IDE para modificar su comportamiento (por ejemplo @SuppressWarnings).
* Añadir etiquetas a parte del código para que sea más entendible.

Es decir, las anotaciones **no afectan al código por sí mismas**.

Las anotaciones se usan con la sintaxis `@Anotacion` antes del elemento anotado; por ejemplo:

```java 
@Service
public class StudentService {
    // contenido de la clase...
}
```

Varias APIs usan anotaciones, y de la misma manera nosotros podemos crear las nuestras. Un ejemplo sería:

```java 
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Service {
    String value() default "";
}
```

## Establecer dónde se puede usar la anotación

La anotación `@Target` indica en qué elementos de Java se puede insertar esta. 

Para ello, recibe un array de `ElementType` con todos los puntos en los que se quiere permitir el uso de la anotación. Cada tipo indica que podremos usar la anotación en las declaraciones de:
* TYPE: clases, interfaces, anotaciones y enums.
* FIELD: campos de clase y constantes de enums.
* METHOD: métodos.
* PARAMETER: parámetros de métodos.
* CONSTRUCTOR: constructores.
* LOCAL_VARIABLE: variables locales.
* ANNOTATION_TYPE: anotaciones.
* TYPE_PARAMETER: parámetros genéricos de TYPE (por ejemplo, `E` en `List<E>`).
* TYPE_USE: cualquier uso de un tipo (es decir, todos los anteriores juntos).
* PACKAGE: paquetes; sólo se usa en package-info, por lo que su uso no es habitual.

## Establecer quién puede acceder a la anotación

Con la anotación `@Retention` se indica quién puede acceder a la anotación.

Para ello, recibe una `RetentionPolicy` con uno de los siguientes valores:
* CLASS: valor por defecto, la anotación se compila pero no es usada por la JVM.
* SOURCE: la anotación no se compila, se usa sólo como marcador a nivel de código; recomendada para anotaciones de uso interno.
* RUNTIME: la anotación se compila y está disponible para la JVM; necesaria para poder usar la anotación por reflexión.

## Parámetros de la anotación

Las anotaciones pueden recibir también parámetros con nombre, y con opción a tener un valor por defecto; por ejemplo:

```java
String value() default "";
```

Los parámetros de la anotación se usan como argumentos con nombre al usarla; por ejemplo:

```java
@Retention(value = RetentionPolicy.RUNTIME)
```

En el caso particular de que el uso de la anotación sólo tenga un parámetro `value` (independientemente de que la anotación tenga más parámetros que no usemos, y del tipo de este parámetro) nos permite omitir el nombre de la variable:

`java
@Retention(RetentionPolicy.RUNTIME)
`

Los parámetros sirven como campos de la anotación para especificar aún más el comportamiento; por ejemplo, una anotación `@Order` con un `value` de tipo `int` se puede usar para recoger todos los elementos anotados y procesarlos en un orden.

# Ejercicios

## Visor reflexivo

Construye un visualizador de clases que, dado el nombre completo de una clase, muestre por pantalla:
* Tipo de clase: clase estándar, interfaz, anotación o enum.
* Miembros: campos, constructores y métodos.
* Accesibilidad de los miembros: publica, privada, protegida.
* Tipos en orden de los parámetros de funciones y constructores.

Intenta usar un formato similar al código fuente para poder mostrar la información de manera compacta

## Extensión de anotaciones para el visor reflexivo

Extiende el visor reflexivo para que también incluya las anotaciones de todos estos miembros y sus parámetros; probablemente tengas que crear tu propia anotación y aplicarla en una clase propia para comprobar su funcionamiento.

## Ejecutor reflexivo

Crea un ejecutable que, dado el nombre completo de una clase:
* Muestre los constructores con firma de manera numerada.
* Pida un número de un constructor.
* Si el método acepta solo parámetros sencillos*, que los pida en orden y cree una instancia de la clase.
* Muestre los métodos con firma de manera numerada.
* Pida un número de un método.
* Si el método acepta solo parámetros sencillos*, que los pida en orden e invoque el método.

\* Los parámetros sencillos pueden ser tipos primitivos (o sus equivalentes boxed), String y Object (que puede pasarse como String para simplificar).

# Enlaces de interés

* [Reflexión (Baeldung)](https://www.baeldung.com/java-reflection)
* [Borrado de tipos (Baeldung)](https://www.baeldung.com/java-type-erasure)
* [Anotaciones por defecto (Baeldung)](https://www.baeldung.com/java-default-annotations)
* [Anotaciones personalizadas](https://www.baeldung.com/java-custom-annotation)
