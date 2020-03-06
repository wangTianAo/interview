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
> 代码膨胀
> 无法随着函数库升级而升级，需要重新编译
> 只是建议，不可控
<br>
###虚函数（virtual）和内联函数（inline）


