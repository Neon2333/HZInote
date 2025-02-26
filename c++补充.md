# 1. std::expected

> std::expected 是 C++23 中引入的一个新特性，但它已经在许多现代C++库（如 Boost.Outcome）中被广泛讨论和使用。
std::expected 是一个模板类，用于表示函数的返回值，它可以包含一个成功值（T）或者一个错误值（E）。它的设计目标是提供一种更安全、更灵活的方式来处理错误情况，相比传统的返回值和异常机制，它有以下特点：
封装成功值和错误值：
如果操作成功，std::expected 包含一个类型为 T 的值。
如果操作失败，std::expected 包含一个类型为 E 的错误值。
生命周期管理：
std::expected 负责管理内部存储的 T 或 E 的生命周期。
内部对象的初始化和销毁由 std::expected 完全控制。
状态跟踪：
std::expected 跟踪内部对象是否已初始化，以及它是成功值还是错误值。
提供方法（如 has_value() 或 has_error()）来检查当前状态。
替代异常处理：
与异常相比，std::expected 是一种更显式的错误处理方式，避免了异常的性能开销和复杂性。
它允许函数返回错误信息，而不是抛出异常。

std::expected 跟踪内部对象是否已经被初始化。
* has_value() 方法可以检查是否包含成功值 T，
* has_error() 方法可以检查是否包含错误值 E。
* 两种状态：
成功状态：包含一个类型为 T 的对象。
错误状态：包含一个类型为 E 的对象。
这两种状态是互斥的，即一个 std::expected 对象在任何时刻只能包含一个成功值或一个错误值。

```cpp
#include <iostream>
#include <expected>
#include <string>

std::expected<int, std::string> divide(int a, int b) {
    if (b == 0) {
        return std::unexpected<std::string>("Division by zero");
    }
    return a / b;
}

int main() {
    auto result = divide(10, 2);
    if (result.has_value()) {
        std::cout << "Result: " << result.value() << std::endl;
    } else {
        std::cerr << "Error: " << result.error() << std::endl;
    }

    auto result2 = divide(10, 0);
    if (result2.has_value()) {
        std::cout << "Result: " << result2.value() << std::endl;
    } else {
        std::cerr << "Error: " << result2.error() << std::endl;
    }

    return 0;
}

# 2. insert_or_assign
---
insert_or_assign()是C++17中引入的一个成员函数模板，用于在std::map、std::unordered_map
等关联容器中进行插入或赋值操作。这个函数模板提供了两种功能：如果指定的键不存在于容器中，则插入该键及其对应的值；如果键已存在，则更新该键对应的值。

# 3. [[nodiscard]]
一个C++17引入的属性，用于指示函数的返回值不应被忽略。如果调用了一个带有 [[nodiscard]] 属性的函数，
但没有使用其返回值，编译器将发出警告。这对于那些其返回值包含重要信息或操作结果的函数特别有用，可以帮助开发者避免由于忘记处理函数返回值而导致的潜在错误。
```cpp
 [[nodiscard]] T start(const size_t i) const
    {
        return static_cast<T>(i * block_size) + first_index;
    }

# 4. inline变量
---
https://blog.csdn.net/janeqi1987/article/details/100108589
https://cloud.tencent.com/developer/article/1979728
使用Inline变量的动机

* 在C++中，类结构中不允许初始化非const静态成员
。例如：

class MyClass {
static std::string name = ""; // 编译错误
};
* 在多个CPP文件中包含头文件定义类结构之外的变量也会导致错误：

// hpp
class MyClass {
static std::string name; // 可以
};
// cpp
MyClass::name = ""; // 链接错误
为了解决这些问题，C++17引入了inline变量

# 5. extern
---
* extern关键字用于在多个文件中声明同一全局变量或函数。当一个文件需要使用另一个文件的某个全局变量时，可以在该文件中通过 extern 关键字来声明这个变量的存在，但不分配内存。
定义处不声明为extern，使用处声明为extern。
* 头文件中声明全局变量经常使用extern关键字。否则每个包含该头文件的源文件都会有一个该全局变量的定义，导致链接错误。



# 6. rapidjson库
---
> https://rapidjson.org/zh-cn/md_doc_pointer_8zh-cn.html
由于一个 Value 可包含不同类型的值，我们可能需要验证它的类型，并使用合适的 API 去获取其值。
```cpp
assert(document.HasMember("hello"));
assert(document["hello"].IsString());
printf("hello = %s\n", document["hello"].GetString());

# 7. hpp头文件
.h文件用于需要兼容C语言的场合，.hpp文件用于纯C++的场合，现代C++项目中通常只用.hpp文件。
使用模板的类或函数通常放在.hpp文件中，因为编译器需要看到模板的定义才能实例化，即定义和声明都要放在.hpp文件中。
具体场景：
* 头文件是纯C++编写的，使用.hpp扩展名。
* 头文件中有类声明，使用.hpp扩展名。

#8. std::tuple
---
> std::tuple 是 C++11 引入的一个标准库类型，用于存储固定数量的元素，每个元素可以有不同的数据类型。它类似于一个轻量级的结构体或联合体，但不具备命名字段的特性。

* 使用场景：
返回多个值：
当函数需要返回多个值时，可以使用 std::tuple 来封装这些返回值。例如，一个函数可以同时返回一个状态码和一个结果数据。
数据结构：
在需要存储不同类型数据的场景下，std::tuple 提供了一种灵活的方式来构建复合数据结构。例如，可以创建一个 tuple 来同时表示一个人的姓名、年龄和地址信息。

* 访问元素：
可以使用 std::get<N>(tuple) 来访问 tuple 中的第 N 个元素，其中 N 是从 0 开始计数的索引。
可以使用 std::get<T>(tuple) 来访问 tuple 中的第一个类型为 T 的元素。（从C++14开始支持）。

```cpp
#include <iostream>
#include <tuple>

int main() {
    // 创建一个包含 int、double 和 std::string 的 tuple
    //std::tuple<int, double, std::string> myTuple = {1, 2.3, "hello"};
    auto myTuple = std::make_tuple(42, 3.14, "Hello");

    // 获取并打印各个元素的值
    std::cout << "Integer: " << std::get<0>(myTuple) << '\n';
    std::cout << "Double: " << std::get<1>(myTuple) << '\n';
    std::cout << "String: " << std::get<2>(myTuple) << '\n';

    return 0;
}

# 9. std::atomic_bool
---
> 原子类型是C++11引入的，用于提供对单个变量的无锁访问和修改。std::atomic_bool 是 std::atomic 的一个特化版本，专门设计用来存储
bool 类型的值，并保证对该值的所有操作都是原子的。
* 读操作：
store(val)：将一个新值存储到原子变量中。
load()：从原子变量中读取当前值。
* 内存顺序：
原子操作还可以指定内存顺序，这对于控制多线程程序中的内存可见性和顺序性至关重要。C++11提供了几种内存顺序选项，如
memory_order_relaxed	最宽松的顺序，仅保证操作的原子性，不保证顺序。
memory_order_acquire	用于读load操作，确保后续操作不会重排到读操作之前。
memory_order_release	用于写store操作，确保前面的操作不会重排到写操作之后。
memory_order_acq_rel	用于读写操作，结合了 acquire 和 release 的特性。
memory_order_seq_cst	最严格的顺序，确保所有操作的顺序对所有线程都是一致的
默认是：memory_order_seq_cst，它提供了最强的内存顺序保证，确保操作的原子性和可见性。

# 10. std::atomic_int64_t 

# 11. std::priority_queue
---
> std::priority_queue 是 C++ STL 中的一个容器适配器，它提供了一个优先队列的实现。在优先队列中，每个元素都有一个优先级，并且总是按照元素的优先级顺序进行排序。它是一个最大堆（默认情况下），这意味着每次访问队列顶部时，都会得到优先级最高的元素。
在默认情况下，std::priority_queue使用std::less<T>作为比较函数，这意味着元素是按照从小到大的顺序排列的（但堆顶元素是最大的）

* 定义：
```cpp
std::priority_queue<int> pq; // 默认情况下，这是一个最大堆

//使用自定义类型，需在类型中定义比较器
struct Compare {
    bool operator()(const int& a, const int& b) const {
        return a > b; // 改为最小堆(按照priority从大到小排，顶部优先级最小)
    }
};
std::priority_queue<int, std::vector<int>, Compare> pqMin;
```
* 基本操作：
push(value)：将一个元素添加到优先队列中。
top()：返回优先级最高的元素的引用（但不移除它）。
pop()：从优先队列中删除优先级最高的元素。
empty()：检查优先队列是否为空。
size()：返回优先队列中的元素数量。

# 12. std::map判断Key是否存在
---
    map.find(key) != map.end()
    map.count(key) > 0
    map.contains(key) // C++20

# 13. std::vector访问首元素
---
`.at(0)`，返回引用，进行越界判断
`[0]`
`.front()`，返回首元素引用。当vector为空时未定义行为，用之前判断`size()>0`或`!empty()`
`begin()`，迭代器需要解引用


# 14. std::vector删除元素
---
erase删除某元素后，原本指向该元素及其后续元素的迭代器、指针和引用都会失效。
在调用erase之后，如果要继续使用容器中的元素，应该重新获取迭代器,iter=erase()。

# 15. 查找
---
* std::find 用于在容器中查找与给定值相等的第一个元素
```cpp
#include <algorithm>

iterator std::find(Iterator first, Iterator last, const T& value);
参数：
first 和 last：定义要搜索的范围 [first, last)。
value：要查找的目标值。
返回值：
返回指向找到的第一个匹配元素的迭代器。
如果没有找到匹配的元素，则返回 last迭代器。
```

* std::find_if 用于在容器中查找满足特定条件的第一个元素。它允许通过一个谓词函数（predicate）来定义查找条件，
```cpp
#include <algorithm>

iterator std::find_if(Iterator first, Iterator last, Predicate pred);
参数：
first 和 last：定义要搜索的范围 [first, last)。
pred：谓词函数，用于判断每个元素是否满足条件。它接受一个参数并返回一个布尔值。
返回值：
返回指向找到的第一个满足条件的元素的迭代器。
如果没有找到满足条件的元素，则返回 last迭代器。
```

# 16. 使用命名空间别名
---
> 不用写那么长的命名空间前缀了。

```cpp
namespace fs = std::filesystem;
```

# 17. 其他类型拼接字符串
* C++20中可以使用`std::format()`函数来拼接字符串。
类似qt中的%1占位符方式。
```cpp
#include <format>
#include <iostream>

int main() {
    int age = 30;
    std::string name = "Alice";
    double pi = 3.14159;

    std::string result = std::format("My name is {0} and I am {1} years old. Pi is {2:.2f}.", name, age, pi);
    std::cout << result << std::endl;
    return 0;
}
```

* 将其他类型通过`std::to_string()`转成字符串后拼接。
```cpp
std::string to_string(int value);
std::string to_string(long value);
std::string to_string(long long value);
std::string to_string(unsigned value);
std::string to_string(unsigned long value);
std::string to_string(unsigned long long value);
std::string to_string(float value);
std::string to_string(double value);
std::string to_string(long double value);
```
* `std::ostringstream`
通过str()函数获取最终的字符串。

轻量级
```cpp
//拼接字符串
#include <sstream>
#include <iostream>

std::ostringstream oss;  // 创建一个 ostringstream 对象
oss << "Hello, " << name << "!" << std::endl;  // 向流中插入数据
std::string result = oss.str();  // 获取最终的字符串
```
* `std::stringstream`
功能更多
```cpp
//拼接字符串同上
std::stringstream ss;
ss << "100" << " " << 200;  // 向流中插入数据
std::string result = ss.str();  // 获取最终的字符串

//提取数据
int a,b;
ss>>a>>b;  // 从流中提取数据
std::cout<<a<<" "<<b;  // 输出提取的数据,100,200

```

# 18. 抛出运行时异常
---
* `.what()`
std::exception类中的一个成员函数，返回一个指向描述异常原因的空终止字符串（C风格字符串）的指针。这个字符串通常用于调试和日志记录目的。
* `std::cerr`
将异常消息输出到标准错误。


```cpp
#include <stdexcept>//头文件
#include <iostream>

int main() {
    try {
        // 可能抛出异常的代码块
        if (true) { // 示例错误条件
            throw std::runtime_error("An error occurred during runtime");
        }
    } catch (const std::runtime_error& e) {
        // 捕获并处理std::runtime_error异常
        std::cerr << "Caught a runtime_error: " << e.what() << std::endl;
    }

    return 0;
}
```


