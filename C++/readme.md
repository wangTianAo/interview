# C++面试资料整理
## 1.inline和宏
* inline是编译器处理，宏是编译前预处理阶段处理。<br>
* 宏是简单的替换，inline是直接把代码放到调用的地方。<br>
* 宏不能访问对象的成员，inline可以。<br>
* inline展开前会安全检查或自动类型转换，宏不会。<br>
* inline可以调试，宏不可以。<br><br>
* inline必须在实现的时候写。<br>
* 类里面直接实现函数默认inline（除了虚函数）。<br>
* inline适合简单逻辑。<br>
* inline只是建议编译器inline，是否inline由编译器决定<br>
* 缺点
  * 代码膨胀
  * 无法随着函数库升级而升级，需要重新编译
  * 只是建议，不可控
### 虚函数（virtual）和内联函数（inline）
* 虚函数可以是内联函数，内联是可以修饰虚函数的，但是当虚函数表现多态性的时候不能内联<br>
* 内联是在编译器建议编译器内联，而虚函数的多态性在运行期，编译器无法知道运行期调用哪个代码，因此虚函数表现为多态性时（运行期）不可以内联。<br>
* 如果编译器知道所调用的对象是哪个类，就可以inline，这只有在编译器具有实际对象而不是对象的指针或引用时才会发生。<br>
e.g.<br>
摘自https://github.com/huihut/interview/blob/master/README.md#-cc
```cpp
#include <iostream>  
using namespace std;
class Base
{
public:
	inline virtual void who()
	{
		cout << "I am Base\n";
	}
	virtual ~Base() {}
};
class Derived : public Base
{
public:
	inline void who()  // 不写inline时隐式内联
	{
		cout << "I am Derived\n";
	}
};

int main()
{
	// 此处的虚函数 who()，是通过类（Base）的具体对象（b）来调用的，编译期间就能确定了，所以它可以是内联的，但最终是否内联取决于编译器。 
	Base b;
	b.who();

	// 此处的虚函数是通过指针调用的，呈现多态性，需要在运行时期间才能确定，所以不能为内联。  
	Base *ptr = new Derived();
	ptr->who();

	// 因为Base有虚析构函数（virtual ~Base() {}），所以 delete 时，会先调用派生类（Derived）析构函数，再调用基类（Base）析构函数，防止内存泄漏。
	delete ptr;
	ptr = nullptr;

	system("pause");
	return 0;
} 
```
## 2.虚函数virtual
摘自https://blog.csdn.net/iFuMI/article/details/51088091<br>
* c++用virtual来实现多态，当子类重新定义了父类的虚函数后，当父类的指针指向子类对象的地址时，[即B b; A a = &b;] 父类指针根据赋给它的不同子类指针，动态的调用子类的该函数，而不是父类的函数（如果不使用virtual方法，请看后面★*），且这样的函数调用发生在运行阶段，而不是发生在编译阶段，称为动态联编。而函数的重载可以认为是多态，只不过是静态的。注意，非虚函数静态联编，效率要比虚函数高，但是不具备动态联编能力。
* 如果使用了virtual关键字，程序将根据引用或指针指向的 对 象 类 型 来选择方法，否则使用引用类型或指针类型来选择方法。
```cpp
    class A{
    private:
        int i;
    public:
        A();
        A(int num) :i(num) {};
        virtual void fun1();
        virtual void fun2();

    };

    class B : public A{
    private:
        int j;
    public:
        B(int num) :j(num){};
        virtual void fun2();// 重写了基类的方法
    };

    // 为方便解释思想，省略很多代码

    A a(1);
    B b(2);
    A *a1_ptr = &a;
    A *a2_ptr = &b;

    // 当派生类“重写”了基类的虚方法，调用该方法时
    // 程序根据 指针或引用 指向的  “对象的类型”来选择使用哪个方法
    a1_ptr->fun2();// call A::fun2();
    a2_ptr->fun2();// call B::fun2();
    // 否则
    // 程序根据“指针或引用的类型”来选择使用哪个方法
    a1_ptr->fun1();// call A::fun1();
    a2_ptr->fun1();// call A::fun1();

```
### 虚函数的底层实现机制
* 实现原理：虚函数表+虚表指针<br>
每个类有一个虚函数表，每个对象有一个虚表指针。基类对象有一个虚表指针，基类对象包含一个虚表指针，指向基类中所有虚函数的地址表。派生类对象也将包含一个虚表指针，指向派生类虚函数表。看下面两种情况：<br>

*如果派生类重写了基类的虚方法，该派生类虚函数表将保存重写的虚函数的地址，而不是基类的虚函数地址。

*如果基类中的虚方法没有在派生类中重写，那么派生类将继承基类中的虚方法，而且派生类中虚函数表将保存基类中未被重写的虚函数的地址。注意，如果派生类中定义了新的虚方法，则该虚函数的地址也将被添加到派生类虚函数表中。

![](https://img-blog.csdn.net/20160407180941333)  
