## [C++11引入了标准化的内存模型，这意味着什么，这将如何影响Ｃ++编程？](https://stackoverflow.com/questions/6319146/c11-introduced-a-standardized-memory-model-what-does-it-mean-and-how-is-it-g)
### 问题
C++11引入了标准化的内存模型，但准确地说这意味着什么呢？这将如何影响Ｃ++编程呢？
有一篇[文章](http://www.theregister.co.uk/2011/06/11/herb_sutter_next_c_plus_plus/page2.html)（作者
是Gavin Clarke，他引用[Herb Sutter](https://en.wikipedia.org/wiki/Herb_Sutter)）说道：
> 内存模型意味着，无论使用何种编译器或者在何种平台上运行，C++ 代码都有一个标准库可供调用。
有一种标准的方式来控制不同的线程如何与处理器的内存进行通信。

> ”当说道将代码按标准分割在不同的核心上时，我们说的就是内存模型。我们会在不破坏以下假设的基础上，对它进行优化。“ Sutter说道。

当然我可以记住这些以及网络上类似的文章（我自打出生以来就有自己的记忆模型），甚至于可以回答别人问的相关问题。
但说实话，我并不确切地理解这些。

C++程序员以前就开发多线程应用，所以是POSIX线程或者Windows线程，或是C++11线程，这有什么关系呢？它的好处在哪里？我想了解底层的细节。

我不清楚多线程的底层是如何工作的，也不清楚通常意义上的内存模型是什么意思，请帮我理解这些概念。

### [回答1](https://stackoverflow.com/a/6319356)

首先，你必须学着像个语言律师那样思考。

C++11的标准并不参考任何特定的编译器、操作系统或者CPU。它参考的是一个通常系统对应的抽象机器。
在语言律师的世界中，程序员的工作就是为抽象机器写代码，编译器的工作就是将代码实现在具体的机器上。
严格参照标准写代码，就能保证不论是今天还是50年后，你的代码能够不经修改地在任何系统上通过兼容的C++编译器来编译和运行。

C++98/C++03标准中的抽象机器基本上是单线程的，因此无法写出处处可用的多线程代码。
该规范甚至没有说明内存加载和存储的原子性或加载和存储可能发生的顺序，更不用说像互斥体这样的东西了。

当然，你在实践中可以为特定的具体系统写多线程代码——比如pthreads或者Windows，但是C++98/C++03并没有一个标准的方式来写多线程代码。

C++11的抽象机器就设计为了多线程，同时具备良好定义的内存模型，也就是说，它规定了当涉及到访问内存时，编译器能做啥不能做啥。

考虑下面这个例子，当一对全局变量被并发的两个线程访问：
```C++
           Global
           int x, y;

Thread 1            Thread 2
x = 17;             cout << y << " ";
y = 37;             cout << x << endl;
```
`Thread2` 可能输出什么呢？

在C++98/C++03标准下，这甚至不是未定义的行为，这问题本身就毫无意义，因为标准不知道什么是thread。

在C++11标准下，这个结果就是未定义的行为。因为通常加载和存储不是原子性的，这看起来并不像是个改进项。事实也确实如此。

但有了C++11，你可以这样写：
```C++11
           Global
           atomic<int> x, y;

Thread 1                 Thread 2
x.store(17);             cout << y.load() << " ";
y.store(37);             cout << x.load() << endl;
```
这下事情变得有意思多了。首先，这里的行为都是有定义的，Thread2可以打印出`0 0`（如果它在Thead1之前运行的话），`37 17`（如果它在Thread1之后运行），或者`0 17`（如果它在Thread1赋值给x之后，在赋值给y之前）。

但它无法打印出`37 0`，因为C++11默认的原子性载入/存储强制顺序一致性。也就是说好像是按照你在各个线程中写的代码顺序执行，实际上包含有线程间交错执行。因此默认的原子性为载入和存储同时提供了原子性和有序性。

如今的现代CPU，要保证顺序一致性的开销很大。特别是，编译器可能会在每次访问内存之间发出完整的内存屏障。但如果你的算法可以容忍载入/存储的无序性，也就是说它要求原子性但不要求有序性，比如它可以允许输出`37 0`，你可以这样写：
```C++
           Global
           atomic<int> x, y;

Thread 1                            Thread 2
x.store(17,memory_order_relaxed);   cout << y.load(memory_order_relaxed) << " ";
y.store(37,memory_order_relaxed);   cout << x.load(memory_order_relaxed) << endl;
```
越是现代化的CPU，这个示例越可以比之前的示例更快。

最后，如果你需要保证特定的载入/存储有序，你可以这样写：
```c++
           Global
           atomic<int> x, y;

Thread 1                            Thread 2
x.store(17,memory_order_release);   cout << y.load(memory_order_acquire) << " ";
y.store(37,memory_order_release);   cout << x.load(memory_order_acquire) << endl;
```
这就又回到了有序的载入/存储，因此`37 0`也就不再可能出现，但它以最小的开销实现了这一点。
（在这个简单的例子中，结果与完整的顺序一致性相同;在较大的程序中则不行。）

当然，如果您想要查看的唯一输出是`0 0`或`37 17`，则可以为原始代码包装互斥锁。
但是如果你已经读到这里了，我打赌你已经知道它确实可行，这个答案已经比我预想的要长:-)。

互斥量很棒，C++11已经将其标准化了。但有时出于性能原因，你需要较低级别的原语（比如经典的[双重检查锁定模式](http://www.justsoftwaresolutions.co.uk/threading/multithreading-in-c++0x-part-6-double-checked-locking.html)）。新标准提供了高级别的小工具，比如互斥锁和条件变量，它同时还提供了低级别的小工具，比如原子类型和各种内存屏障。因此你完全可以使用标准指定的语言编写复杂的高性能并发程序，并且可以确定你的代码将在今天的系统和未来的系统上编译和运行。

然而坦白说，除非你一个研究底层代码的专家，你要做的应该是使用互斥锁和条件变量，我就会这么干。

关于这方面更多的信息，参考[这篇博文](http://bartoszmilewski.wordpress.com/2008/12/01/c-atomics-and-memory-ordering/)。
