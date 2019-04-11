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
引用计数的指针，在对象的生命周期更复杂的时候非常有用，并且它没有直接地绑定在特定的代码段或者另一个对象。
引用计数指针有个弊端 - 可能产生悬挂引用：
```c++
// 在堆中创建智能指针
MyObjectPtr* pp = new MyObjectPtr(new MyObject())
// 由于我们没有释放智能指针，这个对象也就不会被销毁。
```
Another possibility is creating circular references:
```c++
struct Owner {
   boost::shared_ptr<Owner> other;
};
boost::shared_ptr<Owner> p1 (new Owner());
boost::shared_ptr<Owner> p2 (new Owner());
p1->other = p2; // p1 references p2
p2->other = p1; // p2 references p1

// p1 和 p2 的引用计数永远不会为0，两者对象也永远不会被销毁
```
为了解决这个问题，Boost 和 C++11 都定义了 `weak_ptr` 来定义弱(不计)引用数的 `shared_ptr`

----
#### 更新
以上回答有些老了，描述了那个时候好的东西，也就是由Boost库提供的智能指针。 从 C++11 开始，标准库提供了足够的智能指针类型，因此你应该尽量使用 `std::unique_str`, `std::shared_ptr` 和 `std::weak_ptr`。
同时还有 `std::auto_ptr`，它和范围指针非常像，但它有可被复制的危险特性--这也会意外地转换所有权！在最新的标准里，它已经被废弃了，因此不应该使用它，还是使用 `std::unique_ptr` 吧。
```c++
std::auto_ptr<MyObject> p1 (new MyObject());
std::auto_ptr<MyObject> p2 = p1; // 对象拷贝，所有权转换
p2->DoSomething(); // 正常 
p1->DoSomething(); // 很可能报空指针异常
```

### 回答2
对于现代C++，这里有个简单的答案。
- 什么是智能指针？
一种可以被像指针那样使用的类型，但它同时提供自动管理内存的额外特征：当一个智能指针不再被使用时，指针指向的内存就会被释放（更多细节可以看维基百科）。
- 什么时候应该用智能指针？
当需要考虑一块内存空间的所有权、分配或者释放时，智能指针可以让你不需要显式地做这些操作。
- 在
Here's a simple answer for these days of modern C++:

What is a smart pointer? 
It's a type whose values can be used like pointers, but which provides the additional feature of automatic memory management: When a smart pointer is no longer in use, the memory it points to is deallocated (see also the more detailed definition on Wikipedia).
When should I use one? 
In code which involves tracking the ownership of a piece of memory, allocating or de-allocating; the smart pointer often saves you the need to do these things explicitly.
But which smart pointer should I use in which of those cases?
Use std::unique_ptr when you don't intend to hold multiple references to the same object. For example, use it for a pointer to memory which gets allocated on entering some scope and de-allocated on exiting the scope.
Use std::shared_ptr when you do want to refer to your object from multiple places - and do not want your object to be de-allocated until all these references are themselves gone.
Use std::weak_ptr when you do want to refer to your object from multiple places - for those references for which it's ok to ignore and deallocate (so they'll just note the object is gone when you try to dereference).
Don't use the boost:: smart pointers or std::auto_ptr except in special cases which you can read up on if you must.
Hey, I didn't ask which one to use! 
Ah, but you really wanted to, admit it.
So when should I use regular pointers then? 
Mostly in code that is oblivious to memory ownership. This would typically be in functions which get a pointer from someplace else and do not allocatemor de-allocate, and do not store a copy of the pointer which outlasts their execution.
