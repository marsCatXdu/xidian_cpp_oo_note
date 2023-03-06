# C++学习笔记1：浅尝命名空间、输入输出、常量、重载、模板

> 本文全部内容基于西安电子科技大学潘蓉老师的《面向对象程序设计》课程记录而成。更多其他技术类内容可关注我的掘金和知乎： [修复格式问题](https://juejin.cn/user/1996368848621319/posts)、[李经纬 - 知乎 (zhihu.com)](https://www.zhihu.com/people/li-jing-wei-78/posts)
>
> 有其他意见和建议欢迎联系，QQ：1428319077

C++是C的超集，有STL，支持OOP和泛型编程（Generic Programming）

C++特色：

- 有封装；
- 可以通过泛化、继承来实现程序的重用和多态。泛型可以编写一般化、可重用的算法并且对效率没有影响；
- 可以对函数、运算符进行重载。



泛型可以处理不同数据类型的数据。泛型程序可通过模板来实现，模板实现了数据类型、函数定义的参数化。



## 关于 #include

```cpp
#include <...>		// 尖括号写的是 C++ 自带的。编译器会去 include 目录进行搜索
#include "..."		// 一般自己写的。编译器首先到当前工作目录中去查找，再去 include 目录寻找
```



## namespace

- STL 中的东西都是在 std 中声明的，使用类等之前需要指明其所在的命名空间。

```cpp
using namespace std;  // 整个都用 std
using std::cout;      // 只且只有 cout 可以直接写出来而不用带命名空间
using std::cin;	      // 同上
```

- 命名空间可防止重名冲突



命名空间的定义和使用

```cpp
namespace ns1 {	       // declear
    int inflag;
    void g(int);
}
namespace ns2 {
    int inflag;
}
//-------------------------
ns1::inflag = 2;       // use
ns2::inflag = 1;
//-------------------------
using ns1::inflag;
inflag = 666;          // in ns1
ns2::inflag = 123;     // in ns2
ns1::g(123);           // call. namespace required.
//-------------------------
using namespace ns1;
g(456);	               // call. without namespace, for having using above.
```



C++也可以用没名字的命名空间。

这样的空间中的内容无法在其他文件中进行访问，只能在本文件中使用——从声明处开始直到其所在源文件结束处。

```cpp
int i = 666;
namespace {
    int i = 123;
}
int main () {
    cout << i;
}
// 会报错，【多重定义】
```



## CPP 输入输出

IO 由 I/O 流类库提供，cin 和 cout 是两个对象，分别代表标准输入输出设备。

### cout

cout 是一个输出流类的对象，“输出流”指的是从内存向输出设备流动的数据流。cout做的事情就是将要输出的数据插入到输出流对象中。也被称为“插入操作”。

```cpp
cout<<exp1<<exp2<<...<<endl;    // endl，换行。"<<"是“输出运算符”或“插入运算符”
```

### cin

标准输入流对象。输入流是从输入设备流向内存的数据流。

cin是从输入流对象中提取数据，也称为“提取操作”。

```cpp
cin>>var1>>var2>>...<<varn;     // “>>”：“输入运算符”/“提取运算符”
```



## 使用 const 定义常量

**C 中使用宏替换来定义常量。**

定义使用 #define 进行，在预编译时进行字符置换，又称“宏替换”

```c
#define PI 3.14
```

预编译是将C++代码转化为机器代码的第一个步骤。对于上面这行代码，这里的工作之一就是将代码中所有写作“PI”的字符全都换为 3.14，然后再进行后续的编译步骤



**C++ 使用 const 定义常量**

```cpp
const <datatypeName> <constName> = <expression>;  // 常量在定义时进行初始化
const int maxLine = 1000;                         // 初始化是常量赋值的唯一方式

// 下面这种在声明之后赋值的操作是错误的.
const int maxLine;
maxLine = 123;
```



**#define VS const**

- #define 无类型，const 有；

- 一些 IDE 仅能检查、调试 const；

- 使用 #define 可能出现奇怪的错误，如：

  ```cpp
  #define pow a+b
  pow*pow;       // 被换成 a+b*a+b，显然与设想不符
  ```



## 函数原型声明

若进行函数调用的位置在该函数定义位置之前，则须在调用前进行函数声明。

```
// 函数类型 函数名 (参数类型 [参数名称], ...)
```



## 函数重载

在同一作用域中使用同一函数名定义多个函数。这些函数的参数类型和个数（至少有一个）不同。只有返回值类型不同并不可以。

```cpp
int add(int a, int b);
double add(double a, double b);
```



## 函数模板

引入 template 的目的在于：解决大量使用重载时，会产生大量类似的多余代码的问题。

适用于函数体相同、函数参数个数相同、类型不同的情况。

```cpp
template<typename T 或 class T>	// T 是类型名

template<typename T>
T max(T a, T b) {
    return (a>b)?a:b;
}
```

函数模板不是确定的函数，编译器不会将模板本身变成指令。

编译器遇到与模板匹配的函数调用时，自动生成一个专用的重载函数，该函数的函数体与函数模板相同，但涉及类型的部分都被改变了。

例如：

```cpp
int max(int a, int b) {
    return (a>b)?a:b;
}
```

这是一个“模板函数”，意为"从模板产生的函数"，是函数模板的一个实例，只能处理一种类型的数据。

**函数模板是模板，模板函数是函数。**



使用多个类型的参数

```cpp
// template <class T1, class T2, class T3>

template<class T1, class T2>
T1 max(class T1, class T2) {
    return (a>b)?a:(T1)b;    // 返回时需要对 b 进行类型转换
}
```

