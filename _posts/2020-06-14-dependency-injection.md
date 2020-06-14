---
layout: default
title: Inyección de dependencias (DI)
categories: inyeccion-dependencias
order: 1
---

# Inyección de dependencias (DI)

En programas Java de gran tamaño es habitual incluir no sólo las dependencias que necesitamos directamente, 
sino herramientas para gestionar el programa a nivel interno; la más común es un inyector de dependencias.

La inyección de dependencias es un patrón de diseño en el que **un objeto recibe los componentes que necesita desde fuera**; 
es decir, nunca crea instancias de sus dependencias.

Este patrón nos obliga, además, a seguir otro: 
la inversión de control, que consiste en inicializar primero los componentes sencillos y luego los que dependan de estos
(de abajo hacia arriba y no al revés).

## Por qué usar inyección de dependencias

Supongamos que tenemos un proyecto grande, en el que usamos una clase de servicios a lo largo de todo nuestro código.
En un momento, nos llega un requisito: todas las clases deben usar un único servicio en vez de inicializar todos el suyo.
Sin inyección de dependencias, estaríamos obligados a reemplazar el uso de la clase a lo largo de todo el código.

Una primera solución podría ser crear un método estático `instance()` que nos diera instancias, pero eso nos va a dar dos problemas:
* Nuestro código quedaría manchado por llamadas a `Component.instance()` en todas partes.
* No hay manera de instruir en el "uso correcto" del componente, ni a nivel interno (alguien podría no ver otros usos de la clase)
ni epecialmente cuando nuestro proyecto es usado como dependencia por otros.

En resumen, usar inyección de dependencias es útil porque **separa la arquitectura de un proyecto de su funcionalidad**.

## Cómo inyectar dependencias

Las inicializaciones explícitas no pueden ser manejadas desde fuera:
```java
public class MyComponent {

    private final Dependency initByConstructor;
    private final Dependency initByField = new Dependency();
    
    public MyComponent() {
        initByConstructor = new Dependency();
    }
    
}
```

Por eso, el primer paso es que el deshacerse de ellas:
```java
public class MyComponent {

    private final Dependency initByConstructor;
    private Dependency initByField;
    private Dependency initBySetter;
    
    public MyComponent(Dependency initByConstructor) {
        this.initByConstructor = initByConstructor;
    }
    
    public void setInitBySetter(Dependency initBySetter) {
        this.initBySetter = initBySetter;
    }
    
}
```

A partir de aquí, podemos hablar de cómo se inicializan las dependencias.
Los distintos mecanismos se pueden combinar, pero es recomendable ser consistente a la hora de hacer código

### Inicialización por campo
```java
public class MyComponent {

    private Dependency initByField;
    
}
```

Esta es la inicialización más rápida por escribir menos, pero a su vez la menos recomendable por varios motivos:
* Solo podemos guardar la variable, siendo imposible realizar otras operaciones 
(p.e. trazas de log o inicializar otros valores según la dependencia).
* No existe manera de inyectar que no requiera reflexión o proxying, ralentizando el proceso de inyección.
* Dificulta hacer tests de los componentes separados del inyector de dependencias sin reflexión a gran escala y de manera manual.
* Nadie nos impide reasignar esta variable aunque no haya acceso directo a ella.

### Inicialización por método
```java
public class MyComponent {

    private Dependency initBySetter;
    
    public void setInitBySetter(Dependency initBySetter) {
        this.initBySetter = initBySetter;
    }
    
}
```

Esta inicialización nos libra de la mayoría de inconvenientes de la inicialización por campo, pero:
* Sigue permitiendo modificar el campo, agravándolo además al exponer un método para ello.
* Sigue siendo difícil hacer tests que no usen el inyector sin instrucciones manuales.
* Algunos de los principales inyectores de dependencias usan reflexión para llamar a los métodos, 
por lo que muchos métodos a inyectar también afectan al rendimiento.

### Inicialización por constructor
```java
public class MyComponent {

    private final Dependency initByConstructor;
    
    public MyComponent(Dependency initByConstructor) {
        this.initByConstructor = initByConstructor;
    }
    
}
```

Este es el mecanismo recomendado, ya que:
* Garantiza que no se reasignan las dependencias.
* Permite inicializar otros campos en función de la dependencia.
* Minimiza la cantidad de acciones a realizar, especialmente importante en inyectores por reflexión.
* Facilita hacer tests independientes del inyector.

La única desventaja clara es que si usamos herencia para componentes tenemos que pasar los constructores en cada clase.

Hay quien considera una desventaja que cuando tenemos muchas dependencias genera un constructor muy grande que es "incómodo", 
pero eso es de hecho una ventaja porque nos indica cuándo una clase tiene demasiadas dependencias
(lo que significa que seguramente la clase tiene demasiadas responsabilidades y debería ser dividida).

## Provisión de dependencias

Tan importante como definir cómo se inyectan las dependencias de un componente es definir cómo se proveen las dependencias.
Las estrategias habituales cuando se pide una dependencia al inyector son:
* Prototipo: siempre proporciona un nuevo objeto (estrategia por defecto en la mayoría de frameworks pero no en Spring).
* Singleton: siempre properciona el mismo objeto.

Dependiendo de nuestra aplicación, puede ser que nuestras dependencias provean objetos distintos en función de algún estado; por ejemplo, reutilizar una dependencia pero sólo en el marco de un mismo hilo o una misma petición o sesión HTTP.

## El inyector

Existen varias opciones de inyector de dependencias, siendo la más sencilla (e igualmente válida en proyectos pequeños)
inyectar nosotros las dependencias, sea en la misma clase principal o a través de una clase separada.

Un ejemplo de programa "Hello World!" con inyección de dependencias:
```java
public class App {

    public static void main(String... args) {
      new Injector().getApp().run(args);
    }

    private final Greeter greeter;
    
    public App(Greeter greeter) {
        this.greeter = greeter;
    }
    
    public void run(String... args) {
        greeter.greet();
    }
    
}
public class Injector {

    private final App app;
    
    public Injector() {
        Greeter greeter = new Greeter();
        app = new App(greeter);
    }
    
    public App getApp() {
        return app;
    }
    
}
public class Greeter {
    
    public void greet() {
        System.out.println("Hello World!");
    }
    
}
```

## Dependencias de genéricos

Por defecto, los inyectores reconocerán las dependencias por su clase;
sin embargo, aveces nos interesará reconocer distintas dependencias en función del tipo genérico 
(considerar que un `List<String>` no es lo mismo que un `List<Integer>`).

Para estos casos, los inyectores de dependencias suelen utilizar mecanismos relacionados con la reflexión
para reconocer la clase especializada con su tipo genérico en lugar de simplemente el tipo.

## Múltiples dependencias de una misma clase
En ocasiones nos interesará tener varias dependencias de un mismo tipo (p.e. distintos `Properties` para distintas funcionalidades).

La estrategia habitual a seguir es que, además de la clase a inyectar, se acepte de manera opcional un nombre o identificador para distinguir cuál de las depndencias de un mismo tipo querremos realmente.

# Ejercicios
Tal y como se indicaba al principio, es poco habitual utilizar implementaciones propias de un inyector de dependencias, por lo que es más importante tener claros los conceptos generales para cuando se trabaje con cualquier framework; no es necesario hacer una implementación propia de un inyector para entender cómo funcionan.

## Inyector de dependencias
Desarrolla una aplicación que a través de un inyector inyecte en una clase `MainComponent` una dependencia de la clase `SingletonComponent` y otra de una clase `PrototypeComponent` con los alcances de mismo nombre; comprueba que las dependencias inyectadas aplican el alcance correcto (para dos instancias de `MainComponent`, su `SingletonComponent` debe ser el mismo y su `PrototypeComponent` distinto).

Opcionalmente, amplíalo para que soporte genéricos y/o distintas dependencias en función de un String.

# Enlaces de interés
* [Metaprogramación - reflexión y anotaciones] (https://danpintas.github.io/apis-java/2020/06/01/reflection.html)
* [Estándar de inyección de dependencias en Java] (https://github.com/javax-inject/javax-inject)
