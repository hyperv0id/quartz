# C++提高编程

[TOC]

本阶段主要针对泛型编程和STL做详细讲解,探究C++更深层的应用

## 1.	模板

本阶段主要针对c++==泛型编程==和==STL==技术做详细讲解，探讨C++更深层的应用

### 1.1	模板概念



模板就是**通用的模具**，能大大提高**复用性**



![查看源图像](https://pic36.photophoto.cn/20150722/1190119732905698_b.jpg)



**PPT模板**§(*￣▽￣*)§



特点：

- 不可以直接使用，它只是一个框架
- 模板不是万能的







### 1.2	函数模板

- c++的另一种编程思想为==泛型编程==，主要技术实现就是模板
- C++提供两种模板机制**函数模板**和**类模板**

函数模板作用：建立一个通用函数，使函数返回值和形参类型可以不制定，用一个**虚拟的类型**来代表

#### 1.2.1	函数模板语法

语法:

```c++
template<typename YOURNAME>
函数声明或定义
```

template: 声明创建模板

typename：表示其后是一种数据类型，可用class代替

YOURNAME：通用的数据类型，通常为大写字母





#### 1.2.2	函数模板注意事项

- 传入数据类型与之对应的通用数据类型必须相同

```c++
template<typename T>
void mySwap(T& a, T& b) {
	T temp = a;
	a = b;
	b = temp;
}
void test01() {
	int a = 10, b = 20;
	char c = 'a';
	mySwap(a, b);
	//mySwap(a, c);
}
```

- 函数必须确定数据类型才能使用

```c++
template<typename P>
void foo() {
	cout << "nothing,just a joke." << endl;
}

void test02() {
	//foo();
	foo<int>();
	foo<double>();
}
```





#### 1.2.3	案例

实现==冒泡排序==

```c++
template<typename T>
void BubberSort(T arr[], int len)
{
	for (int i = 0; i < len; i++)
	{
		for (int j = 0; j < len-i-1; j++) {
			if (arr[j] < arr[j + 1]) {
				T temp = arr[j];
				arr[j] = arr[j + 1];
				arr[j + 1] = temp;
			}
		}
	}
}
```





#### 1.2.4	普通函数与函数模板的区别



区别

- 普通函数调用时可以发生自动类型转换
- 函数模板显示指定类型时可以发送自动类型转换
- 函数模板在自动类型推导时不可以隐式类型转换

普通函数

```c++
int myAdd(int a,int b) {
	return a + b;
}


void test01() {
	int a = 10; int b = 20; char c = 'c'; 
	cout << myAdd(a, b)<<"\n"<<myAdd(a,c);
};
```

函数模板

```c++
template<typename T>
T _myAdd(T a,T b) {
	return a + b;
}

void test02() {

	int a = 10; int b = 20; char c = 'c';
	cout << _myAdd(a, b) << "\n";
	//cout <<  _myAdd(a, c);
	//编译器没法推导出对应类型，可以return int/char 类型 
	cout << _myAdd<int>(a, c);
};
```





#### 1.2.5	普通函数与函数模板的调用规则

```C++ 
void myAdd(int a,int b) {
	cout << "普通函数的调用" << endl;
}

template<typename T>
void print(T a) {
	cout << "函数模板的调用" << endl;
}
template<typename T>
void print(T a, T b) {
	cout << "函数模板2.0版本调用" << endl;
}
void test01() {
	print(10);
	//通过空模板参数列表，强制调用
    print<>(10);
    
	print(10, 20);
};
```

如果函数模板能产生更好的**匹配**，那么调用函数模板

```C++
void myAdd(int a,int b) {
	cout << "普通函数的调用" << endl;
}
template<typename T>
void print(T a, T b) {
	cout << "函数模板2.0版本调用" << endl;
}
void test01() {
	print('a', 'b');
};
```

总结：有了函数模板还写普通函数搞莫斯





#### 1.2.6	模板局限性

例如

```C++
template<typename T>
void foo(T a, T b) {
	a = b;
}
```

无法对数组进行上述操作



```C++
template<typename T>
void foo(T a, T b) {
	if (a > b) { cout << "a>b" << endl; }
}
```

Person类等等自定义数据类型，运算符未重载无法使用；



==**BUT**==C++为我们提供了特定的重载，做特殊实现；

```C++
template<class T>
bool myCompare(T& a, T& b) {
	if (a == b)return true;
	return false;
}
template<> bool myCompare(Person& a, Person& b) {
	if (a.age == b.age)return true;
	return false;
}

void test01() {
	Person a, b;
	a.age = 18; b.age = 18;
	if (myCompare(a, b)) {
		cout << "a==b" << endl;
	}
};
```

[^cjj]: 可以看成补丁吗



### 1.3	类模板

- c++的另一种编程思想为==泛型编程==，主要技术实现就是模板
- C++提供两种模板机制**函数模板**和**类模板**



#### 1.3.1	类模板语法

作用：

​		建立一个通用类，类中成员数据类型可以不制定，用**虚拟的**来代表

语法：

```C++
template<typename T>
/*your class*/
```



**EXAMPLE**

```c++
template<class NAME,class AGE>
class Person
{
public:
	AGE age;
	NAME name;
	Person(NAME a, AGE b)
	{
		this->age = b;
		this->name = a;
	}
};
void test01() {
	Person<string, int> p("jon", 18);
	cout << p.name << endl << p.age << endl;
};
```





#### 1.3.2	类模板与函数模板区别

1. 类模板没有自动类型推导的使用方式
2. 类模板在模板参数列表中可以有默认参数



**EAXAMPLE**

```c++
template<class NAME,class AGE>
class Person
{
public:
	AGE age;
	NAME name;
	Person(NAME a, AGE b)
	{
		this->age = b;
		this->name = a;
	}
};
void test01() {
	//类模板没有自动类型推导
	//Person p("jon", 18);
	Person<string, int> p("jon", 18);

	cout << p.name << endl << p.age << endl;
};
```



```c++
template<class NAME=string,class AGE=int>
class Person
{
public:
	AGE age;
	NAME name;
	Person(NAME a, AGE b)
	{
		this->age = b;
		this->name = a;
	}
};
void test01() {
	//你没传就用默认的类型
	Person<> p("jon", 18);
	cout << p.name << endl << p.age << endl;
};
```





#### 1.3.3	类模板中成员函数创建时机

类模板成员函数与普通类成员函数创建时机有区别

- 普通类：**一开始**就创建
- 类模板：成员函数**调用时**创建



**示例**

```c++
class NormalClass1
{
public:
	void show1() {
		cout << "NormalClass1" << endl;
	}
};
class NormalClass2
{
public:
	void show2() {
		cout << "NormalClass2" << endl;
	}
};
template<class T>
class myClass
{
public:
	T obj;
	void foo1() {
		obj.show1();
	}
	void foo2() {
		obj.show2();
	}
};
//以上部分能通过编译

void test01() {
	NormalClass1 c1;
	myClass<NormalClass1> mc1;
	mc1.foo1();
	//mc1.foo2();
};
```





#### 1.3.4	类模板对象做函数参数

类模板实例化出对象，向函数传参的方式

|   三种方式   |                            |
| :----------: | -------------------------- |
| 指定传入类型 | 直接显示对象的传入类型     |
|  参数模板化  | 将对象中的参数变为模板传递 |
|   类模板化   | 将对象类型 模板化传递      |

```c++
template<class T1,class T2>
class myClass
{
public:
	T1 name;
	T2 age;
	myClass(T1 name, T2 age) {
		this->age = age;
		this->name = name;
	}
	void show() {
		cout << "name:" << this->name << "\tage:" <<this->age<< endl;
	}
};
//指定传入类型
void print1(myClass<string, int>& P) {
	P.show();
}
//参数模板化
template<class T1,class T2>
void print2(myClass<T1, T2>& P) {
	P.show();
	cout << "T1类型：" << typeid(T1).name() << endl;
	cout << "T2类型：" << typeid(T2).name() << endl;
}
//整个类模板化
template<class T>
void print3(T& P) {
	P.show();
	cout << "T类型：" << typeid(T).name() << endl;
}
void test01() {
	myClass<string, int>p("张三", 18);
	print1(p);
	print2(p);
	print3(p);
};
```

第一种最**直截了当**，在实际开发中也最常见；





#### 1.3.5	类模板与继承

**注意事项**：

1. 子类继承 类模板父类 时，子类的声明得指出父类的类型
2. 接1，如果不指定，则编译器无法为子类分配内存
3. 如果想灵活的指定出父类类型，子类也需变为类模板

**EXAMPLE**

```c++
template<class T>
class Base
{
public:
	T a;
};

//class Son : public Base{
//public:
//};
class Son1 : public Base<int> {
public:
};

template<class T1, class T2>
class Son2 : public Base<T1> {
public:
	T2 age;
	Son2() {
		cout << "T1:\t" << typeid(T1).name() << endl;
		cout << "T2:\t" << typeid(T2).name() << endl;
	}
};

void test01() {
	Son1 s1;
	//int----T1		char----T2
	Son2 <int,char>s2;
}
```





#### 1.3.6	类模板成员函数外实现

```c++
template<class T1,class T2>
class myClass
{
public:
	T1 name;
	T2 age;
	myClass(T1 name,T2 age);
};
template<class T1, class T2>
myClass<T1, T2>::myClass(T1 name, T2 age)
{
	this->age = age;
	this->name = name;
}

void test01() {
	myClass<string, int> a("张三",18);
	cout << a.name << "已经" << a.age<<"岁了" << endl;
}
```

类外实现时需要加上模板参数列表





#### 1.3.7	类模板分文件编写

**问题：**

- 类模板的成员函数创建时机是在调用阶段，导致分文件编写时链接不到

**解决方法：**

- 直接包含.cpp源文件
- 将声明同实现一起写道一个文件中，并更改文件名为.hpp(约定名称，可替换)



**示例**

```c++
//#include"myClass.h"
//编译器只看见了头文件，导致链接阶段时出错
#include"myClass.cpp"
```

​	myClass.hpp代码

```C++
#pragma once
#include<iostream>
#include<vector>
#include<string>

using namespace std;
template<class T1, class T2>
class myClass
{
public:
	T1 name;
	T2 age;
	myClass(T1 name, T2 age);
};

template<class T1, class T2>
myClass<T1, T2>::myClass(T1 name, T2 age)
{
	this->age = age;
	this->name = name;
}
```





#### 1.3.8	类模板与友元		

类模板配合友元函数的类内实现与类外实现



- 全局函数类内实现----直接在类内声明友元
- 全局函数类外实现----提前告知编译器全局函数的存在

**示例**

```c++
//提前让编译器知道Person类的存在
template<class T1, class T2>
class Person;
//提前让编译器知道函数的存在
template<class T1, class T2>
void printPerson2(Person<T1, T2>p) {
	cout << p.name << "已经" << p.age << "岁了" << endl;
}
template<class T1,class T2>
class Person
{
	//全局函数 类内实现
	friend void printPerson1(Person<T1, T2>p) {
			cout << p.name << "已经" << p.age << "岁了" << endl;
		}
	//全局函数 类外实现
    /**加一个空模板的参数列表**/
	friend void printPerson2<>(Person<T1, T2>p);
public:
	Person(T1 name, T2 age) {
			this->name = name;
			this->age = age;
		}
private:
	T1 name;
	T2 age;
};
void test01() {
	Person<string, int>p("张三", 18);
	printPerson1(p);
	p = Person<string, int>("cjj", 19);
	printPerson2(p);
}
```





#### 1.3.9	案例

实现通用数组类，要求如下

1. 对基础数据类型(int char bool...)存储
2. 将数组中数据存储到堆区
3. 构造函数中可以传入数组容量
4. 提供对应拷贝函数及operator=防止浅拷贝问题
5. 尾插法 尾删法 增加和删除数组中的元素
6. 通过下标访问数组中的元素
7. 获取数组中当前元素个数、数组容量



**示例**

hpp中代码

```C++
template<class T>
class MyArr
{
public:
	MyArr(int len);
	MyArr(const MyArr& a);
	~MyArr();
	T& operator[](int val);
	MyArr& operator=(const MyArr& a);
	int MaxLen();
	int size();
	void Push_Back(const T& a);
	void pop_back();
private:
	T* arr;
	int m_MaxLen;
	int m_size;
};

//有参构造
template<class T>
MyArr<T>::MyArr(int len)
{
	arr = new T[len];
	m_MaxLen = len;
	m_size = 0;
}

//拷贝构造
template<class T>
MyArr<T>::MyArr(const MyArr& a)
{
	this->m_MaxLen = a.m_MaxLen;
	this->m_size = a.m_size;
	//深拷贝，防止数据重复释放
	this->arr = new T[a.m_MaxLen];
	for (int i = 0; i < a.m_MaxLen; i++)
	{
		this->arr[i] = a.arr[i];
	}
}

//析构函数
template<class T>
MyArr<T>::~MyArr()
{
	if (this->arr != NULL) {
		delete[] arr;
		this->arr = NULL;
	}
}

template<class T>
T& MyArr<T>::operator[](int val)
{
	return this->arr[val];
}

template<class T>
MyArr<T>& MyArr<T>::operator=(const MyArr& a)
{
	if (this->arr != NULL)
	{
		delete[] this->arr;
		this->arr = NULL;
		this->m_MaxLen = 0;
		this->m_size = 0;
	}
	this->m_MaxLen = a.m_MaxLen;
	this->m_size = a.m_size;
	//深拷贝，防止数据重复释放
	this->arr = new T[a.m_MaxLen];
	for (int i = 0; i < a.m_MaxLen; i++)
	{
		this->arr[i] = a.arr[i];
	}
	return *this;
}

template<class T>
int MyArr<T>::MaxLen()
{
	return this->m_MaxLen;
}

template<class T>
int MyArr<T>::size()
{
	return this->m_size;
}

template<class T>
void MyArr<T>::Push_Back(const T& a)
{
	if (this->m_size == this->m_MaxLen)return;
	this->arr[this->m_size] = a;
	this->m_size += 1;
}


//尾删法
template<class T>
void MyArr<T>::pop_back()
{
	if (this->m_size == 0)return;
	this->arr[this->m_size - 1] = NULL;
	this->m_size -= 1;
}

```



```C++
class Person
{
public:
	Person()
	{
		name = "";
		age = 0;
	}
	Person(string name, int age) {
		this->name = name;
		this->age = age;
	}
	void print() {
		cout << this->name << "have already " << this->age << "years old" << endl;
	}
private:
	string name;
	int age;
};

void test02() {
	MyArr<Person>myarr1(5);
	Person p1("张三", 20);
	Person p2("李四", 23);
	Person p3("王五", 21);
	Person p4("赵六", 29);
	Person p5("丁七", 18);
	myarr1.Push_Back(p1);
	myarr1.Push_Back(p2);
	myarr1.Push_Back(p3);
	myarr1.Push_Back(p4);
	myarr1.Push_Back(p5);
	printPersonArr(myarr1);
};
```

------







## 2	STL初识

### 2.1	STL的诞生



- 软件界妄图建立可重复利用的东西

- C++ **面向对象**和**泛型编程**思想，目的就是为了提高复用性

- 大多数情况下，数据结构和算法都未能有一套标准，被迫重复工作

- 为了建立数据结构和算法的一套标准，诞生了**STL**

  ------

  





### 2.2	STL的基本概念

- STL(Standard Template Library,**标准模板库**)

- STL从广义上分为：**容器(Container)****和**算法(Algorithm)**和**迭代器(Iterator)**

- **容器**和**算法**之间通过**迭代器**进行无缝连接

- STL 几乎所有代码都采用了模板类 或 模板函数

  ------

  





### 2.3	STL六大组件

**容器**、**算法**、**迭代器**、**仿函数**、**适配器**、**空间配置器**

1. 容器：各种数据结构，如vector、list、set、map、deque

2. 算法：常用算法，如sort、find、copy、for_each

3. 迭代器：容器和算法之间的胶合剂

4. 仿函数：行为类似函数，可作为算法的某种策略

5. 适配器：修饰容器或仿函数或迭代器接口

6. 空间配置器：空间配置与管理

   ------

   





### 2.4	STL中容器、算法、迭代器

**容器**；置物之所也，放数据用的

STL**容器**就是将运用最广泛的数据结构实现出来

常用数据结构：数组、链表、树、栈、队列、集合、映射表...

这些容器分为**序列式**和**关联式**：

​		**序列式**：强调值的排序，每个元素均有固定的位置

​		**关联式**：二叉树的结构，各元素间无严格物理上的顺序关系



**算法**：问题之解也

通过有限步骤解决逻辑或数学上的问题

算法分为**质变算法**和**非质变算法**：

​		质变：运算过程中会改变区间内元素的内容。如 拷贝、替换、删除

​		非质变：不会改变区间内元素的内容。如 遍历、计数、查找



**迭代器**：

提供一种方法，使之能依次访问容器中的各个元素，且无需暴露该容器的内部

每个容器有其专属迭代器

迭代器非常类似于指针

| 种类     | 功能                                   | 支持运算                  |
| -------- | -------------------------------------- | ------------------------- |
| 输入     | 只读                                   | 只读，++、==、!=          |
| 输出     | 只写                                   | 只写，++                  |
| 前向     | 读写，并能向前推进迭代器               | 读写，++、==、!=          |
| 双向     | 读写，并能向前向后推进迭代器           | 读写，++、--              |
| 随机访问 | 读写，以跳跃方式访问任意数据，功能最强 | 读写，++，--，[n],-n,<... |

常用的为双向和随机访问迭代器

------







### 2.5	容器算法迭代器初识





#### 2.5.1	vector存放内置数据类型

容器：`vector`

算法：`for_each`

迭代器：`vector<int>::interator`



**示例**

```C++
void myPrint(int val)
{
	cout << val << endl;
}

void test5_1_1()
{
	vector<int> v;

	//尾插数据
	v.push_back(10);
	v.push_back(11);
	v.push_back(12);
	v.push_back(13);
	v.push_back(14);

	vector<int>::iterator itsBegin = v.begin();//起始迭代器 指向第一个数据
	vector<int>::iterator itsEnd = v.end();//结束迭代器 指向最后一个数据的*下一个位置*
	//遍历方法1
	cout << "遍历方法1" << endl;
	while (itsBegin != itsEnd) {
		cout << *itsBegin << endl;
		itsBegin++;
	}
	//遍历方法2
	cout << "遍历方法2" << endl;
	for (vector<int>::iterator i = v.begin(); i < v.end(); i++)
	{
		cout << *i << endl;
	}
	cout << "遍历方法3" << endl;
	//遍历方法3	STL自带遍历方法
	//需包含头文件algorithm
	for_each(v.begin(), v.end(),myPrint);
}
```



**for_each**函数源码

```c++
template <class _InIt, class _Fn>
_CONSTEXPR20 _Fn for_each(_InIt _First, _InIt _Last, _Fn _Func) { // perform function for each element [_First, _Last)
    _Adl_verify_range(_First, _Last);
    auto _UFirst      = _Get_unwrapped(_First);
    const auto _ULast = _Get_unwrapped(_Last);
    for (; _UFirst != _ULast; ++_UFirst) {
        _Func(*_UFirst);
    }

    return _Func;
}
```

------





#### 2.5.2vector存放内置数据类型



```c++
#include "2.5.2 vector存放自定义数据类型.h"
class Person
{
public:
	Person(string name, int age) {
		this->name = name;
		this->age = age;
	}
	string name;
	int age;
private:

};
void printPerson5_1_2(const Person& p) {
	cout << p.name << "已经" << p.age << "岁了\n";
}
void printPerson5_1_2(Person *p) {
	cout << p->name << "已经" << p->age << "岁了\n";
}
void test5_1_2_1()
{
	vector<Person>v;
	Person p1("孙悟空", 9999);
	Person p2("猪八戒", 5000);
	Person p3("我自己", 9999999);
	Person p4("fghj", 678);
	v.push_back(p1);
	v.push_back(p2);
	v.push_back(p3);
	v.push_back(p4);
	for (vector<Person>::iterator i=v.begin(); i != v.end(); i++) {
		printPerson5_1_2(*i);
	}
}

void test5_1_2_2()
{
	vector<Person *>v;
	Person p1("孙悟空", 9999);
	Person p2("猪八戒", 5000);
	Person p3("我自己", 9999999);
	Person p4("fghj", 678);
	v.push_back(&p1);
	v.push_back(&p2);
	v.push_back(&p3);
	v.push_back(&p4);
	for (vector<Person *>::iterator i = v.begin(); i != v.end(); i++) {
		printPerson5_1_2(*i);
	}
}
```

------





#### 2.5.3vector容器嵌套容器

hpp中代码

```c++
#include<iostream>
#include<vector>
using namespace std;
void test5_3_1();

void test5_3_1() {
	vector<vector<int>>matrix(10);
	vector<int>v1;
	vector<int>v2;
	vector<int>v3;
	vector<int>v4;
	for (int i = 0; i < 4; i++)
	{
		v1.push_back(i);
		v2.push_back(i + 4);
		v3.push_back(i + 8);
		v4.push_back(i + 12);
	}
	matrix.push_back(v1);
	matrix.push_back(v2);
	matrix.push_back(v3);
	matrix.push_back(v4);
	for (vector<vector<int>>::iterator i = matrix.begin(); i < matrix.end(); i++) {
		for (vector<int>::iterator j = (*i).begin(); j < (*i).end(); j++) {
			cout << *j << " ";
		}
		cout << endl;
	}
}
```

------







## 3	STL-常用容器



### 3.1	string容器



#### 3.1.1	基本概念

**本质**

- string是C++风格的字符串，本质上是个类

**string 与 char*区别**

- char*是一个指针
- string是一个类，内部封装了char\*，管理这个字符串，是一个char\*类型的类

**特点**

string 类内部封装了很多成员方法(find copy delete insert replace)

------







#### 3.1.2	构造函数

- `string()`
- `string(const char *s)`
- `string(const string& str)`
- `string(int n, char c)`

例子就不写了

------







#### 3.1.3	赋值操作

功能描述：

- 给string字符串赋值



- `string& operator=(const char* s);`

- `string& operator=(const string &s);`

- `string& operator=(char c);`

- `string& assign(const char *s);`

- `string assign(const char *s,int n);`

- `string& assign(const string& str);`

- `string assign(int n, char c);`

  

  

  ```c++
  
  void test() {
  	string str1("qnmd Hello World");
  	string str2 = str1;
  	cout << str2 << endl;
  	string str3 ;
  	str3 = 'a';
  	cout << str3 << endl;
  	string str4;
  	str4.assign("awsl");
  	cout << str4 << endl;
  
  	string str5;
  	str5.assign(str4, 2);
  	cout << str5 << endl;
  
  	string str6;
  	str6.assign(str5);
  	cout << str6 << endl;
  
  	string str7;
  	str7.assign(3, '6');
  	cout << str7 << endl;
  }
  ```

------





#### 3.1.4	字符串拼接

| 语法                                             | 作用                                       |
| ------------------------------------------------ | ------------------------------------------ |
| `string& operstor+=(const char* str)`            | 重载+=运算符                               |
| `string& operstor+=(const char c)`               | 重载+=运算符                               |
| `string& operstor+=(const string& str)`          | 重载+=运算符                               |
| `string& append(const char *s, int n)`           | 字符串s拼接到当前字符串的结尾              |
| `string& append(const string &s)`                | 字符串s的前n个字符串拼接到当前字符串的结尾 |
| `string& append(const string &s, int pos,int n)` | 相当于operator+=                           |





#### 3.1.5	查找和替换

| 函数                                                  | 作用                               |
| ----------------------------------------------------- | ---------------------------------- |
| `int find(const string& str, int pos=0)const;`        | 寻找str第一次出现的位置, 从pos开始 |
| `int find(const char* s, int pos=0)const;`            | 从pos开始寻找s第一次出现的位置     |
| `int find(const char* s, int pos, int n)const;`       | pos开始寻找s前n个字符第一个位置    |
| `int find(const char c, int pos=0)const;`             | c第一次位置                        |
| `int rfind(const string& sstr, int pos=npos)const;`   | npos开始str最后一次位置            |
| `int rfind(const char* s, iint pos,int n)const;`      | npos开始s最后一次位置              |
| `int rfind(const char c, int pos=0)const;`            | c最后一次出现位置                  |
| `string& replace(int pos, int n, const string& str);` | 替换pos开始n个字符 为 str          |
| `string& replace(int pos, int n, const char *s);`     | 替换pos开始n 个字符为 s            |



#### 3.1.6	字符串比较

`int compare(const string& s)const`

`int compare(const char* s)const`





#### 3.1.7	字符串存取



`char& operator[](int n);`	//[]方式存取

`char& at(int n);`	//at方式存取





#### 3.1.8	插入&删除

`string insert(int pos, CODE)`

CODE:

- `const char* s`——插入字符串
- `const string& s`——插入字符串
- `int n,char c`——指定位置插入n个c



`string& erase(int pos, int n=npos)`删除从Pos开始的n个字符





#### 3.1.9	子串



从字符串中获取想要的子串

`string substr(int pos=0,int n=npos)const`

返回有pos开始的n个字符组成的子串





### 3.2	vector容器





#### 3.2.1	vector基本概念

**功能**: 

- vector数据结构和 **数组非常相似**, but **vector**可以**动态扩展**



**动态扩展**:

- 不是扩展新空间, 而是找一块更大的空间拷贝, 删除原有空间







#### 3.2.7	互换容器



`swap(vec)`

**实际用途**

- 巧用swap可以收缩内存空间

```C++

```







### 3.3	deque容器

#### 3.3.1	基本概念

**功能**：

- 双端数组(vector 是单端), 可以对头端插入和删除



**deque** 与 **vector** 区别：

- vector 对于头部插入删除效率低, 数据量越大 效率越低
- **vector** 访问元素比 **deque**快



**deque**内部工作原理：

​	内部有个 **中控器**, 维护每段缓冲区中的内容, 每个缓冲区存放**真实**数据, 中控器维护的是缓冲区的地址

#### 3.3.2	构造函数

跟vector没什么不一样的

```c++
//注意 两个不同iterator

void printDeque(deque<int>& a) {
    for (deque<int>::iterator i = a.begin(); i < a.end(); i++)
    {
        *i = 100;
        cout << *i << endl;
    }
}
void printDeque(const deque<int>& a) {
    for (deque<int>::const_iterator i = a.begin(); i < a.end(); i++)
    {
        //*i = 100;
        cout << *i << endl;
    }
}
void test01() {
    deque<int> a = { 0,1,2,3,4,5,6,7,8,9 };
};
```





#### 3.3.3	赋值操作



```c++
int main()
{
    deque<int> a = { 0,1,2,3,4,5,6,7,8,9 };
    deque<int> b = a;
    for (int i = 0; i < 10; i++)
    {
        b.push_front(i);
    }    
    for (deque<int>::const_iterator i = b.begin(); i < b.end(); i++)
    {
        //*i = 100;
        cout << *i << endl;
    }
    return 0;
}
```







#### 3.3.4	大小操作



resize()	重新指定大小

resize(int a,b);	重新指定大小, 超过的用数据变量b填充

resize()	变小了, 后面的元素会被删除







#### 3.3.5	插入&删除





#### 3.3.6	数据存取





#### 3.3.7	排序

需包含头文件 **algorithm**

`sort(iterator begin, iterator end)`	对区间begin与end的元素进行排序

对于支持随机访问的迭代器的容器, 都可以利用sort算法



### 3.4	stack容器

**先进后出**(First In Last Out, FILO)的数据结构, 它只有出口

![image-stack容器](D:\University\Notes\MCM\Images\image-20210321215315577.png)

只有顶端元素才可以被外界使用, 不能遍历



#### 3.4.1	stack容器通用接口

**构造函数**

- `stcak<T> mystk`	采用模板类实现, stack对象的默认构造形式
- `stack(const stack &stack)`    拷贝构造函数

**赋值操作**

- `stack operator=(const stack &stk)`重载等号运算符

**数据存取**

- `push()`向栈顶添加元素
- `pop()`向栈顶移除第一个元素
- `top()`返回栈顶元素

**大小操作**

- `empty()`判断是否为空
- `size()`返回栈大小



### 3.5	queue容器

#### 3.5.1	queue基本概念

**先进先出**(FIFO)

![image-20210321220438100](D:\University\Notes\MCM\Images\image-20210321220438100.png)

只允许一侧传入数据, 另一侧传出数据

不允许遍历



#### 3.6.2	queue常用接口

**构造函数**

- `queue<T> que`默认构造函数
- `queue(const queue &que)`拷贝构造函数

**赋值操作**

- `queue& operator=(const queue &que)`

**数据存取**

- `push()`向队尾添加
- `pop()`移除队头第一个元素
- `back()`返回最后一个元素
- `front()`返回第一个元素

**大小操作**

- `empty()`判断堆栈是否为空
- `size()`返回栈大小



### 3.6	list容器

#### 3.6.1	list概念

就是链表，链表由一系列**节点**构成，节点存在数据域和指针域，通过指针域访问下一节点。

优点：

- 快速增加与删除

缺点：

- 占用空间大
- 遍历慢

STL中不仅有指向下一节点的指针，还有指向上一节点的指针。

**list**迭代器只支持前移与后移，属于双向迭代器



list插入和删除操作都不会使原有迭代器失效，而vector不可以



#### 3.6.2	list构造函数

- 默认构造函数`list<T> lst`
- 将[begin-end)区间的元素拷贝给自身`list(begin, end)`
- 将n个元素赋值给自身`list(n, element)`
- 拷贝构造`list(const list& lst)`





#### 3.6.3	list赋值与交换

- "="号重载`list<T>lst2 = lst1`
- 特定区间`assign(begin,end)`
- n个值`assign(n,element)`
- 与指定值互换：`swap(lst)`





#### 3.6.4	list大小操作

- `size()`返回个数

- `empty()`判断是否为空

- `resize(num)`重新指定容器长度

- `resize(num,element)`重新指定容器长度，若变长则用element填充

  





#### 3.6.5	list插入删除

- 头尾

  - `push_back(element)`尾部加入元素element
  - `pop_back()`删除尾部元素

  - `push_front(element)`头部添加
  - `pop_front()`头部删除

- 任意

  - `insert(pos, [值|区间|n个值])`在pos位置插入数据
  - `clear()`清空
  - `earse([区间|位置])`删除数据，并返回下一数据位置
  - `remove(element)`删除与element匹配的值









#### 3.6.6	list数据存取

返回首末元素

- `front()`返回第一个元素
- `back()`返回最后一个元素

迭代器只能前移后移，所以不能@某元素^_^





#### 3.6.7	list反转和排序

- 反转`reverse()`
- 排序`sort()`





#### _3.6.7	排序案例

```c++
#include <iostream>
#include<list>
#include<string>
using namespace std;
class Person
{
public:
	Person(string name,int age,int height);

	int age;
	int height;
	string name;
private:
};
bool comparePerson(const Person& p1, const Person& p2) {
		return p1.age > p2.age?
			true
			:(p1.age==p2.age?
				(p1.height>p2.height)
				:false);
}
Person::Person(string name,int age,int height)
{
	this->name = name;
	this->age = age;
	this->height = height;
}

void PracticeList() {
	list<Person>lst;
	lst.push_back(Person("cjj", 18, 173));
	lst.push_back(Person("zbc", 9, 110));
	lst.push_back(Person("hello", 10000, 119));
	lst.push_back(Person("david", 40, 200));
	lst.push_back(Person("C++", 80, 0));
	//不需要加括号
	lst.sort(comparePerson);
	for (list<Person>::const_iterator i = lst.begin(); i != lst.end(); i++)
	{
		cout << "name: "<< i->name <<"\tage: "<< i->age<<"\theight: "<< i->height <<endl;
	}
}

```








### 3.7	set容器

#### 3.7.1	set基本概念

所有元素在插入时会被**自动排序**

set本质上是**关联式容器**，底层结构是 **二叉树**实现





#### 3.7.2	set构造赋值

构造：

- `set<T> st`*默认构造函数*
- `set(const set& st)`*拷贝构造函数*

赋值：

- `set& operstor=(const set& st)`*重载等号运算符*



**EXAMPLE**：

```c++
	set<char> alphabet;
	for (int i = 'A'; i <= 'Z'; i++) {
		alphabet.insert(i);
	}
	for (int i = 'a'; i <= 'z'; i++) {
		alphabet.insert(i);
	}
	for (set<char>::iterator i = alphabet.begin(); i != alphabet.end(); i++)
	{
		cout << char(*i)<<"\t";
	}
```





#### 3.7.3	set大小交换

- `size()`
- `empty()`
- `swap()`

```c++
	set<int> myset;
	if (myset.empty())
	{
		myset.insert(666);
		myset.insert(888);
	}
	printf("%d", myset.size()); 
	set<int>s2;
	myset.swap(s2);
```



#### 3.7.4	set插值删除

- `clear()`清空
- `earse(pos)`删除迭代器所处位置的元素
- `insert(element)`插入元素
- `earse(pos1,pos2)`删除元素



```c++
void printset(set<char>& _set) {
	for (set<char>::iterator i = _set.begin(); i != _set.end(); i++) {
		cout << *i << endl;
	}cout << endl;
}
int main()
{
	set<char>abc;
	abc.insert('a');
	abc.clear();//==abc.erase(abc.begin(),abc.end());
	for (int i = 'A'; i < 'Z'; i++) {
		abc.insert(i);
	}printset(abc);
	abc.erase(abc.begin());
	printset(abc);
	abc.erase('c'); abc.erase('j');
	printset(abc);
}
```





#### 3.7.5	set查找统计

- `find()`
- `count()`对于set而言，结果不是0就是1

```c++
	set<char>abc;
	for (int i = 'a'; i < 'z'; i++) {
		abc.insert(i);
	}
	set<char>::iterator pos = abc.find('c');
	if (pos != abc.end()) { cout << "found it!\nHave: '"<<*pos<<"'X"<<abc.count('c')<< endl; }
	else { cout << "no found" << endl; }
```





#### 3.7.6	set与multiset区别

区别：

- multiset允许重复的元素
- set在插入时会返回插入结果，代表其是否插入成功
- multiset不会检测数据





#### 3.7.7	pair对组创建

`pair<type,type>p(value1, v2);`
`pair<type,type>p = make_pair(v1,v2);`



**示例**：

```c++

```







#### 3.7.8	set排序



1. 自定义排序规则

```c++
class MyCompare {
public:
	bool operator()(int a, int b)const {
		return a > b;
	}
};
void test01(){
	set<int> s1;
	s1.insert(50);
	s1.insert(40);
	s1.insert(30);
	s1.insert(20);
	s1.insert(10);
	for (set<int>::iterator it = s1.begin(); it != s1.end(); it++)
	{
		cout << *it << "  ";
	}cout << endl;

	set<int,MyCompare>s2;
	s2.insert(50);
	s2.insert(40);
	s2.insert(30);
	s2.insert(20);
	s2.insert(10);
	for (set<int, MyCompare>::iterator it = s2.begin(); it != s2.end(); it++)
	{
		cout << *it << "  ";
	}cout << endl;
}
```



2. 对自定义类排序

```C++
class Person {
public:
	int age;
	string name;
	int height;
	Person(int age, int height, string name) {
		this->age = age;
		this->height = height;
		this->name = name;
	}
};
class PersonCompare {
public:
	bool operator()(const Person &a,const Person &b)const {
		if (a.age == b.age)
			return a.height < b.height;
		return a.age > b.age;
	}
};
void test02() {
	set<Person,PersonCompare> group;
	Person p1(18, 173, "cjj");
	Person p2(19, 183, "cjj");
	Person p3(19, 193, "cjj");
	Person p4(20, 163, "cjj");
	Person p5(21, 103, "cjj");
	group.insert(p1);
	group.insert(p2);
	group.insert(p3);
	group.insert(p4);
	group.insert(p5);
	for (set<Person, PersonCompare>::iterator it = group.begin(); it != group.end(); it++) {
		cout << "name:"<< it->name <<" age:"<< it->age <<" height:" << it->height <<endl;
	}
}
```










### 3.8	map容器

**简介**:

- map中所有元素都是pair
- pair中第一个元素(key)起到索引的作用
- 所有元素会根据元素的键值自动排序

**本质**:

- map/multimap属于关联式容器，底层结构通过二叉树实现

**优点**

- 根据key值快速找到value值

map与multimap**区别**：

​	map不允许重复的key，而multimap允许

#### 3.8.1	构造与赋值

```c++
void printMap(map<int, int>& m1) {
	for (map<int, int>::iterator it = m1.begin(); it != m1.end(); it++) {
		cout << it->first << "\t" << it->second << endl;
	}
}
void test01() {
	map<int, int>m1;
    // 通过pair构造
	m1.insert(pair<int, int>(10, 1024));
	m1.insert(pair<int, int>(9, 512));
	m1.insert(pair<int, int>(8, 256));
	m1.insert(pair<int, int>(7, 128));
	m1.insert(pair<int, int>(6, 64));
	printMap(m1);
	// 拷贝构造
	map<int, int>m2(m1);
	printMap(m2);
	// 赋值
	map<int, int>m3;m3 = m2;
	printMap(m3);
}
```



#### 3.8.2	大小和交换

`size()`	返回容器中元素的数目

`empty()`	判断是否为空

`swap(map2)`	交换两个容器

**大小**

```c++
void test01()
{
    map<string, int>m;
    m.insert(pair<string, int>("cjj", 18));
    m.insert(pair<string, int>("jjc", 17));
    m.insert(pair<string, int>("jcj", 19));
    m.insert(pair<string, int>("abc", 1000));
    m.insert(pair<string, int>("China", 5000));
    for (map<string, int>::iterator it = m.begin(); it != m.end(); it++) {
        cout << it->first << "\t" << it->second << endl;
    }
    cout << "m.empty:"<<m.empty() << endl;
    cout << "m的大小为:" << m.size() << endl;
}
```

**交换**

```c++
void test02() {
    map<string, int>m1;
    m1.insert(pair<string, int>("cjj", 18));
    m1.insert(pair<string, int>("jjc", 17));
    m1.insert(pair<string, int>("jcj", 19));
    m1.insert(pair<string, int>("abc", 1000));
    m1.insert(pair<string, int>("China", 5000));
    map<string, int>m;
    m1.swap(m);
    for (map<string, int>::iterator it = m.begin(); it != m.end(); it++) {
        cout << it->first << "\t" << it->second << endl;
    }
}
```



#### 3.8.3	插入和删除

**功能描述**

- map容器插入数据和删除数据

**函数们**

- `insert(ele)`插入元素
- `clear()`清除所有元素
- `erase(position)`删除迭代器所指的元素
- `erase(begin,end)`删除[begin, end]之间的元素
- `erase(key)`删除值为key的元素



#### 3.8.4	查找与统计

- `fid(val)`查找key值为val的元素是否存在, 不存在返回end()
- `count(val)`统计key值为val的元素个数



**查找**

```c++
void find(map<int,char>& m) {
    map<int, char>::iterator pos = m.find(60);
    if(pos!=m.end()){
    cout << pos->first << " " << pos->second << endl;
    }
    else {
        cout << "没找到啊o(TヘTo)"<<endl;
    }
}
```

**统计**

```c++
void count(map<int,char>& m) {
    for (int i = 0; i < 100; i++) {
        int cnt = m.count(i);
        if (cnt != 0)cout << "一共有:" << cnt << "个" << i << endl;
    }
}
```



#### 3.8.5	排序

利用仿函数改变排序规则

同set, 不再赘述



### 案例-员工分组

**案例描述**

- 新招10员工, 需指派部门工作
- 员工信息：name，部门：策划，美术，研发
- 随机分配部门和工资
- multimap插入信息，key(部门), value(员工)
- 分部门显示员工信息



```c++

```







## 4	STL-函数对象

### 4.1	函数对象

#### 4.1.1	概念

- 重载**函数调用操作符**的类, 其对象称为**函数对象**
- **函数对象**使用重载时, 行为类似函数调用, 也叫**仿函数**

**本质**

​		是一个**类**, 不是一个函数

#### 4.1.2	使用

**特点**

- 函数对象在使用时, 可以像普通函数那样调用, 可有参数, 可有返回值
- 函数对象超出普通函数的概念, 函数对象可有自己的状态
- 函数对象可以作为参数传递

**示例**

```c++

```

### 4.2	谓词

#### 4.2.1	概念

- 返回bool类型的仿函数称为**谓词**
- operator()接收n个参数, 那么叫做n元谓词

#### 4.2.2	一元谓词

```c++
class Greater5
{
public:
    // 一元谓词
    bool operator()(int a) {
        return a > 5;
    }
};

void test01() {
    vector<int> arr;
    for (int i = 0; i < 9; i++) {
        arr.push_back(i);
    }
    vector<int>::iterator find = find_if(arr.begin(), arr.end(), Greater5());
    if (find != arr.end())cout << "It is"<<*find << endl;
    else cout << "No found" << endl;
}

```



#### 4.2.3	二元谓词

```c++
class More {
public:
    bool operator()(int a, int b) {
        return a > b;
    }
};
void test02() {
    vector<int> v;
    for (int i = 0; i < 10; i++)v.push_back(i);
    sort(v.begin(), v.end());
    cout << "排序前" << endl;
    for (vector<int>::iterator it = v.begin(); it != v.end(); it++) {
        cout << *it << "\t";
    }cout << endl;
    cout << "排序后" << endl;
    sort(v.begin(), v.end(), More());
    for (vector<int>::iterator it = v.begin(); it != v.end(); it++) {
        cout << *it << "\t";
    }cout << endl;
}
```



### 4.3	内建函数对象

#### 4.3.1	意义

**概念**：STL内建了一堆函数对象

**分类**：关系 算数 逻辑

**用法**：

- 这些仿函数产生的对象、用法和一般函数完全相同
- 使用内建函数对象, 需要引入头文件`functional`

#### 4.3.2	算数仿函数

**四则运算**

- 加`template<class T>T plus<T>`
- 减`template<class T>T minus<T>`
- 乘`template<class T>T multiplies<T>`
- 除`template<class T>T divides<T>`
- 取模mod`template<class T>T modulus<T>`
- 取反`template<class T>T negate<T>`



#### 4.3.3	关系仿函数

**关系对比**

- 等于`template<class T>bool equal_to<T>`
- 不等`template<class T>bool not_equal_to<T>`
- 大于`template<class T>bool greater<T>`
- 大于等于`template<class T>bool greater_equal<T>`
- 小于`template<class T>bool less<T>`
- 小于等于`template<class T>bool less_equal<T>`

#### 4.3.4	逻辑仿函数

1. 或`template<class T>bool logical_and<T>`
2. 与`template<class T>bool logical_or<T>`
3. 非`template<class T>bool logical_not<T>`

## 5	STL-常用算法











