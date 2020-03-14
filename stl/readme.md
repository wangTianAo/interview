# STL
摘自 https://blog.csdn.net/daaikuaichuan/article/details/80717222<br>

## STL部件
STL包含6大部件：容器、迭代器、算法、仿函数、适配器和空间配置器
* 容器：容纳一组元素的对象
* 迭代器：提供一种访问容器中每个元素的方法
* 函数对象：一个行为类似函数的对象，调用它就像调用函数一样
* 算法：包括查找算法、排序算法等等
* 适配器：用来修饰容器等，比如queue和stack，底层借助了deque
* 空间配置器：负责空间配置和管理

## 空间配置器详解
对象构造前的空间配置和对象析构后的空间释放，由<stl_alloc.h>负责，SGI对此的设计哲学如下：
* 向system heap要求空间
* 考虑多线程状态
* 考虑内存不足时的应变措施
* 考虑过多“小型区块”可能造成的内存碎片问题
针对内存碎片问题，SGI设计了双层级配置器
* 第一级直接使用allocate()调用malloc()、deallocate()调用free()，使用类似new_handler机制解决内存不足（抛出异常），配置无法满足的问题（如果在申请动态内存时找不到足够大的内存块，malloc 和new 将返回NULL 指针，宣告内存申请失败）
* 第二级视情况使用不同的策略，当配置区块大于128bytes时，调用第一级配置器，当配置区块小于128bytes时，采用内存池的整理方式：配置器维护16个（128/8）自由链表，负责16种小型区块的此配置能力。内存池以malloc配置而得，如果内存不足转第一级配置器处理
* 第二级空间配置器实际上是一个内存池，维护了16个自由链表。自由链表是一个指针数组，有点类似与hash桶，它的数组大小为16，每个数组元素代表所挂的区块大小，比如free _ list[0]代表下面挂的是8bytes的区块，free _ list[1]代表下面挂的是16bytes的区块…….依次类推，直到free _ list[15]代表下面挂的是128bytes的区块

## 空间配置器存在的问题
* 自由链表所挂区块都是8的整数倍，因此当我们需要非8倍数的区块，往往会导致浪费。
* 由于配置器的所有方法，成员都是静态的，那么他们就是存放在静态区。释放时机就是程序结束，这样子会导致自由链表一直占用内存，自己进程可以用，其他进程却用不了。

## 容器
### vector
### vector的底层原理
* vector底层是一个动态数组，包含三个迭代器，start和finish之间是已经被使用的空间范围，end_of_storage是整块连续空间包括备用空间的尾部
* 当空间不够装下数据（vec.push_back(val)）时，会自动申请另一片更大的空间（1.5倍或者2倍），然后把原来的数据拷贝到新的内存空间，接着释放原来的那片空间【vector内存增长机制】
* 当释放或者删除（vec.clear()）里面的数据时，其存储空间不释放，仅仅是清空了里面的数据
* 对vector的任何操作一旦引起了空间的重新配置，指向原vector的所有迭代器会都失效了

### vector中的reserve和resize的区别
* reserve是直接扩充到已经确定的大小，可以减少多次开辟、释放空间的问题（优化push_back），就可以提高效率，其次还可以减少多次要拷贝数据的问题。reserve只是保证vector中的空间大小（capacity）最少达到参数所指定的大小n。reserve()只有一个参数。
* resize()可以改变有效空间的大小，也有改变默认值的功能。capacity的大小也会随着改变。resize()可以有多个参数。

### vector中的size和capacity的区别
* size表示当前vector中有多少个元素（finish - start），而capacity函数则表示它已经分配的内存中可以容纳多少元素（end_of_storage - start）

### vector的元素类型可以是引用吗？
* vector的底层实现要求连续的对象排列，引用并非对象，没有实际地址，因此vector的元素类型不能是引用

### vector迭代器失效的情况
* 当插入一个元素到vector中，由于引起了内存重新分配，所以指向原内存的迭代器全部失效

### 正确释放vector的内存(clear(), swap(), shrink_to_fit())
* vec.clear()：清空内容，但是不释放内存。
* vector<int>().swap(vec)：清空内容，且释放内存，想得到一个全新的vector。
* vec.shrink_to_fit()：请求容器降低其capacity和size匹配。
* vec.clear();vec.shrink_to_fit();：清空内容，且释放内存。

### vector的常用函数
```cpp
vector<int> vec(10,100);        创建10个元素,每个元素值为100
vec.resize(r,vector<int>(c,0)); 二维数组初始化
reverse(vec.begin(),vec.end())  将元素翻转
sort(vec.begin(),vec.end());    排序，默认升序排列
vec.push_back(val);             尾部插入数字
vec.size();                     向量大小
find(vec.begin(),vec.end(),1);  查找元素
iterator = vec.erase(iterator)  删除元素
```
## list
### list的底层原理
* list的底层是一个双向链表，以结点为单位存放数据，结点的地址在内存中不一定连续，每次插入或删除一个元素，就配置或释放一个元素空间
* list不支持随机存取，如果需要大量的插入和删除，而不关心随即存取

### list常用函数
```cpp
list.push_back(elem)	在尾部加入一个数据
list.pop_back()	        删除尾部数据
list.push_front(elem)	在头部插入一个数据
list.pop_front()	    删除头部数据
list.size()	            返回容器中实际数据的个数
list.sort()             排序，默认由小到大 
list.unique()           移除数值相同的连续元素
list.back()             取尾部迭代器
list.erase(iterator)    删除一个元素，参数是迭代器，返回的是删除迭代器的下一个位置
```
## deque
### deque的底层原理
* deque是一个双向开口的连续线性空间（双端队列），在头尾两端进行元素的插入跟删除操作都有理想的时间复杂度
![](https://img-blog.csdn.net/20150826111941696?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

## 什么情况下用vector，什么情况下用list，什么情况下用deque
* vector可以随机存储元素（即可以通过公式直接计算出元素地址，而不需要挨个查找），但在非尾部插入删除数据时，效率很低，适合对象简单，对象数量变化不大，随机访问频繁。除非必要，我们尽可能选择使用vector而非deque，因为deque的迭代器比vector迭代器复杂很多。
* list不支持随机存储，适用于对象大，对象数量变化频繁，插入和删除频繁，比如写多读少的场景
* 需要从首尾两端进行插入或删除操作的时候需要选择deque

## priority_queue
### priority_queue的底层原理
priority_queue：优先队列，其底层是用堆来实现的。在优先队列中，队首元素一定是当前队列中优先级最高的那一个。

### priority_queue的常用函数
```cpp
priority_queue<int, vector<int>, greater<int>> pq;   最小堆
priority_queue<int, vector<int>, less<int>> pq;      最大堆
pq.empty()   如果队列为空返回真
pq.pop()     删除对顶元素
pq.push(val) 加入一个元素
pq.size()    返回优先队列中拥有的元素个数
pq.top()     返回优先级最高的元素
```

## map 、set、multiset、multimap
* map 、set、multiset、multimap的底层实现都是红黑树

### map 、set、multiset、multimap的特点
* set和multiset会根据特定的排序准则自动将元素排序，set中元素不允许重复，multiset可以重复
* map和multimap将key和value组成的pair作为元素，根据key的排序准则自动将元素排序（因为红黑树也是二叉搜索树，所以map默认是按key排序的），map中元素的key不允许重复，multimap可以重复
* map和set的增删改查速度为都是logn，是比较高效的

### 为何map和set的插入删除效率比其他序列容器高，而且每次insert之后，以前保存的iterator不会失效
* 因为存储的是结点，不需要内存拷贝和内存移动,因为插入操作只是结点指针换来换去，结点内存没有改变。而iterator就像指向结点的指针，内存没变，指向内存的指针也不会变

### 为何map和set不能像vector一样有个reserve函数来预分配数据
* 因为在map和set内部存储的已经不是元素本身了，而是包含元素的结点。也就是说map内部使用的Alloc并不是map<Key, Data, Compare, Alloc>声明的时候从参数中传入的Alloc

### map 、set、multiset、multimap的常用函数
```cpp
it map.begin() 　        返回指向容器起始位置的迭代器（iterator） 
it map.end()             返回指向容器末尾位置的迭代器 
bool map.empty()         若容器为空，则返回true，否则false
it map.find(k)           寻找键值为k的元素，并用返回其地址
int map.size()           返回map中已存在元素的数量
map.insert({int,string}) 插入元素
for (itor = map.begin(); itor != map.end();)
{
    if (itor->second == "target")
        map.erase(itor++) ; // erase之后，令当前迭代器指向其后继。
    else
        ++itor;
}
```

## unordered_map、unordered_set
### unordered_map、unordered_set的底层原理
* unordered_map的底层是一个防冗余的哈希表（采用除留余数法）。哈希表最大的优点，就是把数据的存储和查找消耗的时间大大降低，时间复杂度为O(1)；而代价仅仅是消耗比较多的内存

### unordered_map 与map的区别？使用场景?
* 构造函数：unordered_map 需要hash函数，等于函数;map只需要比较函数(小于函数)
* 存储结构：unordered_map 采用hash表存储，map一般采用红黑树(RB Tree) 实现。因此其memory数据结构是不一样的




