## [`static_cast`, `dynamic_cast`, `const_cast`和`reinterpret_cast`分别应该用在什么场合？](https://stackoverflow.com/questions/332030/when-should-static-cast-dynamic-cast-const-cast-and-reinterpret-cast-be-used)

### 问题
以下关键字正确的用法是什么？
- `static_cast`
- `dynamic_cast`
- `const_cast`
- `reinterpret_cast`
- C-风格转换`(type)value`
- 函数风格转换`type(value)`

应该如何考虑在特定的场合使用呢？

### [回答1](https://stackoverflow.com/a/332086)
`static_cast`应该是首选。它做的是类型之间（`int`->`float`，或者指针到`void*`）隐式转换，它也可以调用显式转换函数（或者隐式转换函数）。在许多场合，显式地声明`static_cast`也不是必须的，但是要记住`T(something)`的语法和`(T)Something`是等价的，但是应该避免这样使用。但`T(something, something_else`是安全的，并且一定会调用构造函数。

`static_cast`也可以在继承层次结构间进行转换。向上（向基类）的转换是不必要的，但是向下转换时，只要它不通过虚继承，就可以使用这种转换。但是，它不会进行检查，并且将继承结构`static_cast`到一个实际上不是该对象类型的类型是未定义的行为。

----

`const_cast`可以用来添加或者移除变量的`const`属性，其他的C++ 转换都无法去掉`const`属性（即便`reinterpret_cast`也不行）。很重要的一点是，如果原始变量是`const`，那么试图修改一个预先就是`const`的值会是未定义的。如果一个`const`引用变量对应的值是非`const`的，那么去掉这个引用的`const`会是安全的。这在重载基于const的成员函数时很有用，比如说，它也可以用以给一个对象加`const`属性，以便调用一个重载的成员函数。

虽然不太常见，但`const_cast`也可以类似`volatile`那样使用。

----

`dynamic_cast`专门用于处理多态性。你可以将指向任何多态类型的指针或引用转换为任何其他类类型（多态类型至少具有一个虚函数，声明或继承）。你不仅仅可以使用它完成向下转换 - 你可以平行转类型或甚至向上转换另一个链。`dynamic_cast`将寻找所需的对象并在可能的情况下返回它。如果不能找到，则在指针的情况下返回`nullptr`，或者在引用的情况下抛出`std::bad_cast`。

但是，`dynamic_cast`有一些局限性。如果继承层次结构中存在多个相同类型的对象（所谓的“可怕的菱形”）并且你没有使用虚继承，则它不起作用。它也只能通过`public`继承 - 它总是无法处理受`protected`或者`private`继承。然而，这很少是一个问题，因为这种形式的继承很少见。

----

`reinterpret_cast`是最危险的转换方式，应该非常谨慎地使用。它将一种类型直接转换为另一种类型 - 例如将值从一个指针转换为另一个指针，或将指针存储在`int`中，或者存储各种其他令人讨厌的东西。很大程度上，`reinterpret_cast`的唯一保证是，通常如果将结果转换回原始类型，你将获得完全相同的值（但如果中间类型小于原始类型则不会）。`reinterpret_cast`也有许多转换也无法做到。它主要用于特别奇怪的转换和位操作，例如将原始数据流转换为实际数据，或将数据存储在对齐内存指针的低位中。

----

C-风格转换和函数风格转换分别使用`(type)object`和`type(object)`，功能上是一致的。它们的定义是以下第一个成功时的转换：
- `const_cast`
- `static_cast`（忽略访问限制）
- 先`static_cast`，然后`const_cast`
- `reinterpret_cast`
- 先`reinterpret_cast`，然后`const_cast`

它们可以作为其他转换方式的替代方案，这也就导致非常危险的情况。因为它们有可能会变为`reinterpret_cast`方式的转换。后者应该是在你想要显式转换时候的首选，除非你确认`static_cast`一定成功或者`reinterpret_cast`一定失败。即便如此，还是应该考虑更长更加显式的方案。

在执行`static_cast`转换时，C风格类型转换也会忽略访问控制，这意味着它们可以执行其他方式无法完成的转换。但这并不是，反而是一个你应该避免使用C风格转换的原因。

### [回答2](https://stackoverflow.com/a/332070/3533902)
使用`dynamic_cast`：转换继承层次结构中的指针/引用。
使用`static_cast`：进行普通类型转换。
使用`reinterpret_cast`：进行底层位模式解析，使用时要格外小心。
使用`const_cast`来去除`const / volatile`。 除非你被不正确的`const API`缠住，否则请避免这样使用。