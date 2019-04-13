## [如何理解move语义](https://stackoverflow.com/questions/3106110/what-are-move-semantics)

### 问题
如题

### [回答1](https://stackoverflow.com/a/3109981)

看几个示例代码会比较容易理解。让我们从一个非常简单的string类开始，该类只保存指向堆分配的内存块的指针：
```c++
class string
{
    char* data;

public:

    string(const char* p)
    {
        size_t size = strlen(p) + 1;
        data = new char[size];
        memcpy(data, p, size);
    }
```
既然我们选择了自己管理内存，那么我们必须遵循[三个原则](http://en.wikipedia.org/wiki/Rule_of_three_%28C++_programming%29)。我先不编写赋值运算符，现在只实现析构函数和复制构造函数：
```c++
    ~string()
    {
        delete[] data;
    }

    string(const string& that)
    {
        size_t size = strlen(that.data) + 1;
        data = new char[size];
        memcpy(data, that.data, size);
    }
```
拷贝构造函数被定义来拷贝一个string对象，参数`const string& that` 绑定了所有string类的表达式，允许我们做这样的拷贝操作：
```c++
string a(x);                                    // Line 1
string b(x + y);                                // Line 2
string c(some_function_returning_a_string());   // Line 3
```
现在来看关于`move`语义的关键见解。请注意，只有在我们复制x的第一行才真正需要深拷贝，因为我们可能希望稍后再访问x，并且希望x不会被意外地改变。你是否注意到我刚刚说了三次x（如果包含这个句子，则是四次）并且每次都表示完全相同的对象？我们称`x`这样的表达式为`lvalues`。
第2行和第3行中的参数不是`lvalues`，而是`rvalues`，因为底层字符串对象没有名称，因此我们无法在以后再次访问它们。`rvalues`表示在下一个分号处被销毁的临时对象（更准确地说：在词法上包含`rvalue`的全表达式的末尾）。这很重要，因为在b和c的初始化期间，我们可以用源字符串做任何我们想做的事情，但对于我们而言这是无区别的！

`C++11`引入了一种名为`rvalue reference`的新机制，它允许我们通过函数重载检测`rvalue`参数。我们所要做的就是编写一个带有右值引用参数的构造函数。
在构造函数内部，只要我们将它保留在某个有效状态，我们就可以对源数据执行任何操作：
```c++
    string(string&& that)   // string&& is an rvalue reference to a string
    {
        data = that.data;
        that.data = nullptr;
    }
```
我们做了什么呢？我们刚刚复制了指针，然后将原始指针设置为null（以防止源对象的析构函数中的`delete []`释放`刚盗得的数据`），而不是深拷贝堆数据。实际上，我们“窃取”了最初属于源字符串的数据。同样，关键的点是在任何情况下客户都无法检测到源已被修改。由于我们在这里没有真正复制，我们将此构造函数称为“移动构造函数”。它的工作是将资源从一个对象移动到另一个对象而不是复制它们。

恭喜，您现在了解移动语义的基础知识！让我们继续实现赋值运算符。如果您不熟悉[复制和交换]()习惯用法，请先去学习它再回来看看，因为它是一个与异常安全相关的很酷的C ++习惯用法。
```c++
    string& operator=(string that)
    {
        std::swap(data, that.data);
        return *this;
    }
};
```
嗯？就这样了吗？关于右值引用有没有什么说法？我的回答是这里不需要它。

请注意，我们通过传值来传递`that`，因此`that`必须像其他string对象那样被初始化。那确切地`that`会如何被初始化呢？在`c++98`里，答案就是通过拷贝构造函数来初始化。在`c++11`中，编译器会依据赋值运算符是左值还是右值来决定使用拷贝构造函数或是移动构造函数。

所以如果你写`a = b;`，那么就会使用拷贝构造函数来初始化`that`（因为`b`是一个`lvalue`）。赋值运算符会使用新创建的深拷贝来交换内容。这正是拷贝与交换的定义——制作副本，用副本交换内容，然后通过离开作用域来删除副本。这里没什么新鲜的。

但如果你写`a = x + y`，那么就会使用移动构造函数来初始化`that`（因为表达式`x + y`是一个`rvalue`）。因此不会涉及深拷贝，只有一次高效的移动。`that`仍然是参数的一个独立对象，但它的结构很简单，因为堆数据不必复制，只需移动即可。没有必要复制它，因为`x + y`是一个`rvalue`，同样，可以从`rvalues`表示的string对象移动也是OK的。

总结而言，拷贝构造函数会制作一份深拷贝，因为它的源数据必须不被污染。而移动构造函数仅仅需要拷贝指针，并将源数据的指针设为`NULL`。这样将源对象置空的操作不会有问题，因为客户端不会再检测其值了。

我希望这个例子解释了重点部分。`rvalue reference`和`move`语义还有很多可说的，我为了简单故意省略了。如果想了解更多详情，请参阅我的[补充答案](https://stackoverflow.com/a/3109981)。