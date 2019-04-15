## [关键字`explicit`是什么意思？](https://stackoverflow.com/questions/121162/what-does-the-explicit-keyword-mean)
### 问题
关键字`explicit`是什么意思？

#### 评论
- 我想告诉那些`C++11`之后才开始开发C++的朋友，`explicti`不仅仅可以用于构造函数，它同样可以用于转换运算符。比如你有个一个类`BigInt`，其转换运算符为`int`，其显式转换运算符确实`std::string`。你可以声明：`int i = myBigInt;`，但你必须显示地转换（建议使用`static_cast`）来声明`std::string s = myBigInt`;

### [回答1](https://stackoverflow.com/a/121163)
编译器可以通过隐式转换来将为函数解析参数。这就是说编译器可以使用单个参数的构造函数来转换一个类型，以便获得该参数正确的类型。

这有一个示例类，带有一个可用于隐式转换的构造函数：
```c++
class Foo
{
public:
  // 单参数构造函数，可用于隐式转换
  Foo (int foo) : m_foo (foo) 
  {
  }

  int GetFoo () { return m_foo; }

private:
  int m_foo;
};
```
以下示例函数中使用一个`Foo`对象作为参数：
```c++
void DoBar (Foo foo)
{
  int i = foo.GetFoo ();
}
```
`DoBar`函数的调用在这里：
```c++
int main ()
{
  DoBar (42);
}
```
这个参数不是`Foo`对象，而是一个`int`。然而，`Foo`还是存在这样的一个构造函数，使用`int`作为参数，以便这个构造函数可以用来将参数转为正确的类型。

对于每个参数，编译器被允许这样做一次。

使用`explicit`关键字前缀的构造函数，不允许编译器使用这样的构造函数来做隐式转换。加上`explicit`的话，上面的代码就会在函数调用`DoBar(42)`时报编译错误，这就需要显示地转换`DoBar(Foo(42))`。

这样做的目的是为了防止隐含的bug。

比如：
你有一个类的构造函数`MyString(int size)`，它使用一个给定的长度值`size`来构造一个字符串。你有一个函数`print(const MyString&)`，你这样调用：`print(3)`(而你实际想要调用的是：`print("3")`)。 你期望打印出`"3"`，但它打印出一个长度为3的空字符串。

#### 评论
- 好回答！或许你应该提到当多参数构造函数有默认值时，它也可以像是一个单参数构造函数。比如`Object( const char* name=NULL, int otype=0)`。
- 我觉得还应该提到的一点是：应该考虑到让单参数构造函数自动加上`explicit`，并且只在想*特意*实现隐式转换时将`explicit`去掉。我觉得应该构造函数应该是默认显示的，然后有个关键字`implicit`来是允许它支持隐式转换。但现在可惜不是如此。