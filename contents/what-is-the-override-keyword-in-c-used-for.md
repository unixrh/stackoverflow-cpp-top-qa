## [override 关键字是做什么用的？](https://stackoverflow.com/questions/18198314/what-is-the-override-keyword-in-c-used-for)
### 问题

> 如题

### [回答1](https://stackoverflow.com/a/18198377/3533902)
`override`关键字有两个作用：
1. 告诉阅读代码的人“这是一个虚函数，是基类虚函数的覆盖实现”
2. 让编译器检查这个函数是否真的是覆盖基类虚函数，而非你自己新写的函数。
```c++
class base
{
  public:
    virtual int foo(float x) = 0; 
};


class derived: public base
{
   public:
     int foo(float x) override { ... do stuff with x and such ... }
}

class derived2: public base
{
   public:
     int foo(int x) override { ... } 
};
```
在`derived2`中，编译器会报错“changing the type”，而如果不带`override`，编译器最多给个warning。