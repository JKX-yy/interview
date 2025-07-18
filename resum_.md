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





### 10. 虚函数表和虚函数表指针

虚函数表和虚函数表指针的创建时机：

虚函数表：

​	内容在编译期生成    

​        全局数据区的只读数据段.rodata 

​	虚函数表存放虚函数的地址的数组

​	编译期发现virtual关键字

虚函数表指针：

​	当为类构建对象的时候（构造函数/默认构造函数），编译器会把vptr给类的对象，存放在前八字节中。

​	如果类没有构造函数 编译器自动为类生成默认构造函数，目的是为类对象初始化vptr

​	继承下虚函数表指针的赋值过程：vptr先=基类虚函数表地址（基类构造函数），然后vptr=子类的虚函数表地址（子类构造函数）

![image-20250715114306844](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715114306844.png)



背景： 用于实现多态  （静态多态 形参返回值不一样  函数名一样  在编译的时候确定）

 （动态多态 虚函数 继承的时候可以重写基类的虚函数 运行时才确定）

虚函数表创建时机：

1.什么时候生成的？（编译的时候生成的  virtual关键字修饰的函数）

2.存在哪里？  编译为可执行程序（存在磁盘里面） 运行时（内存）![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps1.jpg)![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps2.jpg)![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps3.jpg)![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps4.jpg)![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps5.jpg)![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps6.jpg)![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps7.jpg)



虚函数表指针的创建时机：类对象构造的时候 vptr ，把类的虚函数表地址赋值给vptr(类的对象的前八个字节指向虚函数表的地址)

![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps8.jpg) 

继承

B继承a的时候，B自己没有虚函数此时会在磁盘的只读代码段复制一个A的虚函数表。如果B自己写了一个虚函数func和A一样，是接着复制A的虚函数表并把对应的虚函数项的地址指向自己的实现吗。还是重新创建一个自己的虚函数表。、

![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps9.jpg)

（3）继承情况下  虚函数表指针的赋值过程？

![img](file:///C:\Users\jkx-pig\AppData\Local\Temp\ksohtml30668\wps10.jpg) 





### 11. 虚析构函数的作用

✅ 结论（通俗一点讲）：

> **只要你希望子类能“改写”（override）基类的某个函数，并在运行时“正确地调用它”——就要在基类中给这个函数加上 `virtual`。**



解决什么问题：基类对象指针=子类对象指针（在调用析构时 子类对象的析构函数得不到调用）

为什么这么设计：设计某个类的时候不一定是基类，如果该类是基类，我们应该手动将基类的析构函数设计为虚析构。

设置虚析构函数是有代价的，编译器会为类生成虚函数表，每个对象都有虚函数表指针，增大了资源消耗

![image-20250715115204088](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715115204088.png)![image-20250715115504216](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715115504216.png)

作用：继承下 为了使子类的析构函数能正常调用，需要基类的析构设置为虚析构。

​	![image-20250715120043340](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715120043340.png)

当然可以，咱们来**通俗+形象地解释一下**这句代码在干嘛：









🌰 先来个比喻：

把 `Base*` 看成是一个**“动物”类型的指针**，你有很多动物：狗、猫、鸟……

你写了个基类 `Animal`，然后让 `Dog` 和 `Cat` 都继承它：

```c++
cppCopyEditclass Animal {
public:
    virtual void speak() { std::cout << "Animal叫" << std::endl; }
    virtual ~Animal() {}
};

class Dog : public Animal {
public:
    void speak() override { std::cout << "狗汪汪" << std::endl; }
};

class Cat : public Animal {
public:
    void speak() override { std::cout << "猫喵喵" << std::endl; }
};
```

🧠 正常你会这样写：

```
cppCopyEditDog* d = new Dog();
Cat* c = new Cat();
d->speak();  // 狗汪汪
c->speak();  // 猫喵喵
```

这当然没问题，但想象你有很多动物，比如 100 种，要一个一个写太麻烦了！

🎯 现在的代码是这么做的：

```
cppCopyEditstd::vector<Animal*> zoo;        // 创建一个“动物园”
zoo.push_back(new Dog());        // 加一只狗进去（自动转成 Animal*）
zoo.push_back(new Cat());        // 加一只猫进去（自动转成 Animal*）

for (Animal* a : zoo)
    a->speak();   // 调用多态：自动根据实际类型说话
```

🟢 输出是：

```
CopyEdit狗汪汪
猫喵喵
```

🧠 回到你原来的代码

```
cppCopyEditstd::vector<Base*> objs;
objs.push_back(new Drive());
objs.push_back(new AnotherDrive());
```

翻译成人话就是：

> 创建一个“基类指针的容器”（像动物园），然后往里面丢进了两个子类对象（Drive 和 AnotherDrive）。虽然你用的是 `Base*` 类型，但程序**能自动识别每个对象的实际类型**，这就是**多态的魔力**。

🧹 最后别忘了清理！

```
cppCopyEditfor (Base* p : objs)
    delete p;  // 如果Base的析构函数不是virtual，就不会清除掉子类的内容
```

所以：

✅ **总结一句话通俗版：**

> 你用 `Base*` 把子类对象统一装起来，就像用“动物指针”统一装猫狗狮子，然后靠虚函数和多态让它们各自正确地“发出自己的声音”，方便管理也灵活扩展。





### 12. 动态库和静态库的区别

1. 生成方式：

2. 连接方式：

   1. 静态链接  ：把静态库编译进目标文件

   2. 动态链接 ：没有把动态库编译进文件

      ​		 程序运行时动态加载 

      ​		只做语法检查

3. 空间占用：

   1. 静态库占用空间大，存在多个副本
   2. 动态库占用空间小，只存在一个副本共享

4. 使用方式：

   1. 静态库所在程序是直接运行
   2. 动态库所在程序是动态加载的，   程序环境需要制定动态库查找路径   LD_LIBBRARY_PATH

5. 执行速度：

   1. 静态库执行速度快
   2. 动态库执行速度慢

6. 库文件发生变更

   1. 接口改变（参数个数  类型等等） 都需要重新编译
   2. 接口实现改变 ： 静态库和.c文件都需重新编译        只需重新编译动态库

![image-20250715144322206](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715144322206.png)



#### Cmake中的动态库和静态库











### 13.  Cmake  语法  



在build  目录  cmake   然后make 

  #### 01.  set()   file()/aux_source_directory  add_excutable()

基础

构建一个 build目录  进入到buile目录，执行cmake .. *(这个..路径是CMakeLists.txt文件所在的目录)*

![image-20250715145635290](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715145635290.png)

```cmake
CMakeLists.txt
#项目名
project(test)
#指定Cmake的最低版本
cmake_minimum_required(VERSION 3.0)

set(CMAKE_CXX_STANDARD 11)

#设置变量
set(SOR a.cpp  b.cpp  c.cpp)
#文件太多 不能一个一个都写出来  通过两种文件搜索的方式
aux_source_directory(${PROJECT_SOURCE_DIR} SOR) #PROJECT_SOURCE_DIR是cmake .. 的地址，搜索所有的.cpp .c .h 不能递归

file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp) # 在目录下递归搜索，  CMAKE_CURRENT_SOURCE_DIR（搜索指定的.cpp 递归）是CMakeList.txt 文件所在目录

#设置输出路径 宏定义
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR}) 
add_executable(test &{SRC})


```

✅ 总结一句话：

> CMake 不需要 `.h` 文件去参与编译，只要 `.cpp` 文件引用到了正确的 `.h`，编译器就能自动找到并使用。你没写 `.h` 也能编译成功，是完全合理的。





#### 02  include   src    build  

源文件和头文件分开了   要告诉编译器头文件所在的位置  要不找不到  

add_drictories(&{CMAKE_CURRENT_SOURCE_DIR}/include)

![image-20250715150543720](C:\Users\jkx-pig\AppData\Roaming\Typora\typora-user-images\image-20250715150543720.png)

```cmake
 project(test)
cmake_minimum_requiredVERSION 3.0()
set(CMAKE_CXX_STANDARD  11)

file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/sec/*.cpp)
# 设置头文件搜索路径.h
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)  #等同于给编译器加了参数：-I/path/to/your/project/include
#添加库  static  share
#add_library(calc SHARED ${SRC})

set(EXECUTABLE  ${CMAKE_CURRENT_SOURCE_DIR} )
add_executable(text &{SRC})

```



```cmake

# cmake 时构建一个build目录（保存cmake生成的中间产物） cmake ..(这个..路径是CMakeLists.txt文件所在的目录)
#项目名 不写可不可以
project(test)
#指定cmake版本  高于3.0
cmake_minimum_required(VERSION 3.0)

#设置变量
#set(SOR add.cpp div.cpp mult.cpp sub.cpp main.cpp)
#文件搜索的两种方式
#aux_source_directory(${PROJECT_SOURCE_DIR} SOR)  #   PROJECT_SOURCE_DIR（搜索所有的.cpp .c .h文件 不能递归）是cmake 后的地址

#[[注意  1.file只是说把路径列表给SOR   但是并没有给编译器，.h需要给编译器编译器才知道去哪里寻找
file(GLOB SOR ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB SOR_H ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h) 
set(SOR ${SOR} ${SOR_H})  # 合并列表
虽然你把头文件路径放入了 SOR 变量
但 CMake 不会自动 把这些路径加入编译器的头文件搜索路径
编译器仍然不知道去哪里找 head.h

]]
file(GLOB SOR ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)  #   CMAKE_CURRENT_SOURCE_DIR（搜索指定的.cpp 递归）是CMakeList.txt 文件所在目录
# 设置头文件搜索路径
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include) 


#设置输出路径 宏定义
set(EXECUTABLE_OUTPUT_PATH  ${PROJECT_SOURCE_DIR})  #PROJECT_SOURCE_DIR宏是cmake 后面跟的路径 
#生成可执行文件
add_executable(test ${SOR})

```



#### 03 静态库  动态库  



总体流程  一样的

add_library()  //生成库

link_directories()  //告诉库的位置

add_executable() //生成可执行文件

target_link_libraries(test  cale) //  告诉某个可执行文件链接哪个库



```cmake
project(test)
    
cmake_minimum_required(VERSION 3.0)
set(CMAKE_CXX_STANDARD 11)
    
    
file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
#库的输出路径
#  静态库和动态库都可以使用 LIBRARY_OUTPUT_PATH 针对动态库具有可执行权限还可以使用EXECUTABLE_OUTPUT_PATH
set(LIBRARY_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR})

#添加静态库 
add_library(cale_a STATIC ${SRC})

#添加动态库
add_library(cale_s SHARED ${SRC})


link_directories(${CMAKE_CURRENT_SOURCE_DIR}/lib1)# 告诉静态库的文件位置
add_executable(test1 ${SRC})  
target_link_libraries(test1 cale_a)  #必须在add_execurable后面是给某个“已经存在的目标”（target）设置链接库属性。

# 有了库要发布给使用者  动态库和静态库都是二进制文件 .cpp是文本的格式  所以发布的时候还要有include中的.h文件

#测试创建出来的静态库和动态库能不能用  需要先发布 库和头文件
# #动态库的链接
link_directories(${CMAKE_CURRENT_SOURCE_DIR}/lib2)
add_executable(test2 ${SRC})
target_link_libraries(test2  cale) #这里给出动态库的链接  TARGET
    
```

是的，✅ **`target_link_libraries(test1 cale_a)` 必须在 `add_executable(test1 ...)` 之后执行**，这是 **CMake 的语法规定**，不是建议，而是硬性规则。

📌 原因解释（一定要在后面）

CMake 的设计逻辑是：

> `target_link_libraries()` 是给某个“已经存在的目标”（target）设置链接库属性。

所以你必须 **先创建这个目标**（不管是可执行文件 `add_executable()`，还是库 `add_library()`），CMake 才允许你对它设置链接信息。

#### 05  message

打印消息的

```cmake
#打印消息
cmake_minimum_required(VERSION 3.0)
project(message)
set(CMAKE_CXX_STANDARD 11)


file(GLOB  SRC ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
message( "xxxxxxxxxxxxxx")
link_directories(${CMAKE_CURRENT_SOURCE_DIR}/lib2)
message(STATUS "11111xxxxxxxxxxxxxx")
add_executable(test ${SRC})
#输出变量
message(${SRC})
message(STATUS "11111xxxxxxxxxxxxxx")
```



#### 06 list 

```
#[[字符串的操作    
list（APPEND src  item1  item2  ... )   
list(REMOVE_ITEM src  item1  item2  ...)   
获取长度list(LENGTH src  output_var)  
获取元素list(GET SRC -1 output_var)
                list(JOIN src  sep  output_var)
                list(find  src  item  output_var)
                list(insert src index item1 item2 ...)
插入头           list(prepend src item1 item2 ...)
弹出最后一个元素  list(POP_BACK src output_var)
弹出头           list(POP_FRONT src output_var)
移除列表中重复元素 list(REMOVE_DUPLICATES src)
将指定索引删除    list(REMOVE_AT src index)
列表反转         list(REVERSE src)
列表排序         list(SORT src comparator)
]]
#list的其他操作   
```



```cmake
cmake_minimum_required(VERSION 3.0)
project(test)
set(CMAKE_CXX_STANDARD 11)

message("===========配置开始============")
file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB APP ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
#删除绝对路径
list(REMOVE_ITEM SRC ${APP}) 
message("SRC:${SRC}")
message("APP:${APP}")
message("===========配置完成============")

#创建动态库
message("===========创建动态库============")
set(LIBRARY_OUTPUT_PATH  ${CMAKE_CURRENT_SOURCE_DIR}/lib_so)
add_library(calc SHARED ${SRC})
message("===========创建动态库完成============")

link_directories(${LIBRARY_OUTPUT_PATH})
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR})
add_executable(test ${APP})
target_link_libraries(test calc)

```

#### 07 宏定义  

*#进行宏定义  自定义 帮我们把程序执行期间需要的宏定义出来  add_definitions(-D宏)*

很好！我们来详细讲清楚 CMake 中的：

> ✅ `add_definitions(-DDEBUG)` 的含义、作用、使用时机，以及现代替代方式。

✅ 一句话说明：

```
cmake


CopyEdit
add_definitions(-DDEBUG)
```

表示：

> **给所有后续的源文件添加一个预处理宏定义 `DEBUG`，等效于在编译时加上 `-DDEBUG`。**

✅ 它等价于手动 g++ 命令的写法：

```
bash


CopyEdit
g++ -DDEBUG main.cpp -o main
```

这样在 C++ 源文件里你就可以用：

```
cppCopyEdit#ifdef DEBUG
    std::cout << "Debugging info..." << std::endl;
#endif
```

```Cmake
add_difinition(-DDEBUG)

cmake_minimum_required(VERSION 3.0)
project(test)
set(CMAKE_CXX_STANDARD 11)

message("===========配置开始============")
file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB APP ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
#删除绝对路径
list(REMOVE_ITEM SRC ${APP}) 
message("SRC:${SRC}")
message("APP:${APP}")
message("===========配置完成============")

#创建动态库
message("===========创建动态库============")
set(LIBRARY_OUTPUT_PATH  ${CMAKE_CURRENT_SOURCE_DIR}/lib_so)
add_library(calc SHARED ${SRC})
message("===========创建动态库完成============")

link_directories(${LIBRARY_OUTPUT_PATH})
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR})
add_executable(test ${APP})
target_link_libraries(test calc)

```

#### 08 父子关系的复杂目录

*[add_subdirectory(lib)  父子关系的复杂目录  子目录也包含cmakelists.txt* 

*子目录可以使用父目录的变量（全局变量）   父目录不可以使用子目录（局部）* 

*]]*

```cmake
add_definitions(-DDEBUG)
cmake_minimum_required(VERSION 3.0)
project(test)
set(CMAKE_CXX_STANDARD 11)

message("===========配置开始============")
file(GLOB SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB APP ${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp)
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)
#删除绝对路径
list(REMOVE_ITEM SRC ${APP}) 
message("SRC:${SRC}")
message("APP:${APP}")
message("===========配置完成============")

#创建动态库
message("===========创建动态库============")
set(LIBRARY_OUTPUT_PATH  ${CMAKE_CURRENT_SOURCE_DIR}/lib_so)
add_library(calc SHARED ${SRC})
message("===========创建动态库完成============")

link_directories(${LIBRARY_OUTPUT_PATH})
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_SOURCE_DIR})
add_executable(test ${APP})
target_link_libraries(test calc)

```





很好，这次我们讲 CMake 中的一个重要命令：

> ✅ `add_subdirectory` —— 用于**将子目录加入当前构建系统**，支持递归构建多个模块或组件。





✅ 一句话理解

```
cmake


CopyEdit
add_subdirectory(src)
```

表示：

> “请进入 `src/` 目录，加载它的 `CMakeLists.txt` 并把它当作当前项目的一部分来一起构建。”

📘 它的作用：

| 功能                         | 说明                                           |
| ---------------------------- | ---------------------------------------------- |
| 🔨 **模块化管理源码**         | 每个模块或组件独立放在子目录中，便于维护和分工 |
| 🔁 **递归构建**               | 会递归调用子目录下的 `CMakeLists.txt` 文件     |
| 🔗 **支持 target 共享与依赖** | 主目录可以链接子目录中生成的库或目标           |

✅ 示例场景

你的工程结构如下：

```
csharpCopyEditproject/
├── CMakeLists.txt          # 顶层
├── main.cpp
├── math/
│   ├── CMakeLists.txt      # 子目录
│   ├── add.cpp
│   └── add.h
```

顶层 `CMakeLists.txt`

```
cmakeCopyEditcmake_minimum_required(VERSION 3.10)
project(MyProject)

add_subdirectory(math)  # 加入 math 子目录

add_executable(main main.cpp)
target_link_libraries(main mathlib)  # 链接子目录里的库
```

子目录 `math/CMakeLists.txt`

```
cmakeCopyEditadd_library(mathlib add.cpp)
target_include_directories(mathlib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```

main.cpp`

```
cppCopyEdit#include "add.h"
#include <iostream>

int main() {
    std::cout << add(2, 3) << std::endl;
    return 0;
}
```

✅ 执行结果：

- CMake 会递归进入 `math/` 目录
- 编译 `mathlib` 静态库
- 主程序 `main` 链接这个库
- 一起构建完成，目标可用
- 



### 14 .STL 容器  

https://www.nowcoder.com/discuss/724532234882940928

嵌入式八股2】C++：STL容算法

黑马程序员 PDF    

[]: 

[]()

#### 1. STL常见容器及其内部实现的数据结构

序号 名称 描述 存储结构 常用方法和操作

 二、哪些 STL 容器的操作**必须**或**推荐**使用迭代器？



✅string的操作是基于下标的迭代器也可以，但是严格

来说string并不属于容器，一般容器来都是以迭代器

为基准访问元素的 inset   erase



| 容器类型                               | 是否支持随机访问 | 是否推荐使用迭代器    | 原因                                        |
| -------------------------------------- | ---------------- | --------------------- | ------------------------------------------- |
| `std::vector`                          | ✅ 是             | ❌（可以用下标）       | 因为它像数组一样                            |
| `std::string`                          | ✅ 是             | ❌（常用下标）         | 同上                                        |
| `std::deque`                           | ✅ 是             | ❌（下标或迭代器皆可） | 支持随机访问                                |
| `std::list`                            | ❌ 否             | ✅ 必须用迭代器        | 不支持 `operator[]`                         |
| `std::forward_list`                    | ❌ 否             | ✅ 必须用迭代器        | 只能前向遍历                                |
| `std::set` / `std::multiset`           | ❌ 否             | ✅ 必须用迭代器        | 无下标访问                                  |
| `std::map` / `std::multimap`           | ❌ 否             | ✅ 必须用迭代器        | 无下标访问，`[]`只能用于 `map<K, V>` 的写入 |
| `std::unordered_map` / `unordered_set` | ❌ 否             | ✅（多使用迭代器）     | 不支持有序位置                              |



非常棒的问题！

你问的是：

> `std::string::insert` 为什么可以用下标而不是像 `vector` 一样必须用迭代器？是不是其实也可以用迭代器？

答案是：

> ✅ **`std::string::insert` 既可以用下标，也可以用迭代器！**

🔹 一、`std::string::insert` 的两种接口形式

`std::string` 是 STL 中少数提供**下标式插入**和**迭代器式插入**两种方式的容器。

✅ 形式 1：下标插入

```
cppCopyEditstd::string s = "hello";
s.insert(2, "XX");  // 插入到第2个位置，变成 heXXllo
```

✅ 形式 2：迭代器插入（像 vector 一样）

```
cppCopyEditstd::string s = "hello";
s.insert(s.begin() + 2, 'X');  // 插在第2个位置
```

或者插入一段：

```
cppCopyEditstd::string s = "hello";
std::string t = "XX";
s.insert(s.begin() + 2, t.begin(), t.end());  // heXXllo
```

🔹 二、为什么 `string` 提供下标接口，而 `vector` 没有？

✅ 原因 1：字符串操作“以位置为核心”

- 字符串 API 传统上都围绕“第几个字符”展开，比如：
  - `insert(pos, str)`
  - `substr(pos, len)`
  - `replace(pos, len, str)`
- 所以提供下标版本，**符合直觉、常用、简洁**。

✅ 原因 2：`string` 是专门为字符处理而优化的类，不是纯通用容器

- 它是 STL 的 `basic_string<char>`，但也被“当作字符数组”来对待；
- 提供了额外的便捷接口，是**特例**，不是一般 STL 容器都这样。

🔹 三、对比：`string` vs `vector` 的 insert 接口

| 容器          | 支持下标插入 | 支持迭代器插入 | 原因说明                             |
| ------------- | ------------ | -------------- | ------------------------------------ |
| `std::vector` | ❌ 不支持     | ✅ 支持         | 泛型容器，不提供按下标插入的语义     |
| `std::string` | ✅ 支持       | ✅ 支持         | 因为它是字符串，有历史习惯和特殊处理 |





| 序号 | 名称           | 描述                   | 存储结构                       | 常用方法和操作                                               |
| ---- | -------------- | ---------------------- | ------------------------------ | ------------------------------------------------------------ |
| 0    | string         |                        | char*                          | 增：s.insert, s.append   删：s.erase   改：是。replace(),s.assing(),查：find()   ,其他s.substr(),s.size(),at(),[] |
| 1    | vector         | 动态分配的数组         | 顺序数组（array）              | `v.push_back()`, `v.pop_back()`, `v.insert()`, `v.erase()`, `v.capacity()`, `v.size()`, `v.at(idx)`, `v.front()`, `v.back()  ,v.reserve(),v.swap() ,v.resize()，v.assgin(),v.empty()，` |
| 5    | deque          | 双端队列,双端数组      | 分段连续（多个 `vector` 连续） | `d.push_back()`, `d.push_front()`, `d.pop_back()`, `d.pop_front()`, `d.insert()`, `d.erase()  ,d.assgin()/d.empty()/d.size()/d.resize()/容器没有容量的概念，。at(),[],/ d.front()/d.back()/sort()` |
| *    | ****           | ***                    | ****                           | **下面是非随机访问的容器  不可以随即插入删除没有at  []**     |
| 2    | list           | 双向链表               | 离散                           | `lt.push_back()`, `lt.push_front()`, `lt.insert()`, `lt.erase()`, `lt.sort()`, `lt.merge()`, `lt.splice()  ，assgin()  swap()  size()  empty  resize(),  remove(),clear(),reverse(), ` |
| 3    | stack          | 栈                     | 用 `list` 或 `deque` 实现0     | `push()`, `pop()`, `top()  empty(),size(),`，                |
| 4    | queue          | 队列(不允许有遍历行为) | 用 `list` 或 `deque` 实现      | `push()`, `pop()`, `front()`, `back()  empty()  size()`  ，  |
| 6    | priority_queue | 优先级队列             | `vector`                       | `push()`, `pop()`, `top()`                                   |
| ***  | *******        | ********               | ******                         | **********************没有push  pop                          |
| 7    | set            | 集合（有序不重复）     | 红黑树（弱平衡二叉搜索树）     | `insert()  clear() ` , `erase()`, `find()`, `count()`, `clear()  size()  empty() swap() ` |
| 8    | multiset       | 集合（有序可重复）     | 红黑树                         | `insert()`, `erase()`, `find()`, `count()`, `clear()`        |
| 9    | unordered_set  | 集合（无序不重复）     | 哈希表                         | `insert()`, `erase()`, `find()`, `count()`, `clear()`        |
| 10   | map            | 键值对（有序不重复）   | 红黑树                         | `insert() clear()` , `erase()`, `find()`, `count()`, `clear() size() empty() swap() ` |
| 11   | multimap       | 键值对（有序可重复）   | 红黑树                         | `insert()`, `erase()`, `find()`, `count()`, `clear()`        |
| 12   | unordered_map  | 键值对（无序不重复）   | 哈希表                         | `insert()`, `erase()`, `find()`, `count()`, `clear()`        |
| 13   | hash_map       | 哈希表（类似 `map`）   | 哈希表                         | `insert()`, `erase()`, `find()`, `count()`, `clear()`        |

#### 2. deque底层数据结构

![image-20250716100529563](assets/image-20250716100529563.png)

`deque` 底层实现通常是分段连续的内存结构，即由多个 `vector` 组成，允许高效的从两端进行元素的插入和删除。

✅ **deque****与****vector****区别：**

vector对于头部的插入删除效率低，数据量越大，效率越低

deque相对而言，对头部的插入删除速度回比vector快

vector访问元素时的速度会比deque快,这和两者内部实现有关





✅ deque内部工作原理:

deque内部有个**中控器**，维护每段缓冲区中的内容，缓冲区中存放真实数据

中控器维护的是每个缓冲区的地址，使得使用deque时像一片连续的内存空间



![image-20250716100442831](assets/image-20250716100442831.png)

#### list  容器

**功能：**将数据进行链式存储

**链表**（list）是一种物理存储单元上非连续的存储结构，数据元素的逻辑顺序是通过链表中的指针链接实现的

结点的组成：一个是存储数据元素的**数据域**，另一个是存储下一个结点地址的**指针域**

![image-20250716102138570](assets/image-20250716102138570.png)

由于链表的存储方式并不是连续的内存空间，因此链表list中的迭代器只支持前移和后移，属于**双向迭代器**

✅ list的优点：

采用动态存储分配，不会造成内存浪费和溢出

链表执行插入和删除操作十分方便，修改指针即可，不需要移动大量元素

✅list的缺点：

链表灵活，但是空间(指针域) 和 时间（遍历）额外耗费较大

List有一个重要的性质，插入操作和删除操作都不会造成原有list迭代器的失效，这在vector是不成立的。



> ✅ **`std::list` 的一个重要特性是：插入或删除不会使原有的迭代器失效（除非删除的是该迭代器所指的元素本身）**

那我们来详细解释：**为什么 list 可以这么干？其他容器（如 vector、set、map）为什么不行？**

🔹 一、先来看 `std::list` 为什么不失效？

✅ 本质原因：`list` 是链表（通常是双向链表）

- 每个元素是一个节点，存在堆上；
- 节点之间通过指针链接；
- 插入/删除节点，不会移动其它节点的地址；
- 所以之前返回的迭代器（其实就是节点指针）依然有效！

```
cppCopyEditstd::list<int> lst = {1, 2, 3};
auto it = std::next(lst.begin());  // 指向 2
lst.insert(lst.begin(), 0);        // 插入 0
std::cout << *it;                  // 仍然是 2，有效
```

🔹 二、再来看 vector 为什么会失效？

❌ 原因：`vector` 是**连续数组**

- 插入/删除元素会造成后续元素移动（重排）；
- 有时还会导致整个数组重新分配到新的内存块（尤其是插入导致容量不足时）；
- 所以旧的迭代器（本质是指向旧内存地址的指针）会失效！

```
cppCopyEditstd::vector<int> v = {1, 2, 3};
auto it = v.begin();
v.insert(v.begin(), 0);  // 插入会导致元素右移，it 失效
std::cout << *it;        // ❌ 未定义行为！
```

🔹 三、其他容器的迭代器失效规则

我们用表格总结一下各种容器的迭代器失效行为：

| 容器类型                          | 插入是否会导致迭代器失效？       | 删除是否会导致迭代器失效？       | 说明                           |
| --------------------------------- | -------------------------------- | -------------------------------- | ------------------------------ |
| `vector`                          | ✅ 会（尤其是插入中间或 realloc） | ✅ 会（删除点后所有元素右移）     | 数组结构                       |
| `deque`                           | ✅ 有概率失效（特别是首尾插入）   | ✅ 有概率失效                     | 分段连续内存                   |
| `list`                            | ❌ 不会失效（除非删除该元素）     | ❌ 不会失效（除非删除该元素）     | 链表                           |
| `forward_list`                    | ❌ 不会失效（除非删除该元素）     | ❌ 不会失效（除非删除该元素）     | 单向链表                       |
| `set` / `map`                     | ✅ 插入通常不失效                 | ✅ 删除该元素会失效，其他不受影响 | 红黑树，结构稳定但元素可变性差 |
| `unordered_set` / `unordered_map` | ✅ 插入可能会导致迭代器失效       | ✅ 删除该元素会失效               | 哈希表，bucket 可能重排        |

🔹 四、补充说明：为什么容器不统一？

因为每种容器的底层结构不同，迭代器实质也是**对结构的封装**：

| 容器            | 底层结构 | 迭代器实质            |
| --------------- | -------- | --------------------- |
| `vector`        | 连续内存 | 原始指针 `T*`         |
| `list`          | 双向链表 | 指向节点的封装指针    |
| `set/map`       | 红黑树   | 包含指针 + 树结构信息 |
| `unordered_map` | 哈希表   | 包括 bucket 信息      |



因此：**迭代器能不能保留，与结构是否稳定密切相关**。

🔹 五、面试必备总结口诀

> 插删无忧属链表，数组变动请小心；
>  树类结构看元素，哈希装桶也谨慎。



#### set/multiset  容器

**简介：**

所有元素都会在插入时自动被排序

**本质：**

set/multiset属于**关联式容器**，底层结构是用**二叉树**实现。

**set****和****multiset****区别**：

set不允许容器中有重复的元素

multiset允许容器中有重复的元素

set容器插入数据时用insert

set容器插入数据的数据会自动排序



**区别：**

set不可以插入重复数据，而multiset可以

set插入数据的同时会返回插入结果，表示插入是否成功

multiset不会检测数据，因此可以插入重复数据





三、红黑树 vs 哈希表：对比总结

| 特性       | 红黑树             | 哈希表                    |
| ---------- | ------------------ | ------------------------- |
| 查找效率   | O(log n)           | 平均 O(1)，最坏 O(n)      |
| 有序性     | ✅（支持范围查找）  | ❌（无序）                 |
| 内存利用   | 高（无扩容）       | 可能有大量空槽或链表      |
| 扩展性     | 不需要扩容         | 容量需手动扩容/自动扩容   |
| 并发友好性 | 相对较难           | Java 有 ConcurrentHashMap |
| 常用用途   | 有序映射，区间查询 | 快速查找，键值缓存        |

#### 对组 pair

pair<type, type> p = make_pair( value1, value2 );



#### map/multimap容器

**简介：**

map中所有元素都是pair

pair中第一个元素为key（键值），起到索引作用，第二个元素为value（实值）

所有元素都会根据元素的键值自动排序

**本质：**

map/multimap属于**关联式容器**，底层结构是用二叉树实现。

**优点：**

可以根据key值快速找到value值

map和multimap**区别**：

map不允许容器中有重复key值元素

multimap允许容器中有重复key值元素









#### 3. 红黑树

红黑树是一种非严格平衡的二叉查找树，具有自动排序的功能。每个节点存储一个颜色（红或黑），并且通过调整树的结构保持特定的平衡条件，从而保证最坏情况下的查找效率。

### 15.STL 常见算法

主要有三个头文件  <algorithm> <functional> <numeric> 组成

<algorithm>是所有头文件中最大的，范围涉及 比较运算查找、 遍历、 赋值、 修改、等等。

<numeric> 体积很小，只包括几个在序列上面进行简单数学运算的模板函数

<functional> 定义了一些模板类,用以声明函数对象



#### 函数对象

概念：  重载了（）操作符的类，其对象成为函数对象

​		函数对象使用重载的()时，行为类似函数，所以称为仿函数。

本质：

​	函数对象（仿函数） 是一个类   不是一个函数。

**4.1.2** **函数对象使用**

**特点：**

函数对象在使用时，可以像普通函数那样调用, 可以有参数，可以有返回值

函数对象超出普通函数的概念，函数对象可以有自己的状态

函数对象可以作为参数传递

#### 谓词

概念：  返回bool类型的仿函数称为谓词  

​		一元谓词

​		二元谓词

#### 内建函数对象

STL内建了一下函数对象   functional

​	算数仿函数

​	关系仿函数

​	逻辑仿函数

用法：

这些仿函数所产生的对象，用法和一般函数完全相同

使用内建函数对象，需要引入头文件 #include<functional>

1. 算数仿函数

   plus<T>

   minus()

   multiplies()

   divides()

   modulus()

   negate()

   template<class T> T plus<T> //加法仿函数

   template<class T> T minus<T> //减法仿函数

   template<class T> T multiplies<T> //乘法仿函数

   template<class T> T divides<T> //除法仿函数

   template<class T> T modulus<T> //取模仿函数

   template<class T> T negate<T> //取反仿函数

2. 关系仿函数

   template<class T> bool equal_to<T> //等于

   template<class T> bool not_equal_to<T> //不等于

   template<class T> bool greater<T> //大于

   template<class T> bool greater_equal<T> //大于等于

   template<class T> bool less<T> //小于

   template<class T> bool less_equal<T> //小于等于

3. 逻辑仿函数

   



#### 4. 常见排序算法

算法主要是由头文件 <algorithm> <functional> <numeric> 组成。

<algorithm> 是所有STL头文件中最大的一个，范围涉及到比较、 交换、查找、遍历操作、复制、修改等等

<numeric> 体积很小，只包括几个在序列上面进行简单数学运算的模板函数

<functional> 定义了一些模板类,用以声明函数对象。

##### 1.常用遍历算法

for_each

```c++

    for_each(iterator beg, iterator end, _func);
// 遍历算法 遍历容器元素
// beg 开始迭代器
// end 结束迭代器
// _func 函数或者函数对象

```

![image-20250716150636242](assets/image-20250716150636242.png)



transform 

搬运容器到另一个容器中

```c++
transform(iter1 beg ,iter1 end,iter2 beg,fun )
    
transform(iterator beg1, iterator end1, iterator beg2, _func);
//beg1 源容器开始迭代器
//end1 源容器结束迭代器
//beg2 目标容器开始迭代器
//_func 函数或者函数对象

```





##### 2. 常用查找算法

find //查找元素

find_if //按条件查找元素

adjacent_find //查找相邻重复元素

binary_search //二分查找法

count //统计元素个数

count_if //按条件统计元素个数



```c++
find (beg,end,elm)
    vector<int>::iterator it = find(v.begin(), v.end(), 5);
// 按值查找元素，找到返回指定位置迭代器，找不到返回结束迭代器位置
// beg 开始迭代器
// end 结束迭代器
// value 查找的元素    
```

```c++
find_if()  // 只返回第一个满足条件的元素位置（迭代器）。
find_if(iterator beg, iterator end, _Pred);
// 按值查找元素，找到返回指定位置迭代器，找不到返回结束迭代器位置
// beg 开始迭代器
// end 结束迭代器
// _Pred 函数或者谓词（返回bool类型的仿函数）
```

```c++
count
count(iterator beg, iterator end, value);
// 统计元素出现次数
// beg 开始迭代器
// end 结束迭代器
// value 统计的元素
```

```c++
count_if
count_if(iterator beg, iterator end, _Pred);
// 按条件统计元素出现次数
// beg 开始迭代器
// end 结束迭代器
// _Pred 谓词
```





##### 3.常用排序算法

sort //对容器内元素进行排序

random_shuffle //洗牌 指定范围内的元素随机调整次序

merge // 容器元素合并，并存储到另一容器中

reverse // 反转指定范围的元素



```c++
sort(iterator beg, iterator end, _Pred);
// 按值查找元素，找到返回指定位置迭代器，找不到返回结束迭代器位置
// beg 开始迭代器
// end 结束迭代器
// _Pred 谓词

```



```c++
merge
merge(iterator beg1, iterator end1, iterator beg2, iterator end2, iterator dest);
// 容器元素合并，并存储到另一容器中
// 注意: 两个容器必须是有序的
// beg1 容器1开始迭代器 // end1 容器1结束迭代器 // beg2 容器2开始迭代器 // end2 容器2结束迭代器 //
dest 目标容器开始迭代器
```



```c++
reverse(iterator beg, iterator end);
// 反转指定范围的元素
// beg 开始迭代器
// end 结束迭代器
```



##### 4.常用拷贝和替换算法

copy // 容器内指定范围的元素拷贝到另一容器中

replace // 将容器内指定范围的旧元素修改为新元素

replace_if // 容器内指定范围满足条件的元素替换为新元素

swap // 互换两个容器的元素

```c++
copy(iterator beg, iterator end, iterator dest);
// 按值查找元素，找到返回指定位置迭代器，找不到返回结束迭代器位置
// beg 开始迭代器
// end 结束迭代器
// dest 目标起始迭代器
```

```c++
replace(iterator beg, iterator end, oldvalue, newvalue);
// 将区间内旧元素 替换成 新元素
// beg 开始迭代器
// end 结束迭代器
// oldvalue 旧元素
// newvalue 新元素
```

```c++
replace_if(iterator beg, iterator end, _pred, newvalue);
// 按条件替换元素，满足条件的替换成指定元素
// beg 开始迭代器
// end 结束迭代器
// _pred 谓词
// newvalue 替换的新元素
```

```c++
swap(container c1, container c2);
// 互换两个容器的元素
// c1容器1
// c2容器2
```





##### 5. 常用算术生成算法 \#include <numeric>

**总结：**accumulate使用时头文件注意是 numeric，这个算法很实用





accumulate // 计算容器元素累计总和

fill // 向容器中添加元素

```c++
accumulate()
accumulate(iterator beg, iterator end, value);
// 计算容器元素累计总和
// beg 开始迭代器
// end 结束迭代器
// value 起始值
int total = accumulate(v.begin(), v.end(), 0);
```

```c++
fill(iterator beg, iterator end, value);
// 向容器中填充元素
// beg 开始迭代器
// end 结束迭代器
// value 填充的值
```



##### 6. 常用集合算法



set_intersection // 求两个容器的交集

```c++

```

set_union // 求两个容器的并集

set_difference // 求两个容器的差集











##### 4.1 `sort()`

- **适用容器**：仅支持随机访问的容器，如 `vector`、`deque`、`array`。
- **功能**：快速排序。

[复制代码](#)

```
bool func(``int` `a, ``int` `b) {``  ``return` `a > b; ``// 降序排列``}``sort(vec.begin(), vec.end(), func); ``// 对vector进行排序
```

##### 4.2 `partial_sort()`

- **功能**：对部分元素进行排序，使用堆排序实现。
- **参数**：排序范围，默认排序前 `n` 个元素。

[复制代码](#)

```
int` `n = ``4``; ``// 需要排序的元素个数``partial_sort(vec.begin(), vec.begin() + n, vec.end(), func); ``// 排序前4个元素
```

##### 4.3 `is_sorted()`

- **功能**：检查容器是否已排序。
- **返回值**：布尔值，`true` 表示已排序。

[复制代码](#)

```
bool result = is_sorted(vec.begin(), vec.end(), func);
```

##### 4.4 `is_sorted_until()`

- **功能**：返回第一个破坏排序规则的元素位置。

[复制代码](#)

```
auto it = is_sorted_until(vec.begin(), vec.end(), func);
```

#### 5. 查找操作

##### 5.1 `find()`

- **功能**：查找指定元素。

[复制代码](#)

```
vector<``int``> vec{``10``, ``20``, ``30``, ``40``, ``50``};``auto it = find(vec.begin(), vec.end(), ``30``); ``// 查找30``if` `(it != vec.end()) {``  ``cout << ``"查找成功："` `<< *it;``} ``else` `{``  ``cout << ``"查找失败"``;``}
```

##### 5.2 `find_if()`

- **功能**：根据自定义谓词查找元素。

[复制代码](#)

```
bool mycomp(``int` `i) {``  ``return` `(i % ``2``) == ``1``; ``// 查找奇数``}``vector<``int``> myvector{``4``, ``2``, ``3``, ``1``, ``5``};``auto it = find_if(myvector.begin(), myvector.end(), mycomp);
```

#### 6. 使用 `vector` 避免频繁的内存重新分配

`vector` 在扩容时通常会以 2 倍容量增长，这会导致频繁的内存分配和元素拷贝。为了优化性能，可以采取以下策略：

##### 6.1 预分配内存

在创建 `vector` 时，可以使用 `reserve()` 方法预先分配内存，以减少多次扩容。

##### 6.2 合理选择初始容量

根据数据的预计大小选择合适的初始容量，避免不必要的扩容操作。

##### 6.3 优化算法

使用时间复杂度较低的算法，避免在数据量增大时造成性能瓶颈。

#### 7. `vector` 的 `resize()` 与 `reserve()`

##### 7.1 `resize()`

- **功能**：调整容器的大小。
- **效果**：如果 `n` 小于当前大小，则删除尾部元素；如果 `n` 大于当前大小，新的元素会被默认构造并添加到尾部。

##### 7.2 `reserve()`

- **功能**：调整容器的容量，不会改变当前大小。
- **效果**：用于减少扩容次数，确保容器有足够的内存空间。

#### 8. `vector` 扩容原理

- **扩容过程**：每次扩容时，`vector` 会分配新的内存块并将现有元素复制到新内存中，旧内存被释放。
- **扩容系数**：通常情况下，`vector` 容量每次会翻倍或增加 1.5 倍，这可以减少频繁的扩容。

##### 8.1 Linux中的内存管理

在Linux系统中，内存区域以2的倍数扩容，以便进行高效的内存分配。

##### 8.2 Windows中的内存管理

在Windows系统中，内存分配通常会增加1.5倍，以便更好地利用已经释放的内存。

#### 9. 迭代器

迭代器是STL中用于访问容器元素的一种抽象工具。通过迭代器，容器元素的访问具有一致的接口，并且可以实现多态。

**注意**：迭代器只能前进，不能回退。

#### 10. 迭代器失效的情况

迭代器可能会因为容器结构的修改而失效。不同类型的容器失效的情况略有不同：

- **数组型数据结构**（如 `vector`）：`insert` 和 `erase` 操作会使插入或删除点之后的所有迭代器失效。
- **链表型数据结构**（如 `list`）：`erase` 操作只会使指向删除元素的迭代器失效，其他迭代器不受影响。
- **树型数据结构**（如 `map`）：`erase` 操作会使指向删除元素的迭代器失效，其他迭代器不受影响。 `insert` 操作不会使任何迭代器失效。



#### 4.2.1 vector list异同⭐⭐⭐⭐⭐

| 项目       | `vector`              | `list`（`std::list`）         |
| ---------- | --------------------- | ----------------------------- |
| 底层结构   | 连续数组              | 双向链表                      |
| 随机访问   | ✅ 支持（`v[i]`）      | ❌ 不支持                      |
| 插入效率   | ❌ 中间插入开销大      | ✅ 任意位置插入 O(1)           |
| 删除效率   | ❌ 中间删除开销大      | ✅ 任意位置删除 O(1)           |
| 内存局部性 | ✅ 好                  | ❌ 差                          |
| 空间开销   | 小                    | 大（每个节点需要两个指针）    |
| 迭代器失效 | 插入/删除可能全部失效 | 插入/删除不会使其他迭代器失效 |

#### 4.2.2 vector内存是怎么增长的vector的底层实现⭐⭐⭐⭐\

**底层结构**：`vector` 使用一段连续内存 `T* buffer` 来存储数据。

**容量扩容机制**：

- 当你插入新元素超出当前容量（`capacity()`）时，会进行**自动扩容**。
- 扩容策略通常是**2 倍增长**（实现细节可能因编译器而异）。
- 扩容过程包括：
  1. 分配新内存（通常是原容量的两倍）；
  2. 将旧元素 `move` 或 `copy` 到新地址；
  3. 释放旧内存。

**注意**：扩容会导致指针、引用、迭代器**全部失效**！



#### 4.2.3 vector和deque的比较⭐⭐⭐⭐

| 特性         | `vector`                 | `deque`                  |
| ------------ | ------------------------ | ------------------------ |
| 底层结构     | 单块连续数组             | 分段内存数组 + 索引表    |
| 随机访问     | ✅ 快速（`O(1)`）         | ✅ 也支持，但略慢         |
| 首尾插入效率 | ❌ 头部插入慢（O(n)）     | ✅ 头尾插入都是 O(1)      |
| 中间插入     | ❌ 慢                     | ❌ 仍慢                   |
| 内存结构     | 连续块                   | 非连续块                 |
| 适合场景     | 重视随机访问、尾部操作快 | 需要双端插入或删除的场景 |



#### 4.2.4为什么stl里面有sort函数list里面还要再定义一个sort⭐⭐⭐

`std::sort()` 只能作用于**随机访问迭代器**（如 `vector`, `deque`）。

`list` 的迭代器是**双向迭代器**，不支持 `it + n`，所以不能用 `std::sort()`。

因此，`std::list` 提供了成员函数 `.sort()`，使用**归并排序**，时间复杂度为 `O(n log n)`，不需要额外内存，稳定排序。



#### 4.2.5 STL底层数据结构实现⭐⭐⭐⭐

| 容器 | 底层实现结构 |
| ---- | ------------ |

| `vector` | 动态数组（连续内存） |
| -------- | -------------------- |

| `list` | 双向链表 |
| ------ | -------- |

| `deque` | 段式数组 + 中央控制块 |
| ------- | --------------------- |

| `set/map` | 平衡二叉搜索树（红黑树） |
| --------- | ------------------------ |

| `unordered_*` | 哈希表（开放地址或链地址法） |
| ------------- | ---------------------------- |

| `stack/queue` | 通常是基于 `deque` 封装 |
| ------------- | ----------------------- |

#### 4.2.6利用迭代器删除元素会发生什么？⭐⭐⭐⭐

示例：

```
cppCopyEditfor (auto it = v.begin(); it != v.end(); ) {
    if (*it == val) it = v.erase(it);  // 返回下一个有效迭代器
    else ++it;
}
```

**正确做法**：用 `erase` 的返回值更新迭代器，防止失效。

**为什么不能直接 `++it`？**：`erase(it)` 会让当前迭代器失效，继续使用是**未定义行为**。

`list.erase(it)` 删除当前元素不会影响其他迭代器，`vector.erase(it)` 会让**之后所有元素的迭代器失效**。



#### 4.2.7 map是如何实现的，查找效率是多少⭐⭐⭐⭐⭐



**底层结构**：红黑树（self-balancing BST）。

**特点**：

- 自动按 key 排序；
- 插入、删除、查找：**时间复杂度为 `O(log n)`**；
- 每个节点包含 `<key, value>` 对；
- 遍历是**有序的**。

若不需要有序访问，可用 `unordered_map`（哈希表）查找更快（平均 `O(1)`，最坏 `O(n)`）。



#### 4.2.8几种模板插入的时间复杂度 ⭐⭐⭐⭐⭐

| 容器            | 插入位置  | 时间复杂度    | 备注                    |
| --------------- | --------- | ------------- | ----------------------- |
| `vector`        | 尾部      | `O(1)` (摊还) | 若触发扩容则为 `O(n)`   |
| `vector`        | 中间/头部 | `O(n)`        | 元素右移                |
| `list`          | 任意位置  | `O(1)`        | 不包括查找迭代器时间    |
| `deque`         | 首尾      | `O(1)`        | 内部分段控制            |
| `map/set`       | 插入      | `O(log n)`    | 红黑树维持平衡          |
| `unordered_map` | 插入      | 平均 `O(1)`   | 最坏 `O(n)`（哈希冲突） |









### 16.C++多线程机制

#### 2. 创建线程

获得id

```c++
pthread_t pthread_self(void)
```

![image-20250716163210044](assets/image-20250716163210044.png)



![image-20250716163357652](assets/image-20250716163357652.png)

![image-20250716164651679](assets/image-20250716164651679.png)

![image-20250716163537373](assets/image-20250716163537373.png)

#### 3. 线程退出：

```c++
pthread_exit()//不是释放拟内存空间
```

![image-20250716164744685](assets/image-20250716164744685.png)

```
//县城回收

pthread_join()

```

![image-20250716165126121](assets/image-20250716165126121.png)





#### 4. join（）

#include<thread>

```c++
//创建线程
void func(std::string str)
{
    std::cout<< <<endl;

}

int main()
{
	std::thread thread1(func，“dello”);
	
	//主程序等待子线程  子现场还在运行就阻塞 运行完回收
	thread1.join();
    //分离子线程和主线程   detach()
    //thread1.detch()  //主线程结束后   子线程可以持续在后台运行
	

}
```



![image-20250716170733039](assets/image-20250716170733039.png)

```c++
//创建线程
void func(std::string str)
{
    std::cout<< <<endl;

}

int main()
{
	std::thread thread1(func，“dello”);
	
	//主程序等待子线程  子现场还在运行就阻塞 运行完回收
	thread1.join();
    //分离子线程和主线程   detach()
    //thread1.detch()  //主线程结束后   子线程可以持续在后台运行  一般不会使用
	

}
```

#### 5. joinable

```c++
//创建线程
void func(std::string str)
{
    std::cout<< <<endl;
	return ;
}

int main()
{
	std::thread thread1(func，“dello”);
	
	//先判断是否可以使用joinable() 
	bool ret=thread1.joinable();
	if(ret)
    {
		thread1.join(); //是阻塞的  不会任务切换
    	}
	

}
```

![image-20250716171307601](assets/image-20250716171307601.png)



#### 6. 互斥锁  m

![image-20250716172216915](assets/image-20250716172216915.png)



#### 7. lock_guard(); 



lock_guard();  

```c++
只有自动加锁      自动解锁
```



构造函数被调用时 该互斥量会自动被锁定，

当析构函数被调用是，该互斥量会被自动解锁

std::lock_guard对象不能复制或移动，因此他只能在局部作用域中使用。

```c++
std::lock_guard<std::mutex> lg(mtx)
    //mu.lock()
    //mu.unlock()
```

具体分析如下：

```c++
cppCopyEditfor (int i = 0; i < 10000; i++) {
    std::lock_guard<std::mutex> lg(mtx);  // 每次循环都会构造一个 lock_guard
    shared_data++;
}
```

- 这行定义的 `lg` 是一个**局部变量**，其**生命周期从声明位置开始，到所在的作用域末尾为止**；
- 在这里，`lg` 的作用域是 `for` 循环的花括号 `{}`，也就是每次迭代执行一次，然后自动析构，**自动释放锁**；
- 换句话说：**每次 `for` 循环迭代都会加锁、执行 `shared_data++`、然后自动解锁**。



![image-20250716172913027](assets/image-20250716172913027.png) 

![image-20250716173050730](assets/image-20250716173050730.png)



#### 8. 🔍unique_lock(); 

```c++
互斥量封装类： 主要特点是可以对互斥量进行更加零花的管理，包括延迟加锁、条件变量、超时等
```

```c++
for (int i = 0; i < 10000; i++) {
    std::unique_lock<std::mutex> lg(mtx); //枷锁的时候可能会阻塞  可以设置超时
    //lg.lock()  lg(mtx,defer_lock)
    // lg.try_lock_for(std::chrono::secounds(5))超时时间5S钟  我只等待一段时间 超时就返回
    //try_lock_untile()传一个时间点  不常用
    shared_data++;
}
```





![image-20250716194802283](assets/image-20250716194802283.png)

#### 9. 🔍  call_once与其使用场景（只能在线程里面使用）

是C++11标准库中的以哦个函数，用于确保某个函数只会被调用一次，

一个类  全局只需要创建一个对象就可以了，不如日志；-》单例模式

```c++
//禁用拷贝构造和等于号
Class Log
{
public:
Log(const Log&log)=delet
   Log  &operator =(const Log&log)=delet

    static log& GetInstance()
{
    //Static Log log;//全局静态log
    
    std::call_once(once,init)//保证同一时间只有一个调用
    return log
}

    static void init()
    {
        if(!log)log=newLog;
    }
    
    private:
    static std::once_flage once;c  
}



```



![image-20250716200940932](assets/image-20250716200940932.png)

单例模式 多个线程同时调用同一个全局函数  同一个里面的变量，可能会有多次创建，



![image-20250716201123208](assets/image-20250716201123208.png)

![image-20250716201624345](assets/image-20250716201624345.png)





✅   call_once与互斥锁的区别

✅ 一句话区别：

> `std::mutex` 用来**保护一段代码块在多个线程中互斥执行**，而
>  `std::call_once` 用来**确保某个函数只被执行一次**（通常用于线程安全的**一次性初始化**）。

🧩 详细对比：call_once vs mutex

| 比较项       | `std::mutex`                           | `std::call_once`                       |
| ------------ | -------------------------------------- | -------------------------------------- |
| ✅ 功能       | 多线程互斥执行某段代码                 | 多线程下**只执行一次**某段代码         |
| ✅ 使用方式   | 加锁、解锁（推荐用 `std::lock_guard`） | 配合 `std::once_flag` 使用             |
| ✅ 调用次数   | 每次都可以加锁                         | 无论多少线程调用，指定函数只会执行一次 |
| ✅ 适用场景   | 多线程保护共享资源（如读写变量）       | 单例模式、延迟初始化、全局变量初始化   |
| ✅ 性能开销   | 相对较大（每次都要加锁）               | 更轻量，仅执行一次，之后线程直接跳过   |
| ✅ 线程安全性 | 需要正确管理锁（可能死锁）             | 自动线程安全                           |
| ✅ 代码示例   | `lock_guard<mutex> lock(mtx);`         | `call_once(flag, init_func);`          |

📌 示例代码对比：

🔹 `std::mutex` 示例：保护资源

```
cppCopyEditstd::mutex mtx;
int shared_data = 0;

void add() {
    std::lock_guard<std::mutex> lock(mtx);
    shared_data++;  // 所有线程都执行，但互斥
}
```

🔹 `std::call_once` 示例：只执行一次化

```
cppCopyEditstd::once_flag flag;
int config = 0;

void init() {
    config = 42;  // 只初始化一次
}

void load() {
    std::call_once(flag, init);  // 多线程调用但 init 只执行一次
}
```

⚠️ 注意事项

- `std::call_once` 常用于单例模式、懒汉式初始化：

  ```
  cppCopyEditstatic MySingleton& getInstance() {
      static std::once_flag flag;
      static MySingleton* instance = nullptr;
      std::call_once(flag, []() {
          instance = new MySingleton();
      });
      return *instance;
  }
  ```

- 不要用 `mutex` 来代替 `call_once` 实现“只执行一次”，会导致多线程下性能低、逻辑复杂、甚至死锁。

✅ 总结

| 你想要的功能                               | 用什么？      |
| ------------------------------------------ | ------------- |
| 多线程都能执行，但不能同时执行同一段代码   | ✅ `mutex`     |
| 多线程只允许有一个执行一次（一次性初始化） | ✅ `call_once` |

####  10 . condiation_var 条件变量  （生产者消费者模型）

有点像poll唤醒机制（ 生产者唤醒消费者执行）

是的，**`std::condition_variable` 在 `wait(lock)` 时会自动释放锁并进入等待状态**，这正是它的核心机制。

```c++
condition_variable
mutex
//在需要等待条件变量的地方
 unique_lock()
condition_variable::wait()//阻塞
condition_variable::wait_for()
condition_variable::wait_until()
    
//在其他线程中需要通知等待的线程环形   
condation_variable::notify_onec;
condation_variable::notify_all();

```



```c++
condition_variable::wait(lock,true/false)//阻塞   true 不阻塞   false阻塞等待
```

  ![image-20250716204619204](assets/image-20250716204619204.png)









![image-20250716204122305](assets/image-20250716204122305.png)

![image-20250716203712036](assets/image-20250716203712036.png)

std::condition_variable cv;
std::mutex mtx;

std::unique_lock<std::mutex> lock(mtx);
cv.wait(lock);  // ✅ 自动释放锁，等待条件被唤醒后再自动重新加锁

```c++
std::mutex mtx;
std::condition_variable cv;
bool ready = false;

void worker() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, [] { return ready; });  // 自动释放锁并等待
    std::cout << "Worker proceeds!\n";
}

void notifier() {
    std::this_thread::sleep_for(std::chrono::seconds(1));
    {
        std::lock_guard<std::mutex> lock(mtx);
        ready = true;
    }
    cv.notify_one();  // 唤醒 worker
}

```

![image-20250716210707517](assets/image-20250716210707517.png)



✅ **问题 1：为什么 `for` 循环里还有一对 `{}`？**

这对花括号的作用是：

> **限定 `std::unique_lock<std::mutex> lock(mtx);` 的作用域**。

🔍 原因：

- `unique_lock` 是 RAII 风格的锁：进入作用域时加锁，离开作用域时自动解锁。
- 所以这对 `{}` 保证：
  - `queue.push(i)` 和 `cv.notify_one()` 时，锁是持有状态；
  - 一旦 `{}` 结束，`lock` 被析构，**自动释放互斥锁**；
  - 然后再 `sleep_for(...)`，此时锁已经释放，方便其他线程运行。

✅问题 2：`g_cv.notify_one()` 是立即唤醒等待线程吗？**

**不是立即执行，而是立即通知**，被通知的线程需要**等待锁释放后**才能继续执行。

🔍 执行流程简图：

1. `Producer` 线程执行到 `notify_one()`，发出“有新数据啦”的信号；
2. **此时锁还没释放**（还在花括号 `{}` 内）；
3. 等到锁随着 `lock` 离开作用域被释放；
4. 被唤醒的消费者线程尝试获得锁（成功后）；
5. 才真正进入继续执行（如 `wait()` 后面的逻辑）。







notify_once()  🔧 发生了什么（底层流程）：

当你调用：

```
cpp


CopyEdit
cv.notify_one();
```

背后大致会发生以下几个步骤：

1. **条件变量检查是否有线程在 wait() 上阻塞**；
2. 如果有，**唤醒其中一个线程**（是哪个线程由操作系统调度决定）；
3. 被唤醒的线程会尝试重新**获取与 `wait()` 关联的互斥锁**；
4. 如果获取到锁，线程继续执行 `wait()` 之后的代码；
5. 如果锁还被别人占着，它会继续等待锁释放。

#### 11.  跨平台线程池  生产者消费者模型



![image-20250716212712666](assets/image-20250716212712666.png)

 emplace_back() 在原地直接构造一个新变量，不会进行      push_back()->进行拷贝构造

 

这是一个非常专业的问题，**在实现线程池时**，你到底应该用 `emplace_back()` 还是 `push_back()`，要看你如何创建线程对象的 —— **两者都可以用，但推荐 `emplace_back()`，更高效也更自然**。

✅ 先说结论：

> 在线程池中添加线程对象时，一般使用 `std::vector<std::thread>::emplace_back(...)`，因为它：
>
> - **直接在容器内部原地构造线程对象**；
> - 避免了不必要的临时对象拷贝或移动；
> - 更符合 C++11 的推荐写法。

✅ 详细解释：`emplace_back` vs `push_back`

| 方法              | 用法                                       | 是否推荐在线程池中用    |
| ----------------- | ------------------------------------------ | ----------------------- |
| `push_back(t)`    | 你**先创建一个线程对象**，再传进去         | ⚠️ 额外构造+移动，不推荐 |
| `emplace_back(f)` | 你直接把构造线程需要的参数传进去（如函数） | ✅ 推荐，高效原地构造    |

`std::queue<T>` 的 `front()` 返回的是 **左值引用（lvalue reference）**，也就是：

```
cppCopyEditT& front();
const T& front() const;
```

✅ 所以答案是：**左值**

![image-20250716215118142](assets/image-20250716215118142.png)



```c++
#include <vector>    // 用于存储线程
#include <thread>    // 线程相关功能
#include <queue>     // 任务队列
#include <functional> // 函数对象
#include <mutex>     // 互斥锁
#include <condition_variable> // 条件变量

class ThreadPool {
public:
    // 构造函数，创建指定数量的工作线程
    ThreadPool(size_t numThreads) : stop(false) {  // stop标志初始化为false
        for (size_t i = 0; i < numThreads; ++i) {  // 创建numThreads个线程
            threads.emplace_back([this] {  // 每个线程执行以下lambda函数
                while (true) {  // 线程持续工作
                    std::function<void()> task;  // 用于存储从队列取出的任务

                    {  // 加锁区域开始
                        std::unique_lock<std::mutex> lock(mtx);  // 获取互斥锁
                        
                        // 等待条件变量通知，当任务队列不为空或线程池停止时继续
                        condition.wait(lock, [this] {
                            return !tasks.empty() || stop;
                        });

                        // 如果线程池已停止且任务队列为空，则线程退出
                        if (stop && tasks.empty()) {
                            return;
                        }

                        // 从任务队列取出第一个任务
                        task = std::move(tasks.front());
                        tasks.pop();
                    }  // 加锁区域结束，自动释放锁

                    task();  // 执行任务
                }
            });
        }
    }

    // 析构函数，停止所有线程
    ~ThreadPool() {
        {  // 加锁区域开始
            std::unique_lock<std::mutex> lock(mtx);
            stop = true;  // 设置停止标志
        }  // 加锁区域结束，自动释放锁
        
        condition.notify_all();  // 通知所有等待的线程
        
        // 等待所有线程完成
        for (auto& thread : threads) {
            thread.join();  // 等待线程结束
        }
    }

    // 向线程池添加任务
    template<class F, class... Args>
    void enqueue(F&& f, Args&&... args) {
        // 使用bind将函数和参数绑定为一个无参函数对象
        std::function<void()> task = std::bind(std::forward<F>(f), std::forward<Args>(args)...);
        
        {  // 加锁区域开始
            std::unique_lock<std::mutex> lock(mtx);
            tasks.emplace(std::move(task));  // 将任务加入队列
        }  // 加锁区域结束，自动释放锁
        
        condition.notify_one();  // 通知一个等待的线程有新任务
    }

private:
    std::vector<std::thread> threads;  // 存储工作线程
    std::queue<std::function<void()>> tasks;  // 任务队列
    std::mutex mtx;  // 保护任务队列的互斥锁
    std::condition_variable condition;  // 用于线程间通信的条件变量
    bool stop;  // 线程池停止标志
};




```





用单例模式和智能指针实现的线程池

下面我将实现一个使用单例模式、智能指针和call_once的线程池，并逐行解释代码，最后给出main函数的使用示例。



```c++

#include <iostream>
#include <thread>  
#include <functional>  //函数模板  bind
#include <vector>
#include <queue>
#include <mutex>
#include <condition_variable>
#include <memory>  //unique_ptr shared_ptr
#include <chrono>  //时间   
using namespace std;
//手撕线程池   
/*
生产者消费者模型    
线程池有多个创建的线程  等待任务队列里有任务就去执行，这个有谁执行是不确定的 
生产者负责向任务队列里添加任务，消费者负责从任务队列里取任务执行


线程池是全局唯一一个unique_ptr    只初始化一次once_call();，后续调用仍然还是同一个线程池 ，所以需要用单例模式  
*/


class ThreadPool { 
public:
    //首先  创建单例模式  （在单例模式中创建线程池 保障只初始化一次  返回创建的线程池对象指针）
    // 获取线程池单例对象（懒汉式单例，线程安全）
    static ThreadPool& getInstance(size_t number_thread=4)  //static保证外部main可以调用
    {
        static once_flag flag;  //once_flag是一个标志位，表示是否已经执行过某个操作
        static unique_ptr<ThreadPool> instance;  //创建一个空的  使用unique_ptr管理ThreadPool的生命周期

        call_once(flag,[number_thread](){
            instance.reset(new ThreadPool(number_thread));  //创建线程池实例 释放当前对象并管理新的对象

        });
        return *instance;  //返回线程池实例的引用
    }

    //禁用拷贝构造
    ThreadPool(const ThreadPool& )=delete;
    //禁用赋值构造
    ThreadPool& operator=(const ThreadPool& )=delete;


    
    //析构函数
    ~ThreadPool()
    {
        {
            unique_lock<mutex> lock(m_mutex);
            stop = true;  //设置停止标志
        }

        cond.notify_all();  //通知所有等待的线程，线程池即将停止
        for(auto &t: threads)  //遍历线程池中的所有线程
        {
            if(t.joinable())  //如果线程可连接
            {
                t.join();  //等待线程结束
            }
        }
        cout << "ThreadPool destroyed" << endl;  //输出线程池销毁信息
    }

    //添加任务队列函数
    template<class T, class ...Args>
    void addtask(T&&task,Args&&...args)
    {
        function<void()> func=bind(forward<T>(task),forward<Args>(args)...);  //使用bind将任务和参数绑定到一个函数对象上
        {
            //加锁区   要修改等待队列了
            unique_lock<mutex> lock(m_mutex);  //使用unique_lock来管理互斥锁的生命周期
            tasks.emplace(move(func));  //将函数对象添加到任务队列中

        }
        cond.notify_one();  //通知一个等待的线程，任务队列中有新任务
    }
    private:

    
    //默认构造函数  explicit 的作用是防止构造函数或转换函数被隐式调用，从而避免意外的隐式类型转换。
    /*
        Test t1(10);  // 正确：显式调用构造函数
       Test t2 = 10;  // 错误：隐式调用被禁止，因为构造函数是 explicit

    */
    explicit ThreadPool(size_t number_thread): stop(false) //在线程池里面创建线程 几个线程
    {
        for(int i=0;i<number_thread;i++)
        {
            threads.emplace_back([this](){
                while(1)
                {
                    //获取锁 
                    unique_lock<std::mutex> lock(m_mutex);  //使用unique_lock来管理互斥锁的生命周期
                    cond.wait(lock,[this](){
                        return !tasks.empty() || stop;  //等待条件变量，直到任务队列不为空或线程池停止
                    });
                    if(stop && tasks.empty())  //如果线程池停止且任务队列为空，退出线程
                    {
                        return;
                    }
                    //从任务队列中取出任务
                    function<void()>  task(move(tasks.front()));  //这里创建了一个新的 std::function<void()> 对象 task，并将 tasks.front() 的内容移动到它中。
                    tasks.pop();
                    lock.unlock();  //解锁互斥锁，允许其他线程访问任务队列
                    //执行任务
                    task();  //执行任务函数对象
                 
                }



            });
        }

    }


    mutex m_mutex;  //互斥锁
    condition_variable cond;  //条件变量
    queue<function<void()>> tasks;  //任务队列，存储函数对象
    vector<thread> threads;  //线程池，存储工作线程
    bool stop;  //标志线程池是否停止
};

// 示例任务函数
void func(int id) {
    std::cout << "任务 " << id << " 开始执行，线程ID: " 
              << std::this_thread::get_id() << std::endl;

    std::this_thread::sleep_for(std::chrono::seconds(1)); // 模拟耗时任务

    std::cout << "任务 " << id << " 完成" << std::endl;
}

int main() {

    ThreadPool &pool=ThreadPool::getInstance(5);//获取线程池单例对象  只有首次会被构建
    std::this_thread::sleep_for(std::chrono::seconds(5));
    for(int i=10;i>0;i--)
    {
        pool.addtask(func,i);  //添加任务到线程池
        std::cout << "已添加任务 " << i << std::endl;
    }

      // 主线程等待部分任务完成（实际情况应使用同步机制）
    std::cout << "主线程等待任务完成..." << std::endl;
    std::this_thread::sleep_for(std::chrono::seconds(10));

  // 再次获取相同的线程池实例（证明单例有效）
    ThreadPool& samePool = ThreadPool::getInstance();

    samePool.addtask([](){
        std::cout << "再次添加任务" << std::endl;
    });

    std::this_thread::sleep_for(std::chrono::seconds(5));
    std::cout << "程序结束" << std::endl;


    return 0;
}


```

行解释

单例模式实现部分

1. `static ThreadPool& getInstance(size_t numThreads = 4)` - 静态方法获取单例实例
2. `static std::once_flag onceFlag` - 保证线程安全的once_flag
3. `static std::unique_ptr<ThreadPool> instance` - 使用智能指针管理单例实例
4. `std::call_once(onceFlag, [numThreads]() {...})` - 保证线程安全地初始化单例

线程池核心部分

1. `std::vector<std::thread> threads` - 存储工作线程的容器
2. `std::queue<std::function<void()>> tasks` - 任务队列
3. `std::mutex mtx` - 保护共享数据的互斥锁
4. `std::condition_variable condition` - 线程间通信的条件变量
5. `bool stop` - 线程池停止标志

任务处理逻辑

1. `threads.emplace_back([this] {...})` - 创建工作线程
2. `condition.wait(lock, [this] {...})` - 等待任务或停止信号
3. `task = std::move(tasks.front())` - 从队列获取任务
4. `task()` - 执行任务

任务添加逻辑

1. `auto task = std::make_shared<std::function<void()>>(...)` - 使用智能指针包装任务
2. `tasks.emplace([task]() { (*task)(); })` - 将任务加入队列
3. `condition.notify_one()` - 通知一个工作线程

Main函数使用说明

1. `ThreadPool& pool = ThreadPool::getInstance(4)` - 获取有4个线程的线程池实例
2. `pool.enqueue(taskFunction, i)` - 添加任务到线程池
3. 多次调用`getInstance()`返回的是同一个实例
4. 程序结束时自动清理线程池资源

✅ 总结：`unique_ptr` + `call_once` 单例模式优势

| 特性       | 说明                                      |
| ---------- | ----------------------------------------- |
| 延迟初始化 | 第一次用到时才创建对象                    |
| 线程安全   | `std::call_once` 保证只初始化一次         |
| 无内存泄漏 | `unique_ptr` 自动释放对象                 |
| 禁止拷贝   | `unique_ptr` 语义匹配单例“不允许复制”设计 |
| 易于维护   | 不需要手动 delete，析构自动完成           |



#### 12  .async future packaged_task  promise  异步并发  获得线程的返回值

`promise` / `future` 一般用于 **一次性** 通信。   

##### 1. async  future

![image-20250717112818380](assets/image-20250717112818380.png)



![image-20250717113013558](assets/image-20250717113013558.png)

![image-20250717113251218](assets/image-20250717113251218-1752723172713-1.png)

```c++
future<int> future_res=async(launch::asunc,func); //当func传给async时func已经在后台运行可以看成一个线程  不阻塞  在后台运行，返回的结果存在 future_res  。

cout<< future_res.get()<<endl;    //可能早就运行完了  取结果，没运行完就等待结果  也就是说，它会**“死等”**，一直等 func 执行完成。



```

这个例子中，我们使用std:async函数异步执行了一个耗时的计算，这个计算可以在另一个
线程中执行，不会阻塞主线程。同时，我们也避免了手动创建线程和管理线程的麻烦。



```

```



在 C++ 中，`std::async` 和 `std::future` 是用于实现 **异步任务调度与结果获取** 的标准库组件。下面我将具体解释它们的含义、使用方式、底层原理，以及与线程池的对比。





一、基本概念

1. `std::async`

`std::async` 会启动一个异步任务，可能在新线程中运行，也可能在调用线程中延迟运行（lazy evaluation），由策略决定。

```
cppCopyEdit#include <iostream>
#include <future>

int compute(int x) {
    return x * x;
}

int main() {
    std::future<int> fut = std::async(std::launch::async, compute, 10);
    // 可做其他事情
    int result = fut.get();  // 阻塞，直到compute完成并返回结果
    std::cout << "Result: " << result << std::endl;
}
```

2. `std::future`

`std::future<T>` 是一个对象，表示一个 **异步计算操作的结果**。你可以通过 `.get()` 方法来获取结果（如果结果还没出来，它会阻塞）。



##### 2.packaged_task  

![image-20250717114106929](assets/image-20250717114106929.png)

packaged_task  生成一个可调用对象

```c++
packaged_task<int()> task(func);   //包装 生成可调用对象  
auto future_res=task.get_future();  //  返回一个future对象
```

![image-20250717114645288](assets/image-20250717114645288.png)



##### 3.promise  

是一个类模板  用于在一个线程中产生之，在另一个线程中获取值，promise通常与async一起使用，用于实现异步编程

![image-20250717115037939](assets/image-20250717115037939.png)

![image-20250717115417201](assets/image-20250717115417201.png)

 

```c++
#include <iostream>
#include <thread>
#include <future>
using namespace std;

/*


1. std::promise 和 std::future
std::promise<T>：用于在一个线程中设置值

std::future<T>：用于在另一个线程中获取该值

它们之间通过一个共享状态通信。
*/
void producer(promise<int> prom)
{
    this_thread::sleep_for(chrono::seconds(1));
    prom.set_value(10);  //设置promise的值  只设置一次  第二次会抛出异常
}

void comsumer(future<int> fu)
{
    int ret=fu.get(); //会阻塞等待promise设置值   这 并不是“只读”操作，而是“取一次就没了”的操作。只能调用一次！ 调第二次就会抛出异常：
    cout<<"ret:"<<ret<<endl;
}

int main()
{
    promise<int> prom;
    future<int> fu=prom.get_future();

    thread t1(producer , std::move(prom));
    thread t2(comsumer, std::move(fu));

    t1.join();
    t2.join();

return 0;

}


```



2. std::async
std::async 创建一个任务，该任务在后台自动创建线程或延迟调用。

返回一个 std::future<T>，通过它可以获取任务的返回值。

```c++

#include <iostream>
#include <future>

int task(int x)
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
    return x * 2;
}

int main()
{
    std::future<int> fut = std::async(std::launch::async, task, 21);
    std::cout << "等待结果...\n";
    std::cout << "结果是: " << fut.get() << std::endl;
}
```

三者结合使用：promise + future + async

我们可以结合使用 `promise` 与 `future`，并通过 `async` 创建异步任务，完成一个线程间数据传递+异步计算的场景：

✅ 示例：main 线程准备数据 -> 异步线程处理 -> 返回处理结果

```c++
#include <iostream>
#include <future>
#include <thread>

// 异步任务，等待主线程传来的值
void worker(std::future<int> fut, std::promise<int> prom)
{
    int x = fut.get(); // 从主线程接收数据  死等
    int result = x * 10;
    prom.set_value(result); // 将结果返回给主线程
}

int main()
{
    std::promise<int> input_prom;
    std::future<int> input_fut = input_prom.get_future();

    std::promise<int> result_prom;
    std::future<int> result_fut = result_prom.get_future();

    // 异步任务（也可以用 std::thread）
    std::thread t(worker, std::move(input_fut), std::move(result_prom));

    
    
    // 主线程向异步线程发送数据
    input_prom.set_value(7);

    // 主线程获取异步线程返回的结果
    int result = result_fut.get();
    std::cout << "处理结果: " << result << std::endl;

    t.join();
}

```

因为这两个类型：

```
cppCopyEditstd::future<int>
std::promise<int>
```

都**不能被拷贝（copy）**，只能被**移动（move）**！

✅ 1. `std::future` 和 `std::promise` 是 **不可拷贝的（deleted copy constructor）**

```
cppCopyEditstd::future<int> f1 = f2;  // ❌ 编译错误！
std::promise<int> p1 = p2; // ❌ 编译错误！
```

它们内部维护着同步状态和共享状态指针，为了避免误用和数据冲突，标准库禁止你拷贝它们。





总结对比：

| 机制           | 用途                         | 特点                          |
| -------------- | ---------------------------- | ----------------------------- |
| `std::promise` | 设置异步结果（由生产者线程） | 与 `future` 一对一绑定        |
| `std::future`  | 获取异步结果（由消费者线程） | `get()` 会阻塞直到有结果      |
| `std::async`   | 自动创建异步任务             | 返回一个 future；线程自动管理 |

####  13.原子操作   atomic

和互斥锁  全局共享变量



std:atomic 是C++11标准库中的一个模板类，用于实现多线程环境下的原子操作。它
提供了一种线程安全的方式来访问和修改共享变量，可以避免多线程环境中的数据竞争问
题。



![image-20250717121442696](assets/image-20250717121442696.png)

```c++
#include<atomic>
atomic<int> shared_data=0;


#include <iostream>
#include <thread>
#include <mutex>
#include <atomic>

std::atomic<int> shared_data = 0;
void func() {
    for (int i = 0; i < 100000; ++i) {
   
    shared_data++;
    }
}

int main() {
    std::thread t1(func);
    std::thread t2(func);
    t1.join();
    t2.join();
    std::cout << "shared_data = " << shared_data << std::endl;
    return 0;
}
```

![image-20250717122035760](assets/image-20250717122035760.png)

🧱 二、常见操作

✅ 原子读写

```
cppCopyEditstd::atomic<int> x(0);
x = 5;              // 原子写
int y = x.load();   // 原子读
```

✅ 原子加减

```
x++;                // 原子加一
x.fetch_add(2);     // 加2，返回修改前的值
x.fetch_sub(1);     // 减1，返回修改前的值
```

✅ compare_exchange

这是 **无锁编程核心操作**，实现类似乐观锁的机制：

```
int expected = 5;
if (x.compare_exchange_strong(expected, 10)) {
    // 成功：x 原本是 5，现在改成 10
} else {
    // 失败：x 当前不是 5，expected 被更新为 x 的当前值
}
```

🚥 三、memory_order 内存序（可选高级用法）

- `memory_order_relaxed`: 最弱顺序，仅保证原子性。
- `memory_order_acquire / release`: 用于单向同步。
- `memory_order_seq_cst`: 默认，最强顺序，保证所有线程对原子变量的操作具有全序一致性。

```
cppCopyEditx.store(1, std::memory_order_release);
int y = x.load(std::memory_order_acquire);
```

📌 小结

| 操作                 | 方法或语法                     |
| -------------------- | ------------------------------ |
| 原子读写             | `.load()`, `.store()`          |
| 原子加/减            | `.fetch_add()`, `.fetch_sub()` |
| 自增自减             | `++x`, `--x`                   |
| CAS 操作（比较交换） | `.compare_exchange_strong()`   |

#### 14.读写锁

是一种同步机制，与i徐多个读操作同时进行，写操作必须独立访问，通过分离读写操作提高并发性能，是用于读多写少的场景

无读锁时，读写可共享 

无写无读时，才可加写锁

优先级策略优化问题







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





# 上传git



```c
 cd D:/

cd a资料/研究生资料/找工作/C++知识/

 git add resum_.md
 git commit -m "2025/7/17"

git push -u origin master




```



## 1. GIT 指令

```c
//gloabl 关键字表示这台机器上的所有GIT仓库都会使用这个配置 ，也可以为某个仓库制定不同的仓库地址
git config --global  user.name "JKX-yy"
git config --global  user.email "983008880@qq.com"
    
```



## 4

## 4. 创建版本库



创建一个仓库，可以简单理解为一个目录，里面的所有文件都可以被GIT管理起来，每个文件的修改删除 Git都能跟踪，一边任何时刻都可以追踪历史，

```c++
mkdir learngit 
cd learngit
pwd
```



初始化为可管理的仓库,`.git`的目录，这个目录是Git来跟踪管理版本库的，没事千万不要手动修改这个目录里面的文件，不然改乱了，就把Git仓库给破坏了.

```c
git init 
// ll  会出现一个隐藏目录  .git 
    //ls -ah 
```



提交本地仓库





```c
//第一步，用命令git add告诉Git，把文件添加到仓库
git add ..  //(可以多次add)
//第二步，用命令git commit告诉Git，把文件提交到仓库：    
git commit -m "说明"
    
git push -u origin master
```



因为`commit`可以一次提交很多文件，所以你可以多次`add`不同的文件，比如：

```c
$ git add file1.txt
$ git add file2.txt file3.txt
$ git commit -m "add 3 files."

```

## 5.时光穿梭机



使用git add提交后  继续修改文件，

### 1.查看状态

```c
git  status //查看仓库状态
    
    
$ git status
    /*
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

	modified:   readme.txt  //已经修改

no changes added to commit (use "git add" and/or "git commit -a")*/

```

### 2. 查看具体哪里修改了difference

`git diff`顾名思义就是查看difference，显示的格式正是Unix通用的diff格式，可以从上面的命令输出看到，我们在第一行添加了一个`distributed`单词。

但是我的是md的文件调用后并没有输出

```c
git diff 
    
    
```

```
$ git diff readme.txt 
    
diff --git a/readme.txt b/readme.txt
index 46d49bf..9247db6 100644
--- a/readme.txt
+++ b/readme.txt
@@ -1,2 +1,2 @@
-Git is a version control system.
+Git is a distributed version control system.
 Git is free software.

```

### 1.版本退回

Git也是一样，每当你觉得文件修改到一定程度的时候，就可以“保存一个快照”，这个快照在Git中被称为`commit`。一旦你把文件改乱了，或者误删了文件，还可以从最近的一个`commit`恢复，然后继续工作，而不是把几个月的工作成果全部丢失。



查看提交的历史记录

```c
git log
  输出 版本号    
    
```

`git log`命令显示从最近到最远的提交日志，我们可以看到3次提交，最近的一次是`append GPL`，上一次是`add distributed`，最早的一次是`wrote a readme file`。

如果嫌输出信息太多，看得眼花缭乱的，可以试试加上`--pretty=oneline`参数：



```
$ git log
commit 1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master)//head表示当前版本   最新版本
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:06:15 2018 +0800

    append GPL

commit e475afc93c209a690c39c13a46716e8fa000c366
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:03:36 2018 +0800

    add distributed

commit eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 20:59:18 2018 +0800

    wrote a readme file

```

//如果关机了  后悔了 想穿越回来可以  git reflog  用来记录你的每一次命令

```c
git reflog
    
```

```
$ git reflog
adf82ef (HEAD -> master) HEAD@{0}: commit: xx
7c3fa38 HEAD@{1}: commit: 2025/7/17
ac363eb (origin/master) HEAD@{2}: commit: 线程池2
52ee54e HEAD@{3}: commit: 线程池
77c6ce1 HEAD@{4}: commit (initial): c++ 面试可能会问的问题

```

每提交一个新版本，实际上Git就会把它们自动串成一条时间线。如果使用可视化工具查看Git历史，就可以更清楚地看到提交历史的时间线：



上一个版本就是`HEAD^`，上上一个版本就是`HEAD^^`，当然往上100个版本写100个`^`比较容易数不过来，所以写成`HEAD~100`。

```c
git reset --hard  HEAD^  //回退到上一个版本 HEAD~n
```

从21世纪穿越到19世纪  但是只要穿梭机不关（命令行窗口还没有被关掉），可以找到之前的commit id  

版本号没必要写全，前几位就可以了，Git会自动去找。当然也不能只写前一两位，因为Git可能会找到多个版本号，就无法确定是哪一个了。

```c
git reset --hard 1094a
```





🔧 三个核心概念先解释清楚

| 名词                        | 含义                                          |
| --------------------------- | --------------------------------------------- |
| `HEAD`                      | 当前所指向的提交（你正在哪个 commit 上）      |
| `index`（暂存区）           | 用 `git add` 加进去的内容（准备要提交的内容） |
| 工作区（Working Directory） | 你当前看到的项目文件（你在编辑器里改的内容）  |







示例场景（你刚做了两次提交）

假设你有一个 `file.txt` 文件，依次执行了以下操作：

```
bashCopyEditecho "first line" > file.txt
git add file.txt
git commit -m "first commit"

echo "second line" >> file.txt
git add file.txt
git commit -m "second commit"
```

现在：

- `HEAD` 指向 **第二次提交**

- `file.txt` 内容是：

  ```
  first line
  second line
  ```

👇 假设现在你执行：

```

git reset --soft HEAD~1
```

| 区域   | 说明                              |
| ------ | --------------------------------- |
| HEAD   | ⬅️ 回退到第一次提交                |
| index  | ✅ 暂存区 **保留第二次提交的内容** |
| 工作区 | ✅ 你看到的文件内容 **还是有两行** |



👉 意思是：**只是撤回了提交，文件改动还在，并且还在 `add` 状态**

你可以直接：

```

git commit -m "修改过的提交信息"
```

来改写刚才的提交。

👇 如果你执行：

```
bash

git reset --mixed HEAD~1
```

| 区域   | 说明                                        |
| ------ | ------------------------------------------- |
| HEAD   | ⬅️ 回退到第一次提交                          |
| index  | ❌ 暂存区清空（也就是撤回 `git add` 的效果） |
| 工作区 | ✅ 文件内容还是两行                          |



👉 意思是：你撤回了提交，也撤回了 `add`，但改动还在文件里
 你需要重新 `git add file.txt` 才能再提交

👇 如果你执行：

```

git reset --hard HEAD~1
```

| 区域   | 说明                                              |
| ------ | ------------------------------------------------- |
| HEAD   | ⬅️ 回退到第一次提交                                |
| index  | ❌ 暂存区恢复到第一次提交的状态（不包含第二行）    |
| 工作区 | ❌ 你看到的文件也只剩下 `first line`（第二行丢失） |



👉 意思是：**提交、暂存、工作区，全都退回去了，第二次提交的改动丢了**



### 2.工作区和暂存区



工作区是.git 存在的文件夹  



。git(是版本库)  包含

​	stage 暂存区

​	master 创建的一个分支（并由一个指向master的指针）



![git-repo](https://liaoxuefeng.com/books/git/time-travel/working-stage/repo.png)

```c
git  add //添加到暂存区  stage
```

```c
git commit //将暂存区提交到分支
```

所以，`git add`命令实际上就是把要提交的所有修改放到暂存区（Stage），然后，执行`git commit`就可以一次性把暂存区的所有修改提交到分支。

![git-stage-after-commit](https://liaoxuefeng.com/books/git/time-travel/working-stage/commit.png)

### 3.管理修改

跟踪管理的是修改   而不是文件

那怎么提交第二次修改呢？你可以继续`git add`再`git commit`，也可以别着急提交第一次修改，先`git add`第二次修改，再`git commit`，就相当于把两次修改合并后一块提交了：

第一次修改 -> `git add` -> 第二次修改 -> `git add` -> `git commit`

好，现在，把第二次修改提交了，然后开始小结。



### 4. 撤销修改

总之，就是让这个文件回到最近一次`git commit`或`git add`时的状态。

```c
git checkout -- file
```

撤销在文件区的修改

命令`git checkout -- readme.txt`意思就是，把`readme.txt`文件在工作区的修改全部撤销，这里有两种情况：

一种是`readme.txt`自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是`readme.txt`已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。





`git checkout -- file`命令中的`--`很重要，没有`--`，就变成了“切换到另一个分支”的命令，我们在后面的分支管理中会再次遇到`git checkout`命令。



Git同样告诉我们，用命令`git reset HEAD <file>`可以把暂存区的修改撤销掉（unstage），重新放回工作区：

```c
git reset HEAD file //对单个文件的操作 把 readme.txt 从 暂存区（stage/index） 移除（回到工作区），但不会还原工作区的内容（不会改动你编辑的文件），所以只是“取消 add”了这个文件。
it reset --mixed HEAD // 把所有暂存区的变更都撤销掉（即回退到 HEAD 指向的版本）， 但不会修改工作区内容（改过的文件内容还在），  也就是“取消 git add”。
```

`git reset`命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用`HEAD`时，表示最新的版本。





```c
//现在暂存区是干净的，工作区有修改
 git checkout -- readme.txt//丢弃工作区的修改 回到最近一一版的add / commit
```

场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令`git checkout -- file`。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令`git reset HEAD <file>`，就回到了场景1，第二步按场景1操作。



```
git reset HEAD file  //针对撤销add  没有完成commit

*********************下面的三种针对完成  工作区修改+add+commit****************
git reset --hard HEAD^   //完全清空  工作区+暂存区+HEAD移动
git reset --soft  HEAD^
git reset  --mixed HEAD^ 
```

🧠 总结对比表

| 命令                         | HEAD 移动 | 暂存区变化          | 工作区变化         | 备注                     |
| ---------------------------- | --------- | ------------------- | ------------------ | ------------------------ |
| `git reset HEAD <file>`      | ❌         | 将文件 unstage      | ❌ 保留修改         | 用于取消 `git add`       |
| `git reset --soft <commit>`  | ✅         | 不变（保留 staged） | ❌ 保留修改         | 改动仍在暂存区           |
| `git reset --mixed <commit>` | ✅         | 清空暂存区          | ❌ 保留修改         | 默认行为，改动仍可见     |
| `git reset --hard <commit>`  | ✅         | 清空                | ❌ 清空（还原文件） | 改动彻底丢失，请谨慎使用 |









### 5 删除文件  （本地删除了 本地仓库也得删除）

```c
rm file
```

这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，

现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令`git rm`删掉，并且`git commit`：

```c
$ git rm test.txt
rm 'test.txt'

$ git commit -m "remove test.txt"
[master d46f35e] remove test.txt
 1 file changed, 1 deletion(-)
 delete mode 100644 test.txt

```

 注意

从来没有被添加到版本库就被删除的文件，是无法恢复的！

## 6.远程仓库



1. 第1步：创建SSH Key。在用户主目录下，看看有没有.ssh目录，如果有，再看看这个目录下有没有`id_rsa`和`id_rsa.pub`这两个文件，如果已经有了，可直接跳到下一步。如果没有，打开Shell（Windows下打开Git Bash），创建SSH Key：

   

   ```c
   $ ssh-keygen -t rsa -C "youremail@example.com"
   ```

   

2. 登陆GitHub，打开“Account settings”，“SSH Keys”页面：

   然后，点“Add SSH Key”，填上任意Title，在Key文本框里粘贴`id_rsa.pub`文件的内容

### 1. 添加远程仓库

1. 创建远程仓库

   ![github-create-repo-1](https://liaoxuefeng.com/books/git/remote/add-remote/new-repo.png)

2. 在本地工作区关联远程仓库

   ```c
   
   git remote add origin git@。。。。
   ```

3. 推送 

   把本地库的内容推送到远程，用`git push`命令，实际上是把当前分支`master`推送到远程。

   ```c
   git push -u origin master
   
   ```

4. 删除远程仓库 

   如果添加的时候添加错了  像删除

   ```c
   git remote -v //查看
   ```

   

删除远程库

如果添加的时候地址写错了，或者就是想删除远程库，可以用`git remote rm <name>`命令。使用前，建议先用`git remote -v`查看远程库信息：

```c
$ git remote -v
origin  git@github.com:michaelliao/learn-git.git (fetch)
origin  git@github.com:michaelliao/learn-git.git (push)
```



然后，根据名字删除，比如删除`origin`：

```c
$ git remote rm origin
```



此处的“删除”其实是解除了本地和远程的绑定关系，并不是物理上删除了远程库。远程库本身并没有任何改动。要真正删除远程库，需要登录到GitHub，在后台页面找到删除按钮再删除。

小结

要关联一个远程库，使用命令`git remote add origin git@server-name:path/repo-name.git`；

关联一个远程库时必须给远程库指定一个名字，`origin`是默认习惯命名；

关联后，使用命令`git push -u origin master`第一次推送`master`分支的所有内容；

此后，每次本地提交后，只要有必要，就可以使用命令`git push origin master`推送最新修改；

### 2. 从远程仓库克隆



## 7.分支管理

### 1. 创建与合并分支





一开始的时候，`master`分支是一条线，Git用`master`指向最新的提交，再用`HEAD`指向`master`，就能确定当前分支，以及当前分支的提交点：

```
                  HEAD
                    │
                    ▼
                 master
                    │
                    ▼
┌───┐    ┌───┐    ┌───┐
│   │───▶│   │───▶│   │
└───┘    └───┘    └───┘
```

每次提交，`master`分支都会向前移动一步，这样，随着你不断提交，`master`分支的线也越来越长。

当我们创建新的分支，例如`dev`时，Git新建了一个指针叫`dev`，指向`master`相同的提交，再把`HEAD`指向`dev`，就表示当前分支在`dev`上：

```
                 master
                    │
                    ▼
┌───┐    ┌───┐    ┌───┐
│   │───▶│   │───▶│   │
└───┘    └───┘    └───┘
                    ▲
                    │
                   dev
                    ▲
                    │
                  HEAD
```

你看，Git创建一个分支很快，因为除了增加一个`dev`指针，改改`HEAD`的指向，工作区的文件都没有任何变化！

不过，从现在开始，对工作区的修改和提交就是针对`dev`分支了，比如新提交一次后，`dev`指针往前移动一步，而`master`指针不变：

```
                 master
                    │
                    ▼
┌───┐    ┌───┐    ┌───┐    ┌───┐
│   │───▶│   │───▶│   │───▶│   │
└───┘    └───┘    └───┘    └───┘
                             ▲
                             │
                            dev
                             ▲
                             │
                           HEAD
```

假如我们在`dev`上的工作完成了，就可以把`dev`合并到`master`上。Git怎么合并呢？最简单的方法，就是直接把`master`指向`dev`的当前提交，就完成了合并：

```
                           HEAD
                             │
                             ▼
                          master
                             │
                             ▼
┌───┐    ┌───┐    ┌───┐    ┌───┐
│   │───▶│   │───▶│   │───▶│   │
└───┘    └───┘    └───┘    └───┘
                             ▲
                             │
                            dev
```

所以Git合并分支也很快！就改改指针，工作区内容也不变！

合并完分支后，甚至可以删除`dev`分支。删除`dev`分支就是把`dev`指针给删掉，删掉后，我们就剩下了一条`master`分支：

```
                           HEAD
                             │
                             ▼
                          master
                             │
                             ▼
┌───┐    ┌───┐    ┌───┐    ┌───┐
│   │───▶│   │───▶│   │───▶│   │
└───┘    └───┘    └───┘    └───┘
```

真是太神奇了，你看得出来有些提交是通过分支完成的吗？

下面开始实战。

```c
//创建分支并进入
git checkout -b dev
git switch -c dev
//等同于
git branch dev
git checkout dev



```



```c
git branch

```

`it branch`命令会列出所有分支，当前分支前面会标一个`*`号。

然后，我们就可以在`dev`分支上正常提交，



```c
//一定要  add  commit  不然会丢失
git add

git commit

    
//切换回master
    
git checkout master 
git switch master
//合并分支
    
git merge dev
   
    
    
//合并完分支后，甚至可以删除dev分支。删除dev分支就是把dev指针给删掉，删掉后，我们就剩下了一条master分支：
git branch -d dev     
    
```





我们注意到切换分支使用`git checkout <branch>`，而前面讲过的撤销修改则是`git checkout -- <file>`，同一个命令，有两种作用，确实有点令人迷惑。



实际上，切换分支这个动作，用`switch`更科学。因此，最新版本的Git提供了新的`git switch`命令来切换分支：

创建并切换到新的`dev`分支，可以使用：

  



小结

Git鼓励大量使用分支：

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`或者`git switch <name>`

创建+切换分支：`git checkout -b <name>`或者`git switch -c <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`



### 2. 解决冲突



当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

用`git log --graph`命令可以看到分支合并图。



```c
//创建新分支  在新分支上
git switch -c fu1
echo >> "hello" hello.txt
git add 
git commit 
git switch  master
//正常merge
git merge fu1
//但是当还没有merge  接着修改hello.txt
git add
git commit 
//现在，master分支和feature1分支各自都分别有新的提交，变成了这样：


```

现在，`master`分支和`feature1`分支各自都分别有新的提交，变成了这样：

```
                            HEAD
                              │
                              ▼
                           master
                              │
                              ▼
                            ┌───┐
                         ┌─▶│   │
┌───┐    ┌───┐    ┌───┐  │  └───┘
│   │───▶│   │───▶│   │──┤
└───┘    └───┘    └───┘  │  ┌───┐
                         └─▶│   │
                            └───┘
                              ▲
                              │
                          feature1
```

这种情况下，Git无法执行“快速合并”，只能试图把各自的修改合并起来，但这种合并就可能会有冲突，我们试试看：

git merge fu1

果然冲突了！Git告诉我们，`readme.txt`文件存在冲突，必须手动解决冲突后再提交。`git status`也可以告诉我们冲突的文件：

```
$ git status
On branch master
Your branch is ahead of 'origin/master' by 2 commits.
  (use "git push" to publish your local commits)

You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   readme.txt

no changes added to commit (use "git add" and/or "git commit -a")

```

需要手动修改冲突的文件  然后保存

![image-20250717214435479](assets/image-20250717214435479.png)

用带参数的`git log`也可以看到分支的合并情况：



```
$ git log --graph --pretty=oneline --abbrev-commit
```



```c
//删除分支
git branch -d fu1
```

小结

当Git无法自动合并分支时，就必须首先解决冲突。解决冲突后，再提交，合并完成。

解决冲突就是把Git合并失败的文件手动编辑为我们希望的内容，再提交。

### 3. 分支管理策略

通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。



准备合并`dev`分支，请注意`--no-ff`参数，表示禁用`Fast forward`：

```c
$ git merge --no-ff -m "merge with no-ff" dev
```



![image-20250717214925333](assets/image-20250717214925333.png)



```c
$ git log --graph --pretty=oneline --abbrev-commit
*   e1e9c68 (HEAD -> master) merge with no-ff
|\  
| * f52c633 (dev) add merge
|/  
*   cf810e4 conflict fixed
...

```

分支策略

首先，`master`分支应该是非常稳定的，也就是仅用来发布新版本，平时不能在上面干活；

那在哪干活呢？干活都在`dev`分支上，也就是说，`dev`分支是不稳定的，到某个时候，比如1.0版本发布时，再把`dev`分支合并到`master`上，在`master`分支发布1.0版本；

你和你的小伙伴们每个人都在`dev`分支上干活，每个人都有自己的分支，时不时地往`dev`分支上合并就可以了。

 



小结

Git分支十分强大，在团队开发中应该充分应用。

合并分支时，加上`--no-ff`参数就可以用普通模式合并，合并后的历史有分支，能看出来曾经做过合并，而`fast forward`合并就看不出来曾经做过合并。



### 4.Bug分支



软件开发中，bug就像家常便饭一样。有了bug就需要修复，在Git中，由于分支是如此的强大，所以，每个bug都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。

当你接到一个修复一个代号101的bug的任务时，很自然地，你想创建一个分支`issue-101`来修复它，但是，等等，当前正在`dev`上进行的工作还没有提交：(  要离开当前分支   区master或者别的分支创建issue-101分支修复bug   但是当前的工作没做完 不想git add git commit  可以隐藏)

并不是你不想提交，而是工作只进行到一半，还没法提交，预计完成还需1天时间。但是，必须在两个小时内修复该bug，怎么办？

```
幸好，Git还提供了一个stash功能，可以把当前工作现场“储藏”起来，等以后恢复现场后继续工作：
git  stach
```

现在，用`git status`查看工作区，就是干净的（除非有没有被Git管理的文件），因此可以放心地创建分支来修复bug。



首先确定要在哪个分支上修复bug，假定需要在`master`分支上修复，就从`master`创建临时分支：

```c
//在dev分支暂存  把当前工作区和暂存区的改动都保存到一个“临时堆栈”里；当前分支（比如 dev）的状态会恢复成干净状态；改动内容不会丢，只是临时藏起来了。
git stash  //git stash save "name"  当前在 dev 分支，保存当前工作现场
    
//# 切换到 master，修复 bug
git checkout master
git checkout -b issue-101  //创建

 git add readme.txt 
 git commit -m "fix bug 101"
 
 //修复完成后，切换到master分支，并完成合并，最后删除issue-101分支：
 git switch master
 git merge --no-ff -m "merged bug fix 101" issue-101
 git branch -d issue-101 
 
 //太棒了，原计划两个小时的bug修复只花了5分钟！现在，是时候接着回到dev分支干活了！
 git switch dev
 
 //工作区是干净的，刚才的工作现场存到哪去了？用git stash list命令看看：
 git stash list
 
 
 //工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：一是用git stash apply恢复，但是恢复后，stash内容并不删除，你需要用git stash drop来删除；另一种方式是用git stash pop，恢复的同时把stash内容也删了：
git stash pop  

// 把 master 上的修复同步进 dev
git cherry-pick <修复的 commit hash>
```



```c
//你可以多次stash，恢复的时候，先用git stash list查看，然后恢复指定的stash，用命令：
$ git stash list

    /*
stash@{0}: On dev: save dev wip
stash@{1}: On main: fix bug
    */
    
$ git stash apply stash@{0}  //回复指定版本

```

在`master`分支上修复了bug后，我们要想一想，`dev`分支是早期从`master`分支分出来的，所以，这个bug其实在当前`dev`分支上也存在。

那怎么在`dev`分支上修复同样的bug？重复操作一次，提交不就行了？

有木有更简单的方法？



同样的bug，要在`dev`上修复，我们只需要把`4c805e2 fix bug 101`这个提交所做的修改“复制”到`dev`分支。注意：我们只想复制`4c805e2 fix bug 101`这个提交所做的修改，并不是把整个`master`分支merge过来。

为了方便操作，Git专门提供了一个`cherry-pick`命令，让我们能复制一个特定的提交到当前分支：

```c
git branch
* dev
  master
git cherry-pick 4c805e2 //commit id  Git自动给`dev`分支做了一次提交，注意这次提交的commit是`1d4b803`
```

Git自动给`dev`分支做了一次提交，注意这次提交的commit是`1d4b803`，它并不同于`master`的`4c805e2`，因为这两个commit只是改动相同，但确实是两个不同的commit。用`git cherry-pick`，我们就不需要在`dev`分支上手动再把修bug的过程重复一遍。





有些聪明的童鞋会想了，既然可以在`master`分支上修复bug后，在`dev`分支上可以“重放”这个修复过程，那么直接在`dev`分支上修复bug，然后在`master`分支上“重放”行不行？当然可以，不过你仍然需要`git stash`命令保存现场，才能从`dev`分支切换到`master`分支。



修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除；

当手头工作没有完成时，先把工作现场`git stash`一下，然后去修复bug，修复后，再`git stash pop`，回到工作现场；

在`master`分支上修复的bug，想要合并到当前`dev`分支，可以用`git cherry-pick <commit>`命令，把bug提交的修改“复制”到当前分支，避免重复劳动。



总结：什么时候在 dev 修，什么时候在 master 修？

| 场景                           | 修复位置                                                 | 是否需要同步        |
| ------------------------------ | -------------------------------------------------------- | ------------------- |
| 仅 dev 有 bug（新功能引入）    | dev 修                                                   | 不需要同步到 master |
| master 和 dev 都有 bug         | 推荐：在 master 修，再 cherry-pick 到 dev                |                     |
| 只在 dev 修，但想同步到 master | 允许，但要 cherry-pick，只挑出修复，不带入其他未完成开发 |                     |





### 5. Feature分支

添加一个新功能时，你肯定不希望因为一些实验性质的代码，把主分支搞乱了，所以，每添加一个新功能，最好新建一个feature分支，在上面开发，完成后，合并，最后，删除该feature分支。







```c
 git switch -c feature-vulcan
 $ git add vulcan.py

$ git status
     $ git commit -m "add feature vulcan"
     
 $ git switch dev
    /*
    一切顺利的话，feature分支和bug分支是类似的，合并，然后删除。
但是！
就在此时，接到上级命令，因经费不足，新功能必须取消！
虽然白干了，但是这个包含机密资料的分支还是必须就地销毁：
    */
 git branch -d feature-vulcan    
     /*
     销毁失败。Git友情提醒，feature-vulcan分支还没有被合并，如果删除，将丢失掉修改，如果要强行删除，需要使用大写的-D参数。。
     */
     git branch -D feature-vulcan
```



如果要丢弃一个没有被合并过的分支，可以通过`git branch -D <name>`强行删除。

### 6. 多人协作





推送分支

推送分支，就是把该分支上的所有本地提交推送到远程库。推送时，要指定本地分支，这样，Git就会把该分支推送到远程库对应的远程分支上：

```
git  push origin(远程仓库默认名字) master(本地分支)
```

```
git push origin dev
```

但是，并不是一定要把本地分支往远程推送，那么，哪些分支需要推送，哪些不需要呢？

- `master`分支是主分支，因此要时刻与远程同步；
- `dev`分支是开发分支，团队所有成员都需要在上面工作，所以也需要与远程同步；
- bug分支只用于在本地修复bug，就没必要推到远程了，除非老板要看看你每周到底修复了几个bug；
- feature分支是否推到远程，取决于你是否和你的小伙伴合作在上面开发。

总之，就是在Git中，分支完全可以在本地自己藏着玩，是否推送，视你的心情而定！



抓取分支

多人协作时，大家都会往`master`和`dev`分支上推送各自的修改。

现在，模拟一个你的小伙伴，可以在另一台电脑（注意要把SSH Key添加到GitHub）或者同一台电脑的另一个目录下克隆：

```c
//当你的小伙伴从远程库clone时，默认情况下，你的小伙伴只能看到本地的master分支
git branch

//你的小伙伴要在dev分支上开发，就必须创建远程origin的dev分支到本地，于是他用这个命令创建本地dev分支：
$ git checkout -b dev origin/dev
//他就可以在dev上继续修改，然后，时不时地把dev分支push到远程：
git add env.txt
git commit -m "add env"
 git push origin dev

//你的小伙伴已经向origin/dev分支推送了他的提交，而碰巧你也对同样的文件作了修改，并试图推送：

$ git add env.txt
$ git commit -m "add new env"
$ git push origin dev

//推送失败，因为你的小伙伴的最新提交和你试图推送的提交有冲突，解决办法也很简单，Git已经提示我们，先用git pull把最新的提交从origin/dev抓下来，然后，在本地合并，解决冲突，再推送：

$ git pull
 git branch --set-upstream-to=origin/dev dev//git pull也失败了，原因是没有指定本地dev分支与远程origin/dev分支的链接，根据提示，设置dev和origin/dev的链接：
   $ git pull
    
   ///这回git pull成功，但是合并有冲突，需要手动解决，解决的方法和分支管理中的解决冲突完全一样。解决后，提交，再push： 
    $ git commit -m "fix env conflict"
    $ git push origin dev
```

因此，多人协作的工作模式通常是这样：

1. 首先，可以尝试用`git push origin <branch-name>`推送自己的修改；
2. 如果推送失败，则因为远程分支比你的本地更新，需要先用`git pull`试图合并；
3. 如果合并有冲突，则解决冲突，并在本地提交；
4. 没有冲突或者解决掉冲突后，再用`git push origin <branch-name>`推送就能成功！

如果`git pull`提示`no tracking information`，则说明本地分支和远程分支的链接关系没有创建，用命令`git branch --set-upstream-to <branch-name> origin/<branch-name>`。

这就是多人协作的工作模式，一旦熟悉了，就非常简单。

小结

- 查看远程库信息，使用`git remote -v`；
- 本地新建的分支如果不推送到远程，对其他人就是不可见的；
- 从本地推送分支，使用`git push origin branch-name`，如果推送失败，先用`git pull`抓取远程的新提交；
- 在本地创建和远程分支对应的分支，使用`git checkout -b branch-name origin/branch-name`，本地和远程分支的名称最好一致；
- 建立本地分支和远程分支的关联，使用`git branch --set-upstream branch-name origin/branch-name`；
- 从远程抓取分支，使用`git pull`，如果有冲突，要先处理冲突。



```c
为什么后 push 的人需要先 pull？
当多人协作时，假设你和同事都在 dev 分支上工作：

初始状态：远程仓库和你们本地的 dev 分支都是版本 A

同事先推送：

同事修改代码并提交到本地（版本 B）

同事执行 git push origin dev 成功，远程仓库更新为版本 B

你也修改了代码：

你基于版本 A 修改了代码并提交到本地（版本 C）

当你尝试 git push origin dev 时，Git 会拒绝你的推送

Git 为什么会拒绝你的推送？
因为 Git 发现远程仓库的 dev 分支（版本 B）已经不是你上次拉取时的状态（版本 A）了。Git 要求你必须先合并远程的最新更改（版本 B）到你的本地分支（版本 C），然后才能推送。

具体流程
你尝试推送：

bash
git push origin dev
会收到类似这样的错误：

text
! [rejected]        dev -> dev (non-fast-forward)
Git 提示你需要先 pull：

text
hint: Updates were rejected because the tip of your current branch is behind
hint: its remote counterpart. Integrate the remote changes (e.g.
hint: 'git pull ...') before pushing again.
你执行 pull 操作：

bash
git pull origin dev
这相当于：

git fetch origin dev（获取远程最新代码）

git merge origin/dev（将远程变更合并到本地）

这时有两种情况：

没有冲突：Git 会自动创建一个合并提交（版本 D，包含 B 和 C 的修改）

有冲突：需要手动解决冲突（如前一节所述）

解决冲突后（如有），提交合并结果：

bash
git commit -m "Merge remote changes"
最后推送：

bash
git push origin dev
为什么这样设计？
这种机制是 Git 的重要安全特性：

防止意外覆盖他人的工作

确保历史记录的完整性

强制开发者意识到并处理代码变更之间的交互

实际工作中的建议
开始工作前先 pull：git pull origin dev 获取最新代码

频繁提交和推送：减少大范围冲突的可能性

小步提交：每次提交只做一个小功能的修改，冲突更容易解决

这种工作流虽然有时显得繁琐，但能有效防止代码丢失和冲突，是多人协作的必要流程。
```







### 7.Rebase

在上一节我们看到了，多人在同一个分支上协作时，很容易出现冲突。即使没有冲突，后push的童鞋不得不先pull，在本地合并，然后才能push成功。





### 小结

- rebase操作可以把本地未push的分叉提交历史整理成直线；
- rebase的目的是使得我们在查看历史提交的变化时更容易，因为分叉的提交需要三方对比。

## 2. 标签管理

Git 标签（Tag）确实是为了更方便地管理 commit，特别是在版本发布和重要节点标记方面非常有用。下面我会详细解释标签的作用、使用场景以及具体操作。

**Git 标签的作用**

标签（Tag）本质上是一个**指向特定 commit(默认当前HEAD) 的不可变引用**（类似于分支，但不会移动）。它的主要用途包括：

**1. 标记重要版本（如软件发布）**

- 例如 `v1.0.0`、`v2.3.1` 等，方便快速找到某个发布版本对应的代码。
- 比直接记忆 commit hash（如 `a1b2c3d`）更直观。

**2. 方便回滚到指定版本**

- 如果线上版本 `v1.2.0` 出现 Bug，可以直接检出（`git checkout v1.2.0`）进行修复，而不用记住 commit ID。  切换到标签版本





### 1.创建标签

```c
git branch
//切换到需要打标签的分支上
git switch master
git tag V1.0 //然后，敲命令git tag <name>就可以打一个新标签：

git tag   //可以用命令git tag查看所有标签：  打标签是commit上
    
/*
默认标签是打在最新提交的commit上的。有时候，如果忘了打标签，比如，现在已经是周五了，但应该在周一打的标签没有打，怎么办？
方法是找到历史提交的commit id，然后打上就可以了：
*/    
     git log --pretty=oneline --abbrev-commit
    //比方说要对add merge这次提交打标签，它对应的commit id是f52c633，敲入命令
    $ git tag v0.9 f52c633
//再用命令git tag查看标签：
  git  tag
    //注意，标签不是按时间顺序列出，而是按字母排序的。可以用git show <tagname>查看标签信息：
    git show v0.9
```

```
//可以看到，v0.9确实打在add merge这次提交上。
$ git show v0.9
commit f52c63349bc3c1593499807e5c8e972b82c8f286 (tag: v0.9)
Author: Michael Liao <askxuefeng@gmail.com>
Date:   Fri May 18 21:56:54 2018 +0800

    add merge

diff --git a/readme.txt b/readme.txt
...

```

```c
//还可以创建带有说明的标签，用-a指定标签名，-m指定说明文字：
git tag -a v1.0 -m "说明" commit_id
$ git tag -a v0.1 -m "version 0.1 released" 1094adb

```

标签总是和某个commit挂钩。如果这个commit既出现在master分支，又出现在dev分支，那么在这两个分支上都可以看到这个标签。

#### 小结

- 命令`git tag <tagname>`用于新建一个标签，默认为`HEAD`，也可以指定一个commit id；

- 命令`git tag -a <tagname> -m "blablabla..."`可以指定标签信息；

- 命令`git tag`可以查看所有标签。

  

### 2. 操作标签

```c
git tag -d v1.0 //如果标签打错了，也可以删除
    
    //如果要推送某个标签到远程，使用命令git push origin <tagname>：
    git push origin v1.0
    // 一次性推送全部尚未推送到远程的本地标签：
    git push origin --tags
    
    //如果标签已经推送到远程，要删除远程标签就麻烦一点，先从本地删除：
    git tag -d v0.9
    git push origin :refs/tags/v0.9 //然后，从远程删除。删除命令也是push，但是格式如下：
    
```

#### 小结

- 命令`git push origin <tagname>`可以推送一个本地标签；
- 命令`git push origin --tags`可以推送全部未推送过的本地标签；
- 命令`git tag -d <tagname>`可以删除一个本地标签；
- 命令`git push origin :refs/tags/<tagname>`可以删除一个远程标签。





#### 完整示例流程**

**场景：修复旧版本 `v1.0.0` 的 Bug**

1. **查看标签**：

   bash

   ```
   git tag
   ```

   输出：

   text

   ```
   v1.0.0
   v1.1.0
   v2.0.0
   ```

2. **基于 `v1.0.0` 创建修复分支**：

   bash

   ```
   git checkout -b fix-bug-v1.0.0 v1.0.0
   ```

3. **修改代码并提交**：

   bash

   ```
   vim app.js  # 修复 Bug
   git add app.js
   git commit -m "修复 v1.0.0 的登录问题"
   ```

4. **可选：合并回 `master`（如果需要）**：

   bash

   ```
   git checkout master
   git merge fix-bug-v1.0.0
   ```

**总结**

| 操作                       | 命令                              | 说明                              |
| :------------------------- | :-------------------------------- | :-------------------------------- |
| **查看标签**               | `git tag`                         | 列出所有标签                      |
| **基于标签创建分支**       | `git checkout -b 新分支名 标签名` | 从标签创建可修改的分支            |
| **直接切换到标签（只读）** | `git checkout 标签名`             | 进入 `detached HEAD` 状态（慎用） |

**关键点**：
✅ **标签是只读的**，要修改代码必须**先创建分支**。
✅ **新分支和 `master` 无关**，除非你主动合并。
✅ **适用于修复旧版本 Bug**，而不会影响最新代码。

现在你应该清楚整个流程了！如果还有疑问，欢迎继续提问 😊
