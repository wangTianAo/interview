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
代码取自https://github.com/huihut/interview/blob/master/README.md#-cc
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


