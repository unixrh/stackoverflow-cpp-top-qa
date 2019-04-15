## [为什么应该使用指针而非对象本身]()

### 问题
我来自Java背景，正开始使用C++中的对象。但是我遇到一件事是人们经常使用指向对象的指针而非对象本身。比如：
```c++
Object *myObject = new Object;
```
而不是：
```c++
Object myObject;
```
在调用函数时，不这样用：
```c++
myObject.testFunc();
```
而是：
```c++
myObject->testFunc();
```
但我无法弄清楚为什么我们这样做呢。我认为它与效率和速度有关，因为我们可以直接访问内存地址。是酱吗？

### [回答1](https://stackoverflow.com/a/22146244)
你经常看到动态分配是非常不幸的。这只是表明有多少糟糕的C ++程序员。

从某种意义上说，你有两个问题捆绑在一起。首先是我们何时应该使用动态分配（使用`new`）？第二个是我们什么时候应该使用指针？

重要的实用信息是，你应该始终使用适当的工具来完成工作。在几乎所有情况下，都比执行手动动态分配和/或使用原始指针更合适，更安全。

#### 动态分配内存
在你的问题中，您已经演示了两种创建对象的方法。主要区别在于对象的存储持续时间。在执行`Object myObject`时，在块内，对象是使用自动存储持续时间创建的，这意味着当它离开作用域时将自动销毁。当执行`new Object()`时，该对象具有动态存储持续时间，这意味着它将保持占内存的状态，直到明确删除它为止。你应该只在需要时使用动态存储持续时间。也就是说，你应该尽量创建具有自动存储持续时间的对象。

你可能需要动态分配的两种主要情况：
1. **你需要该对象比当前作用域存活更长** - 该特定对象位于该特定内存位置，而不是它的副本。如果你可以复制/移动对象（大多数时候你应该如此），你应该选择自动对象。
2. **你需要分配大量内存**，这可能很容易填满堆栈。如果我们不必关心这个（大部分时间你不必）就好了，因为它实际上超出了C ++的范围，但不幸的是我们必须处理我们正开发的系统的实际情况。

当你绝对需要动态分配时，你应该将其封装在智能指针或执行RAII的其他类型（如标准容器）中。智能指针提供动态分配对象的所有权语义。例如，查看`std::unique_ptr`和`std::shared_ptr`。如果你恰当地使用它们，你几乎可以完全避免执行自己的内存管理（参考[原则零](https://en.cppreference.com/w/cpp/language/rule_of_three)）。

#### 指针
但是，除了动态分配之外，还有其他更常用的原始指针，但大多数都有你应该选择的替代方案。和之前一样，除非你真的需要指针，否则**应该选择替代方案**。
1. **你需要引用语义。**有时你希望使用指针传递一个对象（无论它是如何分配的），因为您希望传递它的函数能够访问该特定对象（而不是它的副本）。但是，在大多数情况下，比起指针，你应该选择引用类型，因为这正是他们被设计的目的。注意，这不一定是将对象的生命周期延长到当前作用域之外，如上面的情况1所示。和之前一样，如果你可以传递对象的副本，则不需要引用语义。
2. **你需要多态性。**你只能通过指针或对象的引用以多态方式（即，根据对象的动态类型）调用函数。如果这是你需要的行为，那么你需要使用指针或引用。同样地，引用应该是首选。
3. **你希望通过允许在省略对象时传递nullptr来表示对象是可选的。**如果它是一个参数，您应该选择使用默认参数或函数重载。否则，您应该选择使用封装此行为的类型，例如`std::optional`（在C++17中引入 - 使用早期的C++标准，使用`boost::optional`）。
4. **你希望解耦编译单元以缩短编译时间。**指针的有用属性是您只需要指向类型的前向声明（要实际使用该对象，你还需要一个完整定义）。这允许你解耦部分编译过程，这可能会显著缩短编译时间。参见[Pimpl用法](http://en.wikipedia.org/wiki/Opaque_pointer)。
5. **你需要与C库或C风格的库进行交互。**此时，你不得不使用原始指针。你能做的最好的事情就是确保你只在最后一刻释放你的原始指针。您可以从智能指针获取原始指针，例如，使用其get成员函数。如果库为你执行某些分配，它希望你通过句柄释放，则通常可以使用自定义删除器将句柄包装在智能指针中，该删除器将适当地释放对象。