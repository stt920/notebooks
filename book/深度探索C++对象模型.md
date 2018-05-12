# 深度探索C++对象模型

## [第一章 关于对象模型](https://www.cnblogs.com/tgycoder/p/5426628.html)

### C++对象模型

​	此模型中，Nonstatic data members被配置于一个class object之内，static data members则被存放在个别的class object之外。Static和nonstatic function members也被存放在个别的class object之外。Virtual functions则分两个步骤支持之：

1. 每一个class产生出一堆指向Virtual functions的指针，放在表格之中。这个表格被成为虚函数表（virtual table,vtbl)；
2. 每一个class object被安插一个指针，指向相关的virtual table。通常这个指针被称为vptr。vptr的设定和重置都由每一个class的构造函数、析构函数和拷贝赋值运算符自动完成。每一个class所关联的type_info object（用以支持runtime type identification,RTTI,运行时类型识别），也经由virtual table被指出来，通常放在表格的第一个slot。

![1.3](./pic/1.3.png)

​	上图为C++对象模型示例。主要**优点**在于它的空间和存取时间的效率；主要**缺点**则是，如果应用程序代码本身未曾改变，但所用到的class objects和nonstatic data members有所改变，那么应用程序代码同样需要重新编译。

### 关键词带来的差异

​	struct和class从语法本质上差别并不大，无非是二者的默认继承和默认成员访问级别不同。但从一般来说，我们习惯用struct来代表一些简单数据的集合，用class来代表更为复杂的封装、继承的数据。

### 对象的差异

​	C++程序设计模型直接支持三种程序设计范式

1. 程序模型；

2. 抽象数据类型模型；

3. 面向对象模型。

   需要多少内存能表现一个class object？（类的大小）一般而言：

- 其nonstatic data members的总和大小；（非静态成员变量）
- 加上任何由于数据对齐而填补上去的空间；
- 加上支持virtual而由内部产生的任何额外负担。（虚函数表指针，4字节）

#### 指针的类型

​	指向不同类型的指针有什么不同？

```c++
ZooAnimal *px；
int *pi;
Array<string> *px;
```

​	从内存的观点来说没有什么不同！它们三个都需要足够的内存来放置一个机器地址。"指向不同类型之各指针"间的差异，既不在其指针表示方法不同，也不在其内容（代表一个地址）不同，**而是在其所寻址出来的object类型不同**。也就是说，“指针类型”会教导编译器如何解释某个特定地址的内存内容及其大小。

```c++
class ZooAnimal{
public:
    ZooAnimal();
    virtual ~ZooAnimal();
    //……
    virtual void rotate();
protected:
    int loc;
    string name;
};

ZooAnimal za("Zoey");
ZooAnimal *pza=&za;
```

- 一个指向地址1000的整数指针，在32位机器上，其涵盖地址空间1000~1003

- 如果string是传统的8-bytes（包含一个4-bytes的字符指针和一个用来表示字符串长度的整数），那么一个ZooAnimal指针将横跨地址空间1000~1015（注：非静态成员变量（int4+string8）+虚函数指针（4）=16）

  ![1.4](./pic/1.4.png)

#### 加上多态后

​	定义一个Bear继承ZooAnimal：

```c++
class Bear:public ZooAnimal{
public:
    Bear();
    ~Bear();
    //……
    void rotate();
    virtual void dance();
    //……
protected:
	enum Dances{……};
    Dances dances_known;
    int cell_block;
};

Bear b("Yogi");
Bear *pb=&b;
Bear &rb=pb;
```

​	Bear object需要24bytes，也就是ZooAnimal的16bytes加上Bear所带的8bytes。可能的内存布局如图所示：

![1.5](./pic/1.5.png)

​	假设我们的Bear object放在地址1000处，一个Bear指针和一个ZooAnimal指针有什么不同

```c++
Bear b;
ZooAnimal *pz=&b;
Bear *pb=&b;
```

​	它们每个指针指向Bear object的第一个byte。其间差别是，**pb所涵盖的地址包含整个Bear object，而pz所涵盖的地址只包含Bear object中的ZooAnimal subobject**。

​	除了ZooAnimal subobject中出现的members，不能使用pz来直接处理Bear中的任何members。**唯一例外是通过virtual机制**（注：具体参考书上内容，virtual实现运行时多态）。

## 第二章 构造函数语义学



