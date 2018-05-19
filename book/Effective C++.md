# Effective C++

## 第一章 让自己习惯C++

### 条款01：视C++为一个语言联邦

为了理解C++，你必须人生的主要次语言：

- C。说到底C++仍以C为基础。区块（blocks）、语句（statements）、预处理器（preprocessor）、内置数据类型（built-in data types）、数组（arrays）、指针（pointers）等都来自C。许多时候C++对问题的揭法其实不过就是高级C的解法，但当你以C++内的C成分工作时，高效编程守则映射出C语言的局限：没有模板、没有异常、没有重载……
- Object-Oreinted C++。这一部分是面向对象设计之古典守则在C++上的最直接实施。类，封装，继承，多态，virtual函数......等等
- Template C++。这是C++泛型编程部分
- STL。STL是个template程序库。主要包括容器（containers），迭代器（iterators），算法（algorithms）以及函数对象（function objects）... 

**请记住：**

- C++高效编程守则视情况而变化，取决于你使用C++的哪一部分。

### 条款02：尽量以const、enum、inline替换#define

​	这个条款或许改为“宁可以编译器替换预处理器”比较好，因为或许 #define 不被视为预言的一部分。

1. 用const替换#define

   当做出这样的事情：

   ```c++
   #define ASPECT_RATIO 1.653
   ```

   ​	几号名称ASPECT_RATIO也许从未被编译器看见；也许编译器开始处理源码之前它就被预处理器移走了。于是记号名称ASPECT_RATIO有可能没进入记号表内。于是当你运用此常量获得一个编译错误信息时，可能会带来困惑，因为编译错误信息可能会提到1.653而不是ASPECT_RATIO。

   ​	解决方法可以是以一个常量替换上述宏（#define）：

   ```c++
   const double AspectRatio=16.53
   ```

   ​	好处有如下几点：a、语言常量肯定会被编译器看到，会进入符号表内。b、使用常量可能比使用#define导致较小量的码，因为预处理器盲目的将宏名称替换会导致目标码出现多份替换后的内容。 

   ​	常量替换#defines要注意两种特殊情况：a、常量指针的定义，如：const char * const authorName =“Scott Meyers”。b、class专属常量，即一个static const member，要在实现文件中定义它。 

2. enum替换#define

   ```c++
   class GamePlayer{  
   private:  
       static const int NumTurns = 5;  
       int scores[NumTurns];  
   };  
   ```

   ​	对于一些编译器，上述代码编译会出错，它们不允许static成员在其声明式上获得初值。如果编译器不支持，则可以将初值放在定义式：

   ```c++
   class GamePlayer{  
   private:  
       static const int NumTurns; //static class 常量声明位于头文件
       int scores[NumTurns];  
   };  
   const int GamePlayer::NumTurns=5;//static class 常量定义位于实现文件
   ```

   ​	但是可以用enum hack的补偿做法实现类内初始化，理论基础是：一个属于枚举类型的数值可权充ints被使用，GamePlayer可定义如下： 

   ```c++
   class GamePlayer{  
   private:  
       enum{ NumTurns = 5 };  
       int scores[NumTurns];  
   };  
   ```

   ​	优点：第一，enum hack的行为某方面来说比较像#define而不像const，有时候这正是你想要的。例如取一个const的地址是合法的，但取一个enum和#define的地址就不合法；第二，enum hack是模板元编程的基本技术，许多代码用了它，看到时必须认识。

3. inline替换#define

   ​	宏看起来像一个函数，但它不会招致函数调用带来的额外开销，同时使用中可能会带来麻烦，例如：

   ```c++
   #define CALL_WITH_MAX(a,b)    f((a) > (b)) ? (a) : (b))  
   ```

   ​	template inline函数可以获得宏带来的效率以及一般函数的所有可预料行为和类型安全性：

   ```c++
   template<typename T>  
   inline void callWithMax(cosnt T &a, cosnt T &b){  
       f(a > b ? a : b);  
   }  
   ```

   ​	有了consts、enums和inlines，我们对预处理器的需求降低了，但不能完全消除。#include仍然是必需品，比如#ifdef/#ifndef页继续扮演者控制编译的重要角色。目前还不到预处理器全面退隐的时候。

请记住：

- 对于单纯常量，最好以const对象或enums替换#defines。
- 对于形似函数的宏，最好改用inline函数替换#defines。

### 条款03：尽可能使用const

1. const成员变量

   ​	如果关键字const出现在星号左边，表示被指物是常量；如果关键字const出现在星号右边，表示指针自身是常量；如果出现再星号两边，表示被指物和指针两者都是常量。 

   ```c++
   char greeting[] = "hello"  
   char* p = greeting;   
   const char* p = greeting;//非常量指针，指针可变，指针指向内容不能变  
   char* const p = greeting;//常量指针，指针不能变，指针指向内容可变  
   const char* const p = greeting;//指针和指针指向内容都不能变  
   ```

2. const STL

   ​	STL迭代器系以指针为根据塑模出来，所以迭代器的作用就像`T*`指针。声明迭代器为const就像声明指针为const一样（即声明一个`T*` const），表示这个迭代器不得指向不同的东西，但它所指的东西的值是可以改动的。如果希望迭代器所指的东西不可被改动（即希望STL模拟一个const T*指针），需要的是const_iterator:

   ```c++
   std::vector<int> vec;  
   const std::vector<int>::iterator iter = vec.begin();  
   *iter = 10; //没问题，改变iter所指物  
   ++iter; //错误，iter是const  
     
   std::vector<int>::const_iterator cIter = vec.begin();  
   *cIter = 10;//错误，*cIter是const  
   ++cIter;//没问题，改变cIter 
   ```

3. const成员函数

   

### 条款04：确定对象被使用前已被初始化

## 第二章 构造/析构/赋值运算

 

