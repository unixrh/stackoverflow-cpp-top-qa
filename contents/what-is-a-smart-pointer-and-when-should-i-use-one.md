## 什么是智能指针，什么时候需要使用智能指针 
### 问题

> 如题

### 回答1：

智能指针是一个封装了“原始指针”的类，它的作用是用来管理指针所指对象的生命周期。不存在单一的智能指针类型，所有智能指针的存在都是为了描述“原始指针”。
我们应该优先选择使用智能指针。当你想要使用指针时（考虑清楚是否真的需要用），你通常最好使用智能指针，因为这可以减少“原始指针”可能带来的麻烦，主要是
忘记释放对象、造成内存泄漏等。

使用智能指针时，程序员必须在不需要用到对象时显示地销毁它。
```c++
// 为了某个需求而创建对象
MyObject* ptr = new MyObject(); 
ptr->DoSomething(); // 使用对象
delete ptr; // 释放对象，完成！
// 等等！如果 DoSomething()抛出异常咋办！
```
相较而言，智能指针定义了一种策略来控制何时销毁对象。当然你还是得创建对象，但是不再需要担心何时销毁它。
```c++
SomeSmartPtr<MyObject> ptr(new MyObject());
ptr->DoSomething(); // Use the object in some way.
// 对象的销毁依赖于智能指针的策略。
// 即便  DoSomething()抛出异常，依然可以正常释放对象
```
使用的最简单的策略与智能指针相关对象的作用域有关，比如`boost::scoped_ptr` 或者 `std::unique_ptr` 的实现
```c++
void f()
{
    {
       boost::scoped_ptr<MyObject> ptr(new MyObject());
       ptr->DoSomethingUseful();
    } // boost::scopted_ptr 离开作用域后，MyObject对象会自动被销毁

    // ptr->Oops(); // 编译错误：`ptr`未定义已经在其作用于之外了
}
```
注意 `scoped_ptr` 实例不能被拷贝，这是为了避免指针被错误地多次释放。但是你可以通过引用将它传递给你调用的函数。
如果你想将你的对象的生命周期限定在一段特定的代码块中，或者你将它作为一个成员数据嵌入其它对象中时，范围指针就非常有用。这个对象将会在退出代码块时，或者父对象销毁时自动销毁。
一种更复杂的策略与指针的引用计数有关。这样允许指针被拷贝。当对象的最后一个“引用”被销毁时，这个对象也同步销毁。这种策略有以下实现：`boost::shared_ptr` 和 `std::shared_ptr`.
```c++
void f()
{
    typedef std::shared_ptr<MyObject> MyObjectPtr; // 简写一下
    MyObjectPtr p1; // 空值

    {
        MyObjectPtr p2(new MyObject());
        // 对象当前有一个引用。
        p1 = p2; // 拷贝指针
        // 对象当前有两个引用
    } // p2 销毁，对象仅剩一个引用
} // p1 销毁，对象剩余0个引用 
  // 对象被销毁
```
        