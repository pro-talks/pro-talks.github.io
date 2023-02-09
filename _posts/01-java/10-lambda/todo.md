# TODO

1. [Java SE 8: Lambda Quick Start](https://www.oracle.com/webfolder/technetwork/tutorials/obe/java/Lambda-QuickStart/index.html)
2. [Functional interfaces](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html)
3. [Functional Interfaces](https://docs.oracle.com/javase/specs/jls/se8/html/jls-9.html#jls-9.8)



## Вопросы

Почему есть проблемы у анонимных/локальных классов, но нет у лямбд?

- ??? Lambda expressions are **lexically scoped**. This means that they **do not inherit any names from a supertype** or introduce a new level of scoping. Declarations in a lambda expression are 