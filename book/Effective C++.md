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

### 条款18：让接口容易被正确使用，不易被误用

- 考虑客户可能做出什么样的错误。

- 除非有好理由，否则应该尽量令你的types的行为与内置types一致。 
- tr1::shared_ptr有一个特别好的性质是：它会自动使用它的“每个指针专属的删除器”，因而消除另一个潜在的客户错误：所谓的“cross-DLL problem”。这个问题发生于“对象在动态连接程序库（DLL）中被new创建，却在另一个DLL内被delete销毁”。在许多平台上，这一类“跨DLL之new/delete成对运用”会导致运行期错误。tr1::shared_ptr没有这个问题，因为它缺省的删除器是来自“tr1::shared_ptr诞生所在的那个DLL”的delete。 

请记住：

- 好的接口很容易被正确使用，不容易被误用。你应该在你的所有接口中努力达成这些性质。
- “促进正确使用”的办法包括接口的一致性，以及与内置类型的行为兼容。
- “阻止误用”的办法包括建立新类型、限制类型上的操作，束缚对象值，以及消除客户的资源管理责任。
- tr1::shared_ptr支持定制删除器。这可防范DLL问题，可被用来自动解除互斥量等等。   

### 条款19：设计class犹如设计type

如何设计高效的classes呢？首先你必须了解面对的问题。几乎每一个class都要求面对以下提问，而你的回答往往导致你的设计规范：

- 新type的对象应该如何被创建和销毁？构造函数和析构函数以及内存分配函数和释放函数的设计。
- 对象的初始化和对象的赋值该有什么样的差别？ 决定于构造函数和赋值操作符的行为，以及其间的差异。
- 新type的对象如果被passed by value（以值传递），意味着什么？ copy构造函数用来定义一个type的passed by value该如何实现。
- 什么是新type 的“合法值”？ 对class的变量而言，通常只有某些数值集是有效的。那些数值集决定了你的class必须维护的约束条件，也就决定了你的成员函数必须进行错误检查工作。它也影响函数抛出异常、以及函数异常明细列。
- 你的新type需要配合某个继承图系(inheritancegraph)吗？ 如果你的class是从别的基类继承下来的，那么会受到基类的束缚。反之，如果你允许其他classes继承你的class，会影响你声明的函数尤其是析构函数是否是virtual。 
- 你的新type需要声明样的转换？显示or隐式。
- 什么样的操作符和函数对此新type而言是合理的？ 这个问题答案决定你将为你的class声明哪些函数。其中某些该是member函数，某些则否。
- 什么样的标准函数应该驳回？ 声明为private者。
- 谁该取用新type的成员？ 
- 什么是新type的“未声明接口”(undeclared interface) 
- 你的新type有多么一般化？ 并非定义一个新type，而是定义一个types家族，应该定义为class template。
- 你真的需要一个新type吗？ 如果只是定义新的derived class以便为既有的class添加机能，那么说不定单纯定义一个或多个non-member函数或templates，更能够达到目标。

请记住：

- Class的设计就是type的设计。在定义一个新的type之前，请确定你已经考虑过本条款覆盖的所有讨论主题。  

### 条款20：宁以pass-by-reference-to-const替代pass-by-value

1、缺省情况下C++以by value方式传递对象至函数。除非你另外指定，否则函数参数都是以实际实参的**副本为初值**，而调用端所获得的亦是返回值的一个副本。这些副本由对象的**拷贝构造函数产生**，这可能使得pass by value成为昂贵的操作。 

```c++
#include<iostream>  
#include<string>  
using namespace std;
class Person {
public:
	Person() { cout << "Person()" << endl; }
	Person(const Person& p) :name(p.name), address(p.address) {
		cout << "Person copy construct" << endl;
	}
	~Person() { cout << "~Person()" << endl; }
private:
	string name;
	string address;
};
class Student :public Person {
public:
	Student() { cout << "Student()" << endl; }
	Student(const Student& s):Person(s), schoolName(s.schoolName), schoolAddress(s.schoolAddress) {
		cout << "Student copy construct" << endl;
	}
	~Student() { cout << "~Student()" << endl; }
private:
	string schoolName;
	string schoolAddress;
};
//pass-by-value
bool validateStudent(Student s) {
	return true;
}
int main() {
	Student s;
	cout << "------------------" << endl;
	bool  b = validateStudent(s);
	return 0;
}
```

运行结果：

```
Person()
Student()
------------------
Person copy construct
Student copy construct
~Student()
~Person()
~Student()
~Person()
```

说明：从运行结果我们可以对validateStudent函数的参数的传递成本是“一次Student拷贝构造函数调用和一次Student析构函数调用”。其实远不止这些，因为上述两个类都含有两个string，他们也相应的调用拷贝构造函数，就是说四次string拷贝构造动作，还有四次析构函数。所以最终结果是以by value方式传递一个Student对象会导致一次Student拷贝构造函数、一次Person拷贝构造函数、四次string拷贝构造函数。当函数内的那个Student副本被销毁时，都要调用对应的析构函数。因此，以by value方式传递一个Student对象总体成本是“六次构造函数和六次析构函数”。 

避免构造和析构的方法，就是pass by reference-to-const。

```c++
bool validateStudent(const Student& s){  
    return true;  
} 
```

 运行结果：

```
Person()
Student()
------------------
~Student()
~Person()
```

说明：这种传递方式的效率高得多：没有任何构造函数或析构函数被调用，因为没有任何新对象被创建。将它声明为const是必要的，防止传入的Student对象被改变。 

2、以by reference方式传递参数可以避免slicing（对象切割）问题。当一个derived class对象以by value方式传递并视为一个base class对象，base class的copy构造函数会被调用，而“造成此对象的行为像个derived class对象”的那些特化性质全被切割掉了，仅仅留下一个base class对象。

以by reference方式传递参数可以避免切割问题。如果窥视c++编译器的底层，references往往以指针实现出来，因此pass by reference通常意味着这真正传递的是指针。对内置类型（例如int）而言，pass by value或pass by reference-to-const时，选择pass by value并非没有道理。这个忠告也适用于STL的迭代器和函数对象，因为习惯上它们都被设计为pass by value。 

slicing（对象切割）例子：

```c++
#include<iostream>  
#include<string>  
using namespace std;

class Window {
public:
	virtual void display()const {
		cout << "window display()" << endl;
	}
};
class WindowWithScrollBars :public Window {
public:
	virtual void WindowWithScrollBars::display() const {
		cout << "WindowWithScrollBars display()" << endl;
	}
};
void pintNaandWind(Window w) {
	w.display();
}

int main() {
	WindowWithScrollBars wwsb;
	pintNaandWind(wwsb);
	return 0;
}
```

输出结果：

```
window display()
```

说明：从运行结果可以看出，我们得到的是Window的display()，而我们本想得到WindowWithScrollBars的display()。它调用的是window对象的display()，而不是子类的，因为pintNaandWind中的参数传递是by value，WindowWithScrollBars的所有特化信息都会被切除。解决办法是 pass by reference-to-const。 

```c++
void pintNaandWind(Window& w){  
    w.display();  
}  
```

运行结果：

```
WindowWithScrollBars display()
```

请记住:

- 尽量以pass-by-reference-to-const替换pass-by-value。前者通常比较高效，并可避免切割问题(slicingproblem)。
- 以上规则并不适合与内置类型，以及STL的迭代器和函数对象。对它们而言，pass-by-value往往比较适合。

### 条款21：必须返回对象时，别妄想返回其reference

在坚定追求pass-by-value（传值）的纯度中，一定会犯下一个致命的错误：开始传递一些references执行其实并不存在的对象。这可不是什么好事。任何时候看到一个reference声明式，都必须立刻问自己，它的另一个名称是什么？因为它一定是某物的另一个名称。

函数创建新对象的途径有二：在栈空间或在堆空间创建。 

在stack空间创建对象例子：

```c++
class Rational{  
public:  
    Rational(int numerator, int denominator);  
    ...  
private:  
    int n, d;  
    friend const Rational operator*(const Rational& lhs, const Rational& rhs);  
};  
const Rational& operator*(const Rational& lhs, const Rational& rhs){  
    Rational result(lhs.n * rhs.n, lhs.d * rhs.d);  
    return result;  
}  
```

你可以拒绝这种做法，因为你的目标是避免调用构造函数，而result却必须像任何对象一样地由构造函数构造起来。更严重的是：这个函数返回一个reference指向result，但result是个local对象，而local对象在函数退出前就被销毁了。

在heap空间创建对象例子：

```c++
const Rational& operator*(const Rational& lhs, const Rational& rhs){  
    Rational* result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);  
    return result;  
}
```

在堆上构造一个对象，并返回reference指向它。会出现问题：谁该对着被你new出来的对象实施delete呢？还有一个问题：如果有计算：Rational result = x*y*z,其中x、y、z都是Rational类型的对象，此时就会产生内存泄露的问题。因为同一语句内调用了两次operator*，因而使用两次new，也就需要两次delete，但却没有合理的办法让operator * 使用者进行那些delete调用，因为没有办法取得operator*返回的references背后隐藏的那个指针。

一个“必须返回新对象”的函数的正确写法是：就让那个函数返回一个新对象。

请记住：

- 绝不要返回pointer或reference指向一个local stack对象，或返回reference指向一个heap-allocated对象，或返回pointer或reference指向一个local static对象而有可能同时需要多个这样的对象。 

### 条款22：将成员变量声明为private

1. 如果成员变量不是public，客户唯一能够访问对象的办法就是通过成员函数。如果public接口内的每样东西都是函数，客户就不需要再打算访问class成员时迷惑地试着记住是否该使用小括号。 

2. 使用函数可以让你对成员变量的处理有更精确的控制。如果你令成员变量为public，每个人都可以读写它，但如果你以函数取得或设定其值，你就可以出现“不准访问”、“只读访问”以及“读写访问”。 

   ```c++
   class AccessLevels{
   public:
       ...
       int getReadOnly() const {return readOnly;}
       void setReadWrite(int value) {readWrite=value;}
       int getReadWrite() const {return readWrite；}
       void setWriteOnly(int value) {writeOnly=value;}
   private:
       int noAccess;       //对此int无任何访问操作
       int readOnly;       //对此int做只读访问
       int readWrite;      //对此int做读写访问
       int writeOnly;      //对此int做惟写访问
   }；
   ```

3. 将成员变量隐藏在函数接口的背后，可以为“所有可能的实现”提供弹性。例如这可使得成员变量被读或写时轻松通知其它对象、可以验证calss的约束条件以及函数的前提和事后状态、可以在多线程环境中执行同步控制......等等 。

4. 从封装的角度观之，其实只有两种访问权限：private（提供封装）和其他（不提供封装）。书中以public或protected成员变量被取消而需要修改大量代码为例，来说明public和protected不具有封装性。

请记住：

- 切记将成员变量声明为private。这可赋予客户访问数据的一致性、可细微划分访问控制、允许约束条件获得保护，并提供class作者以充分的实现弹性。
- protected并不比public更具封装性。

### 条款23：宁以non-member、non-friend替换member函数

1. 面向对象守则要求，数据以及操作数据的那些函数应该被捆绑在一起，这意味着它建议member函数是较好的选择。这个建议并不正确，这是基于面向对象真实意义的一个误解。面向对象守则要求数据应该尽可能被封装，与直观相反，member函数带来的封装性比non-member函数低。

   member函数与non-member函数示例： 

   ```c++
   //功能：web浏览器清除下载元素高速缓存区、清除访问过的url、以及移除系统中所有cookies
   class WebBrower{
   public:
       ...
       void clearCache();
       void clearHistory();
       void removeCookies();
       ...
   };
   //执行所有这些动作，member函数
   class WebBrower{
   public:
       ...
       void clearEverything();
       ...
   };
   //non-member函数
   void clearBrowser(WebBrower& wb)
   {
       wb.clearCache();
       wb.clearHistory();
       wb.removeCookies();
   }
   ```

2. 如果在一个member函数（它不仅可以访问class内private数据，也可以取用private函数、enums、typedefs等等）和一个non-membernon-friend函数（它无法访问上述任何东西）之间做抉择，而且两者提供相同机能，那么，导致较大封装性的是non-member non-friend函数，因为它并不增加“能够访问class内之private成分”的函数数量。 这就解释了为什么clearBrowser比较clearEverything更受欢迎的原因：它导致WebBrowser class有更大的封装性。

3. 将所有便利函数放在多个头文件内但隶属同一个命名空间，意味客户可以轻松扩展这一组便利函数。需要做的就是添加更多non-member non-friend函数到此命名空间内。 

   ```c++
   naspace WebBrowserStuff{
       class WebBrowser{...};
       void clearBrowser{WebBrowser& wb};
   }
   ```

请记住：

- 宁可拿non-member non-friend函数替代member函数。这样做可以增加封装性、包裹弹性和机能扩充性。 

### 条款24：若所欲参数皆需要类型转换，请为此采用non-member函数

假设Rational class：

```c++
class Rational{  
public:  
    Rational(int numerator= 0, int denominator= 1);//构造函数刻意不为explicit，为了隐式类型转换  
    int numerator() const; 
    int denominator() const; 
private:  
    ...
}; 
```

 若要支持例如加法、乘法的算数运算符。例如，operator*写成Rational成员函数的写法：

```c++
class Rational{  
public:  
	const Rational operator*(const Rational& rhs) const;
}; 
```

这个设计可以使得两个有理数轻松相乘：

```c++
Rational oneEighth(1, 8);  
Rational oneHalf(1, 2);  
Rational result = oneHalf*oneEighth;  //正确
result=result*oneEighth//正确
```

但是执行混合运算，却只有一半行得通：

```c++
result=oneHalf*2;//正确
result=2*oneHalf;//错误!
```

重写上述两式：

```c++
result=oneHalf.operator(2);//正确
result=2.operator(oneHalf);//错误!
```

oneHlaf是含有operator*函数的class的对象，所以编译器会调用该函数。而整数2并没有相应的class。

为什么先前那个调用可以成功。注意其第二个参数是整数2，但Rational::operator*需要的实参却是个Rational对象。what happened？

这里发生隐式类型转换。编译器知道你正在传递一个int，而函数需要的是Rational；但它也知道只要调用Rational构造函数并赋予你提供的int，就可以变出一个适当的Rational来。此调用在编译器眼中就像：

```c++
const Rational temp(2);
result=onealf*tmp;
```

当然，只因涉及non-explicit构造函数，编译器才会这样做，否则上述两据都无法编译通过。

让operator*成为一个non-member函数，便允许编译器在一个实参身上执行隐士类型转换：

```c++
class Rational{                 //不包括operator*
}；
const Rational operator*(const Rational& lhs,const Rational& rhs){    //成为一个non-member函数
    return  Rational(lhs.numerator()*rhs.numerator(),lhs.denominator()*rhs.denominator());
}

Rational oneRourth(1,4);
Rational result;
result=oneRourth*2;
result=2*oneRourth;  //都没有问题
```

请记住：

- 如果你需要为某个函数的所有参数(包括this指针所指的那个隐喻参数)进行类型转换，那么这个函数为non-member。

### 条款25：考虑写出一个不抛出异常的swap函数

1. 如果swap的缺省实现码对你的class或class template提供了可接受的效率，你不需要额外做任何事。任何尝试置换那种对象的人都会取得缺省版本，而那将有良好的运作。 

   ```c++
   namespace std{  
       template<typename T>  
       void swap(T&a, T&b)        //std:swap的典型实现  
       {                           //置换a和b的值  
           T temp(a);  
           a = b;  
           b = temp;  
       }  
   }
   ```

2. 如果swap缺省实现版的效率不足，那几乎总是意味着class或class template使用了某种pimpl手法(pointer to implementation)，pimpl手法指“以指针指向一个对象，内含真正数据”那种类型的设计的表现形式。 

   ```c++
   class WidgetImpl{
   public:
   	...
   private:
   	int a, b, c;
   	std::vector<double> v;
   	...
   };
   class Widget{ //这个class使用pimpl手法
   public:
   	Widget(const Widget& rhs);
   	Widget& operator=(const Widget&rhs)
   	{
   		...
   		*pImv = *(rhs.pImpl);
   		...
   	}
   	...
   private:
   	WidgetImpl* pImv; 
   };
   ```

   一旦要置换两个Widget对象值，我们唯一要做的就是置换其pImv指针，但缺省的swap所发并不知道这一点。它不只复制三个Widget，还复制WidgetImpl对象，非常缺乏效率。

   希望告诉std::swap:当Widgets被置换时真正该做的是置换其内部的pImpl指针。

   尝试以下工作：

   a. 提供一个public swap成员函数做真正的置换工作，然后将std::swap特化，令它调用该成员函数。 

   ```c++
   class Widget{  
   public:  
       ...  
       void swap(Widget& other)  
       {  
           using std::swap;  
           swap(pImpl, other.pImpl);  
       }  
       ...  
   };  
   namespace std{
       template<>
       void swap<Widget>(Widget&a,Widget&b)
       {
           a.swap(b);
       }
   }
   ```

   b. 在你的class或template所在的命名空间内内提供一个non-member swap，并令它调用上述swap成员函数。 

   ```c++
   namespace WidgetStuff{  
       ...  
       template<typename T>  
       class Widget { ... }  
       ...  
       template<typename T>  
       void swap(Widget<T>& a, Widget<T>& b)  
       {  
           a.swap(b);  
       }  
   }
   ```

   c. 如果正在编写的是一个class(而非class template),为你的class特化std::swap，并让它调用你的swap member函数。 

   ```c++
   namespace std{  
       template<>  
       void swap<Widget>(Widget& a, Widget& b)  
       {  
           a.swap(b);  
       }  
   }
   ```

   d. 最后，如果你调用swap,请确定包含一个using声明式，以便让std::swap在你的函数中曝光可见，然后不加任何namespace修饰符，赤裸裸地调用swap。 

请记住：

- 当std::swap对你的类型效率不高时，提供一个swap成员函数，并确定这个函数不抛出异常。
- 如果你提供一个member swap，也该提供一个non-member swap用来调用前者。对于class（而非templates），也请特化std::swap。
- 调用swap时应针对std::swap使用using声明式，然后调用swap并且不带任何“命名空间资格修饰”。
- 为“用户定义类型”进行std templates全特化是好的，但千万不要尝试在std内加入某些对std而言全新的东西。   

## 第五章 实现

### 条款26：尽可能延后变量定义式的出现时间

1. 只要你定义了一个变量而其类型带有一个**构造函数或析构函数**，那么当程序的控制流到达这个变量定义式时，你便得承受构造成本；当这个变量离开其作用域时，你便得承受析构成本。即使这个变量最终并为被使用，仍需耗费这些成本，所以应该尽量避免这种情形。 

   考虑下面这个函数，它计算同学密码的加密版本后而返回，前提是密码够长。如果密码短，函数会丢出一个异常，类型为logic_error:

   ```c++
   //改函数过早定义encrypted
   std::string encryptPassword(const std::string& password){
   	using namespace std;
   	string encrypted;
   	if (password.length() < MinimumPasswordLength){
   		throw logic_error("Password is too short");    
   	}
   	...
   	return encrypted;
   }
   ```

   如果有个异常被丢出，对象encrypted就没有被使用，但仍得付出encrypted的构造成本和析构成本。所以最好延后encrypted的定义式，直到确实需要它。 

   ```c++
   //延后encryted的定义，知道真正需要他
   std::string encryptPassword(const std::string& password){  
       using namespace std;  
       if (password.length() < MinimumPasswordLength){  
           throw logic_error("Password is too short");      
       }  
       string encrypted;  
       ...  
       return encrypted;  
   }  
   ```

   “通过默认构造函数构造出一个对象然后对它赋值”比“直接在构造函数时制定初值”效率差。 

2. “尽可能延后”的真正意义是：你不只应该延后变量的定义，直到非得使用该变量的前一刻为止，甚至应该尝试延后这份定义直到能够给它初值实参为止。这样不仅能够避免构造和析构非必要对象，还可以避免无意义的default构造函数。 

3. 对于循环

   ```c++
   //方法A:定义循环外
   Widget w;
   for (int i = 0; i < n; ++i){
   	w = 取决于i的某个值;
   	...
   }     //1个构造函数+1个析构函数+n个赋值操作；
   //方法B：定义循环内
   for (int i = 0; i < n; ++i)
   {
   	Widget w(取决于i的某个值);
   	...
   }     //n个构造函数+n个析构函数
   ```

   以上两周写法的成本：

   - 做法A:1个构造函数+1个析构函数+n个赋值操作
   - 做法B:n个构造函数+n个析构函数

   如果classes的一个赋值成本低于一组构造+析构成本，做法A大体而言比较高效。尤其当n值很大的时候，否则做法B或许较好。此外做法A造成名称w的作用于比B大，有时对程序的可理解性和易维护性造成冲突。除非(1)你知道赋值成本比“构造+析构”成本低，(2)你正在处理代码效率高度敏感的部分，否则应该使用做法B。

请记住：

- 尽可能延后变量定义式的出现。这样做可增加程序的清晰度并改善程序效率。

### 条款27 ：尽量少做转型动作

转型（casts）破坏了类型系统。那可能导致任何种类的麻烦，有些容易辨识，有些非常隐晦。 

C++还提供四种新式转型：

- const_cast通常被用来将对象的常量性转除。它也是唯一有此能力的C++-style转型操作符。
- dynamic_cast主要用来执行“安全向下转型”，也就是用来决定某对象是否归属继承体系中的某个类型。它是唯一无法由旧式语法执行的动作，也是唯一可能消耗重大运行成本的转型动作。
- reinterpret_cast意图执行低级转型，实际动作（及结果）可能取决于编译器，也就表示不可移植。例如将一个pointer to int转型为一个int。
- static_cast用来强迫隐式转换，例如将non-const对象转为const对象，或将int对象转换为double等等。

任何一个类型转换往往真的令编译器编译出运行期间执行的码，例如将int型转换为double型几乎肯定会产生一些代码，因为在大部分计算机体系结构中，int的底层表述几乎不同于double的底层表述。但看下面的例子：

```c++
class Base{ ... };  
class Derived :public Base{ ... };  
Derived d;  
Base* pb = &d; 
```

上述例子建立一个base class指针指向一个derived class对象，但有时候上述两个指针值并不相同。这种情况下会有个偏移量在运行期被施行与`Derived* ` 指针身上，用以取得正确的`Base* ` 指针值。 

 例子：很多应用框架中都要求derived classes内的virtual函数代码第一个动作就是先调用base class对于函数。假设有一个Window base class和一个SpecialWindow derived class，两者都定义了virtual函数onResize。进一步假设SpecialWindow的onResize函数被要求调用Window的onResize。下面实现方式之一看起来正确，实际错误。

```c++
class Window{
public:
    virtual void onResize() {...}
    ...
};
class SpecialWindow:public Window{
public:
    virtual void onResize(){
        static_cast<Window>(*this).onResize();//错误
        ...
    }
    ...
};
```

这段程序将*this转型为Window，对函数onResize的调用也因此调用了Window::onResize。但令人想不到的是，它调用的并不是当前对象上的函数，而是稍早转型动作所建立的一个“ this指针对象的bass class成分“的暂时副本身上的onResize。

之所以需要dynamic_cast，通常是因为你想在一个认定为derived class对象身上执行derived class操作函数，但你却只有一个“指向base”的pointer或reference。 

请记住：

- 如果可以，尽量避免转型，特别是在注重效率的代码中避免dynamic_casts。如果有个设计需要转型动作，试着发展无需转型的替代设计。
- 如果转型是必要的，试着将它隐藏于某个函数背后。客户随后可以调用该函数，而不需要将转型放进他们自己的代码内。
- 宁可使用C++-style（新式）转型，不要使用旧式转型。前者很容易辨识出来，而且也比较有着分们别类的职掌。

### 条款28：避免返回handles指向对象内部成分 

1.例子：

```c++
#include<iostream>  
#include<memory>  
using namespace std;  
  
class Point{ //用来描述“点”  
public:  
    Point(int xVal, int yVal) :x(xVal), y(yVal){}  
    void setX(int newVal){ x = newVal; }  
    void setY(int newVal){ y = newVal; }  
    int getX()const{ return x; }  
    int getY()const{ return y; }  
private:  
    int x;  
    int y;  
};  
struct RectData{ //用来描述“矩形”  
    RectData(const Point& p1, const Point& p2) :ulhc(p1), lrhc(p2){}  
    Point ulhc;//坐上  
    Point lrhc;//右下  
};  
class Rectangle{  //矩形类  
public:  
    Rectangle(RectData data) :pData(new RectData(data)){}  
    Point& upperLeft()const{ return pData->ulhc; }  
    Point& lowerRight()const{ return pData->lrhc; }  
private:  
    shared_ptr<RectData> pData;  
};  
  
int main(){  
    Point coord1(0, 0);  
    Point coord2(100, 100);  
    RectData data(coord1, coord2);  
    const Rectangle rec(data);  
    rec.upperLeft().setX(50);  
    cout << rec.upperLeft().getX() << " " << rec.upperLeft().getY() << endl;//左上角坐标从(0,0)变为了(50,0)  
  
    system("pause");  
    return 0;  
}  
```

 上述例子中upperLeft的调用者能够使用被返回的reference（指向rec内部的Point成员变量）来更改成员，但rec应该是不可变的（const）。我们可以得到以下两个结论：

a、变量的封装性最多等于“返回其引用”的函数的访问级别。

b、如果const成员函数传出一个reference，后者所指数据与对象自身有关联，而它又被储存于对象之外，那么这个函数的调用者可以修改那笔数据。

2、对象的引用、指针、迭代器都是所谓的handles，而返回一个“代表对象内部数据”的handle，会降低对象的封装性。在上述例子中，只要在upperLeft和lowerRight函数的返回类型加上const即可： 

```c++
const Point& upperLeft()const{ return pData->ulhc; }  
const Point& lowerRight()const{ return pData->lrhc; }  
```

即便如此，upperLeft和lowerRight函数还是返回了代表对象内部的handles，有可能在其他场合带来问题，如它可能导致dangling handles，即这种handles所指东西不复存在。 

请记住:

- 避免返回handles(包括引用，指针，迭代器)指向内部对象。遵守这个条款可增加封装性，帮助const成员函数的行为像个const，并将发生“虚吊号码牌”(dangling handles)的可能性降至最低。

转自：https://blog.csdn.net/ruan875417/article/details/47281885

### 条款29：为“异常安全”而努力是值得的

当异常抛出时，带有异常安全的函数会：

- 不泄露任何资源（条款13：资源管理类）
- 不允许数据败坏

异常安全函数提供以下三个保证之一：

- 基本承诺：如果异常被抛出，程序内的任何事物仍然保持在有效状态下。没有任何对象或数据结构会因此而败坏，所有对象都处于一种内部前后一致的状态。然而程序的现实状态不可预料。 
- 强烈保证：如果异常被抛出，程序状态不改变。 
- 不抛掷保证：承诺绝不抛出异常，因为他们总能完成它们原先承诺的功能。 

copy and swap会导致强烈保证。原则是为你打算修改的对象（原件）做出一份副本，然后在那副本身上做一切必要修改。若有任何修改动作抛出异常，原对象仍保持未改变状态。待所有改变都成功后，再将修改过的那个副本和原对象在一个不抛出异常的操作中置换（swap）。 

请记住：

- 异常安全函数（Exception-safefunctions）即使发生异常也不会泄漏资源或允许任何数据结构败坏。这样的函数区分为三种可能的保证：基本型、强烈型、不抛异常型。
- “强烈保证”往往能够以copy-and-swap实现出来，但“强烈保证”并非对所有函数都可实现或具备现实意义。
- 函数提供的“异常安全保证”通常最高只等于其所调用之各个函数的“异常安全保证”中的最弱者。

### 条款30：彻透了解inlining的里里外外

1. inline函数，看起来像函数，比宏好得多，可以调用它们又不需蒙受函数调用所招致的额外开销。

2. inline函数背后整体的整体观念是，将“对此函数的每一个调用”都以函数本体替换之。这样做可能增加目标码的大小。在一台内存有限的的机器上，过度热衷inlining会造成程序体积太大，即使有虚拟内存，inline造成的代码膨胀也会导致额外换页行为，降低指令高速缓存装置的击中率，以及伴随这些而来的效率损失。

3. inline只是对编译器的一个申请，不是强制指令。这项申请可以隐喻提出，也可以明确提出。隐喻方式是将函数定义于class定义式内： 

   ```c++
   class Person{
   public:
   	...
   	int age() const { return theAge; } //隐喻申请
   	...
   private:
   	int theAge
   };
   ```

   这样的函数通常是成员函数，friend函数也可被定义于class内，如果真是那样，它们也是被隐喻声明为inline。

   明确声明inline函数的做法则是在其定义式钱加上关键字inline。例如标准的max template：

   ```c++
   template<typename T>
   inline const T& std::max(const T& a, const T& b){
   	return a < b ? b : a;
   }
   ```

4. 大部分编译器拒绝将过于复杂（例如带有循环或递归）的函数inlining，而所有对虚函数的调用也都会使inlining落空。因为虚函数直到运行期才确定调用哪个函数，而内联函数意味执行前先将调用动作替换为被调用函数的本体。 

5. 一个表面上看似inline的函数是否真是inline，取决于你的建置环境，主要取决于编译器。编译器通常不对“通过函数指针而进行的调用”实施inlining。 

6. 构造函数和析构函数往往是inlining的糟糕候选人。（由编译器于编译期代为产生并安插在程序中的代码，可能存在于构造函数和析构函数中）。

7. 程序库设计者必须评估“将函数声明为inline”的冲击：inline函数无法随着程序库的升级而升级。例如f是程序库内的一个inline函数，客户将“f函数本体”编进其程序中，一旦程序库设计者决定改变f，所有用到f的客户端程序必须重新编译。但如果f是non-inline函数，客户端只需重新连接即可；如果是动态链接库，升级版函数甚至可以不知不觉地被应用程序吸纳。 

8. 大部分调试器面对inline函数都束手无策，因为你不能在一个不存在的函数内设立断点。 

请记住：

- 将大多数inlining限制在小型、被频繁调用的函数身上。这可使日后的调试过程和二进制升级(binary upgradability)更容易，也可使潜在的代码膨胀问题最小化，使程序的速度提升机会最大化。
- 不要只因为function templates出现在头文件，就将它们声明为inline。

### 条款31：将文件的编译依存关系降至最低

如果没有取得其实现代码所用到的class string，Date和Address的定义式，那么class Person无法通过编译。 

```c++
class Person{
public:
	Person(const std::string& name, const Date& birthday,
		const Address& addr);
	std::string name() const;
	std::string birthday() const;
	std::string address() const;
	...
private:
	std::string theName; //实现细节
	Date theBirthday;    //实现细节
	Address theAddress;  //实现细节
};

```

所以Person定义文件的最上方可能存在： 

```c++
#include<string>
#include"date.h"
#include"address.h"
```

 这么一来使得Person定义文件和其含入文件之间形成了一种编译依赖关系。如果这些头文件中有任何一个被改变，或者这些头文件依赖的其他头文件有任何改变，那么每一个含入Person class的文件就得重新编译，任何使用Person class的文件也必须重新编译。 

Handle classes可以解除接口和实现之间的耦合关系，从而降低文件间的编译依存性。 

解除接口和实现之间的耦合关系，从而降低文件间的编译依存性的方法还有Interface classes。 

请记住：

- 支持“编译依存性最小化”的一般构想是：相依于声明式，不要相依于定义式。基于此构想的两个手段是Handle classed和Interface classes。
- 程序库头文件应该以“完全且仅有声明式”（full and declaration-only forms）的形式存在。这种做法不论是否涉及templates都适用。 

