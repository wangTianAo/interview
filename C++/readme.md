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
有函数模板和类模板<br>
部分摘自https://blog.csdn.net/lezardfu/article/details/56852043<br>
优点：
* 代码重用，可扩展性
缺点：
* 模板的数据类型只能在编译时才能被确定。因此，所有用基于模板算法的实现必须包含在整个设计的头文件中
* 有些C++编译器还不支持模板
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

## 7.C++内存布局与对象的生命周期
在C++语言中内存主要分为如下5个存储区：<br>
* 栈(Stack)：位于函数内的局部变量（包括函数实参），由编译器负责分配释放，函数结束，栈变量失效
* 堆（Heap)：就是那些由malloc等分配的内存块，用free来结束自己的生命。
* 自由存储区（free store）:由new申请的内存，且由delete或delete[]负责释放。
* 全局区/静态区(Global Static Area)： 全局变量和静态变量存放区，程序一经编译好，该区域便存在。
* 常量存储区： 这是一块比较特殊的存储区，专门存储不能修改的常量(一般是const修饰的变量，或是一些常量字符串)。
* 程序代码区： 存放函数体的二进制代码

### free store和heap不等价：
new和delete的时候先malloc，再构造，delete的时候先析构，再free，这时候对象可以说同时在堆和自由存储区，但是程序员也可以通过重载操作符，改用其他内存来实现自由存储，例如全局变量做的对象池，这时自由存储区就不位于堆上了。<br>
free store是个概念，heap是一块物理存在的特殊内存，用于程序的内存动态分配。<br>

### malloc/free 和 new/delete 区别：
* malloc/free是库函数，new/delete是操作符
* malloc/free只分配内存，new/delete还负责构造/析构对象
* malloc/free分配失败会返回NULL，new/delete分配失败会返回bad_alloc
* new/delete分配可以自动计算需要的字节数，malloc/free需要人为指定

## 8.C++ 拷贝构造函数
拷贝构造函数是一种特殊的构造函数，函数的名称必须和类名称一致，它必须的一个参数是本类型的一个引用变量。<br>
### 调用时机
* 对象以值传递的方式传入函数参数
* 对象以值传递的方式从函数返回
* 对象需要通过另外一个对象进行初始化

### 浅拷贝和深拷贝
指的是在对象复制时，只是对对象中的数据成员进行简单的赋值，上面的例子都是属于浅拷贝的情况，默认拷贝构造函数执行的也是浅拷贝。但是如果有指针的话，就会指向同一个区域。<br>
深拷贝完全拿出一块区域，重新动态分配空间，指针指向不同区域。<br>
区别：
* 在未定义显示拷贝构造函数的情况下，系统会调用默认的拷贝函数——即浅拷贝，它能够完成成员的一一复制。当数据成员中没有指针时，浅拷贝是可行的；但当数据成员中有指针时，如果采用简单的浅拷贝，则两类中的两个指针将指向同一个地址，当对象快结束时，会调用两次析构函数，而导致指针悬挂现象，所以，此时，必须采用深拷贝。
* 深拷贝与浅拷贝的区别就在于深拷贝会在堆内存中另外申请空间来储存数据，从而也就解决了指针悬挂的问题。简而言之，当数据成员中有指针时，必须要用深拷贝

## 9.C++动态链接库、静态链接库
静态连接库就是把(lib)文件中用到的函数代码直接链接进目标程序，程序运行的时候不再需要其它的库文件；<br>
动态链接库就是把调用的函数所在文件模块（DLL）和调用函数在文件中的位置等信息链接进目标程序，程序运行的时候再从DLL中寻找相应函数代码，因此需要相应DLL文件的支持。<br>

静态库在程序编译时会被连接到目标代码中，静态库的代码在编译过程中已经被载入可执行程序， 程序运行时将不再需要该静态库，因此体积大。<br>
静态库优点:<br>
* 代码装载速度快，执行速度略比动态链接库快
静态库缺点:<br>
* 静态链接库是将全部库中的内容都导入到生成的exe文件中，导致exe文件体积变大

动态库在程序编译时并不会被连接到目标代码中，动态库的代码是在可执行程序运行时才载入内存的，在编译过程中仅简单的引用，因此生成的exe文件体积较小，但是在程序运行时还需要动态库存在
动态库优点:<br>
* dll节省内存，减少交换操作
动态库缺点:<br>
* 使用动态链接库的应用程序不是完整的的，它依赖的dll库也要存在

## 10.C++函数传参的方式和优缺点
* 值传递
* 指针传递
* 引用传递
### 指针传递、引用传递的区别
指针：变量，独立，可变，可空，替身，无类型检查<br>
引用：别名，依赖，不变，非空，本体，有类型检查<br>
相同点：
* 都是地址的概念
不同点：
* 指针是一个实体，而引用仅是个别名
* 引用只能在定义时被初始化一次，之后不可变；指针可变；引用“从一而终”，指针可以“见异思迁”
* 引用没有const，指针有const，const的指针不可变
* 引用不能为空（有本体，才有别名），指针可以为空
* sizeof 引用，得到的是所指向变量的大小；sizeof 指针，得到的是指针的大小
* 指针 ++，是指指针的地址自增；引用++是指所指变量自增
* 引用是类型安全的，引用过程会进行类型检查；指针不会进行安全检查

## 11.C++ static的作用
* 扩展生存期
存放在静态存储区,生存期是整个程序的生存期
* 限制作用域
static全局变量和函数，其作用域为当前cpp文件，其它的cpp文件不能访问该变量和函数。如果有两个cpp文件声明了同名的全局静态变量，那么他们实际上是独立的两个变量
* 数据唯一性

* 全局静态变量，在全局变量前加上关键字static，全局变量就定义成一个全局静态变量.静态存储区，在整个程序运行期间一直存在。作用域：全局静态变量在声明他的文件之外是不可见的，准确地说是从定义之处开始，到文件结尾。
* 局部静态变量，在局部变量之前加上关键字static，局部变量就成为一个局部静态变量。内存中的位置：静态存储区。作用域：作用域仍为局部作用域，当定义它的函数或者语句块结束的时候，作用域结束。但是当局部静态变量离开作用域后，并没有销毁，而是仍然驻留在内存当中，只不过我们不能再对它进行访问，直到该函数再次被调用，并且值不变；
* 静态函数，在函数返回类型前加static，函数就定义为静态函数。函数的定义和声明在默认情况下都是的，但静态函数只是在声明他的文件当中可见，不能被其他文件所用。不要再头文件中声明static的全局函数，不要在cpp内声明非static的全局函数，如果你要在多个cpp中复用该函数，就把它的声明提到头文件里去，否则cpp内部声明需加上static修饰；
* 类的静态成员。在类中，静态成员可以实现多个对象之间的数据共享，并且使用静态数据成员还不会破坏隐藏的原则，即保证了安全性。因此，静态成员是类的所有对象中共享的成员，而不是某个对象的成员。对多个对象来说，静态数据成员只存储一处，供所有对象共用
* 类的静态函数。在静态成员函数的实现中不能直接引用类中说明的非静态成员，可以引用类中说明的静态成员（这点非常重要）。如果静态成员函数中要引用非静态成员时，可通过对象来引用。

### 全局变量和全局静态变量的区别
* 全局变量是不显式用 static 修饰的全局变量，全局变量默认是有外部链接性的，作用域是整个工程，在一个文件内定义的全局变量，在另一个文件中，通过 extern 全局变量名的声明，就可以使用全局变量。
* 全局静态变量是显式用 static 修饰的全局变量，作用域是声明此变量所在的文件，其他的文件即使用 extern 声明也不能使用。

## 12.C++ 动态绑定
只有涉及虚函数的地方才存在动态绑定<br>

### 静态类型
是指不需要考虑表达式的执行期语义，仅分析程序文本而决定的表达式类型。静态类型仅依赖于包含表达式的程序文本的形式，而在程序运行时不会改变。通俗的讲，就是上下文无关，在编译时就可以确定其类型
### 动态类型
是指由一个左值表达式表示的左值所引用的最终派生对象的类型,一般地讲，基类的指针和基类引用有可能为动态类型，就是说在运行之前不能够确定其真实类型。
### 静态绑定
编译时绑定，通过对象调用
### 动态绑定
运行时绑定，通过地址实现
```cpp
/************************************************************************
* 动态绑定与静态绑定,摘自https://blog.csdn.net/iicy266/article/details/11906509
************************************************************************/
class Base {
 public:
   void func() {
      cout << "func() in Base." << endl;
   }
   virtual void test() {
     cout << "test() in Base." << endl;
  }
};
 
class Derived : public Base {
  void func() {
     cout << "func() in Derived." << endl;
  }
  virtual void test() {
     cout << "test() in Derived." << endl; 
  }
};
 
int main() {
  Base* b;
  b = new Derived();
  b->func();
  b->test();
}
```
<br>
![](https://img-my.csdn.net/uploads/201212/08/1354976889_1255.jpg)
由运行结果可以看到，b是一个基类指针，它指向了一个派生类对象，基类Base里面有两个函数，其中test为虚函数，func为非虚函数。因此，对于test就表现为动态绑定，实际调用的是派生类对象中的test，而func为非虚函数，因此它表现为静态绑定，也就是说指针类型是什么，就会调用该类型相应的函数。

## 13.C++友元
摘自https://blog.csdn.net/jackystudio/article/details/11799777<br>
类的友元函数是定义在类外部，但有权访问类的所有私有（private）成员和保护（protected）成员,友元函数并不是成员函数,友元可以是一个函数，该函数被称为友元函数；友元也可以是一个类，该类被称为友元类，在这种情况下，整个类及其所有成员都是友元。<br>

友元提供了一种 普通函数或者类成员函数 访问另一个类中的私有或保护成员 的机制。也就是说有两种形式的友元
* 友元函数：普通函数对一个访问某个类中的私有或保护成员。
* 友元类：类A中的成员函数访问类B中的私有或保护成员。

### 优缺点:
* 优点：提高了程序的运行效率
* 缺点：破坏了类的封装性和数据的透明性

### Tips:
* 友元关系没有继承性.假如类B是类A的友元，类C继承于类A，那么友元类B是没办法直接访问类C的私有或保护成员
* 友元关系没有传递性.假如类B是类A的友元，类C是类B的友元，那么友元类C是没办法直接访问类A的私有或保护成员，也就是不存在“友元的友元”这种关系

## 14.C++ final和override
* C++ 11添加了两个继承控制关键字：override和final。override确保在派生类中声明的重载函数跟基类的虚函数有相同的签名。final阻止类的进一步派生和虚函数的进一步重载。<br>
* 非虚函数不能用final标识符修饰<br>
代码摘自https://blog.csdn.net/jirryzhang/article/details/82961654
```cpp
class A{
    virtual void foo1() final;
    virtual void foo2();
    void bar() final;//错误，只能修饰虚函数
};
 
class B final:public A
{
    virtual void foo2();
    void foo1();//错误，final修饰的虚函数不能重写
};
 
class C:B//错误，final修饰的类不能被继承
{
    void foo2();
};
 
class Base
{
    virtual void foo1(int a,double b, bool c){};
    virtual void foo2(int a,double b, bool c){};
};
 
class Derived:public Base
{
    virtual void foo1(int x,int y,bool z) override{};//错误，签名不一致
    void foo2(int x,int y,bool z) override{};//错误，缺少virtual关键字声明
};
 
void test()
{
    C c;
    Derived d;
}
```

## 15.C++代码的编译过程
预处理——编译——汇编——链接。预处理器先处理各种宏定义，然后交给编译器；编译器编译成.s为后缀的汇编代码；汇编代码再通过汇编器形成.o为后缀的机器码（二进制）；最后通过链接器将一个个目标文件（库文件）链接成一个完整的可执行程序（或者静态库、动态库）<br>

## 16.C++中四种cast转换
* const_cast：用于将const变量转为非const
* static_cast：用于各种隐式转换，比如非const转const，void*转指针等, static_cast能用于多态向上转化，如果向下转能成功但是不安全，结果未知；
* dynamic_cast：用于动态类型转换。只能用于含有虚函数的类，用于类层次间的向上和向下转化。只能转指针或引用。向下转化时，如果是非法的对于指针返回NULL，对于引用抛异常。要深入了解内部转换的原理。
* reinterpret_cast：几乎什么都可以转，比如将int转指针，可能会出问题，尽量少用；

## 17.C++智能指针
智能指针的作用是管理一个指针，因为存在以下这种情况：申请的空间在函数结束时忘记释放，造成内存泄漏。使用智能指针可以很大程度上的避免这个问题，因为智能指针就是一个类，智能指针的类都是栈上的对象，当超出了类的作用域是，类会自动调用析构函数，析构函数会自动释放资源。所以智能指针的作用原理就是在函数结束时自动释放内存空间，不需要手动释放内存空间。

* auto_ptr:C++11已弃用。
```cpp
auto_ptr< string> p1 (new string ("I reigned lonely as a cloud.”));
auto_ptr<string> p2;
p2 = p1; //auto_ptr不会报错.
```
此时不会报错，p2剥夺了p1的所有权，但是当程序运行时访问p1将会报错。所以auto_ptr的缺点是：存在潜在的内存崩溃问题！<br>

* unique_ptr:unique_ptr实现独占式拥有或严格拥有概念，保证同一时间内只有一个智能指针可以指向该对象。它对于避免资源泄露(例如“以new创建对象后因为发生异常而忘记调用delete”)特别有用。另外unique_ptr还有更聪明的地方：当程序试图将一个 unique_ptr 赋值给另一个时，如果源 unique_ptr 是个临时右值，编译器允许这么做；如果源 unique_ptr 将存在一段时间，编译器将禁止这么做，比如
```cpp
unique_ptr<string> pu1(new string ("hello world"));
unique_ptr<string> pu2;
pu2 = pu1;                                      // #1 not allowed
unique_ptr<string> pu3;
pu3 = unique_ptr<string>(new string ("You"));   // #2 allowed
```

* shared_ptr：shared_ptr实现共享式拥有概念。多个智能指针可以指向相同对象，该对象和其相关资源会在“最后一个引用被销毁”时候释放。从名字share就可以看出了资源可以被多个指针共享，它使用计数机制来表明资源被几个指针共享。可以通过成员函数use_count()来查看资源的所有者个数。除了可以通过new来构造，还可以通过传入auto_ptr, unique_ptr,weak_ptr来构造。当我们调用release()时，当前指针会释放资源所有权，计数减一。当计数等于0时，资源会被释放。

* weak_ptr：weak_ptr 是一种不控制对象生命周期的智能指针, 它指向一个 shared_ptr 管理的对象. 进行该对象的内存管理的是那个强引用的 shared_ptr. weak_ptr只是提供了对管理对象的一个访问手段。weak_ptr 设计的目的是为配合 shared_ptr 而引入的一种智能指针来协助 shared_ptr 工作, 它只可以从一个 shared_ptr 或另一个 weak_ptr 对象构造, 它的构造和析构不会引起引用记数的增加或减少。weak_ptr是用来解决shared_ptr相互引用时的死锁问题,如果说两个shared_ptr相互引用,那么这两个指针的引用计数永远不可能下降为0,资源永远不会释放。它是对对象的一种弱引用，不会增加对象的引用计数，和shared_ptr之间可以相互转化，shared_ptr可以直接赋值给它，它可以通过调用lock函数来获得shared_ptr。

### 智能指针的实现
* shared_ptr允许拷贝和赋值，其底层实现是以"引用计数"为基础的,需要实现构造，析构，拷贝构造，=操作符重载，重载*-和>操作符
原文链接：https://blog.csdn.net/LF_2016/java/article/details/52421909
```cpp

template<typename T>
class SharedPtr            //采用引用计数，实现一个可以有多个指针指向同一块内存的类模板，SharedPtr是类模板，不是智能指针类型
{
public:
       SharedPtr(T* ptr);
       SharedPtr(const SharedPtr<T>& sp);
       SharedPtr<T>& operator=(SharedPtr<T> sp);
       T& operator*();
       T* operator->();
       ~SharedPtr();
       int Count()
       {
              return *_pCount;
       }
private:
       void Release()
       {
              if (--(*_pCount) == 0)
              {
                     delete _ptr;
                     delete _pCount;
                     _ptr = NULL;
                     _pCount = NULL;
              }
       }
private:
       T* _ptr;
       int* _pCount;             //指向引用计数的空间
};
template<typename T>
SharedPtr<T>::SharedPtr(T* ptr)
:_ptr(ptr)
, _pCount(new int(1)){}
template<typename T>
SharedPtr<T>::SharedPtr(const SharedPtr<T>& sp)
{
       _ptr = sp._ptr;
       _pCount= sp._pCount;
       ++(*_pCount);
}
template<typename T>
SharedPtr<T>& SharedPtr<T>::operator=(SharedPtr<T> sp)
{
       std::swap(sp._ptr,_ptr);
       std::swap(sp._pCount,_pCount);
       return *this;
}
template<typename T>
T& SharedPtr<T>::operator*()
{
       return *_ptr;
}
template<typename T>
T* SharedPtr<T>::operator->()
{
       return _ptr;
}
template<typename T>
SharedPtr<T>::~SharedPtr()
{
       Release();
}
```
## 18.C++构造析构顺序
* 构造：1）基类构造 2）对象成员构造 3）类本身构造
* 析构：1）类本身析构 2）对象成员析构 3）基类析构

## 19.++i和i++的区别
++i先自增1，再返回，i++先返回i,再自增1
###  ++i 实现：
```cpp
int&  int::operator++（）
{
*this +=1；
return *this；
}
```

### i++实现
```cpp
const int  int::operator（int）
{
int oldValue = *this；
++（*this）；
return oldValue；
}
```

## 20.extern
* extern声明的全局变量可以在其他文件使用
* C++调用C函数需要extern C，因为C语言没有函数重载。

## 21.C++字节对齐
https://blog.csdn.net/chenhanzhun/article/details/39641489<br>
* 结构体变量的首地址能够被其最宽基本类型成员的大小所整除；
* 结构体每个成员相对于结构体首地址的偏移量都是成员大小的整数倍，如有需要编译器会在成员之间加上填充字节
* 结构体的总大小为结构体最宽基本类型成员大小的整数倍，如有需要编译器会在最末一个成员之后加上填充字节。
* 成员函数和静态成员在全局静态区，所以不用计算
* 在类中虚函数会占用4个字节(虚指针)

## 22.Lambda
https://www.cnblogs.com/DswCnblog/p/5629165.html
