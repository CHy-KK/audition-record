# 1 设计模式
[C++各类设计模式及实现详解 - linux的文章 - 知乎](https://zhuanlan.zhihu.com/p/431714886)

## 工厂模式系列

工厂模式属于创建型模式，其基础思路在于，将获取产品对象指针由一个工厂类来返回，而不是显示的调用产品类的构造函数。

* 简单工厂：有一个产品基类，现要生产A B C三个派生类产品，那么我们就需要一个单独的工厂类来生产这些产品，用户通过调用工厂类方法，传入需要的产品的标识来决定返回的是哪个产品的对象。每个产品的生产都在一个函数内实现。

* 工厂方法模式：简单工厂的缺点在于违反了开闭原则，即可以扩展代码，但是不能修改原有的代码。即如果要增加新的产品，需要对这个单独的工厂类的生产函数进行修改。而改进思路也很简单，我对每一个产品单独用一个工厂基类的派生工厂类即可。这样在增加新产品时我不会改动原来的类的代码，而是直接增加新类即可。但这样也有个问题在于，增加的产品很多之后会有过多的类。

* 抽象工厂模式：在产品A B C的基础上，新增A+ B+ C+ 三种产品，由一个新的产品基类派生而出，但产品X和产品X+由于本质上是同一系列的产品，所以他们还是由同一家工厂生产。因此不需要新建工厂而是在原有工厂基础上增加一个返回X+产品对象指针的接口即可。

## 策略模式

我对策略模式的理解就是，其实就是一个标准的纯虚函数类（或者叫纯虚类即抽象类也有可能）和派生类的模式，对于基类只给出接口不给出具体实现，把具体实现延迟到派生子类中。也是面向对象中"接口interface"的思想。可以看一下软光栅项目里那个shader类的实现。其实工厂模式里面的产品类和工厂类都是策略模式。

## 单例模式

对一个类维持在进程中只有唯一的一个实例对象。具体实现思路为，将构造函数置为private，同时保存private的static实例instance；只public获取实例的方法getInstance()，在getInstance方法中，如果instance还是未被创建（NULL）那么调用构造函数，然后返回实例，否则直接返回实例。
```c++
  //Singleton.h
  class Singleton    
  {  
  private:  
      Singleton() {}  
      static Singleton *singleton;  
  public:  
      static Singleton* GetInstance();  
  };  

  //Singleton.cpp  
  Singleton* Singleton::singleton = NULL;  
  Singleton* Singleton::GetInstance()  
  {  
      if(singleton == NULL)  
          singleton = new Singleton();  
      return singleton;  
  }
```

## 适配器模式

联想STL的deque、queue、stack
* queue用到了deque的push_back()但是封装为push()，用了deque的pop_front()但是封装为pop()
* stack用到了deque的push_back()但是封装为push()，用了deque的pop_back()但是封装为pop()
但是queue和stack并不继承自deque，而是继承自一个拥有push和pop方法的抽象类Sequence()，同时queue和stack内部保存一个private的deque数据结构。deque就是该模式中的那个适配器
```c++
  class Deque  
  {  
  public:  
      void push_back(int x) { cout<<"Deque push_back"<<endl; }  
      void push_front(int x) { cout<<"Deque push_front"<<endl; }  
      void pop_back() { cout<<"Deque pop_back"<<endl; }  
      void pop_front() { cout<<"Deque pop_front"<<endl; }  
  };  
  //顺序容器  
  class Sequence  
  {  
  public:  
      virtual void push(int x) = 0;  
      virtual void pop() = 0;  
  };  
  //栈  
  class Stack: public Sequence  
  {  
  public:  
      void push(int x) { deque.push_back(x); }  
      void pop() { deque.pop_back(); }  
  private:  
      Deque deque; //双端队列  
  };  
  //队列  
  class Queue: public Sequence  
  {  
  public:  
      void push(int x) { deque.push_back(x); }  
      void pop() { deque.pop_front(); }  
  private:  
      Deque deque; //双端队列  
  };
```

# 2 智能指针

## unique_ptr

[unique_ptr 的底层实现是什么样的？ - 培风而图南的回答 - 知乎](https://www.zhihu.com/question/519457964/answer/2446096630)

unique_ptr包含两个类型参数，第一个是原生指针类型，一个是析构器
```c++
  template <typename _Tp, typename _Dp = default_delete<_Tp>>
  class unique_ptr
  {
      // 使用__uniq_ptr_impl管理要管理的heap对象
      // _Tp为管理对象类型，_Dp为析构器
      __uniq_ptr_impl<_Tp, _Dp> _M_t;
  public:
      using pointer    = typename __uniq_ptr_impl<_Tp, _Dp>::pointer;
      using element_type  = _Tp;
      using deleter_type  = _Dp;
  };
```

*  __unique_ptr如何保证唯一性？__

    使用了c++11的delete特性，将拷贝构造和赋值函数都置为delete，只保留移动构造函数和接收右值引用为传入参数的赋值函数，在移动构造中会将源unique_ptr对象release掉。
  
    ```c++
    unique_ptr(const unique_ptr&) = delete;
    unique_ptr& operator=(const unique_ptr&) = delete;

    // 传入右值引用
    unique_ptr& operator=(unique_ptr&& __u) noexcept {
      ...
    }
    ```

* __如何使用__

    初始化传入的原生指针或unique_ptr必须是一个临时变量或者通过move转化为右值，不能传入一个左值。赋值时也必须通过move转化为右值。


# 3 callable对象

## lambda

lambda函数即匿名函数的别称，是一种在代码的逻辑位置编写函数的简单方法，与传统的函数相比，他可以在代码的逻辑位置上定义，如向各类容器传入lambda表达式作为参数，并且不会污染任何命名空间。
具体语法这里不做详述，请[参考链接](https://www.cnblogs.com/chaichengxun/p/15529016.html)

## std::function

[函数对象包装器之std::function](https://blog.csdn.net/yizhiniu_xuyw/article/details/115187141)

C++11提出了std::function和std::bind，并统一了可调用对象的概念，可调用对象有以下几个定义：
* 是一个函数指针，参考C++函数指针和函数类型
* 是一个具有operator()成员函数的类的对象
* 可被转换成函数指针的类对象
* 一个类成员函数指针
举个例子，函数可以有以下几种写法
```c++
// 普通函数
int add(int a, int b) { return a + b; }

// lambda表达式
auto mod = [](int a, int b) { return a % b; }

// 函数对象
struct divide {
  int operator() (int denominator, int divisor) { return denominator / divisor; }
}
```

上述三种可调用对象虽然类型不一样，但是共享了同一种调用形式：
```c++
int(int, int)
```
std::function就可以将上述类型保存起来，如下：
```c++
std::function<int(int, int)> a = add;
std::function<int(int, int)> a = mod;
std::function<int(int, int)> a = divide;
```

总结来说，std::function是一个模板化对象，可以存储和调用任何可调用类型，例如函数、函数指针、对象、lambda和std::bind的结果。


### __std::function与lambda区别__

* lambda函数可以捕获定义处的变量，而std::function只能封装已有的函数对象
* lambda通常用于简化代码，std::function则用于提供通用的函数封装和调用方式
* lambda只能使用auto关键字推导返回值类型，而std::function必须定义时指定返回类型。


# 4 多线程

## std::thread

## std::future

std::future通常由某个provider创建，provider即一个异步任务的提供者，provider会在某个线程中设置共享状态的值，与该共享状态相关联的std::future对象调用get（通常在另一个线程中）获取该值，如果该共享状态的标志不是ready，则调用std::future::get者将会被阻塞，直到provider设置了共享状态的值，其标志位变为ready。

一个有效的std::future对象通常由以下三种provider创建，并和某个共享状态相联：

* std::async函数
* std::promise::get_future，get_future是promise类成员函数
* std::packaged_task::get_future，get_future是packaged_task类成员函数

### __std::promise__

std::promise将数据和future绑定起来，以下是和future配合使用范例。输出结果是先打印thread working，然后在输入值之后打印value: xxxx。

从这个范例中我们可以看出来，在func线程中，我们传入了一个包裹int参数的future对象，并调用future::get函数，由于与之绑定的promise在主线程中还没有得到输入，promise没有调用set_value，所以future::get一直被阻塞；直到我们输入值，promise调用set_value，func线程中的future对象获知包裹的int参数已经是ready状态，所以get不再被阻塞，得到参数值并打印。
```c++
void func(std::future<int>& fut) {
    cout << "thread working" << endl;
    int x = fut.get();
    cout << "value: " << x << endl;
}

int main() {
    std::promise<int> prom;
    std::future<int> fut = prom.get_future();
    std::thread t(func, std::ref(fut));
    int res;
    cin >> res;
    prom.set_value(res);
    t.join();
    return 0;
}
```

### __std::packaged_task__

同样定义于future头文件中，包装任何可调用callable目标。其返回值或抛出的异常都能存储与std::future对象访问的共享状态中，简而言之，就是将一个普通的可调用函数对象转换为异步执行的任务。模板声明如下，其中fn是可调用目标即function name，Args是函数参数类型。

```c++
template<class R, class ...Args>
class packaged_task<fn(Args...)>;
```
packaged_task包装的是一个函数。如果我们的需求是获取线程中的某个值，可以使用promise；当需要获取该线程函数的返回值，就需要用到packaged_task，以下是配合future使用的范例。输出是延迟2s之后打印7。注意packaged_task传入thread时需要move。
```c++
int delayAdd(int a, int b) {
    Sleep(2000);
    return a + b;
}

int main() {
    std::packaged_task<int(int, int)> task(delayAdd);
    std::future<int> result = task.get_future();

    thread td(move(task), 2, 5);
    td.join();
    cout << result.get() << endl;
    return 0;
}
```

### __std::async__

简单来看async是高级packaged_task，使用起来非常方便，先放个简单示例在这里：
```c++
int delayAdd(int a, int b) {
    Sleep(2000);
    return a + b;
}

int main() {
    auto res = std::async(delayAdd, 2, 5);
    cout << res.get() << endl;
    return 0;
}
```

## Thread Pool线程池

基础思路：线程池类的基本方法和成员包括：
* 构造函数：初始化一个<font color="#dd0000)">线程列表</font>，每个线程在<font color="#dd0000)">任务队列</font>为空时阻塞，在<font color="#dd0000)">stop标志位</font>为false时阻塞。工作时会从任务队列中获取头部任务进行处理。
* enqueue函数：外部需要执行的任务通过此函数加入<font color="#dd0000)">任务队列</font>，每个任务通过packaged_task进行包装，返回task的future对象。外部通过调用future.get()来获取结果。
* 析构函数：将<font color="#dd0000)">stop标志位</font>置为true，调用condition.notify_all()通知所有线程开始工作，然后将<font color="#dd0000)">线程列表</font>中的每个线程join。
* <font color="#dd0000)">线程列表</font>：std::vector< std::thread > workers;
* <font color="#dd0000)">任务队列</font>：std::queue< std::function<void()> > tasks;
* 条件变量：用来查询<font color="#dd0000)">stop标志位</font>和tasks.empty()状态
* 互斥变量mutex
* 控制变量<font color="#dd0000)">stop标志位</font>

在程序最开始就预先分配好一个线程列表，然后在使用时将需要完成的任务作为callable对象传入线程池