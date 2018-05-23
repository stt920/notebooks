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

   ​	记号名称ASPECT_RATIO也许从未被编译器看见；也许编译器开始处理源码之前它就被预处理器移走了。于是记号名称ASPECT_RATIO有可能没进入记号表内。于是当你运用此常量获得一个编译错误信息时，可能会带来困惑，因为编译错误信息可能会提到1.653而不是ASPECT_RATIO。

   ​	解决方法可以是以一个常量替换上述宏（#define）：

   ```c++
   const double AspectRatio=16.53
   ```

   ​	好处有如下几点：a、语言常量肯定会被编译器看到，会进入符号表内。b、使用常量可能比使用#define导致较小量的码，因为预处理器盲目地将宏名称替换会导致目标码出现多份替换后的内容。 

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

   ​	有了consts、enums和inlines，我们对预处理器的需求降低了，但不能完全消除。#include仍然是必需品，比如#ifdef/#ifndef页继续扮演着控制编译的重要角色。目前还不到预处理器全面退隐的时候。

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

**请记住:**

- 编译器可以暗自为类创建default构造函数，copy构造函数，copy assignment操作符，以及析构函数。

### 条款06：不想使用编译器自动生成的函数，就该明确拒绝

1. 声明为private阻止编译器自动生成的函数

   ​	所有编译器产出的函数都是public。为阻止这些函数被创建出来，需自行声明，因此可以将拷贝构造函数和拷贝赋值运算符**声明为private**。藉由明确声明一个成员函数，你阻止了编译器暗自创建其专属版本；而令这些函数为private，使你得以成功阻止人们调用。

   ​	只声明为private的做法并不绝对安全，因为member函数和friend函数还是可能调用你的private函数。只需不去定义它们，**“将成员函数声明为private而且故意不实现它们”**。 

   例子：

   ```c++
   class HomeForSale{  
   public:  
       ...  
   private:  
       ...  
       HomeForSale)(const HomeForSale&);  
       HomeForSale& operator=(const HomeForSale&)  
   };  
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
   class HomeForSale:private Uncopyable{  
       ...  
   };
   ```

**请记住:**

- 为驳回编译器自动（暗自）提供的机能，可将相应的成员函数声明为private并且不予实现。使用像Uncopyable这样的base class也是一种做法。

### 条款07：为多态基类声明virtual析构函数

1. 如果基类的析构函数不为虚函数，则delete一个**指向派生类对象的基类指针**将产生未定义行为。

```c++
#include <iostream.h>
class Base 
{ 
public: 
Base() { mPtr = new int; } 
~Base() { delete mPtr; cout<<"Base::Destruction"<<endl;} 
private: 
  int* mPtr; 
} ;

class Derived : public Base 
{ 
public: 
  Derived() { mDerived = new long; } 
  ~Derived() { delete mDerived; cout<<"Derived::Destruction"<<endl;} 
private: 
  long* mDerived; 
} ;

void main() 
{ 
  Base* p = new Derived; //父类指针指向子类对象，指向派生类的基类指针
  delete p; 
}
```

输出结果只有：Base::Destruction

​	以上代码会产生内存泄露，因为new出来的是Derived类资源，采用一个基类的指针来接收，析构的时候，编译器因为只是知道这个指针是基类的，所以只将基类部分的内存析构了，而不会析构子类的，就造成了内存泄露，如果将基类的析构函数改成虚函数，就可以避免这种情况，因为虚函数是后绑定，其实就是在虚函数列表中，析构函数将基类的析构函数用实际对象的一组析构函数替换掉了，也就是先执行子类的虚函数再执行父类的虚函数，这样子类的内存析构了，父类的内存也释放了，就不会产生内存泄露。

2. 任何类只要带有virtual函数都几乎确定应该也有一个virtual析构函数。如果一个类不含virtual函数，通常表示它并不意图被用做一个基类，当类不企图被当做基类的时候，令其析构函数为virtual往往是个馊主意。因为实现virtual函数，需要额外的开销（指向虚函数表的指针vptr）。 

 **请记住：**

- 带有多态性质的基类应该声明一个virtual析构函数。如果一个类带有任何virtual函数，它就应该拥有一个virtual析构函数。
- 一个类的设计目的不是作为基类使用，或不是为了具备多态性，就不该声明virtual析构函数。  

### 条款08：别让异常逃离析构函数

1. C++并不禁止析构函数吐出异常，但它不鼓励你这样做。因为在析构函数中吐出异常可能会使程序过早结束或出现不明确行为。

2.  析构函数发生异常解决办法：

   - 如果抛出异常，就结束程序。通常通过调用abort完成。  

   ```c++
   DBConn::~DBConn(){  
       try{ db.close(); }  
       catch(…){  
           //制作运转记录，记下对close的调用失败;  
           std::abort();  
       }  
   }  
   ```

   - 吞下因调用close而发生的异常。 

   ```c++
   DBConn::~DBConn(){  
       try{ db.close(); }  
       catch(…){  
           //制作运转记录，记下对close的调用失败;  
       }  
   }  
   ```

3. 如果某个操作可能在失败时抛出异常，而又存在某种需要必须处理该异常，那么这个异常必须来自析构函数以外的某个函数。 

**请记住：**

- 析构函数绝对不要吐出异常。如果一个被析构函数调用的函数可能抛出异常，析构函数应该捕捉任何异常，然后吞下它们（不传播）或结束程序。
- 如果客户需要对某个操作函数运行期间抛出的异常做出反应，那么类应该提供一个普通函数（而非在析构函数中）执行该操作。

### 条款09：决不在构造和析构过程中调用virtual函数

​	本条款重点：你不应该在构造函数和析构函数期间调用virtual函数，因为这样的调用不会带来你预想的效果。如下例，假如有个class继承体系，用来塑造股票交易如买进、卖出的订单等等，这样的交易一定要经过审计，所以每当创建一个交易对象，在审计日志中也需要创建一笔适当的记录:

```c++
class Transaction{                                //所有交易的base class
pubilc:
    Transaction();    
    virtual void logTransaction() const=0;        //做出一份因类型不同而不同的日志
}；
Transaction::Transaction                          //base class构造函数实现
{
	...
	logTransaction();                            //最后动作是志记这笔交易
}

class BuyTransaction:public Transaction{          //drive class
public:
	virtual void logTransaction() const;          //志记此类型交易
}
class SellTransaction:public Transaction{          //drive class
public:
	virtual void logTransaction() const;          //志记此类型交易
}
```

当执行以下操作：

```c++
BuyTransaction b;
```

首先会调用Transaction构造函数（derived class对象内的base class成分会在derived class自身成分构造之前先构造妥当），这是调用的logTransaction是Transaction内的版本，不是BuyTransaction内的版本，而目前即将建立的对象类型是BuyTransaction版本。**即derived class在构造函数中调用的virtual函数是基类版本而不是derived class的版本** 。Bases class构造期间virtual函数绝不会下降到derived classes阶层。取而代之的是，对象的作为就像隶属base类型一样。非正式说法：在base class构造期间，virtual函数不是virtual函数。

原因：由于base class构造函数执行更早于derived class构造函数，当base class构造函数执行时derived class的成员变量尚未初始化。如果此期间调用virtual函数下降至derived classes阶层，derived class的函数几乎必然取用local成员变量，而那些成员变量尚未初始化。所以C++不会走这条路。

**请记住**：

- 在构造函数和析构期间不要调用虚函数，因为这类调用从不下降至derived class（比起当前执行构造函数和析构函数的那层）。

### 条款10：令operator=返回一个reference to *this

对于赋值操作符，我们常常要达到这种类似效果，即连续赋值：  

```c++
int x, y, z;  
x = y = z = 15;  
```

为了实现“连锁赋值”，赋值操作符必须返回一个“引用”指向操作符的左侧实参。

```c++
class Widget{
public:
	...
	Widget& operator=(const Widget& rhs)
	{
		...
		return *this;
	}
	...
}
```

这个协议不仅适用于以上标准赋值形式，也适用于所有相关运算，如+=，-=，*=。

**请记住** ：

- 令赋值操作符返回一个reference to *this。

### 条款11：在operator=中处理“自我赋值”

“自我赋值”指对象被赋值给自己。 

```c++
class Widget{ ... };  
Widget w;  
...  
w = w;//自我赋值  
  
a[i] = a[j]; //如果i等于j，这个就是自我赋值  
  
*px=*py //当px和py指向同一个对象时，就是自我赋值  
  
class Base{ ... };  
class Derived:public Base{ ... };  
void doSomething(const Base& rb,Derived* pd) //rb和*pd有可能是同一对象  

```

"自我赋值"带来的风险：可能“在停止使用资源之前意外释放了它”；

下面operator=实现代码，表面上看起来合理，但自我赋值出现并不安全：

```c++
class Bitmap{ ..... };  
class Wdiget{  
public:  
    ...  
    Wdiget& operator=(const Wdiget& rhs){  
        delete pb;  
        pb = new Bitmap(*rhs.pb);  
        return *this;  
    }  
private:  
    Bitmap* pb;  
};  
```

这里自我赋值问题是，operator=函数内的*this（赋值目的端）和rhs有可能是同一对象。果真如此delete就不只是销毁当前对象的bitmap，它也销毁rhs的bitmap。

为解决这种错误，传统做法是藉由operator=最前面的一个“证同测试”达到“自我赋值”的检验目的

```c++
Wdiget& operator=(const Wdiget& rhs){  
    if (&rhs == this) return *this;  
    delete pb;  
    pb = new Bitmap(*rhs.pb);  
    return *this;  
}  
```

这样做可行，但如果在new Bitmap的时候出现异常，Widget最终会持有一个指针指向一块被删除的Bitmap，这样的指针有害。下面是另外的一种解决办法： 

```c++
Wdiget& Wdiget::operator=(const Wdiget& rhs){  
    Bitmap* pOrig = pb;  
    pb = new Bitmap(*rhs.pb);  
    delete pOrig;  
    return *this;  
};  
```

这样当如果new Bitmap出现异常，pb保持原状。

**请记住**：

- 确保当对象进行自我赋值的时候有良好的行为，其中包括来源对象和目标对象的比较，设计良好的赋值顺序，以及copy and swap技术。
- 要确保当一个函数操作对个对象，并且多个对象可能是同一个对象的时候行为也是准确的。

### 条款12：复制对象时勿忘其每一个成分

1. 当你拒绝编译器为你写出copying函数，如果你的代码不完全，编译器也不会告诉你。如果你为类添加一个成员变量，你必须同时修改copying函数（所有的构造函数，拷贝构造函数以及拷贝赋值操作符）。 
2. **在派生类的构造函数，拷贝构造函数和拷贝赋值操作符中应当显示调用基类相对应的函数**。
3. 当你编写一个copying函数，请确保：（1）复制所有local成员变量，（2）调用所有基类内的适当copying函数。

```c++
PriorityCustomer::PriorityCustomer(const PriorityCustomer& rhs)
  : Customer(rhs) ,                                //调用base class的拷贝构造函数
    priority(rhs.priority)
{}；
PriorityCustomer&
PriorityCustomer::operator=(const PriorityCustomer& rhs)
{
    Customer::operator=(rhs);               //对base class成分进行赋值操作
    priority=rhs.priority;
    return *this;
}
```

但是，我们不该令拷贝赋值操作符调用拷贝构造函数，也不该令拷贝构造函数调用拷贝赋值操作符。

**请记住**：

- Copying函数应该确保复制“对象内的所有成员变量”及“所有基类成分”；
- 不要尝试以某个copying函数实现另一个copying函数。应该将共同机能放进第三个函数中，并由两个copying函数共同调用。  

## 第三章 资源管理

### 条款13：以对象管理资源

假设我们使用一个用来塑模投资行为的程序库，其中各式各样的投资类型继承自一个root class Investment：

```c++
class Investment{……}；
```

进一步假设，这个程序库通过一个工厂函数供应我们某特定Investment对象：

```c++
Investment* createInvestment();
```

createInvestment的调用端使用了函数返回的对象后，有责任删除之。现在考虑有个f函数履行这个职责：

```c++
void f()
{
	Investment* pInv=createInvestment();
    ...
    delete pInv;
}
```

上述f中在若干情况下可能无法删除它得自createInvestment的投资对象，因为“...”区块可能导致过早退出，并没有执行delete，比如return、continue、goto或者异常等情况，因此会产生潜在的内存泄漏风险。

为确保createInvestment返回的资源总是被释放，我们需要将资源对象放进对象内，当控制流离开f，该对象的析构函数会自动释放那些资源。例如以下，使用标准库auto_ptr只能指针，其析构函数自动对其所指对象调用delete，从而避免内存泄漏：

```c++
void f()
{
    std::auto_ptr<Investment> pInv(createInvestment());
}
```

这个例子示范“以对象管理资源”的两个关键想法：

- 获得资源后立即放进管理对象内；
- 管理对象运用析构函数确保资源被释放；

由于auto_ptr被销毁时会自动删除它所指之物，所以不能让多个auto_ptr同时指向同一对象。所以auto_ptr若通过拷贝构造函数或拷贝赋值操作符复制它们，它们会变成NULL，而复制所得的指针将取得资源的唯一拥有权。

auto_ptr的替代方案是“引用计数型智能指针”（reference-counting smart pointer；RCSP），它可以持续跟踪共有多少对象指向某笔资源，并在无人指向它时自动删除该资源。TR1的tr1::shared_ptr就是一个"引用计数型智能指针"。 

auto_ptr和tr1::shared_ptr都在其析构函数内做delete而不是delete[]，也就意味着在动态分配而得的数组身上使用auto_ptr或tr1::shared_ptr会使资源得不到释放。 

请记住： 

- 为防止资源泄漏，请使用RAII（Resource Acquisitoin Is Initialization,资源取得实际便是初始化时机）对象，它们在构造函数中获得资源并在析构函数中释放资源。
- 两个常被使用的RAII类分别是tr1::shared_ptr和auto_ptr。前者通常是较佳选择，因为其copy行为比较直观。若选择auto_ptr，复制动作会使他（被复制物）指向NULL。

### 条款14：在资源管理类中小心copying行为

当一个RAII对象被赋值时，会发生声明事情？

- 禁止复制
- 对底层资源祭出“引用计数法”
- 复制底部资源（深度拷贝）
- 转移底部资源的拥有权

请记住：

- 复制RAII对象必须一并复制它所管理的资源，所以资源的copying行为决定RAII对象的行为。
- 普遍而常见的RAII类的copying行为是：禁止copying，施行引用计数法。不过其他行为也都可能被实现。

### 条款15：在资源管理类中提供对原始资源的访问

1. 许多APIs直接指涉资源，需要直接访问原始资源。这时候需要一个函数可将RAII对象（如tr1::shared_ptr）转换为其所内含之原始资源。有两种做法可以达成目标：显示转换和隐式转换。
2. tr1::shared_ptr和auto_ptr都提供一个get成员函数，用来执行显示转换，也就是返回智能指针内部的原始指针（的复件）。就像所有智能指针一样， tr1::shared_ptr和auto_ptr也重载了指针取值操作符（operator->和operator*），它们允许隐式转换至底部原始指针。

**显式转换： **

条款13导入一个观念，使用智能指针入auto_ptr或tr1::shared_ptr保持factory函数如createInvestement的调用结果：

```c++
std::tr1::shared_ptr<Investment> pInv(createInvestment());
```

假如希望以某个函数处理Investment对象，像这样：

```c++
int dayHeld(const Investment* pi);
```

你想这么调用它：

```c++
int days=dayHeld(pInv);    //错误
```

通不过编译，因为dayHeld需要的是Investment*指针，传给它的却是个类型为`tr1::shared_ptr<Investment> `的对象。

auto_ptr和tr1::shared_ptr都提供一个get成员函数，用来执行显式转换，也就是它会返回智能指针内部的原始指针（的复件）。同时它们也重载了指针取值操作符（operator->和operator*），允许隐式转换至底部原始指针。

**隐式转换： **

```c++
class Font{  
public:  
    explicit Font(FontHandle fn) :f(fn){}  
    ....  
    FontHandle get() const{  
        return f;  
    }  
    operator FontHandle() const{         //隐式转换  
        return f;  
    }  
    ...  
    ~Font(){ realseFont(f); }  
private:  
    FontHandle f;  
};  
```

请记住：

- APIs往往需要取得RAII的原始资源，所以每一个RAIIclass应该提供一个“取得其所管理之资源”的方法。
- 对原始资源的访问可能经由显示转换或隐式转换。一般而言显示转换比较安全，但隐式转换对客户比较方便。

### 条款16：成对使用new和delete时采取相同形式

1. 当你使用new动态生成一个对象，有两件事发生：第一，内存被分配。第二，针对此内存会有一个(或更多)构造函数被调用。当你使用delete，也有两件事发生：针对此内存会有一个(或更多)析构函数被调用，然后内存被释放。
2. 单一对象的内存布局一般而言不同于数组的内存布局。数组所用的内存通常还包括“数组大小“的记录，以便delete知道需要调用多少次析构函数。

```c++
string* stringpPtr1=new std::string;
string* stringpPtr2=new std::string[100];
...
delete stringpPtr1;
delete [] stringpPtr2;
```

3. 尽量不要对数组形式做typedef动作。

请记住：

- 如果你在new表达式中使用[]，必须在相应的delete表达式中也使用[]。如果你在new表达式中不使用[]，一定不要在相应的delete表达式中使用[]。 

### 条款17：以独立语句将newed对象置入智能指针

```c++
int priority();  
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority); 
```

采用如下形式调用processWidget

```c++
processWidget(std::tr1::shared_ptr<widget> pw(new widget), priority());  
```

虽然在此使用“对象管理式资源”，上述调用可能出现资源泄露。因为在函数中，参数的调用顺序会因为编译器的不同而不同，例如上面的顺序可能是：

1. 执行”new widget”
2. 调用priority
3. 调用tr1::shared_ptr构造函数

这样可能出现的问题就是当new widget成功后，如果priority()函数调用导致异常，new widget返回的指针将会遗失，因为new widget未能放入到智能指针中，导致内存泄漏。 

避免这类问题办法很简单：使用分离语句：

```c++
std::tr1::shared_ptr<Widget> pw(new Widget);    //在单独语句内以智能指存储newd所得对象  
processWidget(pw, priority()); //这个调用动作绝不至于造成泄漏  
```

请记住：

- 以独立语句将newed对象存储于（置入）智能指针内。如果不这样做，一旦异常抛出，有可能导致难以察觉的资源泄漏。   

## 第四章 设计与声明