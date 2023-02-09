---
layout: post
title: "Stream API"
date: 2019-07-04 12:00:00 +0300
categories: java
permalink: stream-api
---

# Stream API

Stream API - технология, появившаяся в java 8 под влиянием идей:

- функциональное программирование (`stream-style`),
- параллельная обработка,
- ленивая обработка,
- оптимизация вычислений (`short-circuit operations`).

## Характеристики Stream

- "A multipilicity of values"

- Не структура данных (no storage)

- Выполнение операций отложено до последнего (lazy)

- Может быть бесконечным

- Не мутирует источник

- Одноразовый

- Ordered/Unordered

- Parallel/Sequential

- Примитивые специализации: IntStream, DoubleStream, LongStream.

  

## Stream-style

`Stream-style` подзволяет сократить количество строк кода в определенных случаях, например.

#### Java 7

```java
Set<Group> groups = new HashSet<>();
for (People p : people) {
    if (p.getAge() >= 65)
        groups.add(p.getGroup());
}
List<Group> sorted = new ArrayList<>(groups);
Collections.sort(sorted, new Comparator<Group>() {
    @Override
    public int compare(Group o1, Group o2) {
        return Integer.compare(o1.getSize(), o2.getSize());
    }
});
for (Group g : sorted) {
    System.out.println(g.getName());
}
```

#### Java 8

```java
people.stream()
    .filter(p -> p.getAge() >= 65)
    .map(People::getGroup)
    .distinct()
    .sorted(comparing(Group::getSize))
    .map(Group::getName)
    .forEach(System.out::println);
```



## Параллельная обработка

`Stream API` позволяет выполнять параллельные вычисления минимальными усилиями. Достаточно использовать `parallelStream()` вместе `stream()`, чтобы включить парреллельную обработку. Под капотом `Stream API` работает на `ForkJoinPool`.

````java
people.parallelStream() // or .stream().parallel()
    ..
    .forEach(System.out::println);
````

Параллельное вычисление не включается автоматически, необходимо явно указать где нужна параллельная работа. Это сделано потому что, сложно определить в каком случае параллельное вычисление принесет явный выигрыш в производительности.

Но зачастую имеет смысл проверить `parallelStream()`, когда:

- есть операции "жрущие процессорное время",
- работа занимает больше чем 10^5 наносекунд.

Также нужно помнить, что  `Stream API` - это не магия, это много кода под капотов. Существует множестов кейсов, где `Stream API` будет работать медленней, чем "код java 7".



## Ленивые вычисления

Выполнение операций в `Stream API` отложено до последнего! Это возможно благодарю дизайну с использованием `pipline`.

`Source -> op -> op -> op -> sink`

"source": collection, iterators, channels, ...
"operations": filter, map, reduce, ...
"sink": collections, locals, ...

Операции делятся на два типа: промежуточные и терминальные операции. Промежуточные операции возвращают тип `Stream`, терминальные - `не Stream` и совершают какое-то конечное действие.



## Short-circuit operations

`Short-circuit operations` - это операции, которые завершают вычисления, когда находят ответ, например, `findFirst()` или `limit(n)`.

> Такие операции можно использовать при работе с бесконечными потоками.



## Создание Stream

Важно, что `Stream` использует информацию об источнике, а также поддерживает его свойства: `sized`, `ordered`, `distinct`, `sorted`. То есть если источник был упорядочен, то `Stream` будет соблюдать свойство `ordered`. А если `Stream` был создан из `HashSet`, то операция `distinct` ничего не будет стоить.

### Factories, builders

````java
// Stream from array
Stream<String> s1 = Arrays.stream(arr);

// Stream from values
Stream<String> s2 = Stream.of(v1, v2, v3, v4);

// Stream builder
Stream<Object> s3 = Stream.builder()
    .add(v1).add(v2).add(v3).add(v4)
    .build();

// Stream generation
Stream<Integer> s4 = IntStream.range(1, 4)
    .boxed();

Stream.of(s1, s2, s3, s4)
    .reduce(Stream::concat)
    .orElseGet(Stream::empty)
    .forEach(System.out::println);
````

### Generators

````java
// Stream.generate
AtomicInteger init = new AtomicInteger(0);
Stream<Integer> s1 = Stream.generate(init::getAndIncrement);

// Stream.iterate
Stream<Integer> s2 = Stream.iterate(0, i -> i + 1);
````



## Промежуточные операции

````java
Stream<S> s;
Stream<S> s.filter(Predicate<S>);
Stream<T> s.map(Function <S, T>);
Stream<T> s.flatMap(Function<S, Stream<T>>);
Stream<S> s.peek(Consumer<S>);
Stream<S> s.sorted();
Stream<S> s.distinct();
Stream<S> s.limit(long);
Stream<S> s.skip(long);

Stream<S> s.unordered();
Stream<S> s.sequential();
Stream<S> s.parallel();  
````

- `Stream<S> s.filter(Predicate<S>) - фильтрует по входному `Predicate<S>`;

    ````java
    // Посчитать сумму чётных чисел от 0 до 99
    int sum = Stream.iterate(0, i -> i + 1)
        .limit(100)
        .filter(i -> i % 2 == 0)
        .mapToInt(i -> i)
        //.peek(System.out::println)
        .sum();
    System.out.println("Sum is " + sum);
    ````

- `Stream<T> s.map(Function <S, T>)` - изменяет тип `Stream` с помощью функции (`Function <S, T>`);

    ````java
    // Получить множество возврастов людей из списка людей 
    Set<Integer> collect = people.stream()
        .map(People::getAge)
        .collect(Collectors.toSet());
    ````

- `Stream<T> s.flatMap(Function<S, Stream<T>>)` - позволяет объединить и обработать вложенные `Stream`;

    ````java
    // Объединить элементы из вложеных коллекций
    List<List<People>> list = Arrays.asList(people, people);
    list.stream()
        .flatMap(Collection::stream)
        .forEach(p -> System.out.println(p.getName()));
    ````

- `Stream<S> s.peek(Consumer<S>)` - выполняет действие, используя `Consumer<S>`;

    >  Важно помнить, что `peek` - это промежуточная, а значит ленивая операция, которая будет выполняться только при наличии терминальной операции в конце.
    
    ````java
    /** Такой код не респечатает элементы */
    Stream.of(1, 2, 3).peek(System.out::println);
    
    /** Такой код респечатает! */
    Stream.of(1, 2, 3)
        .peek(System.out::println)
        .collect(Collectors.toList());
    ````

- `Stream<S> s.sorted()` - отсортирует элементы `Stream`;

- >  Если `S` имплементирует  `java.lang.Comparable` , если нет - вернет `ClassCastException` c сообщением `"..clazz" cannot be cast to java.lang.Comparable`.

- `Stream<S> s.sorted(Comparator<? super T> comparator)` - отсортирует элементы `Stream`, используя *comparator*;

    ````java
    people.stream()
        //.sorted()
        .sorted(comparing(People::getAge))
        .forEach(p -> System.out.println(p.getAge()));
    ````

- `Stream<S> s.distinct()` - возвращает `Stream` без дубликатов;

    ````java
    // Удалить дубликаты в Stream
    List<List<People>> list = Arrays.asList(people, people);
    list.stream()
        .flatMap(Collection::stream)
        .distinct()
        .forEach(p -> System.out.println(p.getName()));
    ````

- `Stream<S> s.limit(long)` - возвращает "обрезанный с конца" `Stream` ;

- `Stream<S> s.skip(long)`  - возвращает "обрезанный с начала" `Stream` ;

  ````java
  // Обрезать Stream с конца
  people.stream()
      .limit(3)
      .forEach(p -> System.out.println(p.getName()));
  
  // Обрезать Stream с начала
  people.stream()
      .skip(3)
      .forEach(p -> System.out.println(p.getName()));
  ````

- `Stream<S> s.unordered()` - говорит стриму "забудь об упорядоченности", чтобы получить производительность;

  > Stream по дефолту поддерживает сортироку, если источник поддерживал ее. Однако поддержка сортировки требует работы, `unordered` позволяет отключить сортировку и повысить производительность.

- `Stream<S> s.sequential()` - однопоточная обработка `Stream`;

- `Stream<S> s.parallel()`  - многопоточная обработка `Stream`;

  ````java
  // Убрать поддержку порядка для многопоточной обработки
  people.stream()
      .parallel()
      .unordered()
      .forEach(p -> System.out.println(p.getName()));
  ````



## Терминальные операции

Терминальные операции можно разбить на следующие категории:

- итерация: forEach, iterator;

- поиск: findFirst, findAny;

- проверка: allMatch, anyMatch, noneMatch;

- агрегаторы: 

  - reduction,
  - collectors.

### Collectors

Терминальная операция, которая объединяет входные параметры в какой-либо результат.

````java
// Конкатенaция имен людей через запятую
String s1 = people.stream()
    .map(People::getName)
    .limit(3)
    .collect(Collectors.joining(", "));
	//.collect(Collectors.toList());
````

### Reduction

Терминальная операция, которая возвращает результат вычисленый по формуле.

````java
// Сложить элементы последовательности
Integer value = Stream.iterate(1, i -> i + 1)
    .peek(System.out::println)
    .limit(2)
    .reduce((x, y) -> x % y)
    .orElseThrow(() -> new RuntimeException("Something went wrong"));
````

> В формуле необходимо использовать ассоциативные операции! Если будут использованы неассоциативные операции, результаты будут плавающими.



## Вопросы

1. Зачем добавили `Stream API`? Какой профит можно получить?

2. Как спроектировано `Stream API`?

3. Как `Stream API` работает с многопоточностью?

4. Что такое промежуточные и терминальные операции?

5. Как работают методы:

   - `flatMap`,
   - `peek`
   - `distinct`,
   - `skip`,
   - `unordered`?

6. Оцените алгоритмическую сложность метода `s.distinct`?

7. Как работает метод `.reduce()`? Что будет если операция не ассоциативная?

8. Что делает данный код?

   ````java
   Stream<People> stream = people.stream();
   stream.forEach(p -> System.out.println(p.getName()));
   List<String> collect = stream.map(People::getName).collect(toList());
   ````

9. Как переобразовать список строк в одну с разделителем?
   
   - Как преобразовать 5 отдельных строк в одну с разделителем?
   
10. Зачем `Stream.forEach`, когда есть `Iterable.forEach` ?

11. Зачем сделали `Stream.forEachOrdered`?

12. Что легче параллелизовать `ArrayList` или `LinkedList`, почему?

13. Как происходит параллезация, если `Stream` создается от `Iterator`?

14. Потокобезопасен ли `Stream.forEach` ?

15. Когда стоит использовать параллельную обработку на `Stream`?

16. Что такое `Spliterator` и как он работает?

17. Какие характеристики `Stream'a` (`Spliterator'a`) существуют?

18. Как работает метод `sum` для сложения плавающих чисел (не ассоциативной операции) ?



##  Источники

2. [Java Doc - Stream](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)
3. [Java Doc - Stream Summary](https://docs.oracle.com/javase/8/docs/api/java/util/stream/package-summary.html)
3. [Java Tutorial - Stream Aggregate Operations](https://docs.oracle.com/javase/tutorial/collections/streams/index.html)
4. [Сергей Куксенко - Stream API](https://www.youtube.com/watch?v=O8oN4KSZEXE)



## [Скачать приложение]({{ site.baseurl }}/download/01-java/stream-api.zip)
