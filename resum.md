# 1. C++基础

###  1.智能指针

智能指针为了解决什么问题：

智能指针是自动管理内存，不需要手动释放内存，随对象释放时自动释放，要和深拷贝浅拷贝结合。

什么是浅拷贝：

```c
浅拷贝：拷贝对象的每个字段值，包括指针的地址，不复制指针指向的数据。    两个对象指向同一片内存

```

什么是深拷贝

```c
深拷贝：拷贝值对象，指针对象开辟新的地址存放值，内个对象拥有自己的副本
```

![image-20250714110658850](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714110658850.png)

问题  ：

浅拷贝当一个对象析构时，会造成野指针，二次释放程序会崩溃



C++中注意事项

如果类中包含指针成员，或资源你必须（三法则）

1.实现自定义的拷贝构造函数（深拷贝）

2.自定义赋值运算符

3.实现析构函数

```c++
#三法则：
class MYclass{
    public:
    int *data;
    Myclass(int val) //默认构造
    {
        data=new int(val);
    }
    //深拷贝构造
    Myclass(const Myclass&other)
    {
        data=new int(other->data);
    }
    //重载赋值运算符
    Myclass & operator=(const Myclass &other)
    {
        if(this!=other) //不能删除自己
        {
            delete data;
            date =new (int other->data)
        }
        return *this
    }
    //析构函数
    ~Myclass()  
    {
        delete data;//手动是地方
    }
}
```

 我能不能自定义类中对象的时候用智能指针解决这个问题，不进行三法则，还是必须进行三法则，但现代 C++ 中，**智能指针（如 `std::unique_ptr`、`std::shared_ptr`）可以替代手动资源管理，从而避免显式实现三法则**，但需根据场景选择合适方案：

### **1. 使用智能指针（推荐现代 C++ 方式）**

**适用场景**：类的唯一目的是管理动态资源的生命周期。
**优点**：

- **自动释放资源**：智能指针会在析构时自动释放内存，无需手动实现析构函数。
- **禁用或默认拷贝语义**：
  - `std::unique_ptr`：默认禁用拷贝（移动语义替代），直接避免浅拷贝问题。
  - `std::shared_ptr`：默认支持深拷贝（引用计数），无需手动实现。

### 1.智能指针

种类

```c++
std::unique_ptr  //独占指针 移动语义 禁止拷贝
std::shared_ptr  //计数指针  默认实现深拷贝 计数值++
std:: weak_ptr //弱指针  
    
//使用时必须加memory头文件
#include <memory>
    //栈上资源自动释放
    //堆上资源手动释放
    
    
```

#### 1.智能指针概述

自动分配内存

![image-20250714113636153](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714113636153.png)

![image-20250714113729628](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714113729628.png)



#### 2. unique_ptr（）



不希望多个类共享该资源

✅ 总结一句话：

> **`unique_ptr` 用于资源的独占管理，自动释放内存，防止泄漏和误用，特别适合对象生命周期清晰、不需要共享的场景。**

![image-20250714113945480](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714113945480.png)

三种创建方式
通过已有裸指针创建
通过new来创建
**通过std::make_unique创建**
**(推荐）**
unique_ptr可以通过get(获取地址）
unique_ptr实现了->与‘
可以通过->调用成员函数
可以通过*调用dereferencing

![image-20250714114134196](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714114134196.png)



![image-20250714115251523](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714115251523.png)

局部作用域{}

```c++
//方式一：  原始指针创建独占指针
cat *cat1=new cat("mimi");
unique_ptr<cat> cat2{cat1};//已经共享一块指针地址了
delete cat1; //释放后野指针  直接置空不需要再释放
cat1=nullptr;
```

通过原始指针赋值后要销毁原始指针，因为他会跟原始指针改变，达不到独占的效果。

![image-20250714115835941](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714115835941.png)

```
❗ 为什么不应该这样写？
1. unique_ptr 是独占所有权的智能指针
当你写：

cpp
Copy code
std::unique_ptr<cat> cat2{cat1};
实际上是把一个 已经由 new 创建并且仍被 cat1 管理 的对象，交给了 unique_ptr 来托管。这会有两个问题：

❌ 错误一：cat1 仍然持有裸指针，和 cat2 共享同一块堆内存
cat2 现在拥有这块内存的释放权。

cat1 仍然可以访问它，但不应再负责释放，否则就是重复释放。

❌ 错误二：你后面还手动 delete cat1
cpp
Copy code
delete cat1;
这相当于手动释放了一块已经交给 unique_ptr 管理的堆内存，等 cat2 被销毁时，它会再次 delete，这将导致二次释放（double free）错误！
```

```c++
//方式二：  显示写
unique_ptr<cat> cat2{new cat("cat")}
```

```c++
//方式三  make_ptr  推荐
unique_ptr<car> cat=make_unique<cat>("mini")
```

![image-20250714121242323](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714121242323.png)

自动释放  不需要手动delet

```c++
//get()  函数 打印地址

```



![image-20250714121521555](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714121521555.png)

![image-20250714121556965](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714121556965.png)



#### 3.unique_ptr与函数

因为我是独占指针  不可能有copy的副本

1.做为参数传递

​	传value  move（）//作为i右值  不可以做为左值copy，

​	作为引用

```c++
void do_trans_val(unique_ptr<cat> b)
{
	b.reset() //情空指针所有地方都不能访问
}

void do_trans_cue(const unique_ptr<cat> &c)//引用不涉及所有权转移
{
//const  修饰的是独占指针  任然可以修改值
    c->set()
        //不可以调用 c.reset() 情空指针
}

unique_ptr retern_()
{
    unique_ptr<cat>  c=make_unique<cat>("c");
    return c;
}
int main()
{
	unique_ptr<cat>  a=make_unique<cat>("a");
    //1.值传递
    do_trans_val(a); // 错这里既然作为左值传参  报错会拷贝
    do_trans_val(std::move(a))//只能通过move的格式将独占指针移动给另一个对象  以后就不能再访问a了因为已经被移动
     //2.引用  修改值
    do_trans_cue(a);//作为引用  并不会设及拷贝，  后面还可以访问
    //3.作为返回值  链式引用
    retern_->info();
    
    a.get()//返回的是指向内存的地址
    &a 返回的是独占指针的地址。
}
```





#### 4. 计数指针 shared_ptr

共享指针，计数值阵（可以copy）

赋值的是指针  ，多个对象指向同一块地址 （那不是浅拷贝吗  但是释放后只是计数值.get_count--，并不会有野指针出现 ）  

🟩 即使多个 `shared_ptr` 指向同一块内存，**只有最后一个会 delete 它**。

✅ 3. 内存释放是引用计数管理的（非浅拷贝）

```c++
std::shared_ptr<Cat> p1 = std::make_shared<Cat>("mimi");  // count=1
std::shared_ptr<Cat> p2 = p1;                             // count=2
p1.reset();                                               // count=1
p2.reset();                                               // count=0 → 释放内存

```

每次 `.reset()` 或作用域结束，引用计数 `--`，当计数为 0 时才 delete 内存。

//如果改变a  b也会变  改变b  a也会变

![image-20250714145535816](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714145535816.png)

![image-20250714145607096](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714145607096.png)



```c++
shared_ptr<int> a=make_shared<int>(10);
a.use_count();//获取计数值  1


shared_ptr<int> b=a; //cpoy
a.use_count();//获取计数值  2
b.use_count();//获取计数值  2 
//如果改变a  b也会变  改变b  a也会变

```



![image-20250714150654221](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714150654221.png)





自定义类型

```c++
//数据永远只有一套   只有最后的计数值为0才会销毁

shared_ptr<cat> cat=make_shared<cat>("cat");
cat.get_count();
shared_ptr<cat> cat2=cat;
shared_ptr<cat> cat3=cat;

cat.reset();
cout<<cat.get_count();



```





#### 5. shared_ptr与函数

![image-20250714151140267](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714151140267.png)

1。参数

​	可以直接传值 valeu  （出了函数  就--   局部作用域结束   但是在函数里改变值肯定也会变 共享内存）

​	ref

2.函数返回值



- `at_p` 是一个局部 `shared_ptr`，引用计数 = 1。
- `return` 的时候，**cat_p 所管理的资源的引用计数会被“转交”给外部**，引用计数仍然保持在 1（未变为 2）。
- 由于是返回值优化（RVO）或 move 语义，所以**没有增加引用计数**。

```c++
std::shared_ptr<Cat> get_shared_ptr()
{
    std::shared_ptr<Cat> cat_p = std::make_shared<Cat>("local cat");
    return cat_p;
}



```

```c++
std::shared_ptr<Cat> c_p = get_shared_ptr();  // 第一次 get_shared_ptr()
c_p->cat_info();  // 调用 cat_info()

get_shared_ptr()->cat_info();  // 第二次 get_shared_ptr()
cout << "------ yz ------" << endl;

```

因为你调用了 **`get_shared_ptr()` 两次**，每次都构造了一个新的 `Cat("local cat")` 对象。

每个对象都有自己的生命周期，**分别构造、分别销毁**。

#### 6. shared_ptr和unique_ptr的转化

将unique_ptr  转化share_ptr

![image-20250714152753571](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714152753571.png)

 ![image-20250714152919576](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714152919576.png)



![image-20250714153054603](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714153054603.png)

![image-20250714153228509](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714153228509.png)

#### 7. weak_ptr是shared_ptr的补充

循环以来问题

弱引用

![image-20250714153320299](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714153320299.png)



![image-20250714153419804](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714153419804.png)



![image-20250714153452818](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714153452818.png)

我不理解   没懂为什么会锁死

ChatGPT said:

太好了，你问得非常关键，我们这就用最通俗的语言和一步步代码来看清楚 —— **为什么 `shared_ptr` 的循环引用会导致“锁死”（内存永远不释放）**。

🧠 **先理解 `shared_ptr` 的核心机制：引用计数**

```
cppCopyEditstd::shared_ptr<Cat> p1 = std::make_shared<Cat>("mimi");
std::shared_ptr<Cat> p2 = p1;  // 引用计数 +1
```

现在这块 `Cat` 对象的引用计数是 2，只有当计数减为 0 时，`Cat` 才会被析构和释放



```
struct B;  // 前向声明

struct A {
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

struct B {
    std::shared_ptr<A> a_ptr;
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    std::shared_ptr<A> a = std::make_shared<A>();  // A引用计数 = 1
    std::shared_ptr<B> b = std::make_shared<B>();  // B引用计数 = 1

    a->b_ptr = b;  // B引用计数 = 2（a 拥有 b）
    b->a_ptr = a;  // A引用计数 = 2（b 拥有 a）

    // main() 结束，a 和 b 离开作用域，计数 -1
    // 但是 a->b_ptr 和 b->a_ptr 仍存在，引用计数各为 1！
}

```

🔄 内部发生了什么？

| 对象 | 被谁持有               | 引用计数 |
| ---- | ---------------------- | -------- |
| `a`  | `main()` 和 `b->a_ptr` | 2        |
| `b`  | `main()` 和 `a->b_ptr` | 2        |



- `main()` 结束后，`a` 和 `b` 的引用计数各减 1
- 但 `a->b_ptr` 和 `b->a_ptr` 还活着 → 计数仍然是 1

💥 **所以：`A` 和 `B` 对象都“锁在对方手里” → 没人能释放它们，析构函数也不会调用！**



```c
#include <iostream>
#include <memory>

struct B;  // 前向声明

struct A {
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destroyed\n"; }
};

struct B {
    std::weak_ptr<A> a_ptr;  // ❗关键：不参与引用计数
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    std::shared_ptr<A> a = std::make_shared<A>();  // A: use_count = 1
    std::shared_ptr<B> b = std::make_shared<B>();  // B: use_count = 1

    a->b_ptr = b;            // B: use_count = 2（a->b_ptr 加1）
    b->a_ptr = a;            // A: use_count = 1  不变（a_ptr 是 weak）

    // 离开作用域，a 和 b 被销毁
}

```

✅ 第三步：main() 离开作用域（开始释放）

1. `main()` 中的 `shared_ptr<A> a` 被销毁，`A` 的引用计数 -1 → 0
   - 触发 `A` 的析构函数，里面的 `b_ptr` 被销毁
2. `b_ptr` 是 `shared_ptr<B>`，它是 `B` 的第 2 个引用，现在也被销毁
   - `B` 的引用计数 -1 → 从 2 变成 1
3. `main()` 中的 `shared_ptr<B> b` 离开作用域
   - `B` 的引用计数再 -1 → 变成 0

✅ 此时：**引用计数 = 0 → 自动析构 B**





```c++
shared_ptr<cat> s=make_shared<cat>("cc")
weak_ptr<c> w(s);

s.get_count();//1
w.get_count();//1  不会加1 不可以用->  * 
shared_ptr s2=w.lock();
s.get_count();//2
w.get_count();//2  

```

![image-20250714160018551](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714160018551.png)



![image-20250714160315252](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714160315252.png)

最下面的改成weak_ptr就好了   weak_ptr=shared_ptr是可以的



![image-20250714160333234](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714160333234.png)

##### 1. waker_ptr解决了什么问题

循环引用问题，

两个或多个对象中互相拥有对方的shared_ptr时，会出现计数值不为0， 内存泄漏，对象没有被释放

![image-20250714210304156](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714210304156.png)



可以使用裸指针解决，但是时刻要关注是否对象被释放

![image-20250714210439796](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714210439796.png)

![image-20250714210452124](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714210452124.png)

##### 2. weaker_ptr的底层原理

计数值并不会—++；没有* 和->

![image-20250714210547058](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714210547058.png)

共享控制块和引用计数机制：

1.   Muse_count 记录shared_ptr的引用计数，管理对象生命周期
2. M_weak_count记录weak_ptr的引用计数+Muse_count!=0?1:0,管理生命周期
3. 创建weak_ptr时，指向与之关联的shared_ptr相同的控制块并源自递增M_weak_count；
4. 析构时源自递减
5. weak_ptr不操作M_use_count，故不影响对象生命周期

![image-20250714211235579](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714211235579.png)

 







##### 3. weak_ptr和shared_ptr的区别

shared_ptr  多个对象共享一个资源，维护一个计数值,get_counter，最后一个对象销毁时调用一次delet 具有*，-》功能；



weak_ptr可以转化为shared_ptr(强转或者  .lock())

![image-20250714211439052](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714211439052.png)

##### 4.如何使用weak_ptr

两个类中有一个使用weak_ptr  另一个shared_ptr;   

lock





### 2. llamaba 表达式  function   lambda bind之间的关系

![image-20250714175250289](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714175250289.png)



function 类模板

lambda 表达式

bind  函数适配器

#### 1.function  类模板  C++11 

void (*fun)(void)  函数指针

有很多不同的函数对象

仿函数  llambda  bind  产生的函数对象  用一个类型约束  std::function是用来描述函数对象的

​	function是一个抽象了函数参数和函数返回值的类模板；

- 抽象：**把任意函数包装成一个对象，该对象可以保存 传递 复制**

​	   动态绑定，只需修改该对象（赋值不同的function对象），实现类似多态的效果

##### 1. 用途：保存普通函数、类静态成员函数static

✅ 类的静态成员函数和类的普通成员函数的区别

🧠 总体区别一句话总结：

> **普通成员函数** 依赖对象调用，能访问类的实例变量；
>  **静态成员函数** 属于类本身，不依赖对象，也**不能访问实例变量**。

```c++
class MyClass {
public:
    int value;                    // 实例变量
    static int count;            // 静态变量

    void normalFunc() {
        std::cout << "value = " << value << std::endl;    // ✅可以访问实例成员
        std::cout << "count = " << count << std::endl;    // ✅也可以访问静态成员
    }

    static void staticFunc() {
        // std::cout << "value = " << value << std::endl; // ❌错误，不能访问实例变量
        std::cout << "count = " << count << std::endl;    // ✅可以访问静态成员
    }
};

int MyClass::count = 0;

int main() {
    MyClass obj;
    obj.value = 42;
    MyClass::count = 10;

    obj.normalFunc();      // ✅ 需要对象调用
    MyClass::staticFunc(); // ✅ 不需要对象，可以用类名调用
}

```



> ![image-20250714171007080](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714171007080.png)

​	![image-20250714171023262](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714171023262.png)

![image-20250714171040036](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714171040036.png)

```c++
function<void(int)> f_hello=hello;
f_hello(1);
function<void(int)> f_hello_addr=&hello;
f_hello_addr(2);
function<void(int)>  f3=&StaticFunc::hello;



```



![image-20250714171455548](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714171455548.png)

##### 2. 保存仿函数 闭包（可以有状态的函数  比如操作i 幂不等的操作）

仿函数：重载（）是一个重载了（）的类

特征： 可以有状态，通过成员变量存储状态

​	    有状态的函数对象成为闭包

![image-20250714175212773](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714175212773.png)

![image-20250714171418938](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714171418938.png)

```c
function(void<int>) f4=Hello();//仿函数  函数对象
f4(4);  //4
f4(4);  //8
function(void<int,int>) f5=Hello();//构造函数 生成仿函数对象  根据参数找到重载符



```

##### 3.普通类成员函数

成员函数的封装需要创建一个该类的实例  所以需要类指针和参数类型   ，这一点和静态成员函数区别；

```c
function<void(CHello*,int)>    h6=&CHELLO::hello;
Chello c;
h6(&c,7);
```

![image-20250714173242389](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714173242389.png)





##### 4.保存llambda表达式

是什么：语法塘->方便创建匿名函数对象的**语法塘**

构成： [捕获列表 外部变量]（参数列表）-> void 返回值{  函数体}

​		[]  捕获列表 ：  指定函数内部可以访问的外部变量  

​			本质是：外部变量将转变为类的成员变量

​		（）参数列表：

​		->  返回值  ：可以省略  因为有类型推导

​		{} 函数体：

原理：编译的时候将lambda转化为函数对象

​	  根据参数列表重载operator()  如下  (下面的&  得去掉  是值捕获)

![image-20250714174854836](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714174854836.png)

编译器编译的时候会展开如下：

![image-20250714174821233](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714174821233.png)

![image-20250714174940525](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714174940525.png)

 



✅值补或  不可以修改外部变量  要修改的话需要加mutable     int i=0;    [i] (int count ) mutable->void{ i++;  //函数调用之后并不会修改外部的i的值   只是内部维护一个值  闭包}

```c++
int i=0;
auto h7=[i](int  count){
i++;
    cout<<i<<count;    //闭包
}
h7(8); //内部i=1  外部i=0;
h7(8)//内部i=2   外部i=0;
```





✅  引用捕获    //想要修改外部变量   可读可写

[&i] () {}

{ //外部的值也会修改}

![image-20250714175150634](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714175150634.png)



##### 5. 保存bind返回的函数对象

![image-20250714181600582](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714181600582.png)

bind函数适配器：用来通过**绑定函数以及函数参数**的方式生成函数对象；生成				闭包的

​			     提供占位符；实现灵活的参数绑定；



```c++
//绑定普通函数
auto f9=bind(&hello,9)// 将参数和函数绑定好
f9();

//绑定成员函数  有实例
auto f10=bind(&Chello::hello,&c,9)；
f10();

//占位符
auto f10=bind(&Chello::hello,&c,placeholders::_1)；
f10(1000);
```

编译时展开

绑定普通函数

![image-20250714180648925](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714180648925.png)

绑定类成员函数



![image-20250714181341806](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714181341806.png)



##### 总结



### 3. 自动类型推导auto 、decltype

![image-20250714202754495](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714202754495.png)



#### 1. auto（就是个赋值的运算）

![image-20250714201017193](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714201017193.png)

 推到变量类型

原理：c++强类型语言  ，编译器根据强制给变量初始化值进行类型推导

规则：

 	1. auto a=10 必须初始化
 	2. 没有cv语义  const  volital ;   const int &a=10;  auto b=a
 	3. 保留引用或者cv语义可以用auto &
 	4.    不可以推到数组类型  回推导出指针  int a[]={}; auto b=a;
 	5. 定义多个变量必须为同一类型
 	6. 万能引用  auto && 根据初始值的属性判断是左值引用还是右值引用

应用：

匿名函数返回值

模板函数

容器时

![image-20250714202224048](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714202224048.png)



#### 2. decltype

用于推到表达式类型，编译器之分析表达式类型而不参与运算

exp:

​	是表达式 推到表达式类型

​	函数返回值类型

​	左值-》左值引用

![image-20250714202244653](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714202244653.png)

![image-20250714202340206](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714202340206.png)

![image-20250714202438135](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714202438135.png)

返回左值的引用：

![image-20250714202609299](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714202609299.png)

![image-20250714202653587](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714202653587.png)





### 4. final、overwrte（virtual） 关键字

为什么引入：

​	（1） 虚函数复写时产生的问题：

​		1. 不能阻止某一个虚函数进一步被重写，

​		2. 本意想写一个新的函数，错误重写基类虚函数（子类virtual关键字可以省略）

​		3. 由于签名不一致，造成了在子类重新构建了虚函数（参数个数  类型...  ）本意是重写

​	（2） 类继承时产生的问题：

1. 不能阻止某个类进一步派生

   





#### 1. final

指定某个虚函数不能再派生类中被覆盖、某给类不能派生

![image-20250714204512547](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714204512547.png)



![image-20250714204601597](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714204601597.png)



![image-20250714204704289](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714204704289.png)









#### 2.overwrite

指定子类的一个虚函数复写基类的虚函数

保证重写函数和基类函数具有相同的签名

![image-20250714205027954](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714205027954.png)



![image-20250714204258789](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714204258789.png)





#### 总结

![image-20250714204803328](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714204803328.png)



### 5.左值引用  右值引用

左值引用与右值引用的区别？  右值引用的意义

#### 1. 左值引用和右值引用定义

左值：具有名，可寻址，可以在等号左边

​           举例：变量名

​			返回左值引用的函数调用

​			前置自增，前置自减 

​			赋值运算，复合赋值运算i+=10

​			解引用表达式 *a=

​			![image-20250714213533461](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714213533461.png)



右值：只能放在等号右边，不具名  不能取地址

​	纯右值：

​		字面值

​		返回非引用对象

​		后置自增，自减

​		算术表达式

​		逻辑表达式

​	将亡值：

​		c++11新引入的与右值引用相关的值类型

​		用来触发移动构造或者移动赋值构造，进行资源转移，之后将亡值调用析构

​		通过std::move\

​		![image-20250714214044660](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714214044660.png)

![image-20250714214323724](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714214323724.png)![image-20250714214354075](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714214354075.png)











#### 2. 左值引用与右值引用的区别



![image-20250714214605096](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714214605096.png)

功能差异：左值引用避免对象的拷贝（传参  函数返回值）

​		  右值引用（完美转发  移动语义）



const左值的引用能指向右值，局限是不能修改 （右值不能修改）要解决这个问题 引入右值引用

右值引用通过std::move（）转化为左值

![image-20250714214916288](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714214916288.png)

![image-20250714214932422](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714214932422.png)

移动语义解决深拷贝问题  （ 移动后源资源为null）

![image-20250714215206101](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714215206101.png)

![image-20250714215328376](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714215328376.png)



![image-20250714215723790](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714215723790.png)



移动语义  避免多次拷贝资源  

#### 3.实现完美转发

![image-20250714215857833](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714215857833.png)

函数模板可以将自己的参数完美的转发给内部调用的其他函数

完美是指：

![image-20250714220045661](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714220045661.png)



![image-20250714220604903](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714220604903.png)

![image-20250714220652324](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250714220652324.png)





### 6.菱形继承

什么是菱形继承   怎么解决菱形继承 

C++独有（多重继承特性）

​	定义：一个子类可以继承多个父类，多个父类可以继承相同的父类

​	问题：A会被构造两次，浪费存储空间 造成二义性

![image-20250715093747848](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715093747848.png)

![image-20250715093907336](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715093907336.png)

有两个number,说明构造了两个A

怎么解决菱形继承：

虚继承：子类只继承一次父类的父类  virtual 关键字

![image-20250715094046601](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715094046601.png)

![image-20250715094123220](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715094123220.png)

虚继承底层实现原理：  g++ -fdump-class-hierarchy   *.cpp   gcc8.0前

​					g++ -fdump-lang-class   *.cpp    gcc8.0后

![image-20250715094434765](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715094434765.png)



原理：  利用虚表偏移实现虚继承	父类的vptr都有到共同基类的偏移量从而让子类多继承时指向同一个父类的父类。（VTable  可以使B1 B2都指向同一个A）

虚继承会为A仍然生成一个虚函数表，B1和B2都会指向同一个虚函数表，类A和A的虚表一一绑定。

![image-20250715095249783](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715095249783.png)



![image-20250715095310382](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715095310382.png)

![image-20250715095347738](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715095347738.png)



![image-20250715095804997](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715095804997.png)



### 7. 面向对象三大特性

封装  继承  多态

#### 1.封装

隐藏实现细节   实现模块化功能（提供统一的接口  外部访问内部资源）

特性：访问权限

​	public:对所有对象开放

​	protected:只对子类对象开放

​	private:只对自己开放  通过友元类打破限定 friend class B

![image-20250715100312960](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715100312960.png)

属性：



方法：

#### 2.继承

定义：无需要修改原有类，通过继承实现功能扩展

特性：基类成员在子类中的最高权限

​	public:原  public  \protected不变

​	protected:原来public变成protected ,原来的protected不变

​	private:原  public  \protected变成private.

![image-20250715100926716](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715100926716.png)



using修改权限:基类成员在子类中的权限

public:

​	using A::C;

![image-20250715101137337](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715101137337.png)



多继承

接口继承

#### 3. 多态

定义：一个接口多种形态：

​	静态多态：函数重载 函数名相同（参数不同） 编译期确定

​	动态多态 ：  虚函数重写实现，  基类的指针=子类的对象   Anm * a= cat;

### 8. 简述多态实现原理

#### 1. 静态多态：

函数重载：编译期确定的，允许在同意作用域中声明多个功能类似的同名函数，这些函数参数列表不同（参数个数，类型，顺序）

注意：没有返回值。

原理：

函数名修饰：

编译过程： 

1. 预编译 ：把头文件中的函数声明拷贝到源文件中，避免编译过程的语法分析找不到函数定义。

2. 编译：语法分析，同时符号汇总生成符号表（函数名）

3. 汇编： 生成函数名-》函数地址的映射方便通过函数名找到函数位置，从而执行。

4. 连接：将多个文件中的符号表汇总合并，	

![image-20250715102523619](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715102523619.png)



意思就是说在编译阶段为不同的函数重载生成不同的符号区分。在汇编阶段映射函数名和具体的函数地址。



1A  3sum   E  dd

类字数   函数名字数   参数列表首字母

 

objdump -t *.o   查看符号表            ——ZN+类长度—+类名+函数长度函数名+E+参数列表类型的首字母

![image-20250715103314776](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715103314776.png)



#### 2. 动态多态： 虚函数重写

运行时确定

在基类函数前加上virtual关键字，在派生类重写该函数

运行时将会根据对象的类型来调用相应的函数

如果类型是基类，调用基类的函数

如果对象类型是派生类调用派生类函数



原理：

​	早绑定：编译器就确定了对象调用的函数的地址

​	晚绑定： 

​	看类使用了virtual函数会为类生成虚函数表-》一维数组存放着虚函数的地址，类对象构造时会初始化指向虚表的指针，vptr,

​	虚函数表指针vptr在构造函数中初始化

虚函数表指针：vptr

在类对象的前8个字节 是虚函数表指针

![image-20250715104340177](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715104340177.png)

![image-20250715104734596](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715104734596.png)

✅ 是的 —— **确实有两个虚函数表（vtable）**

当子类 **重写（override）** 基类的虚函数时，编译器会为子类生成一个**新的虚函数表**。这两个表分别属于：

- `Base` 的虚函数表：指向 `Base::func`
- `Drive` 的虚函数表：指向 `Drive::func`

🧠 为什么需要两个虚函数表？

因为**动态绑定**的机制是通过对象中的 **vptr（虚函数表指针）** 来实现的。

看下面的场景：

```
cppCopyEditBase* p1 = new Base();
Base* p2 = new Drive();  // 向上转型
```

此时：

- `p1` 指向的对象的 `vptr` 指向的是 `Base` 的 vtable ⇒ 第一个函数是 `Base::func`
- `p2` 指向的对象的 `vptr` 指向的是 `Drive` 的 vtable ⇒ 第一个函数是 `Drive::func`

因此调用：

```
cppCopyEditp1->func(); // 调用 Base::func
p2->func(); // 调用 Drive::func
```

即使 `p2` 是 `Base*` 类型，仍然会通过它的 `vptr` 找到 `Drive::func`，这就是**多态性**的基础。

🔬 内存示意图（简化）

```
cppCopyEditclass Base {
    virtual void func();
};

class Drive : public Base {
    void func() override;
};
```

创建对象后：

```
cppCopyEditBase对象内存布局：
┌────────┐
│ vptr ──┼──► [ &Base::func ]
└────────┘

Drive对象内存布局：
┌────────┐
│ vptr ──┼──► [ &Drive::func ]
└────────┘
```

🧩 vtable 实际长什么样？

通常它是个函数指针数组（伪代码）：

```
cppCopyEdit// Base 的 vtable
Base_vtable = {
    &Base::func
};

// Drive 的 vtable
Drive_vtable = {
    &Drive::func
};
```

✅ 总结

| 问题                                 | 回答                                                    |
| ------------------------------------ | ------------------------------------------------------- |
| 子类重写虚函数会生成新的虚函数表吗？ | ✅ 会。每个类有自己的 vtable。                           |
| 如何判断调用基类还是子类函数？       | 通过对象内的 vptr 指针，它指向当前类对应的 vtable。     |
| 为啥两个对象访问的是 `[0]`？         | 因为 `[0]` 是第一个虚函数的位置，但 vtable 本身不一样。 |

### 9.继承下的构造函数和析构函数顺序

继承下：构造函数依照以来连，从上往下构造

析构：按照依赖链从下往上析构

单继承：

1. 先构造父类Base-》子类Driver   析构子类-》父类

2. 父类Base有成员BASE1：

​	先构造BASE1->BASE->DRIVER

​	析构：driver->base->base1

3. 父类Base有成员BASE1,driver有成员DRIVER1(先有爸爸-》 先有器官  属性 -》 再有自己)

   ​	先构造BASE1->BASE->DRIVER1->DRIVER

   ​	析构：DRIVER1->driver->base->base1



多继承：

先父亲在器官在自己（顺序执行）

![image-20250715111306618](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715111306618.png)



















### 3.C++11用过哪些新特性



> [!IMPORTANT]
>
> 语法塘：
>
> ​	关键字：auto 、decltype
>
> ​		      final、overwrte
>
> ​	语法： function 函数对象
>
> ​		     bind
>
> ​		     lambda
>
> STL容器:
>
> ​	array
>
> ​	forward_list
>
> ​	unordered_map
>
> ​	unordered_set
>
> 智能指针：
>
> ​	shared_ptr/weak_ptr
>
> ​	unique_ptr
>
> 多线程：
>
> ​	thread
>
> ​	mutex、lock_ground
>
> ​	condition_variable
>
> ​	atomic
>
> 右值引用
>
> ​	T&&：
>
> ​	将亡值：
>
> ​			移动构造
>
> ​			移动拷贝构造

​	    实现移动语义：std::move()

​            实现完美转发：万能引用 T&&

​					forward





​	



# 2.C基础





# 3. linux基础命令

