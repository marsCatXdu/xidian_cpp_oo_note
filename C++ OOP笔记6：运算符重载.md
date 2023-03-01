# C++ OOP笔记6：运算符重载，友元

> 本文全部内容基于西安电子科技大学潘蓉老师的《面向对象程序设计》课程记录而成。更多其他技术类内容可关注我的掘金和知乎： https://juejin.cn/user/1996368848621319/posts、[李经纬 - 知乎 (zhihu.com)](https://www.zhihu.com/people/li-jing-wei-78/posts)
>
> 有其他意见和建议欢迎联系，QQ：1428319077

## 运算符重载：重新定义运算符

本质是函数的重载，我们需要为每个重载的运算符定义一个【运算符重载函数】，运算符该做的事情实际上会交给这个函数来做。这样的函数可以是类的成员函数或友元函数。

运算符的重载不会改变其本来的优先级和结合性

### 重载为类的成员函数

```
<函数类型> operator <运算符>(<参数表>) {	// 参数表中是要运算的对象，最多只有一个参数
	// 一个（右）操作数是参数，另一个（左操作数）则是调用该函数的对象
	函数体
}
```

### 单目运算符的重载

一般只有++和--

但注意，这俩运算符的  先自加 / 后自加  的效果是不同的，也就是说重载运算符函数的返回值不同，重载时需要区分。

```
<类型> operator ++()		// 前置
<类型> operator ++(int)	// 后置. 写个 int 仅仅用于区分，无实际用途
```

## 以定义复数的计算等进行说明

### 复数类

包含普通双目运算符的重载

```cpp
class Complex {
public:
	Complex(double r=0.0, double i=0.0) {
		real = r;
		imag = i;
	}
    const double Real() {return real; }
    const double Imag() {return imag; }
    Complex operator+(Complex &c);		// 重载加减
    Complex operator+(double &c);
    Complex operator-(Complex &c);
    Complex operator-(double &c);
private:
	double real, imag;
};

Complex Complex::operator+(Complex &c) {
    Complex tmp;
    tmp.real = real + c.real;
    tmp.imag = imag + c.imag;
    return tmp;
}

// 用法如下
Complex c1(...);
Complex c2(...);
Complex c3;
c3 = c1 + c2;	// 实际上是：c3 = c1.operator+(c2);
				// 左操作数必须是对象，左操作数用右操作数为参数调用运算符
```

双目运算符重载为成员函数时，仅能有一个显示指出的参数，另外还会隐含一个 this 指针，用以指向调用它的那个对象（比如上面的 c3 = c1.operator+(c2)），隐含的就是指向c1的。

因此，这种重载的算符是无法进行 c3 = 12 + c1 这样的计算的。如果想要进行这样的计算，则可将该双目算符重载为全局的友元函数，让参与运算的两个数都变成参数即可。



### 单目运算符重载

例：

```cpp
class A {
	float x, y;
	...
	A operator++() {
		A t;
		t.x = ++x;
		t.y = ++y;
		return t;
	}
	A operator++(int) {
		A t;
		t.x = x++;
		t.y = y++;
		return t;
	}
    /* 简便写法。如下例中，调用这个函数的就是 a，this指向的也是这个a本身。也就是把自己返回了
    A operator++() {
		++x;
		++y;
		return *this;
	}
	*/
};

A a(2, 3), b;
b = ++a;	// b = a.operator++(); 重载的运算符内没有参数
```



### 运算符重载为友元函数

运算符重载，本质上是一个全局函数。

这样的情况下，参数中必须有一个自定义对象（因为重载运算符的意义就在于可以使用“常见的”运算符来对自定义的类型进行运算），但并不再必须是左值——**参与运算的对象全部成为函数的参数**。

例如，一个双目运算符的两边全都是参数。因为这样重载算符的友元函数一般是全局函数，不存在普遍意义上的“属于谁”的概念。

```cpp
A a, b, c;
c = a + b;	// c = operator+(a, b)
c = ++a;	// c = operator++(a)
c += a;		// operator+=(c, a)
```

过去要求左操作数一定是对象，主要是因为重载的运算符函数属于左操作数所属类的成员函数，故要依赖于左操作数用右操作数为参数调用运算符。但现在，左操作数是什么都可以了。因为全局函数不需要依赖类的实例来进行调用。

##### 算符重载为类的友元函数

最多只能有两个参数

``` cpp
friend <函数值类型> operator <运算符>(<参数表>) {
	...
}
```

如果重载的是双目运算符，则第一个参数是左操作数，第二个是右操作数

例：

```cpp
class A {
private:
    int i;
public:
	...
	friend A operator+(A &, A &);	// 虽然声明写在 A 里面，但并不是 A 的成员函数
									// 这么写仅仅是说：有这么一个函数，它是A的友元函数
};
A operator+(A &a, A &b) {			// 并不属于 A ，所以不用加作用域算符
	A t;
	t.i = a.i + b.i;				// 计算示例
	return t;
}

// 用法
A a1, a2, a3;
a3 = a1 + a2;	// a3 = operator+(a1, a2);
```

友元单目例：

```cpp
// A operator++(A &a) ++为前置运算符
// A operator++(A &a, int) ++为后置运算符
class A {
private:
    int i;
public:
    friend A operator++(A &a) {		// 定义放在类定义中，仍然可以保证自己是个全局函数
        a.i++; 
        return a;
    }
    friend A operator++(A &a, int) {
        A t;
        t.i = a.i;
        a.i++; 
        return t;
    }
};

A a1, a2, a3;
a2 = ++a1;		// a2 = operator++(a1)
a3 = a1++;		// a3 = operator++(a1, 3)  这后边这个 3 其实没啥影响的样子
```



## 友元

**友元——private 的东西可以给和自己关系好的看**，友元关系是单向的，而且不能传递。

被声明为友元的函数 / 类可以直接访问当前类的 private 成员。

友元，可以是一个全局函数、另一个类的成员函数（友元函数）、或者是整个类（友元类），友元类的所有函数都是友元函数。

实际应用中应该尽量避免使用友元。

### 友元函数

**谨慎使用：友元函数常用于取数据成员值而不进行修改**

```
// 友元的声明只能出现在被访问类的定义中——我自己来决定谁是我的友
friend <函数类型> <函数名>(<参数表>);
friend class <类名>;
```

例：

```cpp
class A {
private:
	float x, y;
public:
	float Sum() {return x+y;}
	friend float Sum(A &a) { return a.x + a.y; }	//全局友元函数，需要指明对谁进行操作
};

A t1(3, 4), t2(10, 20);
t1.Sum();		// 成员函数调用
Sum(t2);		// 全局友元函数调用。它不属于任何类所以不用指明属于谁，但需要指明操作谁
```

友元函数无 this 指针，因此需要指明被操作的对象作为参数

访问权限关键字（public private protect）对友元函数没有影响，也就是说声明在哪里都一样用



**多数情况下，一个类的友元函数是某个类的成员函数，这样就能够实现类和类的通信**

```cpp
class A {
public:
	void fun(B &);
private:
    float x, y;
};

class B {
public:
	friend void A::fun(B &);
private:
    float m, n;
};

void A::fun(B &b) {	// 这样，A 类的函数就可以随便访问 B 了
    x = b.m + b.n;
}
```



### 友元类

两个类紧密耦合，一个类中的好多函数都需要访问另一个类中的成员

```cpp
class A {	// 被访问类
...
	friend class B;
};

class B {	// 可以对 A 的实例为所欲为
	...
    void showA(A &a) {	// 一定要在参数中指明实例才能操作——总不能操作一个概念吧
        ...
    }
};

A a1;
B b1;
b1.showA(a1);	// 使用
```



## 输入输出运算符的重载

此处将说明如何实现对于对象的输入和输出

注意，**只能重载为友元函数**——原因如下：

对于 cin>>a, cout<<a，这里的 >> 、<< 其实都是双目运算符。我们要做的其实是  cin.operator>>(a)，然而 cin 并不是 A 类的对象，自然不可能调用 A 类的函数。重载为全局的友元函数，则运算符两端的值都将作为参数填入运算符重载函数。即如： operator>>(cin, a)

```cpp
// 友元重载算符一般格式
friend istream& operator>>(istream &, MyClassName &)
// 返回值是一个 istream&， 也就是对输入流的对象的引用
// 两个参数，右操作数是我们要进行输入操作的对象的引用
istream& operator>>(istream &is, MyClassName &f) {...}
cin>>a;		// 本质：operator>>(cin, a)
cin>>a>>b;	// 可以连续使用，这样就是调用两次，cin>>a 的返回值就是相对于 b 的第一个参数

// 输出重载
friend ostream& operator<<(ostream &, MyClassName &)
cout<<a;	// operator<<(cout, a);


class A {
	float x, y;
public:
    ...
    friend istream& operator>>(istream &, A &);
    friend ostream& operator<<(ostream &, A &);
};
istream& operator>>(istream &is, A &a) {
    cout<<"input a: "<<endl;
    is>>a.x>>a.y;				// 系统定义的输入运算符。a.x、a.y都是基本数据类型
    return is;					// 把引用返回回去
}
ostream& operator<<(ostream &os, A &a) {
    cout<<"Object is: "<<endl;
    os<<a.x<<"\t"<<a.y<<endl;	// \t是 tab
    return is;
}
```



## 类型转换运算符重载

### 基本类型到类

直接将基本类型数据赋值给类时，会对这个基本类型进行强制类型转换——通过构造函数进行此操作。

进行此任务的构造函数，应该是一个仅有一个参数的构造函数，被称为【转换构造函数】。这个参数的类型就是被转换的类型。

比如，将一个单一的实数赋值给一个复数，在复数类中就需要有这样的一个构造器：

```cpp
Complex(double r) {
    cout<<"调用构造器"<<endl;
	real = r;
	imag = 0;
}
~Complex() {
    cout<<"调用析构器"<<endl;
}

// 使用
Complex c1(1.1);
Complex a(2.2);
a = 3.3;	// 如果有只有一个参数的构造器，可以用等号强制赋值
			// 这一句话会先进行 Complex(3.3)，生成一个临时对象，
			// 该临时对象将被赋给 a，然后临时对象调用析构器解构
```



### 类到基本类型

有一种叫做【类型转换函数】的东西，可以实现这样的转换。这个函数本质上是一个**类型转换运算符重载函数**，专门用于将类转换为基本类型。

由于是对特定的类进行操作，故这种函数只能被重载为某个类的成员函数。

```cpp
operator <返回的基本类型名> () {	// 最前面不加返回值类型。调用属于隐含调用，故不能加参数。
	...
	return <基本类型值>;
}
```



```cpp
class A {
	int i;
public:
	A(int a=0) {
        i=a;
	}
    operator int();
};
A::operator int() {
    return i;
}

...
A a(10);
cout<<a;		// 就可以直接印出一个10。这里进行了对类型转换函数的隐含调用
```

