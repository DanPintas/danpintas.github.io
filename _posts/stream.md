---
layout: default
title: Stream
categories: funcional
order: 2
---
# Stream

La clase `Stream<T>` es un **procesador de secuencias de elementos** que aplica operaciones en modo funcional.

Es importante tener en cuenta que **los streams son de uso único**, e intentar reutilizarlos provocará lanzamiento de excepciones o efectos secundarios silenciosos; 
esto se debe a que los streams no han sido diseñados para guardar elementos.

Los streams se evalúan de manera vaga, lo que quiere decir que las operaciones definidas en un `Stream` no se ejecutan hasta que se cierre.
  
## Crear objetos Stream
* `Stream.empty()` para streams vacíos.
* `Stream.of(T value)` para un stream de un solo objeto.
* `Stream.of(T... values)` para arrays y varargs.
* `collection.stream()` para objetos que implementen `Collection<T>` por ejemplo objetos `List` y `Set`.
* `StreamSupport.stream(iterable.spliterator(), false)` para objetos que implementen `Iterable<T>`.
* `Stream.concat(Stream<? extends T> a, Stream<? extends T> b))` para combinar dos streams en uno.
  * Si queremos combinar varios streams, una alternativa más usable sería:
    ```java
    Stream.of(stream1, stream2, stream3, stream4).flatMap(i -> i)
    ```
  * Si queremos generar un stream a partir de elementos individuales, una alternativa más usable sería:
    ```java
   Stream.<String>builder().add("a").add("b").build()
    ```
* `Stream.generate(Supplier<T> s)` y `Stream.iterate(T seed, UnaryOperator<T> f)` para streams infinitos.

### Streams concurrentes
Permitir habilitar la ejecución con en paralelo de `Stream` basta con llamar al método `parallelStream()`.

## Operaciones intermedias
* `distinct()`: elimina elementos iguales según los métodos `equals` y `hashCode` (aunque la documentación no indique el uso del segundo).
* `filter(Predicate<? super T> predicate)`: elimina elementos para los que el predicado retorna `false`.
* `skip(long n)`: elimina los primeros n elementos del stream.
* `limit(long maxSize)`: limita el stream al número indicado de elementos.
* `sorted()`, `sorted(Comparator<? super T> comparator)`: ordenan los elementos de acuerdo a su ordenación natural o al comparador.
* `map(Function<? super T,? extends R> mapper)`: transforma todos los elementos del stream aplicando la función.
  * `flatMap(Function<? super T,? extends Stream<? extends R>> mapper)` es una especialización para "aplanar" streams.
    ```java
    // Si tuvieramos un stream de listas que queremos convertir a uno de elementos
    Stream<String> stringStream = stringListStream.flatMap(Collection::stream)
    ```
* `peek(Consumer<? super T> action)`: ejecuta la acción para cada elemento del stream sin modificarlo.

TO DO

## Operaciones terminales

Estas operaciones retornan objetos que no son `Stream`, disparando la ejecución de sus operaciones y cerrando este para que no sea reutilizado.

* `allMatch/anyMatch/noneMatch(Predicate<? super T> predicate)`: indica si el predicado retorna `true` para todos, alguno o ninguno de sus miembros.
* `count()`: número de elementos.
* `forEach/forEachOrdered(Consumer<? super T> action)`: ejecuta la acción para cada elemento del stream, en orden si este aplica.
   Igual que `peek` pero cerrando el stream.
* `max/min(Comparator<? super T> comparator)`: retorna el máximo/mínimo del stream según el comparador.
* `toArray()`, `toArray(IntFunction<A[]> generator)`: guarda los elementos en un array `Object[]` o del tipo dado por el generador.

* `collect` y `reduce`: TO DO

## Streams de primitivos

Si queremos trabajar con primitivos, disponemos de las interfaces `IntStream`, `LongStream` y `DoubleStream`.

Estas clases ponen a nuestra disposición, además de las maneras ya indicadas de crear un stream,
los métodos `range` y `rangeClosed` para generar streams con rangos numéricos. 
`Stream` también tiene métodos `mapToX` y `flatMapToX` para generarlos a partir de streams normales.

Un stream de primitivos puede convertirse en uno normal a través de los métodos `boxed` o `mapToObj`.

TO DO

# Ejercicios
Utilizando la API de `Stream`:
* Obten los enteros que sean cuadrados perfectos entre 100 y 1000

TO DO

# Enlaces de interés
* [Introducción a Stream (Baeldung)](https://www.baeldung.com/java-8-streams-introduction)
* [Tutorial de la API de Stream (Baeldung)](https://www.baeldung.com/java-8-streams)
