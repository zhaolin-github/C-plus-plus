# C++新特性
### 自动类型推断
自动类型推断从易用性和表达能力来看的改进角度来看，是c++11的一项重要改进。
## auto
编译器根据表达式类型。
自动决定变量的类型（C++14支持函数的返回值类型），使用auto的变量，其类型是是在编译期就确定了，由编译器自动填充。
#### 不使用auto:
```c++
for (vector<int>::iterator it = v.begin(),end = v.end();it != end; ++it)DefaultTestData
{
    func();
}

```
#### 使用auto:
```c++
for (auto it = v.begin(),end = v.end();it != end; ++it)
{
    func();
}

```
当容器类型未知时
不使用auto:
```c++

template <typename T>
void foo(const T& container)
{
  for (typename T::const_iterator it = v.begin(),end = v.end();it != end; ++it)//注意此处 const 引用要写 const_iterator 作为迭代器的类型
  {
    func();
  } 
}
```
如果 begin 返回的类型不是该类型的 const_iterator 嵌套类型的话，那实际上不用自动类型推断就没法表达:比如，如果我们的遍历函数要求支持C数组，不用自动类型推断，代码只能使用两个不同的重载：
```c++

template <typename T, std::size_t N>
void foo(const T (&a)[N])//
{
  typedef const T* ptr_t;
  for (ptr_t it = a, end = a + N;
       it != end; ++it) {
    // 循环体
  }
}

template <typename T>
void foo(const T& c)
{
  for (typename T::const_iterator
         it = c.begin(),
         end = c.end();
       it != end; ++it) {
    // 循环体
  }
}
```
使用 `auto`，再加上 C++11 提供的`全局 begin `和 `end `函数，上面的代码可以统一成：
```c++

template <typename T>
void foo(const T& c)
{
  using std::begin;
  using std::end;
  // 使用依赖参数查找（ADL）；
  for (auto it = begin(c),
       ite = end(c);
       it != ite; ++it) {
    // 循环体
  }
}
```

自动类型推断不仅降低了代码的啰嗦程度，也提高了代码的抽象性，使我们可以用更少的代码写出通用的功能
auto 实际使用的规则类似于[函数模板参数的推导规则](https://en.cppreference.com/w/cpp/language/template_argument_deduction)。当你写了一个含 auto 的表达式时，相当于把 auto 替换为模板参数的结果
* auto a = expr; 意味着用 expr 去匹配一个假想的 template <typename T> f(T) 函数模板，结果为值类型;
* const auto& a = expr; 意味着用 expr 去匹配一个假想的 template <typename T> f(const T&) 函数模板，结果为常左值引用类型;
* auto&& a = expr; 意味着用 expr 去匹配一个假想的 template <typename T> f(T&&) 函数模板，结果是一个跟 expr 值类别相同的引用类型。
## decltype
auto推断会丢失引用类型信息，而decltype推断保留引用类型信息。但auto和decltype的用法不同，auto直接用于定义变量，decltype获得一个表达式的类型结果可以跟类型一样使用。

两个基本用法:   
* decltype(变量名) 可以获得变量的精确类型。   
  
      如参数是无括号，则返回表达式实体的类型,如没有这样实体，或参数指定是重载函数则程序格式错误               
      a）如参数是未括号id-expression命名结构化绑定，则返回引用类型（C++ 17）   
      b）如参数是未括号id-expression命名非类型模板参数，则返回模板参数类型C++ 20） 
* decltype(表达式) （表达式不是变量名，但包括 decltype((变量名)) 的情况）可以获得表达式的引用类型.除非表达式的结果是个纯右值（prvalue），此时结果仍然是值类型。    
  
      a）如表达式值类别是xvalue(亡值)则返回T&&; 
      b）如表达式值类别为lvalue(左值)则返回T&; 
      c）如表达式值类别为prvalue(纯右值)则返回T 	
      d）如表达式是函数调用返回函数的返回数值的类型 
      e）表达式的内容是解引用操作，得到的是引用类型
```c++
int a;，
decltype(a) //会获得 int（因为 a 是 int）
decltype((a)) //会获得 int&（因为 a 是 lvalue）
decltype(a + a) //会获得 int（因为 a + a 是 prvalue）
```
auto 来声明变量限制是你需要在写下` auto `时就决定你写下的是个引用类型还是值类型。使用 `auto` 不能通用地根据表达式类型来决定返回值的类型。不过，`decltype(expr)` 既可以是值类型，也可以是引用类型。因此：
```c++
decltype(expr) a = expr;
decltype(x + y) z =x+y;
```
表达式很长的情况下，这种写法有些冗余，而且，有代码重复的潜在的问题。为此，C++14 引入了 `decltype(auto)` 语法:
```c++
decltype(auto) a = expr; 
decltype(auto) z =x+y; 
```
上述代码主要用在通用的转发函数模板中：你可能根本不知道你调用的函数是不是会返回一个引用。
泛型编程中结合auto，用于追踪函数的返回值类型:
```c++
template<typename T, typename U>
auto add(T t, U u) -> decltype(t + u) // return type depends on template parameters
                                      //  C++14:返回值可以用auto自动推导
{
    return t + u;
}
```

### 函数返回值类型推断
从 C++14 开始，函数的返回值也可以用 `auto` 或 `decltype(auto)` 来声明。同样的，用 `auto` 可以得到值类型，用 `auto&` 或 `auto&&` 可以得到引用类型；而用 `decltype(auto)` 可以根据返回表达式通用地决定返回的是值类型还是引用类型。在你不知道返回值类型是个reference还是个value并且你想把返回的类型forward给别人的时候你就可以用decltype(auto)：
```c++
// perfect forwarding of a function call must use decltype(auto)
// in case the function it calls returns by reference
template<class F, class... Args>
decltype(auto) PerfectForward(F fun, Args&&... args) 
{ 
    return fun(std::forward<Args>(args)...); 
}

```
### [类模板的模板参数推导](https://en.cppreference.com/w/cpp/language/class_template_argument_deduction)
```c++
pair<int, int> pr{1, 42};
auto pr = make_pair(1, 42);
```
这是因为函数模板有模板参数推导，使得调用者不必手工指定参数类型；但 C++17 之前的类模板却没有这个功能，也因而催生了像 make_pair 这样的工具函数。在进入了 C++17 的世界后，这类函数变得不必要了。现在我们可以直接写：
```c++
pair pr{1, 42};
```
 C++17 中 array 能像 C中 数组一样自动从初始化列表来推断数组的大小。虽然不能只提供一个模板参数，但可以两个参数全都不写：
 ```c++
 
int a1[] = {1, 2, 3};
array<int, 3> a2{1, 2, 3}; //c++11:std::arry 需要指定：类型　大小
// array<int> a3{1, 2, 3}; 不行
array a{1, 2, 3};//c++17
// deduce array<int, 3>
 ```
 c++17中的类模板自动推导机制
 *  编译器根据构造函数来自动生成(Implicitly-generated deduction guides)
```c++

template <typename T>
struct MyObj {
  MyObj(T value);
  …
};

MyObj obj1{string("hello")};
// 得到 MyObj<string>
MyObj obj2{"hello"};
// 得到 MyObj<const char*>
```
* 用户定义的推导向导（User-defined deduction guides）
```c++

template <typename T>
struct MyObj {
  MyObj(T value);
  …
};

MyObj(const char*) -> MyObj<string>;

MyObj obj{"hello"};
// 得到 MyObj<string>
```
### [结构化绑定](https://en.cppreference.com/w/cpp/language/structured_binding)
c++11/14之前，当返回值为pair:
```c++

multimap<string, int>::iterator
  lower, upper;
std::tie(lower, upper) =
  mmp.equal_range("four");
```
声明了两个变量，然后使用 tie 来接收返回的pair，在 C++17 引入了一个新语法**结构化绑定**（structure binding declearation）我们可以把上面的代码简化为：
```c++
auto [lower, upper] = 
  mmp.equal_range("four");
```
### 二进制字面量
C++ 的 0x 前缀： `0xFF //十六进制字面值常量`
0 后面直接跟 0–7 的数字，表示八进制的字面量： `123//十进制｀　而　　　　
｀0123//八进制`
C++14 开始，我们对于二进制也有了直接的字面量:
```c++
unsigned mask = 0b111000000;
```
注意：I/O streams 里只有 dec、hex、oct 三个manipulator，而没有 bin，因而输出一个二进制数不能像十进制、十六进制、八进制那么直接。一个间接方式是使用 bitset，但调用者需要手工指定二进制位数：
```c++

#include <bitset>
#include<iostream>
#include<climits>
using namespace std;
 
int main(){
    cout<<mask<<endl;//dec
    cout<<hex;
    cout<<mask<<endl;//hex
    cout<<oct;
    cout<<mask<<endl;//oct
return 0;
}
cout << bitset<9>(mask) << endl;
```







