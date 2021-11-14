### priority_queue应用

priority_queue是现在#include <queue> 中, 但是他可以定义自己的优先级, 本质上使用堆实现, 默认是大顶堆

定义`template <typename T, typename Container=std::vector<T>, typename Compare=std::less<T>> class priority_queue`

priority_queue 实例默认有一个 vector 容器, 函数对象类型 less<T> 是一个默认的排序断言定义在头文件 function 中, 决定了容器中最大的元素会排在队列前面.

### priority_queue初始化与创建

生成一个空的优先级队列与使用基础类型初始化优先级队列,

```c++
std:: priority_queues<string> words;  //生成空的优先级队列

string words[] {"one", "two", "three"} //使用适当的类型初始化
priority_queues<string> q {begin(words), end(words)};
```

同时可以使用拷贝构造函数用于生成与目标对象相同的一个新的对象.

```c++
priority_queue<string> copy_words {words}
```

当需要对容器进行反向排序时, 需要同时指定容器的三个元素, 同时只要容器有成员函数front(), push_back(), size(), empty()都可以作为优先级队列保存元素的基础容器, 因此申明一个反向排序的容器可以如下所示

```c++
std:: string wrds[] {"one", "two", "three", "four"};
std::priority_queue<std::string, std::vector<std::string>,std: :greater<std::string>> words1 {std::begin (wrds) , std:: end (wrds) }  //使用第三参数比较操作符
std::priority_queue<std::string, std::deque<std::string>> words {std::begin(wrds), std::end(wrds)};   //使用本身反序的结构
```

### 成员函数

- push(const T& obj)：将obj的副本放到容器的适当位置，这通常会包含一个排序操作。
- push(T&& obj)：将obj放到容器的适当位置，这通常会包含一个排序操作。
- emplace(T constructor a rgs...)：通过调用传入参数的构造函数，在序列的适当位置构造一个T对象。为了维持优先顺序，通常需要一个排序操作。
- top()：返回优先级队列中第一个元素的引用。
- pop()：移除第一个元素。
- size()：返回队列中元素的个数。
- empty()：如果队列为空的话，返回true。
- swap(priority_queue<T>& other)：和参数的元素进行交换，所包含对象的类型必须相同。

48G5sl82p92
