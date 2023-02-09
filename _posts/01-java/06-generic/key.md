# Вопросы

1. Зачем используются generic?

   - Проверка типов на стадии компиляции. 
   - Устранение ручного кастования.
   - Возможность писать обобщенные алгоритмы.

2. Почему нужно проверять типы на стадии компиляции?

   - Существуют `Compile` и `Runtime` ошибки. `Compile` ошибки в разы дешевле, чем `Runtime`.

3. Как происходит кастование с использование generic?

   - При использовании `generic` указывается тип в `<...>`, который используется компилятором для кастования автоматически.

     ```java
     List<String> list2 = new ArrayList<>();
     list2.add("hello");
     String s2 = list2.get(0); //without (String) list.get(0);
     ```

4. Что такое обобщенные алгоритмы?

   - Обобщенные алгоритмы - это такие алгоритмы, которые применяются к различным типам данных, не меняя алгоритмы.

5. Сколькими типами может быть параметризован класс? (`public class Box<...>`)

   - [Максимальным числом полей в калссе! -> 2^16=65535](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.11)

6. Какие можно давать имена параметрам? А какие нужно?

   - Можно давать различные имена!

     ```java
     public class GenericBox<HelloWorld> {
     
         private HelloWorld value;
     
         public void setValue(HelloWorld o) {
             this.value = o;
         }
     
         public HelloWorld getValue() {
             return this.value;
         }
     }
     ```

   - Имена параметров должны быть одиночными заглавным буквами, например:

     > E - Element,
     >
     > K - Key,
     >
     > N - Number,
     >
     > T - Type,
     >
     > V - Value.

7. Что будет?

   ```java
   public class GenericBox<T> {
   
       private T value;
   
       public void setValue(T o) {
           this.value = o;
       }
   
       public T getValue() {
           return this.value;
       }
   }
   
   //run this
   public void createBox() {
       GenericBox<GenericBox<GenericBox<?>>> box = new GenericBox<>();
       box.setValue(new GenericBox<>());
   }
   ```

   - Скопмилируется нормально.

8. Какая может быть максимальная вложенность?

   - Любая.
   - Можно этот вопрос свести так "Какая самая длинная иерархия объектов может быть"? Логично, что должна поддерживаться любая иерархия объектов на сколько хватает памяти.

9. Что такое `raw type`?

   - `Raw type` (сырой тип) - это, когда параметризованный класс создан без указания параметра. 

10. Что такое `unchecked warning` и в каких случаях выбрасываются?

- `Unchecked warning` выбрасывается при использовании сырых типов, а именно:
  - если параметризованному типу присвоить сырой тип;
  - если используются параметризованные методы объекта, но объект создан без указания типа параметра.
- `Unchecked warning` НЕ выбрасываются:
  - если сырому типу присвоить параметризованный тип;
  - если используются не параметризованные методы, например, `toString()`.

11. Как отключить `unchecked warning`?

    - `@SuppressWarnings("unchecked")`

12. Как создать параметризованный метод?

    - Нужно указать параметры в угловых скобках до возвращаемого значения.

13. Может ли быть параметризованный конструктор или статический метод?

    - Может.

14. Чем отличается параметризованный класс от параметризованного метода?

    - Область видимости для параметризованного метода только в пределах метода.

15. Могут ли быть несколько классов после `extends`?

    - Нет, правило "один класс, много интерфейсов".

16. Что может быть использовано после `extends`, классы или интерфейсы?

    - Класс и интерфейсы.

17. Важен ли порядок перечисления классов и интерфейсов?

    - Важен. Сначала класс, а потом интерфейсы.

18. Можно ли создать параметризованный класс `Car<T super Number>`

    - Нет, синтаксис `super Number` используется только для `wildcard`.

19. Что нужно сделать чтобы можно было сравнивать параметризованные элементы?

    ```java
    public static <T> int countGreaterThan(T[] anArray, T elem) {
        int count = 0;
        for (T e : anArray)
            if (e > elem)  // compiler error
                ++count;
        return count;
    }
    ```

    - Объекты нельзя сравнивать с помощью `>`, только примитивы, поэтому нужно использовать `Comparable`.

      ```java
      public static <T extends Comparable<T>> int countGreaterThan(T[] anArray, T elem) {
          int count = 0;
          for (T e : anArray)
              if (e.compareTo(elem) > 0)
                  ++count;
          return count;
      }
      ```

20. Что будет, если вызвать `genericMethod1()`?

    ```java
    public void genericMethod1() {
        GenericBox<Integer> box = new GenericBox<>();
        boxTest(box);
        GenericBox<Double> box2 = new GenericBox<>();
        boxTest(box2);
    }
    
    private void boxTest(GenericBox<Number> box) { 
        System.out.println(box.getValue());
    }
    ```

    - Не скомпилируется. Так как `boxTest` принимает тип `GenericBox`, параметризованный только `Number`. 
      - Можно пофиксить так `GenericBox<? extends Number>`

21. Что будет, если вызвать `genericMethod2()`?

    ```java
    public void genericMethod2() {
    	Serializable s = pick("d", new ArrayList<String>());
    }
    
    <T> T pick(T a1, T a2) {
    	return a2;
    }
    ```

    - Нормально отработает и вернет `ArrayList` в ссылку `Serializable`

22. Что будет, если вызвать `processStringList(Collections.emptyList())`?

    ```java
    void processStringList(List<String> stringList) {
        // process stringList
    }
    
    //FYI - implementation emptyList
    public static final <T> List<T> emptyList() {
        return (List<T>) EMPTY_LIST;
    }
    ```

    - В java 8 скомпилируется.
    - В java 7 нет, нужно передавать `Collections.<String>emptyList()`

23. Что будет, если вызвать `genericMethod2()`?

    ```java
    public void genericMethod2() {
        List<Integer> intList = Collections.emptyList();
        printList(intList);
    }
    
    public void printList(List<Object> list) {
        for (Object elem : list)
            System.out.println(elem + " ");
        System.out.println();
    }
    ```

    - Не скомпилируется, `printList(List<Object> list)` принимает только `Object`! Можно заменить на `List<?> list`.

24. Что будет, если вызвать `genericMethod2()`?

    ```java
    public void genericMethod2() {
        printList(Collections.emptyList());
    }
    
    public void printList(List<Object> list) {
        for (Object elem : list)
            System.out.println(elem + " ");
        System.out.println();
    }
    ```

    - Скомпилируется в java 8, в java 7 - нет

25. Как написать generic метод, который будет принимать `List<Integer>`, `List<Double>`, `List<Number>`?

    ```java
    private <T extends Number> void getCollection(List<T> list) {
        list.forEach(System.out::println);
    }
    
    private void getCollection2(List<? extends Number> list) {
        list.forEach(System.out::println);
    }
    ```

26. В чем разница между `wildcard` и `generic`?

    ```java
    private <T extends Number> void getCollection(List<T> list) {
        list.forEach(System.out::println);
        list.add((T) new Double(123.0)); //possible add item
    }
    
    private void getCollection2(List<? extends Number> list) {
        list.forEach(System.out::println);
        list.add(new Double(123.0)); //not possible add item
    }
    ```

27. Какие типы аргументов может принимать данный метод?

    ```java
    public static void addNumbers(List<? super Integer> list) {
        for (int i = 1; i <= 10; i++) {
            list.add(i);
        }
    }
    ```

    - `List<Integer>`, `List<Number>`, `List<Object> `

28. Какая разница между `List<Object>` and `List<?>`?

    - В `List<Object>` можно добавить любой объект, а в `List<?>` только `null`.

29. Почему в `Collection<?>` разрешается класть только null, а `Collection<? super Object>` - любой объект?

    - `Collection<? super Object>` - `wildcard`'ы `<? super ..>` предназначены для операций записи.
    - `Collection<?>` - `wildcard`'ы `<?>` и `<? extends ..>`  - условно `read only`.

30. Что будет и почему?

    ```java
    Collection<? extends String> list = new ArrayList<>();
    list.add("");
    ```

    - Ошибка компиляции.
    - Так как коллекции с типом `<? extends ..>` условно `read only`.

31. Что значить что коллекции с типом `<? extends ..>` условно `read only`?

    - Нельзя напрямую добавить элемент.
    - Но можно добавить `null`; удалить элемент, используя `Iterator`; вызвать `.clear()`, изменить используя, параметризованный метод (`wildcard capture`).

32. Какие `wildcard`'ы выбирать для чтения , а какие для записи?

    - `<? extends ..>` для чтения
    - `<? super..>` для записи 

33. Какие `wildcard`'ы нужно использовать, для операций чтения и записи?

    - Никакие, лучше использовать `generic`'и.

34. Когда следует использовать `<?>`?

    - Если предполагается только чтение, и кастование к `Object`.

35. Почему `<? super ..>` не удобные для чтения?

    - Так как при извлечения объекты кастуются к `Object`.

36. Какая разница между `<T>` и `<?>`?

    - Например, объект типа `T` можно добавить в коллекцию `List<T>`, а `<?>` - нельзя.

37. Что такое `Erasure`?

    - Процесс стирания параметризованных типов (`generic`'ов)  во время компиляции.

38. Что такое `Non-Reifiable Types` и `Reifiable Types`?

    - `Not-reifialbe type` - тип данных, информация о котором не доступна в полной мере в `Runtime` - `generic`'и. 
    - `Reifialbe type` - тип данных, информация о котором доступна в полной мере в `Runtime` - примитивы, не `generic`, сырые типы, `unbounded wildcard`.

39. Что такое `Heap Pollution`?

    - `Heap pollution` процесс когда объект параметризованного типа ссылается на объект, который не является этим типом. Что приведет, к `ClassCastException`.

40. Как избежать `Heap Pollution`?

    - `Heap pollution` не произойдет, если код не бросает `unchecked warning`.

41. Почему нельзя использовать массивы с `generic`?

    - Использовать можно! Нельзя создавать с указанием параметризованного типа.

      ```java
      List<String>[] arr = new List[3]; // Ok
      List<String>[] arr = new List<String>[3]; // Compile-time error
      ```