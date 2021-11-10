* debug fault:`invalid specifier outside a class declaration` 
use virtual befor declearation,don't add virtaul befor defination,otherwise the complier will give `error: invalid specifier outside a class declaration`
        
```c++

        #include<iostream>
        #include<string>
        using namespace std;

        class employee
        {
        public:
            virtual void Print();
        };

        virtual void employee::Print() // ERROR
        {

        }
        int main()
        {
            return 0;
        }
```
* class xxxx has virtual method but non-virtual destructor 
        定义父类的指针指向子类的对象时，先调用父类的构造函数再调用子类的构造函数，释放内存时，若父类的析构函数不是虚析构函数，则只释放调用父类的析构函数而不调用子类的析构函数，从而造成**内存泄漏**
```c++
#ifndef SHAPE_H
#define SHAPE_H
#include <iostream>
using namespace std;
class Shape
{
    public:
      Shape();
      virtual ~Shape();//父类有虚方法，需要虚析构，若没，comlier会提示：class xxxx has virtual method but non-virtual destructor
      virtual double calcArea(int a,double b);
    //private:
};
#endif
```

```c++
#include "Shape.h"
#include <iostream>
using namespace std;

Shape::Shape()
{
    cout<<"Shape()"<<endl;
}
Shape::~Shape()
{
    cout<<"~Shape()"<<endl;
}
double Shape::calcArea(int a,double b)
{
    cout<<"shape-calcArea()"<<endl;
    return 0;
}
```

```c++
#include "Shape.h"
class Circle:public Shape
{
    public:
        Circle(double r);
        ~Circle();
        double calcArea();//overload not override,覆盖了父类的方法，静态绑定
        int    calcArea(int a,double b); //overload not override
        double calcArea() override;//加了override会提示参数列表不符
        double calcArea(int a,double b) override;//加了override，表示子类中与父类同名的改函数是父类方法的重写，编译器会检查
    protected:
        double m_dR;
};
```
```c++
#include "Circle.h"
#include <iostream>
using namespace std;

Circle::Circle(double r)
{
    cout<<"Circle()"<<endl;
    m_dR=r;
}
Circle::~Circle()
{
    cout<<"~Circle()"<<endl;
}
double Circle::calcArea()
{
     cout<<"Circle--Circle()"<<endl;
     return 3.14*m_dR*m_dR;
}

```
```c++
#include <iostream>
#include <stdlib.h>
#include "Circle.h"
using namespace std;

int main(void)
{
    //在堆上实例化一个Shape类型的对象指针，以子类的对象给父类的
    //对象指针初始化。
    Shape *p1=new Circle(5);
    //使用该指针访问calcArea()函数，若Shape类的calcArea函数不是虚函数，
    //则调用Shape类的该函数，若是虚函数，则调用子类的calcArea函数。
    p1->calcArea();
    delete p1;
    p1=NULL;
    system("pause");
    return 0;
}
```
* overrid
       * 加上override后，建议把子类的virtual关键字去掉，并且以后都遵循上边提到的规范：只有顶层的虚函数加上virtual，其余子类在覆写时全部用override来标记
       * 只有加了 virtual 关键字的虚函数才能被继承类 overriding
 在派生类中，重写 (override) 继承自基类成员函数的实现 (implementation) 时，要满足如下条件：     
        1. 一虚：基类中，成员函数声明为虚拟的 (virtual)
        2. 二容：基类和派生类中，成员函数的返回类型和异常规格 (exception specification) 必须兼容
        3. 四同：基类和派生类中，成员函数名、形参类型、常量属性 (constness) 和 引用限定符 (reference qualifier) 必须完全相同  

如此多的限制条件，导致了虚函数重写如上述代码，极容易因为一个不小心而出错
C++11 中的 override 关键字，可以显式的在派生类中声明，哪些成员函数需要被重写，如果c程序员忘记重写该函数，则编译器会报错。

  

