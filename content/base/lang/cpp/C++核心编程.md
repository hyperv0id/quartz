[TOC]

# 1-内存分配模型

# 2-引用
### 2.1-引用的基本使用

​	**作用**：给变量起别名

​	**语法**：`数据类型 &别名 = 原名`;

```C++
int main() {
    // 数据类型 &别名 = 原名；
    int a = 10;
    //创建引用
    int& b = a;
    printf("a=%d,b=%d\n",a, b);
    b = 999;
    printf("a=%d,b=%d\n", a, b);
    system("pause");
    return 0;
}
```

### 2.2-引用注意事项

- 引用**必须初始化**
- 引用初始化后，**不可改变**



```C++
int main()
{
    int a = 10;
    int b = 20;
    //int &c;//ERROR！！必须初始化
    int &c = a;
    c = b;//这里是赋值操作，并不是更改引用
}
```

### 2.3-引用做函数参数

**作用**：函数传参时，可以利用引用的技术让形参修饰实参

**优点**：简化指针修改实参

```C++
//值传递
void swap1(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
    cout << "01,a=" << a << endl;
    cout << "01,b=" << b << endl;
}
//指针传递
void swap2(int *a,int *b) {
    int* t = a;
    a = b;
    b = t;
    cout << "02,a=" << *a << endl;
    cout << "02,b=" << *b << endl;
}
//引用传递
void swap3(int &a,int &b) {

    int t = a;
    a = b;
    b = t;
    cout << "03,a=" << a << endl;
    cout << "03,b=" << b << endl;
}
int main() {

    int a = 0;
    int b = 9;
    swap1(a,b);
    cout << "main:a=" << a << endl;
    cout << "main:b=" << b << endl;
    swap2(&a,&b);
    cout << "main:a=" << a << endl;
    cout << "main:b=" << b << endl;
    swap3(a,b);
    cout << "main:a=" << a << endl;
    cout << "main:b=" << b << endl;
    system("pause");
    return 0;
}
```
**输出的结果为**：

> 01,a=9
> 01,b=0
> main:a=0
> main:b=9
> 02,a=9
> 02,b=0
> main:a=0
> main:b=9
> 03,a=9
> 03,b=0
> main:a=9
> main:b=0
> 请按任意键继续. . .


### 2.4-引用做函数返回值

注意：**不要返回局部变量引用**

用法：函数调用做左值

```C++
//返回局部变量
int& test01(int n) {
    int a = n; //存放在 栈区
    return a;
}
int& test02(){
    static int a = 666;/*静态变量，放在全局区，在程序结束后释放*/
    return a;
}
    int a = 0;
    int b = 9;
    int& c = test01(b);
    printf("%d\n", c);
    printf("%d\n", c);
    int& d = test02();
    printf("%d\n", d);
    printf("%d\n", d);
    test02() = 1000;//对a赋值为1000
    printf("%d\n", d);
    printf("%d\n", d);
    system("pause");
    return 0;
}
```

### 2.5-引用的本质

本质：在C++内部实现一个**指针常量**



```C++
//发现是引用，转换为int* const ref = &a;
void func(int &ref){
    ref = 100;
}
int main()
{
    int a = 10;
    //自动转换为int* const ref = &a;
    int &ref = a;
    ref = 20;
    cout << "a:" << a << endl;
    cout<< "ref:" <<ref<< endl;
    func(a);
    return 0;
}
```


















# 3-函数提高


### 3.1-函数默认参数

函数的形参列表中是可以有默认值的

语法：==返回值类型 函数名(参数=默认值){}==

```C++
int func(int a, int b = 10, int c = 10){
    return a+b+c;
}
//1.默认的必须是从右到左连续的，一个带默认值，右边全都得带默认值
//2.声明和实现只能有一个有默认参数
int func(int a=2,int b = 9);
int func(int a=2, int b = 9){return 0;}
//报错：func2：重定义默认参数
```









### 3.2-函数占位参数

C++中函数的形参列表可以有占位参数，用作占位，调用时必须填补该位置

**语法**：==返回值类型 函数名(数据类型){}==

现阶段意义不大，不过以后会用到该技术

```C++
void func(int a, int,int c) {
    cout << "this is a 占位函数" << endl;
}
//占位参数可以有默认值
void fun2(int a, int = 996, int c = 10) {
    cout << "this is a 占位函数可以有 默认值" << endl;
}
int main()
{
    func(0, 1, 2);
    fun2(10);
    return 0;
}
```









### 3.3-函数重载

#### 3.3.1概述

**作用**：函数名可以相同，提高复用性

**函数重载满足的条件：**

- 同一个作用域下

- 函数名相同

- 参数不同(**类型** / **个数 ** / **顺序**)

  **示例**：都在全局作用域，函数名为func，参数不同

  ```c++
  void func(int a) {
      cout << "func的调用\t" <<  "int a" <<endl;
  }
  void func(int a, int b) {
      cout << "func的调用\t" << "int a, int b" << endl;
  }
  void func(int a, double b) {
      cout << "func的调用\t" << "int a, double b" <<endl;
  }
  void func(double a, int b) {
      cout << "func的调用\t" << "double a, int b" << endl;
  }
  int main()
  {
      func(1);
      func(1, 2);
      func(1, 1.1);
      func(0.1, 1);
      return 0;
  }
  
  ```

  

**注意**：函数的返回值不可以作为函数重载的条件

```C++
int func(int a){
    return a;
}
//报错
```

#### 3.3.2注意事项

- 引用

```c++
void func(int &a) {//int &a=100;不合法
    cout << "func的调用\t" <<  "int &a" <<endl;
}
void func(const int& a) {const int &a=100;合法
    cout << "func的调用\t" << "const int& a" << endl;
}//两个函数属于类型不同
int main()
{
    int b = 10;
    func(b);
    const int c = 100;
    func(c);
    return 0;
}
```

- 默认参数

```C++
void func(int a,int b=99) {
    cout << "func的调用\t" <<  "int a,int b=99" <<endl;
}
void func(int a) {
    cout << "func的调用\t" << "int a" << endl;
}
//编译器蒙圈了
//func(10);//可以调用两个函数
```











# 4-类和对象
C++面向对象的三大特性为：==封装、继承、多态==
		C++认为万物皆有对象，对象上有其==属性==和==行为==
**例如**
	人作为对象，有姓名、身高体重...行为有吃喝玩乐
	神作为对象，有姓名，能力...行为有赐福、保护
	具有相同性质的对象，我们可以抽象为类，人属于人类，神属于神 类


### 4.1-封装

#### 4.1.1-封装的意义
封装是C++面向对象的三大特性之一
		封装的意义：

1. 将属性和行为作为一个整体，表现生活中的事物
2. 将属性和行为加以权限控制

##### 成员

**示例**1.1：创建一个⚪类，输出它的周长

```C++
const double PI = 3.141593;
class circle {
    //访问权限
public://公共权限
    //属性
    int m_r;
    //行为
    double calC() {//求周长
        return 2 * PI * m_r;
    }
};

int main()
{
    circle circle1;
    circle1.m_r = 10;
    cout << "周长为：" << circle1.calC() << endl;
    return 0;
}
```
**示例**1.2：学生类，属性：姓名、学号，可以赋值并打印学号姓名

```c++
class Student
{
public:
    string name;
    int ID;
    void set(){
        cout << "name:";
        cin >> name;
        cout << "ID:";
        cin >> ID;
    }
    void show() {
        cout << "name:" << name << endl << "ID:" << ID << endl;
    }
};
int main()
{
    Student a;
    a.set();
    a.show();
    return 0;
}
```

##### 访问权限

三种权限：

1. 公共权限    ==public==
2. 保护权限    ==protected==   类外不可访问
3. 私有权限    ==private==    类外不可访问

```c++
class library
{
public:
    string name;
    void func() {
        name = "BJTU_Library";
        book = "Nature";//类内可访问
        password = 88888888;//类内可访问
    }
protected:
    string book;
private:
    int password;
};
int main()
{
    library BJTL;
    BJTL.name = "wqer";
    //BJTL.book = "习近平谈治国理政";//类外不可访问
    //BJTL.password=188;//类外不可访问
    return 0;
}
```




#### 4.1.2-struct和class的区别

struct和class的 **唯一区别** 是： ==默认的访问权限不同==；

区别：

- struct默认为公共
- class默认为私有

```c++
懒得写了
```




#### 4.1.3-成员属性设置为私有

优点：

1. 可以自己==控制读写权限==
2. 对于写的权限，我们可以检测==数据的有效性==

```c++
class person {
public:
	void set_name(string NAME) {
		name = NAME;
	}
	void get_name() {
		cout << name << endl;
	}
	int get_age() {
		return age ;
	}
	void set_password(int a) {
		password = a;
	}
private:
	string name;//读写
	int age = 18;//只读
	int password;//只写
};

int main()
{
	person p;
	p.set_name("cjj");
	p.set_password(123456);
	cout << p.get_age() << endl;
	p.get_name();
}
```




### 4.2-对象的初始化和清理



#### 4.2.1构造函数和析构函数

- 一个对象或对象没有初始状态，对其使用的后果是未知的；

- 使用完一个对象或变量，没有及时清理，会造成安全问题



编译器会自动提供**构造**和**析构**函数，但是全部为**空实现**



- **构造函数**：创建对象时对成员==赋值==，由编译器自动调用，无需手动调用

- **析构函数**：对象==销毁前==系统自动调用，执行一些清理工作



**构造函数语法**：`类名(){}`

1. 构造函数没有返回值也不写void
2. 函数名称与类名相同
3. 可以有参数，一次可以发送重载
4. 程序在调用对象时会自动调用构造**一次**，无须手动调用，仅调用一次



析构函数语法**：`~类名(){}`

1. 析构函数没有返回值也不写void
2. 函数名称与类名相同，但在之前加上~
3. 析构函数不可以带参数，因此无法发生重载
4. 程序在**销毁**对象时会自动调用构造**一次**，无须手动调用，仅调用一次



```C++
class person {
public:
	person() {
		cout << "构造函数的调用" << endl;
	}
	~person() {
		cout << "析构函数的调用" << endl;
	}
};
int main()
{
	person p;
}
/*输出为：
构造函数的调用
析构函数的调用*/
```

**小思考**：

为什么没有析构函数的调用呢？

```c++
class person {
public:
	person() {
		cout << "构造函数的调用" << endl;
	}
	~person() {
		cout << "析构函数的调用" << endl;
	}
};
int main()
{
	person p;
    system("pause");
}//还没有到要销毁的时候呢，关闭的一瞬间会有“析构函数的调用”
```



#### 4.2.2构造函数的分类和调用

分类方式：

- 按**参数**分：有参构造和无参构造
- 按**类型**分：普通构造和拷贝构造



调用方式：

- 括号法
- 显示法
- 隐式转换法

分类**示例**：

```c++
class person {
public:
	int age;
	//构造函数
	person() {
		cout << "普通无参  构造函数的调用" << endl;
	}
	person(int a) {
		cout << "普通有参  构造函数的调用" << endl;
	}
	person(const person &p) {//写法为此
		age = p.age;
		cout << "拷贝  构造函数的调用" << endl;
	}
	~person() {
		cout << "析构函数的调用" << endl;
	}
};
```

调用**示例**：

```c++
void test1() {
//1.括号法
	person p1;//默认构造函数的调用
	person p2(10);//普通有参函数
	person p3(p2);//拷贝

	/*person p1();//错误
	编译器认为此为一个函数的声明，使用调用默认析构函数时不要加()*/


//2.显示法
	cout << endl << "接下来是显示法" << endl ;
	person p4;//默认
	person p5 = person(10);//有参构造
	person p6 = person(p5);//拷贝
	cout << endl << "匿名对象：" << endl;
	person(10);//当前行执行完后系统立即回收
	//别用拷贝构造函数初始化匿名对象
	//person(p6);//编译器认为是    person(p6)=person p6;


//3.隐式转换法
	cout << endl << "隐式转换法：" << endl ;
	person p7 = 10;//相当于person p4=person(10)
}
```



#### 4.2.3拷贝构造函数调用时机



- 利用原对象，初始化一个新对象
- 值传递给函数参数传值
- 以值方式返回局部变量

**EXAMPLE**：

```c++
//利用原对象，初始化一个新对象
void test01() {
	person p1(18);
	person p2(p1);
}
//值传递给函数参数传值
void func(person p) {

}
void test02() {
	person p;
	func(p);
}
//以值方式返回局部变量
person func2() {
	person p1;
	return p1;
}
void test03() {
	person p = func2();
}
```

#### 4.2.4构造函数调用规则

默认情况下，C++编译器至少给类添加3个函数

1. 默认构造（无参，函数体为空）
2. 默认析构（无参，函数体为空）
3. 默认拷贝，对属性值进行拷贝



构造函数调用规则：

- 用户定义有参，c++不在提供无参
- 用户定义拷贝，C++不再提供其他函数

```c++
void test01() {
	//拷贝函数被注释
	person p1;
	person p2 = person(p1);
	cout << "p2 age:" << p2.age << endl;//p2 age:18
}
```

#### 4.2.5==深拷贝与浅拷贝==

浅拷贝：简单的赋值拷贝操作

深拷贝：在堆区重新申请空间，进行拷贝操作

**EXAMPLE**：


```C++
class person {
public:
	int age;
	int* height;
	//构造函数
	person() {
		cout << "普通无参  构造函数的调用" << endl;
		age = 18;
	}
	person(int agee,int _height) {
		age = agee; 
		height = new int(_height);//创建在堆区
		cout << "普通有参  构造函数的调用" << endl;
	}
	//person(const person &p) {//写法为此
		//age = p.age;
		//cout << "拷贝  构造函数的调用" << endl;
	//}
	~person() {
		if (height != NULL) {
			delete height;
			height = NULL;
		}
		cout << "析构函数的调用" << endl;
	}
};

void test01() {
	//拷贝函数被注释
	person p1(18,178);
	cout << "p1:" << p1.age <<"height:"<<*p1.height<< endl;
	person p2(p1);
	cout << "p2 age:" << p2.age << "height:" << *p1.height << endl;//p2 age:18
}//调试报错
```

![](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/06/41559729_1144_20210115-123926-ae5f75.png)

浅拷贝的问题得由深拷贝解决；

```c++
class person{	
public:
    person(const person &p) {
        /*your code*/
	}
};
```



#### 4.2.6初始化列表

初始化属性

**语法**：`构造函数()：属性1(值)，属性2(值)...{}`

```C++
	person(int a,int b):height(a),age(b) {
		cout << "初始化 函数的调用" << endl;
	}
```


#### 4.2.7类对象作为类成员

C++中类的成员可以是另一个类的对象，我们称之为`对象成员`

但是在**创建与销毁**时到底谁先谁后？

```C++
class Phone {
public:
	Phone(string a) :model(a) {
		cout << "phone 构造函数正在调用" << endl;
	}
	~Phone() {
		cout << "phone 析构函数正在调用" << endl;
	}
	string model;
};

class Person {
public:

	Person(string a, string b):name(a),myphone(b) {
		cout << "person 构造函数正在调用" << endl;
	}
	~Person() {
		cout << "person 析构函数正在调用" << endl;
	}
	string name;
	Phone myphone;
};

void test01() {
	Person p("张三","华为");
}
```
> 输出为：
>
> phone 构造函数正在调用
> person 构造函数正在调用
> person 析构函数正在调用
> phone 析构函数正在调用

形象的比喻：先有零件后有装置，先拆毁装置，再销毁零件



#### 4.2.8静态成员

静态成员就是在成员变量和成员函数加上关键字static，称为静态成员

静态成员分为：

- 静态成员变量
  - 所有对象共享同一份数据
  - 在编译阶段分配内存
  - 类内声明，类外初始化
- 静态成员函数
  - 所有对象共享同一个函数
  - 静态成员函数只能访问静态成员变量



**静态成员变量**

```C++
class Person {
public:
	~Person() {
		cout << "person 析构函数正在调用" << endl;
	}
	static int age;//类内实现
private:
	static int height;//可有访问权限
};
//类外初始化
int Person::age = 18;
int Person::height = 50;

void test01(){
	Person p;
	Person p2;
	cout <<"p:"<< p.age << endl;
	cout <<"p2:"<< p2.age << endl;
	p2.age = 99;
	cout << "p:"<<p.age << endl;
	cout << "p2"<<p2.age << endl;
}

void test02() {
	//因为共享数据，所以有2种访问方式
	Person p;
	//1.通过对象访问
	cout << p.age << endl;
	//2.通过类名访问
	cout << Person::age << endl;
}
```
**静态成员函数**

```c++
class Person {
public:
	static int age;//类内实现
	static void func() {
		age = 100;
		//height = 100;//不可访问非静态成员变量，无法区分到底是哪个对象的属性
		cout << "static void func 的调用" << endl;
	}
private:
	int height;
};
//类外初始化
int Person::age = 18;

void test01() {
	Person p;
	//通过对象访问
	p.func();
	cout << p.age << endl;
	//通过类名访问
	Person::func();
	cout << p.age << endl;
}
```




### 4.3-C++对象模型和this指针


#### 4.3.1成员变量和成员函数分开存储

在c++中只有非静态成员变量才属于类的对象

```c++
class none {
};
class person {
	int a;
	int func() { return 0; }//函数和变量分开存储
	static int b, c, d, e, f, g;//不属于类的对象
};
void test01() {
	none p;
	//空对象占用内存空间为：1
	//C++编译器会给每个空对象也分配一个字节空间，是为了区分空对象占位
	cout << sizeof(p) << endl;
}
void test02() {
	person p;//size==4
	cout << sizeof(p) << endl;
}
```



#### 4.3.2this指针

通过4.3.1我们知道在C++中成员变量和成员函数是分开存储的

每一个非静态成员函数只会诞生一份函数实例，也就是说多个同类型的对象会共用一块代码那么问题是：这一块代码是如何区分那个对象调用自己的呢？



C++通过提供特殊的对象指针，this指针，解决上述问题。this指针**指向被调用的成员函数所属的对象**



this指针是隐含每一个非静态成员函数内的一种指针

this指针不需要定义，直接使用即可



this指针的用途：

- 当形参和成员变量同名时，可用this指针来区分


- 在类的非静态成员函数中返回对象本身，可使用return *this

```c++
class person {
public:
	int age;
	person(int age) {
		//age = age;同名无法赋值
		this->age = age;
	}
	person& addage(person& p) {
		this->age += p.age;
		return *this;
	}
};
void test01() {
	person p2(10);person p1(10);
	p2.addage(p1).addage(p1).addage(p1).addage(p1).addage(p1).addage(p1).addage(p1).addage(p1);
	cout << p2.age; 
}//输出90😄
```



#### 4.3.3空指针访问成员函数

c++中空指针可以调用成员函数，但是得注意有没有用到this

```c++
class Person {
public:
	void show_class_name() {
		cout << "this is class person" << endl;
	}
	void showage() {
		if (this == NULL) { return; }//加上此条语句后不会报错
		cout << "age=" << age << endl;
		
	}
	int age;
};
void test01() {
	Person* p = NULL;
	p->show_class_name();//没崩
	p->showage();//引发异常
}
```



#### 4.3.4const修饰成员函数



**常函数**：

- 成员函数后加const后我们称之为常函数
- 常函数内不可以修改成员属性
- 成员属性加关键字**mutable**后，在常函数中依然可以修改



**常对象**：

- 声明对象前加const称该对象为常对象
- 常对象只能调用常函数



**常函数**

```C++
class Person {
public:
	//this指针的本质是 指针常量 指向不可修改
	void showperson()const//const修饰this指针的指向，使其指向的值无法修改
	{
		//age = 100;//相当于this->age=100;
		height = 188;
	}
	int age;
	mutable int height;
};
```
**常对象**

```c++
class Person {
public:
	int age;
	mutable int height;
	void func()const {
		height = 188;
	}
	void func2() {
		height = 188;
	}
};
void test01() {
	const Person p;//变为常对象
	//p.age = 100;
	p.height = 1000;
	//p.func2();//常对象只能调用常函数
}
```



### 4.4友元

你家有客厅(public)，有卧室(private)

客人可以随便进入客厅，但是不能随便进入你的卧室，但是你也可以允许你的好基友进去



在程序中，有些私有属性 也想让类外的一些特殊的函数或者类进行访问，就需要用到**友元**(friend)的技术



友元的目的是让一些函数或类 访问另一个类中的私有成员



三种实现：

- 全局函数做友元
- 类做友元
- 成员函数做友元



#### 4.4.1全局函数做友元

```c++
class home {
	//关键标签
	friend void goodgay(home& myhome);
public:
	string setting_room;
	home() {
		setting_room = "客厅";
		Bedroom = "卧室";
	}
private:
	string Bedroom;
};
void goodgay(home &myhome) {
	cout << "goodgay 函数正在访问：" << myhome.setting_room << endl;
	//未加friend标签无法访问
	cout << "goodgay 函数正在访问：" << myhome.Bedroom << endl;
}

void test01() {
	home home_;
	goodgay(home_);

}
```



#### 4.4.2类做友元

**函数类外实现**：`类名::函数名(){}`

```c++
class home {
	friend class goodgay;
public:
	string setting_room;
	home();

private:
	string Bedroom;
};

class goodgay {
public:
	string name;
	home* room;
	goodgay();
	void visit();
};

//类外实现成员函数
home::home() {
	setting_room = "客厅";
	Bedroom = "卧室";
}
goodgay::goodgay(){
	//创建建筑物对象
	room = new home;
}
void goodgay::visit() {
	cout << "goodgay 正在访问" << room->setting_room << endl;
	//加入friend可访问卧室
	cout << "goodgay 正在访问" << room->Bedroom << endl;
}


void test01() {
	goodgay xiao_hao;
	xiao_hao.visit();
}
```



#### 4.4.3成员函数做友元

```C++
class home ;//先声明 类，在后面实现
class GoodGay {
public:
	string name;
	home* room;
	GoodGay();
	void visit();
};
class home {
	friend void GoodGay::visit();
public:
	string setting_room;
	home();
private:
	string Bedroom;
};


//类外实现成员函数
home::home() {
	setting_room = "客厅";
	Bedroom = "卧室";
}
GoodGay::GoodGay(){
	//创建建筑物对象
	room = new home;
}
void GoodGay::visit() {
	cout << "goodgay 正在访问" << room->setting_room << endl;
	//加入friend可访问卧室
	cout << "goodgay 正在访问" << room->Bedroom << endl;
}
```


### 4.5运算符重载

#### 4.5.1	"+ - x /"

作用：实现两个自定义数据类型相加的运算

**示例**：加号重载

```c++
class score {
public:
	int math;
	int english;
	int chinese;
	score();
	score(int m, int e, int c);
	score operator+(score& p);
};
score::score() {
	this->math = 60;
	this->english = 60;
	this->chinese = 60;
}
score::score(int m, int e, int c) {
	this->math = m;
	this->english = e;
	this->chinese = c;
}
score score::operator+(score& p) {
	score temp(0,0,0);
	temp.chinese = this->chinese + p.chinese;
	temp.english = this->english + p.english;
	temp.math = this->math + p.math;
	return temp;
}

void test01() {
	score cjj(100, 100, 100);
	score jcj(100, 100, 100);
	score jjc(100, 100, 100);
	cout << "三人总分为" << "\n语文:" << (cjj + jcj + jjc).chinese << "分\n数学"<< (cjj + jcj + jjc).math<<"分\n英语："<< (cjj + jcj + jjc).english <<"分"<<endl;
}
```

当然也可以这样实现(`不过比较鸡肋`)

```c++
class score {
public:
	int math;
	int english;
	int chinese;
	score();
	score(int m, int e, int c);
	//score operator+(score& p1, score& p2);
};
score operator+(score& p1, score& p2) {
	score temp(0,0,0);
	temp.chinese = p1.chinese + p2.chinese;
	temp.english = p1.english + p2.english;
	temp.math = p1.math + p2.math;
	return temp;
}
```

运算符重载 同时 发生函数重载

```c++
class score {
public:
	int math;
	int english;
	int chinese;
	score();
	score(int m, int e, int c);
	score operator+(int num);
};
score score::operator+(int num) {
	score temp(this->chinese + num,this->math+num,this->english+num);
	return temp;
}
score operator+(score& p1, score& p2) {
	score temp(0,0,0);
	temp.chinese = p1.chinese + p2.chinese;
	temp.english = p1.english + p2.english;
	temp.math = p1.math + p2.math;
	return temp;
}
score::score() {
	this->math = 60;
	this->english = 60;
	this->chinese = 60;
}
score::score(int m, int e, int c) {
	this->math = m;
	this->english = e;
	this->chinese = c;
}



void test01() {
	score cjj(100, 100, 100);
	cout << "三人总分为" << "\n语文:" << (cjj + (100 + 100)).chinese << "分\n数学"<< (cjj + (100+ 100)).math<<"分\n英语："<< (cjj + (100 + 100)).english <<"分"<<endl;
}
```





#### 4.5.2	"<<"左移运算符重载



可以输出用户自定义的数据类型



```c++
class score {
public:
	int math;
	int english;
	int chinese;
	score(int m, int e, int c);
};
//只能通过全局函数实现
void operator<<(ostream &cout,score &p) {
	//cout << p;
	cout << "语文成绩：\t" << p.chinese << endl;
	cout << "数学成绩：\t" << p.math << endl;
	cout << "英语成绩：\t" << p.english << endl;
}
score::score(int m, int e, int c) {
	this->math = m;
	this->english = e;
	this->chinese = c;
}
void test01() {
	score cjj(100,100,100);
	cout << cjj ;
}
```

以上代码存在瑕疵，当想执行以下代码时报错：

```c++
void test01(){
	score cjj(100,100,100),jjc(100,100,100);
    cout << cjj << jjc << endl;
}
```

该进如下：

```c++
//cout 可改成out、啥啥啥的
ostream& operator<<(ostream &cout,score &p) {
	//cout << p;
	cout << "语文成绩：\t" << p.chinese << endl;
	cout << "数学成绩：\t" << p.math << endl;
	cout << "英语成绩：\t" << p.english << endl;
	return cout;
}//成功实现链式编程
```

tip：可以通过添加友元实现输出私有属性

作业：实现右移运算符重载(cin>>)(●'◡'●)



#### 4.5.3	"++ --"

模拟实现整型变量

```c++
class inter {
	friend ostream & operator<<(ostream &cout, inter num);
public:
	inter() {
		myint = 0;
	}
	//重载++i运算符
	inter& operator++() {
		++myint;
		return *this;
		//返回引用，使函数可以连续使用，且每次都是对同一个数进行操作
	}
	//重载i++运算符
	//int 代表占位参数 重载函数
	inter operator++(int) {
		inter temp = *this;
		myint++;
		return temp;
	}
private:
	int myint;
};
ostream& operator<<(ostream& cout, inter num)
{
	cout << num.myint;
	return cout;
}
void test01() {
	inter myint1,myint2;
	cout << myint1++ << endl;
	cout << myint1++ << endl;
	//论返回引用的好处↓
	cout << ++(++(++(++myint2))) << endl;
	cout << myint2 << endl;
}
```



#### 4.5.4	"="

c++编译器至少给一个类4个函数：

1. 默认构造函数
2. 默认析构函数
3. 默认拷贝构造函数
4. 赋值运算符 operator=， 对属性值拷贝

如果类中有属性指向堆区，赋值时也会出现深浅拷贝的问题

```c++
class inter {
public:
	inter(int num) {
		this->num = new int(num);
	}
	~inter() {
		if (num != NULL) {
			delete num;
			num = NULL;
		}
	}
	inter& operator=(inter &myinter2) {
		//判断是否有数据在堆区，如果有则释放
		if (num != NULL) {
			delete num;
			num = NULL;
		}
		//深拷贝
		this->num = new int(*myinter2.num);
		return *this;
	}
	int* num;
};
void test01() {
	inter int1(666),int2(888),int3(999);
	int3 = int2 = int1;//赋值操作
	
	cout << *int1.num << endl << *int2.num << endl << *int3.num << endl;
//原本崩溃，因为在析构函数中堆区的内存重复释放
}
```



#### 4.5.5	"关系"

自定义比较

**EXAMPLE:**

```c++
class score {
public:
	int math;
	int chinese;
	int english;
	score(int a, int b, int c) {
		chinese = a;
		math = b;
		english = c;
	}
	bool operator==(score s) {
		if (s.chinese == chinese && s.english == english && s.math == math) {
			return true;
		}
		return false;
	}
};

void test01() {
	score p1(100, 100, 100);
	score p2(100, 100, 100);
	if (p1 == p2) {
		cout << "same" << endl;
	}
}
```



#### 4.5.6	仿函数

- 函数调用符()也可以重载
- 由于重载后方式非常像函数的调用，因此称之为仿函数
- 仿函数非常灵活，没有固定的写法



**示例**

```C++
class myprint {
public:
	//重载函数调用运算符
	void operator()(string test) {
		cout << test << endl;
	}
};

void test01() {	
    //匿名对象
    myprint()("qwertyukjbvcxz");
	myprint myprint;
	myprint("hello world");
}
```









### 4.6继承

**继承是面向对象三大特性之一**

有些类与类之间存在特殊的关系：

![image-20210118114240549](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/06/image-20210118114240549-ad6769.png)

我们发现，定义这些类时，下级别成员除了拥有上一级的共性，还有自己的特性。

这个时候我们就可以考虑利用继承的技术减少重复代码

#### 4.6.1继承的基本语法

我们看见很多网页中，都有公共的头部、底部、左侧列表，只有中心内容不同

接下来我们分别利用普通写法和继承写法来实现网页中内容，看看继承存在的意义以及好处



**普通实现**

```c++
class java {
public:
	void header() {
		cout << "公共头部" << endl;
	}
	void footer() {
		cout << "公共页脚" << endl;
	}
	void left() {
		cout << "java Python " << endl;
	}
	void content() {
		cout << "java 视频" << endl;
	}
};
class python {
public:
	void header() {
		cout << "公共头部" << endl;
	}
	void footer() {
		cout << "公共页脚" << endl;
	}
	void left() {
		cout << "java Python " << endl;
	}
	void content() {
		cout << "python 视频" << endl;
	}
};
void test01() {
	cout << "java 视频界面如下：" << endl;
	java ja;
	ja.header();
	ja.footer();
	ja.content();
	ja.left();
	cout << "python 视频界面如下：" << endl;
	python py;
	py.header();
	py.footer();
	py.content();
	py.left();
}
```

重复代码太多了，一看就是垃圾

**继承实现**

**语法**：class 子类: 继承方式 父类{}；

子类也称为派生类，父类也称为基类

```c++
class base {
public:
	void header() {
		cout << "公共头部" << endl;
	}
	void footer() {
		cout << "公共页脚" << endl;
	}
	void left() {
		cout << "java Python " << endl;
	}
};
class java:public base {//继承基础页面
public:
	void content() {
		cout << "java 视频" << endl;
	}
};
class python :public base {//继承基础页面
public:
	void content() {
		cout << "python 视频" << endl;
	}
};
void test01() {
	cout << "java 视频界面如下：" << endl;
	java ja;
	ja.header();
	ja.footer();
	ja.content();
	ja.left();
	cout << "python 视频界面如下：" << endl;
	python py;
	py.header();
	py.footer();
	py.content();
	py.left();
}
```

==成功减少重复代码==	ψ(｀∇´)ψ



#### 4.6.2继承方式

继承语法：`class 子类: 继承方式 父类`

**继承方式有3种**：

- 公共继承public
- 保护继承protected
- 私有继承private

![image-20210118120902227](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/06/image-20210118120902227-b12aa2.png)

**代码验证**一下:

```c++
class father {
public:
	int a;
protected:
	int b;
private:
	int c;
};
class son1:public father {
public:
	void func() {
		a = 10;//父类中public子类可访问
		b = 10;//父类中protect子类可访问
		//c = 10;
	}
};
class son2 :protected father {
public:
	void func() {
		a = 10;//父类中public子类可访问
		b = 10;//父类中protect子类可访问
		//c = 10;
	}
};
class son3 :private father {
public:
	void func() {
		a = 10;//父类中public子类可访问
		b = 10;//父类中protect子类可访问
		//c = 10;
	}
};
void test01() {
	son1 s1;
	s1.a = 100;
	//s1.b = 100;//保护权限类外访问不到，所以还是保护权限；
	son2 s2;//还是保护权限
	//s2.a = 100;//变为保护权限，类外无法访问
	//s2.b = 100;
	son3 s3;
	/*s3.a = 100;
	s3.b = 100;
	s3.c = 100;*/
}

```

子类的子类也满足此规则：

```c++
class grandson1 :public son1 {
public:
	void func() {
		a = 100;
		b = 100;
		//c = 100;
	}
};
class grandson2 :public son2 {
public:
	void func() {
		a = 100;
		b = 100;
		//c = 100;
	}
};
class grandson3 :public son3 {
public:
	void func() {
		//a = 100;
		//b = 100;
		//c = 100;
	}
};
```



#### 4.6.3继承中的对象模型









#### 4.6.4继承中构造和析构顺序

先有父后有子，销毁时后进先出

```c++
class father {
public:
	father() {
		cout << "father 构造" << endl;
	}
	~father() {
		cout << "father 析构" << endl;
	}

};
class son:public father {
public:
	son() {
		cout << "son 构造" << endl;
	}
	~son() {
		cout << "son 析构" << endl;
	}
};
void test01() {
	son s;
}
```





#### 4.6.5继承同名函数处理方式

当子类和父类出现同名成员，如何通过子类对象，访问到子类或父类中的成员



- 访问**子类**同名成员：直接访问
- 访问**父类**同名成员，加作用域



`代码奉上`：

```c++
class father {
public:
	int age = 40;

};
class son:public father {
public:
	int age = 18;
};
void test01() {
	son s;
	cout << "son age:" << s.age << endl;
    //父类函数被隐藏，需要加作用域，即使存在重载
	cout << "father age:" << s.father::age << endl;
}
```

[^cjj]: 父类成员、函数被编译器隐藏，无法直接访问，需要加作用域



#### 4.6.6继承同名静态成员处理方式

与非静态成员一致


```C++
class father {
public:
	static int age;
};
int father::age = 40;
class son:public father {
public:
	static int age;
};
int son::age = 18;
void test01() {
	//通过类对象访问
	son s;
	cout << "son age:" << s.age << endl;
	cout << "father age:" << s.father::age << endl;
	//通过类名访问
	cout << "son age:" << son::age << endl;
	cout << "father age:" << son::father::age << endl;
}
```

看懂`son::father::age`了吗：

- 前一个::表示通过**son访问父类信息**
- 后一个::表示访问**父类作用域下成员**












#### 4.6.7多继承语法

C++允许一个类继承多个类，也就是一个儿子认多个爹(吕布)

语法：`class 子类: 继承方式 父类1, 继承方式 父类2......`

多继承可能会引起父类中有同名成员，需要加作用域



> ==**C++实际开发中不建议用多继承**==


```C++
class father1 {
public:
	int age = 100;
};
class father2 {
public:
	int age = 40;
};
class son:public father1,public father2 {
public:
	int age = 18;
};
void test01() {
	//通过类对象访问
	son s;
	cout << "son age:" << s.age << endl;
	cout << "father1 age:" << s.father1::age << endl;
	cout << "father2 age:" << s.father2::age << endl;
}
```

多继承中父类出现同名，子类使用时要加作用域






#### 4.6.8菱形继承(钻石继承)

概念：

​		两个派生类继承同一个基类

​		又有同一个继承着两个派生类



![image-20210119191010164](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/06/image-20210119191010164-4944f7.png)



**继承问题**：

1. 🐏继承了动物数据，🐫继承了动物数据，当`草泥马`使用数据时，产生二义性
2. 草泥马继承动物数据继承了2份，但只需要继承一份即可



```C++
class animal {
public:
	int age;
};
class sheep :public animal {};
class camel :public animal {};
class CNM :public sheep, public camel {};
void test01() {
	CNM cnm;
	//cnm.age = 28;
	cnm.sheep::age = 18;
	cnm.camel::age = 18;
	//两个父类有相同数据
	cout << "sheep age:    " << cnm.sheep::age << endl;
	cout << "camel age：    " << cnm.camel::age << endl;
}
```

![image-20210120120752693](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/06/image-20210120120752693-6487ed.png)

需要用**虚继承**解决问题：

```c++
class animal {
public:
	int age;
};
class sheep :virtual public animal {};
class camel :virtual public animal {};
class CNM :public sheep, public camel {};
void test01() {
	CNM cnm;
	cnm.age = 28;
	cout << "sheep age:    " << cnm.sheep::age << endl;
	cout << "camel age：    " << cnm.camel::age << endl;
	cout << "age：" << cnm.age << endl;
}
```

![image-20210120121112022](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/06/image-20210120121112022-bec3d6.png)

`vbptr`:	虚基类指针，virtual base pointer；它指向`vbtable`

















### 4.7多态

#### 4.7.1多态的基本概念

**多态是C++面向对象三大特性之一**

多态分为两类：

- 静态多态：函数重载 和 运算符重载 属于静态多态，复用函数名
- 动态多态：派生类 和 虚函数 运行时多态



静态多态和动态多态的区别：

- 静态多态的函数地址早绑定----编译阶段确定函数地址
- 动态多态的函数地址晚绑定----运行阶段确定函数地址



**EXAMPLE**

```C++
class animal {
public:
	void speak() {
		cout << "animal is speaking;" << endl;
	}
};
class Cat:public animal {
public:
	void speak() {
		cout << "喵喵喵" << endl;
	}
};

//地址早绑定 在编译时确定函数地址
void doSpeak(animal& ani) //父类的指针可以直接指向子类对象
{
	ani.speak();
}
void test01() {
	Cat cat;
	doSpeak(cat);
}
```

修改方式：

```c++
class animal {
public:
    //虚函数
	virtual void speak() {
		cout << "animal is speaking;" << endl;
	}
};
class Cat:public animal {
public:
	void speak() {
		cout << "喵喵喵" << endl;
	}
};
class Dog :public animal{
public:
	void speak() {
		cout << "汪汪汪" << endl;
	}
};

void doSpeak(animal& ani) //父类的指针可以直接指向子类对象
{
	ani.speak();
}
void test01() {
	Cat cat;
	doSpeak(cat);
	Dog dog;
	doSpeak(dog);
	animal ani;
	doSpeak(ani);
}
```

![](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/06/image-20210120133243383-e7f524.png)

![image-20210120133455254](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/06/image-20210120133455254-1523de.png)

**总结：**

多态满足条件：

- 有继承关系
- 子类重写父类中的虚函数

多态使用条件

- 父类指针或引用指向子类对象



重写：`函数返回值类型 函数名 参数列表`完全一致，内容可以不一样；









#### 4.7.2多态案例一：计算器类

案例描述：分别用普通写法和多态技术设计实现两个操作数进行运算





多态优点：

- 代码组织机构清晰
- 可读性强
- 利于前期和后期拓展与维护



**普通写法**：


```C++
class Calculator {
public:
	
	double getRes(string str) {
		double ans = 0;
		if (str == "+") { ans = a + b; }
		else if(str=="-"){ ans = a - b; }
		else if (str == "*") { ans = a * b; }
		else if(str=="/"){ ans = a / b; }
		else { cout << "暂时无法计算此类复杂运算" << endl; }
		return ans;
	}
	double a, b;
};

void test01() {
	Calculator cal;
	cal.a = 24;
	cal.b = 6;
	cout << cal.a << "+" << cal.b << "=" << cal.getRes("+") << endl;
	cout << cal.a << "-" << cal.b << "=" << cal.getRes("-") << endl;
	cout << cal.a << "*" << cal.b << "=" << cal.getRes("*") << endl;
	cout << cal.a << "/" << cal.b << "=" << cal.getRes("/") << endl;
}
```

**多态写法**：


```C++
class Calculator {
public:
	double a, b;
	virtual double getRes() {
		return 0;
	}
};
class Add :public Calculator {
public:
	double getRes() {
		return a + b;
	}
};
class Minus :public Calculator {
public:
	double getRes() {
		return a - b;
	}
};
class Product :public Calculator {
public:
	double getRes() {
		return a * b;
	}
};
class Divide :public Calculator {
public:
	double getRes() {
		if (b == 0) { cout << "被除数不能是0" << endl; }
		return a / b;
	}
};
void test01() {
	Calculator* cal = new Add;
	cal->a = 24; cal->b = 6;
	cout << cal->a << " + " << cal->b << " = " << cal->getRes() << endl;
	delete cal;
	cal = new Minus;
	cal->a = 24; cal->b = 6;
	cout << cal->a << " - " << cal->b << " = " << cal->getRes() << endl;
	delete cal;
	cal = new Product;
	cal->a = 24; cal->b = 6;
	cout << cal->a << " * " << cal->b << " = " << cal->getRes() << endl;
	delete cal;
	cal = new Divide;
	cal->a = 24; cal->b = 6;
	cout << cal->a << " / " << cal->b << " = " << cal->getRes() << endl;
}
```

calculator忘加virtual了，懵得一匹







#### 4.7.3纯虚函数 和 抽象类

在多态中，通常父类中虚函数的实现是毫无意义的，主要都是调用子类中重写的内容

所以干脆搞成**纯虚函数**算了~(￣▽￣)~*



纯虚函数语法：`virtual 返回值类型 函数名(XXX) = 0;`

当类中有了纯虚函数，这个类也称为==抽象类==



**抽象类特点**：

- 无法实例化对象
- 子类必须重写抽象类中的纯虚函数，否则也属于抽象类



**举个栗子**

```C++
class Base {
public:
	virtual void func() = 0;
};
class son :public Base{
public:
	void func() {
		cout << "good bye" << endl;
	}
};
void test01() {
	//无法实例化对象
	//Base base;
	//new Base;
	Base* base = new son;
	base->func();
}
```









#### 4.7.4多态案例二：制作饮品

制作饮品大致流程为：煮水 ~ 冲泡 ~ 倒入杯中 ~ 加入辅料

![image-20210121103508996](https://cdn.jsdelivr.net/gh/hooooolyshit/Pictures/2022/06/06/image-20210121103508996-01c25a.png)


```C++
class Base {
public:
	virtual void Boil() = 0;
	virtual void Brew() = 0;
	virtual void PourInCup() = 0;
	virtual void PutSth() = 0;
	void MakeDrink() {
		Boil();
		Brew();
		PourInCup();
		PutSth();
	}
};
class Coffee :public Base{
public:
	virtual void Boil() {
		cout << "煮农夫山泉ing" << endl;
	};
	virtual void Brew() {
		cout << "冲泡咖啡ing" << endl;
	};
	virtual void PourInCup() {
		cout << "目前已倒入星巴克杯中" << endl;
	};
	virtual void PutSth() {
		cout << "加入糖和牛奶" << endl;
	};
};
class Tea : public Base{
public:
	virtual void Boil() {
	cout << "煮山泉ing" << endl;
};
	  virtual void Brew() {
		  cout << "冲泡龙井ing" << endl;
	  };
	  virtual void PourInCup() {
		  cout << "目前已倒入瓷杯中" << endl;
	  };
	  virtual void PutSth() {
		  cout << "加入枸杞" << endl;
	  };
};
void test01() {
	cout << "煮咖啡" << endl;
	Base* base = new Coffee;
	base->MakeDrink();
	delete base;
	cout << "\n煮茶" << endl;
	base = new Tea;
	base->MakeDrink();
    delete base;
}
```









#### 4.7.5虚析构和纯虚析构



多态使用时，如果子类中有属性开辟到堆区，那么父类指针在释放时无法调用到子类的析构代码



将父类中的析构函数改为**虚析构**或**纯虚析构**



**共性**

- 都可以解决父类指针释放子类对象
- 都需要具体函数*实现*

**区别**：

- 如果是纯虚析构，该类属于抽象类无法实例化对象




```C++

```









#### 4.7.6多态案例三：电脑组装









```C++

```












# 5.文件操作

头文件<==fstream==>

操作文件：

1. ofstream: 写
2. ifstream:   读
3. fstream： 读写



### 5.1文本文件

写文件步骤如下：

1. 包含头文件: `#include<fstream>`
2. 创建流对象：ofstream ofs;
3. 打开文件：ofs.open("文件路径",打开方式);
4. 写数据：ofs<<"";
5. 关闭文件：ofs.close();

文件打开方式

| 打开方式    | 解释                       |
| :---------- | -------------------------- |
| ios::in     | 为读文件而打开文件         |
| ios::out    | 为写文件而打开文件         |
| ios::ate    | 初始位置：文件尾部         |
| ios::app    | 追加方式写文件             |
| ios::trunc  | 如果文件存在先删除，再创建 |
| ios::binary | 二进制方式打开             |

PS：文件打开方式可以配合使用，利用 ==|== 操作符



```c++
#include<fstream>
using namespace std;


void test01() {
	ofstream ofs;
	ofs.open("text.txt", ios::out);
	ofs << "1234567865433" << endl;
	ofs.close();
}
```




### 5.2二进制文件


```C++

```