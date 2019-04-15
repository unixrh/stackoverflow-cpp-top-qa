## [`nullptr`到底是什么？](https://stackoverflow.com/questions/1282295/what-exactly-is-nullptr)

### 问题
`C++11`有许多新特性，一个有趣又令人疑惑（至少我疑惑）的特性是新的`nullptr`。

好吧，再也不需要令人厌烦的宏`NULL`了。
```c++
int* x = nullptr;
myclass* obj = nullptr;
```
但我还是不知道`nullptr`是如何工作的。比如，[wiki文章](http://en.wikipedia.org/wiki/C%2B%2B11#Null_pointer_constant)说道：
> `C++11`通过引入一个特别的null指针常量`nullptr`作为新的关键字，来解决这个问题。它的类型是`nullptr_t`，可以和任何指针类型或者指向成员的类型作隐式转换以及比较操作。除了`bool`以外，它不可以与整型做隐式转换或比较。

它怎么可以同时是关键字和类型的实例？

有没有一个例子（除了上面的wiki里的）能够说明`nullptr`比以前的`0`更好？

### [回答1](https://stackoverflow.com/a/1282345)

> 它怎么可以同时是关键字和类型的实例？

这没什么值得奇怪的，`true`和`false`都同时是关键字以及类型（`bool`）。`nullptr`是`std::nullptr_r`一个指针字面量，同时也是`prvalue`（你无法通过`&`取得它的地址）。

- `4.10`关于指针转换说到`std::nullptr_t`类型的`prvalue`是一个空指针常量，并且一个整型空指针可以转换为`std::nullptr_t`，反向的转换是不允许的。这就允许为指针和整型重载函数，通过传递`nullptr`来选择指针版函数。而通过传递`NULL`则会迷惑地选择`int`版函数。
- 将`nullptr_t`转换为整型的操作需要`reinterpret_cast`，并且这和将`(void*)0`转换为整型有着相同的语义。`reinterpret_cast`无法将`nullptr_t`转化为任何指针类型。如果可能的话，可以依赖隐式转换或者`static_cast`。
- 标准要求将`sizeof(nullptr_t)`应该变为`sizeof(void*)`。

#### 评论补充
- 注意，`NULL`甚至不能保证为`0`。它可以是`0L`，这时调用`void f(int)`，`void f(char *)`就会有歧义。`nullptr`总是会选择指针的版本，而不会选择整型的版本。同时还要注意`nullptr`可以转换为`bool`。