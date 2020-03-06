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

## 3.虚函数、纯虚函数
* 虚函数里面可以有实现，可以在子类里面override，编译器就后期绑定来达到多态。纯虚函数里面不能有实现，需要在子类里实现
* 带纯虚函数的类叫抽象类，这种类不能直接生成对象，而只有被继承，并重写其虚函数后，才能使用。抽象类被继承后，子类可以继续是抽象类，也可以是普通类。
* c++中没有接口的概念，与之对应的是纯虚类，即只含有纯虚函数的类，c++抽象类的概念是含有纯虚函数成员的类。这是因为c++提供多继承，而像 java、c#这些只提供单继承（避免多继承的复杂性和低效性）的语言为了模拟多继承功能就提供了接口概念，接口可以继承多个。

### 构造函数和析构函数可以是虚函数吗？
构造函数不能是虚函数，析构函数可以是虚函数且推荐最好设置为虚函数。<br>
* 虚函数通过对象内存中虚指针实现(vptr),构造函数是对象内存中的值做初始化操作，构造函数完成之前，也即还没有进行初始化，此时vptr是没有值的，也就无法通过vptr找到作为构造函数和虚函数所在的代码区，所以构造函数只能以普通函数的形式存放在类所指定的代码区中。<br>
* 而对于析构函数，当我们delete(a)的时候，如果析构函数不是虚函数，那么调用的将会是基类base的析构函数。而当继承的时候，通常派生类会在基类的基础上定义自己的成员，此时我们当然希望可以调用派生类的析构函数对新定义的成员也进行析构。

### 抽象类
称带有纯虚函数的类为抽象类<br>

## 4.多态
* 重载多态：函数重载，操作符重载(编译期)
* 子类型多态：虚函数(运行期)
* 参数多态：类模板，函数模板(编译器)
* 强制多态：基本类型转换、自定义类型转换(编译期、运行期)

## 5. C++模板
有函数模板和类模板
摘自https://blog.csdn.net/lezardfu/article/details/56852043<br>
### 1)函数模板
普通函数和成员函数都可以模板化<br>
e.g.
```cpp
template<typename T>
    void print(const T& t) {
        cout << t <<endl;
    }
```
#### 为什么成员函数模板不能是虚函数(virtual)？
因为c++ compiler在parse一个类的时候就要确定vtable的大小，如果允许一个虚函数是模板函数，那么compiler就需要在parse这个类之前扫描所有的代码，找出这个模板成员函数的调用（实例化），然后才能确定vtable的大小，而显然这是不可行的，除非改变当前compiler的工作机制。

#### 函数模板重载
函数模板之间，函数模板与普通函数之间可以重载。编译器会根据调用时提供的函数参数，调用能够处理这一类型的最特殊的版本。在特殊性上，一般按照如下顺序考虑：
* 普通函数
* 特殊模板（限定了T的形式的，指针、引用、容器等）
* 普通模板（对T没有任何限制的）
```cpp
template<typename T>
void func(T& t) { //通用模板函数
    cout << "In generic version template " << t << endl;
}

template<typename T>
void func(T* t) { //指针版本
    cout << "In pointer version template "<< *t << endl;
}

void func(string* s) { //普通函数
    cout << "In normal function " << *s << endl;
}

int i = 10;
func(i); //调用通用版本，其他函数或者无法实例化或者不匹配
func(&i); //调用指针版本，通用版本虽然也可以用，但是编译器选择最特殊的版本
string s = "abc";
func(&s); //调用普通函数，通用版本和特殊版本虽然也都可以用，但是编译器选择最特化的版本
func<>(&s); //调用指针版本，通过<>告诉编译器我们需要用template而不是普通函数
```
### 推断实例化
为了方便使用，除了直接为函数模板指定类型参数之外，我们还可以让编译器从传递给函数的实参推断类型参数，这一功能被称为模板实参推断。<br>
说人话就是编译器帮你推断参数是什么类型<br>
```cpp
compare(1, 2); //推断T的类型为int
compare(1.0, 2.0); //推断T的类型为double
p.print("abc"); //推断T的类型为const char*
```

### 2)类模板
摘自https://blog.csdn.net/lezardfu/article/details/57416241
* 类模板不能推断实例化<br>
* 类模板的成员函数既可以定义在内部，也可以定义在外部。定义在内部的被隐式声明为inline，定义在外部的类名之前必须加上template的相关声明。
```cpp
template<typename T>
class Printer {
public:
    explicit Printer(const T& param):t(param){}
    string&& to_string();
    //定义在内部
    void print() {
        cout << t << endl;
    }
private:
    T t;
};

//定义在外部
template<typename T>
string&& Printer<T>::to_string() {
    strstream ss;
    ss << t;
    return std::move(string(ss.str()));
}

Printer p(1); //error
Printer<int> p(1); //ok
```

### 类模板中的static成员
类模板中可以声明static成员，在类外定义的时候要增加template相关的关键词。另外，需要注意的是：每个不同的模板实例都会有一个独有的static成员对象。比如一个类模板里面有个属性叫value，<int>和<double>里面的value是独立的，因为static成员属于实例化后的类，不同的实例化类有不同的static成员
```cpp
template<typename T>
class Printer {
public:
    explicit Printer(const T& param):t(param){}
    static int s_value;
private:
    T t;
};

template<typename T> //注意这里的定义方式
int Printer<T>::s_value = 1;

Printer<int> pi(1);
Printer<int> pi2(1);
Printer<double> pd(1.0);
pi.s_value += 1; //pi和pi2中的改变了，pd的没改变
```
### 类模板别名
* typedef为类模板的某个实例定义一个别名
* 也可以使用using语句固定一个或多个类型参数
	
## 6.特化与偏特化
* 模板机制为C++提供了泛型编程的方式，在减少代码冗余的同时仍然可以提供类型安全。 特化必须在同一命名空间下进行，可以特化类模板也可以特化函数模板，但类模板可以偏特化和全特化，而函数模板只能全特化。 模板实例化时会优先匹配”模板参数”最相符的那个特化版本。
* 提供了所有类型实参的特化是完全特化，只提供了部分类型实参或者T的类型受限（例如：T*）的特化被认为是不完整的，所以也被称为偏特化。完全特化的结果是一个实际的class，而偏特化的结果是另外一个同名的模板。
### 模板特化
有一个简单的通过==判断是否equal的方法，如果传进来的是两个char*指针，如果指针里的地址不同就会false，所以需要对char*特殊处理，即特化。可以特化类也可以特化函数。(全特化)
### 偏特化
* 类模板可以偏特化，当有多个模板形参时，我们可以只特化一部分形参而不是全部。
* 严格来说，函数模板并不支持偏特化，但由于可以对函数进行重载，所以可以达到类似于类模板偏特化的效果。
### 调用顺序
* 有对应的特化就调用对应的特化，没有就调用次优先的特化，还没有就调用普通特化

## 7.C++程序运行周期
