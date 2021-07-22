### pair的应用

* pair是蒋2个数据组合成一组数据, stl中map就是将key与value组合在一起
* 但需要返回两个数据时, 可以选择pair, pair的实现是一个结构体, 两个主要的成员变量为first与second, 可以直接使用

标准库在`#include <utility>`中, 定义如下所示

模版类: template<class T1,class T2> struct pair

参数：T1是第一个值的数据类型，T2是第二个值的数据类型

定义

```c++
pair<T1, T2> p1;            //创建一个空的pair对象（使用默认构造），它的两个元素分别是T1和T2类型，采用值初始化。
pair<T1, T2> p1(v1, v2);    //创建一个pair对象，它的两个元素分别是T1和T2类型，其中first成员初始化为v1，second成员初始化为v2。
make_pair(v1, v2);          // 以v1和v2的值创建一个新的pair对象，其元素类型分别是v1和v2的类型。
p1 < p2;                    // 两个pair对象间的小于运算，其定义遵循字典次序：如 p1.first < p2.first 或者 !(p2.first < p1.first) && (p1.second < p2.second) 则返回true。
p1 == p2；                  // 如果两个对象的first和second依次相等，则这两个对象相等；该运算使用元素的==操作符。
p1.first;                   // 返回对象p1中名为first的公有数据成员
p1.second;                 // 返回对象p1中名为second的公有数据成员
```



### pair的创建与初始化

在创建pair对象时, 必须提供两个类型名, 两个类型名类型不必相同

初始化方式有多种

* 定义时初始化
* 变量间接赋值

```c++
pair<string, string> author("James","Joy");    // 创建一个author对象，两个元素类型分别为string类型，并默认初始值为James和Joy。
pair<string, int> name_age2(name_age);    // 拷贝构造初始化

//typedef使用
typedef pair<string,string> Author;
Author proust("March","Proust");


pair<int, double> p1(1, 1.2);
pair<int, double> p2 = p1;   //拷贝构造函数赋值
```



### pair对象的操作

可以通过first与second成员访问两个元素

```c++
pair<int ,double> p1;
 
p1.first = 1;
 
p1.second = 2.5;
```



### 生成新的pair对象

使用make_pair生成新的pair对象

```c++
pair<int, double> p1;
p1 = make_pair(1, 1.2);
 
cout << p1.first << p1.second << endl;
 
//output: 1 1.2
```



### 通过tie获取pair元素的值

若函数以pair作为返回值, 可以直接通过std::tie进行接收

```c++
std::pair<std::string, int> getPreson() {
    return std::make_pair("Sven", 25);
}
 
int main(int argc, char **argv) {
    std::string name;
    int ages;
    std::tie(name, ages) = getPreson();//直接通过tie进行接收
    std::cout << "name: " << name << ", ages: " << ages << std::endl;
 
    return 0;
}
```

