## 列表初始化：list-initialization
* className objectName{}
```c++
class Foo
{
public:
	Foo(int) {}
private:
	Foo(const Foo &);
};
 
int _tmain(int argc, _TCHAR* argv[])
{
	Foo a1(123); //调用Foo(int)构造函数初始化
	Foo a2 = 123; //error Foo的拷贝构造函数声明为私有的，该处的初始化方式是隐式调用Foo(int)构造函数生成一个临时的匿名对象，再调用拷贝构造函数完成初始化
 
	Foo a3 = { 123 }; //初始化列表对类进行列表初始化　　列表初始化，私有的拷贝并不影响它
	Foo a4 { 123 }; //列表初始化　　列表初始化，私有的拷贝并不影响它
 
	int a5 = { 3 };
	int a6 { 3 };
	return 0;
}
```
* new操作等圆括号进行初始化的场合用**列表初始化**
```c++
int* a = new int { 3 };
double b = double{ 12.12 };
int * arr = new int[] {1, 2, 3};
```
[new和malloc](https://www.cnblogs.com/ywliao/articles/8116622.html)
  类初始化可以使用列表初始化需要满足：        
>（1）用户自定义构造函数。
（2）无私有或者受保护的非静态数据成员
（3）无基类
（4）无虚函数
（5）无{}和=直接初始化的非静态数据成员
* 用户自定义构造函数
**对于一个类声明了自己的构造函数的情形，在这种情况下使用初始化列表是编译器是不会报错的，操作系统会给变量一个随机的值**对于代码是很大的一个**坑**
```c++
struct Foo
{
	int x;
	int y;
	Foo(int, int){ cout << "Foo construction"; }//用户定义了构造函数
};
 
int _tmain(int argc, _TCHAR* argv[])
{
	Foo foo{ 123, 321 };
    //输出结果为：Foo construction -858993460 -858993460
    //有用户自定义构造函数的类使用初始化列表其成员初始化后变量值是一个随机值，因此用户必须以用户自定义构造函数来构造对象
	cout << foo.x << " " << foo.y;
	return 0;
}
```
* 类包含有私有的或者受保护的非静态数据成员
```c++
struct Foo
{
	int x;
	int y;
protected:
	double z;
};
 
int _tmain(int argc, _TCHAR* argv[])
{
	Foo foo{ 123,456,789.0 };//error C2440: 'initializing' : cannot convert from 'initializer-list' to 'Foo
    //z变量声明为static则可以用列表初始化
	cout << foo.x << " " << foo.y;
	return 0;
}
////////////////////z变量声明为static则可以用列表初始化////////////////////////
struct Foo
{
	int x;
	int y;
protected:
	static double z;
};
 
int _tmain(int argc, _TCHAR* argv[])
{
	Foo foo{ 123,456};//输出123 456
	cout << foo.x << " " << foo.y;
	return 0;
}

```
* 类含有基类或者虚函数
```c++
struct Foo
{
	int x;
	int y;
	virtual void func(){};
};
 
int _tmain(int argc, _TCHAR* argv[])
{
	Foo foo {123,456};//error C2440: 'initializing' : cannot convert from 'initializer-list' to 'Foo
	cout << foo.x << " " << foo.y;
	return 0;
}
```
* 基类
```c++
struct base{};
struct Foo:base
{
	int x;
	int y;
};
 
int _tmain(int argc, _TCHAR* argv[])
{
	Foo foo {123,456};//error C2440: 'initializing' :cannot convert from 'initializer-list' to 'Foo'
	cout << foo.x << " " << foo.y;
	return 0;
}
```
* 类中不能有{}或者=直接初始化的非静态数据成员
```c++
struct Foo
{
	int x;
	int y= 5;
};
 
int _tmain(int argc, _TCHAR* argv[])
{
	Foo foo {123,456};
	cout << foo.x << " " << foo.y;
	return 0;
}
```
**列表初始化防止类型收窄**
```c++
int a = 1.1; //OK
int b{ 1.1 }; //errorC++11中，使用列表初始化的类型收窄编译将会报错
 
float f1 = 1e40; //OK
float f2{ 1e40 }; //error
 
const int x = 1024, y = 1;
char c = x; //OK
char d{ x };//error
char e = y;//error
char f{ y };//error
```
## 初始化列表：std::initialzer-list


## 构造函数初始值列表　constructor initialize list
```c++
struct Foo
{
	int x;
	int y= 5;
	virtual void func(){}
private:
	int z;
public:
	Foo(int i, int j, int k) :x(i), y(j), z(k){ cout << z << endl; }//构造函数初始值列表
};
 
int _tmain(int argc, _TCHAR* argv[])
{
	Foo foo {123,456,789};//可以使用列表初始化方法
    Foo foo1(123,456,789);//构造函数初始值列表
	cout << foo.x << " " << foo.y;
	return 0;
}

```

[C++构造函数体内初始化与列表初始化的区别](https://blog.csdn.net/weixin_41675170/article/details/93485655)

