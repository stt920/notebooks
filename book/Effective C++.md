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

   - const修饰成员函数目的

     - 让函数的权限为只读，它无法改变成员变量的值；
     - 如果一个对象为const，它只有权力调用const函数，因为成员变量不能被改变。

   - const可构成成员函数重载

     即两个成员函数只是常量性不同，可以被重载。例如：

     ```c++
     class TextBlock{  
     public:  
         ...  
         const char& operator[](std::size_t position) const{  
             return text[position];  
         }  
         char& operator[](std::size_t position){  
             return text[position];  
     
         }  
     private:  
         std::string text  
     }; 
     ```

     TextBlock的operator[]可以被这么使用：

     `TextBlock tb("hello");`

     `std::cout<<tb[0];`               //调用non-const TextBlock::operator[]

     `const TextBlock ctb("hello");`

     `std::cout<<ctb[0];`               //调用const TextBlock::operator[]

     但是对于以下操作会有错误：

     `ctb[0]='x'`;                             //写一个const TextBlock

   - mutable（可变的）

     mutable（可变的）关键字可以释放掉non-static成员变量的bitwise constness约束，被mutable修饰的成员变量可能总是会被更改，即使在const成员函数内。 

   - 在const和non-const成员函数中避免重复

     non-const成员函数调用const成员函数是一个避免代码重复的安全做法。相反，const成员函数调用non-const成员函数是一种错误行为，因为对象有可能因此被改动。 

请记住：

- 将某些东西声明为const可帮助编译器侦测出错误用法。const可被施加于任何作用域的对象,函数参数,函数返回类型,成员函数本体。
- 编译器强制实施bitwise constness,但你编写程序时应该使用"概念上的常量性"(conceptual constness)。
- 当const和non-const成员函数有着实质等价的实现时,令non-const版本调用const版本可避免代码重复。

### 条款04：确定对象被使用前已被初始化

1. 永远在使用对象之前先将它初始化。对于无任何成员的内置类型，必须手工完成。对于内置类型以外的任何东西，初始化责任落在构造函数上，确保每一个构造函数都将对象的每一个成员初始化。 

2. C++规定，对象的成员变量初始化动作发生在进入构造函数本体之前，因此”在构造函数内初始化“（准确地说是赋值，不是初始化）并不是理想选择。构造函数较佳的写法是使用所谓的member intialization list（成员初始化列表）替换赋值操作。member intialization list效率较高，基于赋值版本的构造函数，首先调用dafault构造函数为成员变量设置初值，然后立刻再对它们赋予新值。default构造函数的作为因此浪费了，member intialization list的做法避免了这一问题。

3. C++有着十分固定的”成员初始化次序“。

   base classes更早于其derived classed被初始化，而calss的成员变量总是以其声明次序被初始化（与member intialization list中次序无关）

4. 不同编译单元内定义的non-local static对象初始化次序

   编译单元是指产出单一目标文件的那些源码。基本上它是单一源码文件加上其所含入的头文件。C++对于“定义于不同的编译单元内的non-localstatic对象”的初始化相对次序并无明确定义。 

   ```c++
   class FileSystem{  
   public:  
       ...  
       std::size_t numDisks() const;  
       ...  
   };  
   extern FileSystem tfs;  
   ```

   ```c++
   class Directory{  
   public:  
       Directory(params);  
       ...  
   };  
   Directory::Directory(params){  
       ...  
       std::size_t disks = tfs.numDisks();  
       ...  
   }  
   Directory tempDir(params);  
   ```

   现在初始化次序显得很重要：除非tfs在tempDir之前被初始化，否则tempDir的构造函数会用到尚未初始化的tfs。但是上述程序无法确定初始化顺序。

    为了解决上述问题，将每个non-local static对象搬到自己的专属函数内，该对象在此函数内被声明为static，此函数返回一个引用指向它所含的对象。然后用户调用这些函数，而不直接指涉这些对象。这个方法的基础在于：c++保证函数内的local static对象会在该函数被调用期间首次遇上该对象之定义式时被初始化。 

   ```c++
   class FileSystem{  
   public:  
       ...  
       std::size_t numDisks() const;  
       ...  
   };  
   FileSystem& tfs(){  
       static FileSystem fs;  
       return fs;  
   }  
   ```

   ```c++
   class Directory{  
   public:  
       Directory(params);  
       ...  
   };  
   Directory::Directory(params){  
       ...  
       std::size_t disks = tfs().numDisks();  
       ...  
   }  
   Directory& tempDir(){  
       static Directory td;  
       return td;  
   }  
   ```

请记住：

- 为内置对象进行手工初始化，因为C++不保证初始化它们。
- 构造函数最好使用成员初始化列表（member intialization list），而不要在构造函数本体内使用赋值操作。初始化列表列出的成员变量，其排列次序应该和它们在类中的声明次序相同。
- 为免除“跨编译单元之初始化次序”问题，请以local static对象替换non-local static对象。

## 第二章 构造/析构/赋值运算

### 条款05：了解C++默默编写并调用哪些函数

1. 什么时候empty class（空类）不再是个empty class？当C++处理过它后。如果没自己的声明，编译器就会为它声明一个copy构造函数、一个copy assignment操作符和一个析构函数。如果没有声明任何构造函数，编译器也会为你声明一个default构造函数。所有这些函数都是public且inline。

因此，如果写下：

```c++
class Empyt{}；
```

就好像写下这样的代码：

```c++
class Empyt{
public:
    Empyt(){……}
    Empyt(const Empty& rhs){……}
    ~Empyt(){……}   
    Empyt& operator=(const Empyt& rhs){……}
}；
```

2. 编译器阻止产生拷贝赋值运算符情况
   - 类内含有reference成员。如果需要类含有reference的class支持赋值操作，必须自己定义一个拷贝赋值运算符。因为C++不允许让reference改指向不同对象。
   - 类内含有const成员。更改const成员不合法，所以编译器不知道如何在自己生成的赋值函数里面对它们。
   - 将拷贝赋值运算符声明为private。编译器将拒绝为其派生类生成一个拷贝赋值运算符。

请记住:

- 编译器可以暗自为类创建default构造函数，copy构造函数，copy assignment操作符，以及析构函数。

### 条款06：不想使用编译器自动生成的函数，就该明确拒绝

1. 声明为private阻止编译器自动生成的函数

   ​	所有编译器产出的函数都是public。为阻止这些函数被创建出来，需自行声明，因此可以将拷贝构造函数和拷贝赋值运算符**声明为private**。藉由明确声明一个成员函数，你阻止了编译器案子创建其专属版本；而令这些函数为private，使你得以成功阻止人们调用。

   ​	只声明为private的做法并不绝对安全，因为member函数和friend函数还是可能调用你的private函数。只需不去定义它们，**“将成员函数声明为private而且故意不实现它们”**。 

   例子：

   ```c++
   
   ```

2. 为驳回编译器自动（暗自）提供的机能，使用像Uncopyable这样的base class也是一种做法。 

   ```C++
   class Uncopyable{  
   public:  
       Uncopyable();  
       ~Uncopyable();  
   private:  
       Uncopyable(const Uncopyable&);  
       Uncopyable& operator=(const Uncopyable&);  
   };  
   ```

   为求阻止HomeForSale对象被拷贝，唯一要做的就是继承Uncopyable：

   ```c++
   
   ```

请记住:

- 为驳回编译器自动（暗自）提供的机能，可将相应的成员函数声明为private并且不予实现。使用像Uncopyable这样的base class也是一种做法。

### 条款07：为多态基类声明virtual析构函数

 请记住：

- 带有多态性质的基类应该声明一个virtual析构函数。如果一个类带有任何virtual函数，它就应该拥有一个virtual析构函数。
- 一个类的设计目的不是作为基类使用，或不是为了具备多态性，就不该声明virtual析构函数。  

### 条款08：别让异常逃离析构函数

