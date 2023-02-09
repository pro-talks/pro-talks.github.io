# Generic - TODO

## 1 - Почему?

```java
public class SumBox<T extends Integer, U extends Integer> {

    private T t;
    private U u;
    
    public SumBox(T t, U u) {
        this.t = t;
        this.u = u;
    }
    
    public int sum() {
        return Integer.sum(t.intValue(), u.intValue());
        //return t + u; //not compile
    }
}

@Test
public void boundExtend() {
    SumBox<Integer, Integer> integerBox = new SumBox<>(5, 10);
    int sum = integerBox.sum();

    assertEquals(15, sum);
}

Error:(12, 60) java: valueOf(int) in java.lang.Integer is defined in an inaccessible class or interface
Error:(12, 63) java: valueOf(int) in java.lang.Integer is defined in an inaccessible class or interface

//FIX
new SumBox<>(new Integer(5), new Integer(10));
```



## 2 - Generic в Runtime

"Все равно в рантайме не понятно какой внутри тип так как все стирается." -> Какая?



## 3 - Что будет

Что будет?

```java
Integer i = 10;
Class<Integer> aClass = i.getClass();
```

- Ошибка компиляции, почему?



## 4 - Материалы

- [Неочевыдные дженерики](https://www.youtube.com/watch?v=mNyQYTp-Njw)
- [Questions and Exercises: Generics](https://docs.oracle.com/javase/tutorial/java/generics/QandE/generics-questions.html)
- [Lesson: Generics](https://docs.oracle.com/javase/tutorial/extra/generics/index.html)
- [Generics in the Java Programming Language](https://www.oracle.com/technetwork/java/javase/generics-tutorial-159168.pdf)
- [Chapter 10. Arrays](https://docs.oracle.com/javase/specs/jls/se8/html/jls-10.html)