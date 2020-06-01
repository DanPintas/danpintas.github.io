---
layout: default
title: Interfaces funcionales y lambdas
categories: funcional
order: 1
---

# Interfaces funcionales

El primer concepto a entender para trabajar en programación funcional en Java es el de las interfaces funcionales.
Una interfaz funcional es **cualquier interfaz con un único método abstracto**.

Se recomienda, pero no es obligatorio, que las interfaces funcionales estén anotadas con `@FunctionalInterface`.

Una interfaz funcional puede tener cualquier número de métodos default (métodos que la propia interfaz implementa).

## Por qué usar programación funcional

Las principales ventajas de la programación funcional son:
* Favorecer la encapsulación de lógica, facilitando que nuestra lógica sea reusable y más fácil de testear.
* Hacer código más legible, tanto para programadores como no-programadores.

Por ejemplo, digamos que quiero coger una lista de Strings y obtener una lista de ellos en formato "texto-ejemplo".
La opción tradicional sería:

```java
List<String> strings = Arrays.asList("valor 1", null, "valor 2", "");
List<String> tags = new ArrayList<>();
for (String string: strings) {
    if (string != null && !string.isEmpty()) {
        String tag = string.toLowerCase().replaceAll("\\s+", "-");
        tags.add(tag);
    }
}
System.out.println(String.join(", ", tags));
```

Y la opción funcional:
```java
List<String> strings = Arrays.asList("valor 1", null, "valor 2", "");
List<String> tags = strings.stream()
        .filter(StringUtils::isNotEmpty)
        .map(this::toTag);
        .collect(Collectors.toList());
```

## Interfaces funcionales comunes

* `Runnable`: la lambda más simple, ni entradas ni salidas, para comportamientos autocontenidos; por ejemplo, anotar un timestamp).
* `Function<I, O>`: función una entrada y una salida de los tipos que queramos.
* `BiFunction<I, J, O>`: función de dos argumentos, pudiendo elegir los tipos de cada argumento y el retorno.
* `Supplier<O>`: retorna un tipo sin argumentos de entrada; llamar a un constructor sería el ejemplo más básico.
* `Consumer<I>`: acepta un objeto sin retornar nada; normalmente se usa para efectos secundarios como imprimir un resultado.
* `Predicate<I>`: como `Function<I, boolean>`; su uso más habitual es para filtrados o búsquedas.
* `Operator<IO>`: como `Function<IO, IO>`; suele usarse para transformaciones, especialmente en inmutables como los `String`.

Existe una gama de especializaciones para los tipos primitivos `double`, `int` y `long` como `ToIntFunction` o `LongUnaryOperator`.
Si sabemos que podemos trabajar con tipos primitivos es bueno usarlos por su menor huella de memoria y mejor rendimiento.

## Modos de uso: de menos a más funcional

Para ilustrar las maneras de implementar interfaces funcionales usaremos como ejemplo un `Predicate` en el siguiente código:

```java
List<String> strings = Stream.of("a", null, "b", null, "c")
        .filter(predicate)
        .collect(Collectors.toList());
```

`Predicate<T>` es una interfaz funcional cuyo SAM es `boolean test(T)`, es decir, un booleano en función del objeto a analizar.

Haremos uso de la programación funcional para cribar los nulos de la lista.

### Implementación explícita

La opción clásica es generar una clase que implemente nuestra operación:
```java
public class NotNullPredicate<T> implements Predicate<T> {
    @Override
    public boolean test(T t) {
        return t != null;
    }
}
```

Y usar una nueva instancia en el código:
```java
List<String> strings = Stream.of("a", null, "b", null, "c")
        .filter(new NotNullPredicate<>())
        .collect(Collectors.toList());
```

Esto puede funcionar bien con ejemplos sencillos, 
pero si necesitamos otros objetos para evaluar nuestro predicado los tendremos que pasar al constructor,
pudiendo resultar en monstruosidades como `new ValidItemPredicate(servicioValidacion, objetosYaExistentes, cache, indice)`,
que son difíciles de entender, y por lo tanto de mantener.

### Implementación anónima

La otra opción que teníamos disponible antes de Java 8 es la implementación anónima, disponible para cualquier tipo de interfaz:
```java
List<String> strings = Stream.of("a", null, "b", null, "c")
        .filter(new Predicate<String>() {
            @Override
            public boolean test(String t) {
                return t != null;
            }
        })
        .collect(Collectors.toList());
```

Con respecto a la implementación explícita ganamos el no tener que pasar argumentos a un constructor y no mantener otra clase,
pero declarar implementaciones anónimas puede hacer que nuestro código sea menos legible.

### Lambda

La implementación anónima puede aligerarse en el caso de las interfaces funcionales, aprovechando que ya sabe qué interfaz y método rellena:
```java
List<String> strings = Stream.of("a", null, "b", null, "c")
        .filter((String t) -> {
            return t != null;
        })
        .collect(Collectors.toList());
```

Podemos ir varios pasos más allá para minimizar la expresión:
* Se sabe que el método a implementar necesita un String, así que `(String t)` se puede abreviar a `(t)`.
* Si el SAM tiene un sólo argumento, los paréntesis son opcionales y `(t)` se abrevia `t`.
* Si la implementación tiene una sóla línea, se puede eliminar (todo junto) los corchetes, el `return` y el `;`.

El resultado de aplicar estas abreviaturas es el siguiente:
```java
List<String> strings = Stream.of("a", null, "b", null, "c")
        .filter(t -> t != null)
        .collect(Collectors.toList());
```

### Referencia de método

Supongamos que queremos encapsular nuestra lógica para reutilizarla en otros puntos, y creamos para ello un método `isNotNull`:
```java
List<String> strings = Stream.of("a", null, "b", null, "c")
        .filter(t -> isNotNull(t))
        .collect(Collectors.toList());
```

Cuando nuestra función consiste en pasar los argumentos tal cual a un método podemos convertir la lambda en una referencia de método:
```java
List<String> strings = Stream.of("a", null, "b", null, "c")
        .filter(this::isNotNull)
        .collect(Collectors.toList());
```

Esto también aplica a métodos estáticos, como `Objects.nonNull`:
```java
List<String> strings = Stream.of("a", null, "b", null, "c")
        .filter(Objects::nonNull)
        .collect(Collectors.toList());
```

Y también se puede utilizar cuando el SAM tiene un sólo argumento para que llame a un método de su propia clase
```java
List<Integer> lengths = Stream.of("a", null, "b", null, "c")
        .filter(Objects::nonNull)
        .map(String::length) // equivalente a implementar str -> str.length()
        .collect(Collectors.toList());
```

# Ejercicios

TO DO

# Enlaces de interés
* [Interfaces funcionales (Baeldung)](https://www.baeldung.com/java-8-functional-interfaces)
* [Tipos primitivos contra objetos (Baeldung)](https://www.baeldung.com/java-primitives-vs-objects)
