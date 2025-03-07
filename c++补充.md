

# 1. 命名空间

---

> 命名空间可以帮助组织代码，使其更清晰、更有条理。这使得代码更容易阅读和维护。
>
> - **按功能划分**：可以将不同的功能模块放在不同的命名空间中。例如，把数学相关的函数放在 `math` 命名空间，把文件操作相关的函数放在 `filesystem` 命名空间。
> - 比用类的静态成员封装更加**轻量**：命名空间比类更轻量级。它不需要像类那样有构造函数、成员函数和成员变量的概念，也不会涉及对象的实例化。

* 全局命名空间

  C++ 中的程序通常包含多个模块，每个模块可能包含大量的变量、类和函数。如果这些变量和函数都直接放在全局命名空间中，很容易发生命名冲突。

  所有没有被显式放入其他命名空间的代码（如变量、函数和类）都属于全局命名空间。

  而用户定义的命名空间（如 `namespace AA`）可以用来将代码进一步组织和隔离。

* 匿名命名空间

  一种特殊的命名空间，它没有名称，因此只能在当前文件（包括同一翻译单元）中访问。

  作用：

  替代C风格的`static`，限制（变量、函数等）只能在当前文件中访问，不能被其他文件访问。

  

# 1. std::expected

---

> std::expected 是 C++23 中引入的一个新特性，但它已经在许多现代C++库（如 Boost.Outcome）中被广泛讨论和使用。
std::expected 是一个模板类，用于表示函数的返回值，它可以包含一个成功值（T）或者一个错误值（E）。**它的设计目标是提供一种更安全、更灵活的方式来处理错误情况，**相比传统的返回值和异常机制，它有以下特点：
封装成功值和错误值：
如果操作成功，std::expected 包含一个类型为 T 的值。
如果操作失败，std::expected 包含一个类型为 E 的错误值。
生命周期管理：
**std::expected 负责管理内部存储的 T 或 E 的生命周期。**
**内部对象的初始化和销毁由 std::expected 完全控制。**
状态跟踪：
std::expected 跟踪内部对象是否已初始化，以及它是成功值还是错误值。
提供方法（如 has_value() 或 has_error()）来检查当前状态。
**替代异常处理：**
**与异常相比，std::expected 是一种更显式的错误处理方式，避免了异常的性能开销和复杂性。**
**它允许函数返回错误信息，而不是抛出异常。**

std::expected 跟踪内部对象是否已经被初始化。
* `has_value() `判断是否包含成功值 T，通过`value()`获取成功值。

* `has_error() `判断是否包含错误值 E，通过`error()`获取错误值。

* 重载了!运算符

  ```cpp
  bool flag = !std::expected<int,string>
  包含错误值时flag为true，包含成功值时为false
  ```

* 两种状态：
成功状态：包含一个类型为 T 的对象。
错误状态：包含一个类型为 E 的对象。
**这两种状态是互斥的，即一个 std::expected 对象在任何时刻只能包含一个成功值或一个错误值。**

```cpp
#include <iostream>
#include <expected>
#include <string>

std::expected<int, std::string> divide(int a, int b) 
{
    if (b == 0) 
    {
        //这3种写法都行
        return std::unexpected<std::string>("Division by zero");	
        return std::expected<int,std::string>(std::unexpected<string>("Division by zero"));
        return make_unexpected("Division by zero");
    }
    return a / b;
}

int main() 
{
    auto result = divide(10, 2);
    if (result.has_value())
    {
        std::cout << "Result: " << result.value() << std::endl;
    } 
    else 
    {
        std::cerr << "Error: " << result.error() << std::endl;
    }

    auto result2 = divide(10, 0);
    if (result2.has_value()) 
    {
        std::cout << "Result: " << result2.value() << std::endl;
    }
    else 
    {
        std::cerr << "Error: " << result2.error() << std::endl;
    }

    return 0;
}
```

* 指定成功或失败

  ```cpp
  int sock = 1;
  //设置状态为成功，携带成功值为sock
  return tl::expected<int,string>(sock);		
  //设置状态为失败，携带失败值为"failed"
  string str="failed";
  //std::unexpected可传入std::unexpected的构造函数
  return tl::expected<int,string>(std::unexpected<string>(str));
  return std::unexpected<string>(str);
  ```

  ```cpp
  //封装make_unexpected
  template <class E>
  unexpected<typename std::decay<E>::type> make_unexpected(E &&e) 
  {
    return unexpected<typename std::decay<E>::type>(std::forward<E>(e));
  }
  
  //使用
  catch (fs::filesystem_error &e)
  {
   	cout << e.what();
   	return tl::make_unexpected(string("DataFrame::remove error for class_id ") +
                              std::to_string(classId()) + ":" + e.what());
   } 
  catch (sql_exception &e)
  {
      cout << e.what();
      return tl::make_unexpected(string("DataFrame::remove error for class_id ") +
                                 std::to_string(classId()) + ":" + e.what());
  } 
  catch (std::exception &e) 
  {
      cout << e.what();
      return tl::make_unexpected(string("DataFrame::remove error for class_id ") +
                                 std::to_string(classId()) + ":" + e.what());
  }
  ```
* 根据返回不同执行不同操作。链式写法：
  
  map执行成功时操作，lambda捕获返回值T
  ma'p执行错误时操作，lambda捕获返回值E
  
  ````cpp
  std::expected<int, std::string> divide(int a, int b) 
  {
      if (b == 0) 
      {
          //这3种写法都行
          return std::unexpected<std::string>("Division by zero");	
          return std::expected<int,std::string>(std::unexpected<string>("Division by zero"));
          return make_unexpected("Division by zero");
      }
      return a / b;
  }  
  
  divide(10,2).map([&](int val)){cout<<val<<endl;}
                .map_error([](const std::string &err){cout<<err<<endl;});
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
```
# 4. inline变量
---
https://blog.csdn.net/janeqi1987/article/details/100108589
https://cloud.tencent.com/developer/article/1979728
使用Inline变量的动机

* 在C++中，类结构中不允许初始化非const静态成员

  ```cpp
  class MyClass {
  static std::string name = ""; // 编译错误
  };
  ```

* 按照一次定义原则，一个变量或者实体只能出现一个编译单元内，除非这个变量或者实体使用了inline进行修饰。如下面的代码。如果在一个类中定义了一个静态成员变量，然后在类的外部进行初始化，本身符合一次定义原则。但是如果在多个CPP文件同时包含了该头文件，在链接时编译器会报错。
```cpp
//.hpp
class MyClass {
static std::string msg;
...
};
// 如 果 被 多 个CPP文 件 包 含 会 导 致 链 接ERROR
std::string MyClass::msg{"OK"};
```

* C++17中内联变量的使用可以帮助我们解决实际编程中的问题而又不失优雅。使用inline后，即使定义的全局对象被多个文件引用也只会有一个全局对象。如下面的代码，就不会出现之前的链接问题。

```cpp
class MyClass {
inline static std::string msg{"OK"};
...
};
inline MyClass myGlobalObj;
```

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
```
# 7. hpp头文件
.h文件用于需要兼容C语言的场合，.hpp文件用于纯C++的场合，现代C++项目中通常只用.hpp文件。
使用模板的类或函数通常放在.hpp文件中，因为编译器需要看到模板的定义才能实例化，即定义和声明都要放在.hpp文件中。
具体场景：
* 头文件是纯C++编写的，使用.hpp扩展名。
* 头文件中有类声明，使用.hpp扩展名。

# 8. std::tuple

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
```

# 9. std::atomic_bool
---
> 原子类型是C++11引入的，用于提供对单个变量的无锁访问和修改。std::atomic_bool 是 std::atomic 的一个特化版本，专门设计用来存储
> bool 类型的值，并保证对该值的所有操作都是原子的。
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
> 在默认情况下，std::priority_queue使用std::less<T>作为比较函数，这意味着元素是按照从小到大的顺序排列的（但堆顶元素是最大的）

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

# 13. std::vector

* 访问首元素

  `.at(0)`，返回引用，进行越界判断
  `[0]`
  `.front()`，返回首元素引用。当vector为空时未定义行为，用之前判断`size()>0`或`!empty()`
  `begin()`，迭代器需要解引用

* std::vector删除元素

  erase删除某元素后，原本指向该元素及其后续元素的迭代器、指针和引用都会失效。
  在调用erase之后，如果要继续使用容器中的元素，应该重新获取迭代器,iter=erase()。

* 插入元素

  ```cpp
  insert(iterator pos, const T& val);	//在pos处插入val
  insert(iterator pos, int count, const T& val);//pos处插入count个val
  insert(iterator pos, iterStart, iterEnd);//在pos前插入[iterStart,iterEnd)元素
  ```

  

# 15. STL算法
---
> https://learn.microsoft.com/zh-cn/cpp/standard-library/algorithm?view=msvc-170

## （1）std::find

* 用于在容器中查找与给定值相等的第一个元素

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

## （2）std::min_element和std::max_element

在区间内查找最值

```cpp
//在指定区间[start,end)内查找最小值，返回迭代器
iterator pos = std::min_element(iterator start, iterator end);
int posIndex = pos-start;//index
auto minVal = *pos;//值
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

# 19. unique_ptr
---
`get()`获取裸指针

重载了!运算符，`if(!uptr)`和`if(uptr==nullptr)`等价，都是判断是否指向空。

# 20. 智能指针使用make_unique<>创建智能指针比用new更安全，因为它可以确保资源在异常发生时也能被正确释放。

# 21.executeQuery占位符 
auto rslt1 = conn.executeQuery(
        "select count(*) from e_chns_config where dev_id = ? and type_id = "
        "1",
        devicemap.first);
第二个参数是替换占位符?的值。

# 22. 链式处理函数返回值
---
* 通过`and_then()`方法，可以链式调用多个函数，并在每个函数的返回值上执行后续操作。如果任何一个中间步骤返回了一个错误（expected的错误分支），整个链式处理就会停止
```cpp
static tl::expected<int, string> init(int argc,
                                      std::shared_ptr<const char *[]> argv) {
  if (argc == 4) {
    return Config::make(string(argv[0])).and_then([&](Config conf) {
      config = conf;
      cout << "config file: " << argv[0] << endl;
      for (int k = 0; k < config.devicesMap.size(); k++) {
        hzi::sysDevMask += (int)exp2(k);
      }
      if (string(argv[1]) == "--verbose")
        config.verbose = true;
      if (string(argv[2]) == "--disableMslocating")
        config.disableMslocating = true;
      if (string(argv[3]) == "--disableAppResis")
        config.disableAppResis = true;
      return initChanelsConfig().and_then(
          [&](auto r) { return initChnLabels(); });
      // .and_then([&](auto r) {
      //     if (config.ms_locating && config.ms_experimental)
      //         std::thread(setupMSRegression).detach();
      //     return tl::expected<int, string>(0);
      // });
      // .and_then([&](auto r) { return readChnLocations(); });
    });
  }
  return 0;
}
```
* 通过`.map()`和`.map_error()`方法，可以对函数的返回值进行转换或处理错误。
     `.map()`：对函数的返回值进行转换，如果函数返回的是`expected<T, E>`类型（即返回值为成功），则可以使用`.map()`获取返回值并执行后续逻辑。
     `.map_error()`：对函数的错误返回值进行转换，如果函数返回的是`expected<T, E>`类型(即返回值为失败)，则可以使用`.map_error()`获取返回值并执行后续逻辑。

# 23. #pragma region和#pragma endregion
代码折叠

# 24. 时间
---
## （0）std::time_t时间戳

> `std::time_t`是C语言中表示时间的方式，通常是一个整数类型，表示自1970年1月1日以来秒的数（Unix时间戳）。
>
> ```cpp
> #include <ctime>
> ```
>
> - **简单通用**：广泛用于C和C++的遗留代码中，兼容性好。
> - **精度较低**：通常只支持秒级精度。
> - **无类型安全**：直接操作整数，容易出现单位混淆。

```cpp
#include <ctime>
#include <iostream>

int main()
{
    // 获取当前时间戳
    std::time_t now_time = std::time(nullptr);
    std::cout << "Current time (std::time_t): " << std::ctime(&now_time);

    // 转换为uint64_t
    uint64_t timestamp = static_cast<uint64_t>(now_time);
    std::cout << "Timestamp (seconds): " << timestamp << std::endl;

    return 0;
}
```

## （2）`uint64_t`时间戳

> `uint64_t`是一个无符号64位整数类型，常用于存储时间戳。它可以表示更大的范围，通常用于需要高精度或长时间跨度的场景。
>
> - **灵活性高**：可以存储任何时间单位（秒、毫秒、微秒等）。
> - **无类型安全**：需要手动管理时间单位，容易出错。
> - **范围大**：64位整数可以表示非常大的数值，适合长时间跨度。
>
> 

## （3）std::chrono

```cpp
#include<chrono>
using namespace std;
using namespace chrono;	//这一行一定写在using namespace std之后，否则报错
```

`std::chrono` 是 C++11 引入的用于处理时间的库，支持不同的时钟源，如`system_clock`（系统时钟）、`steady_clock`（单调时钟）和`high_resolution_clock`（高分辨率时钟）

```cpp
uint64_t timestamp;	//时间戳
std::chrono::milliseconds ms(timestamp);//时间戳初始化chrono时间点
std::chrono::system_clock::time_point timePoint(ms);
```

## （4） `std::chrono::system_clock`

`std::chrono::system_clock` 是系统时间，是不稳定的时钟，是可以自行设置的。它表示从某个固定时间点（通常是epoch，即 1970 年 1 月 1 日 00:00:00 UTC，这是类 Unix 系统中广泛使用的标准时间起点）到当前时间的持续时间。它通常用于获取当前时间点或计算时间间隔。

### `std::chrono::system_clock::time_point`

`std::chrono::system_clock::time_point` 是 `system_clock` 的时间点类型，表示从**系统时钟的起始点（epoch）**开始的时间点。它是一个强类型的时间点，可以进行时间运算和比较。

```cpp
auto now = system_clock::now();
```

### ``std::chrono::system_clock::duration`

```cpp
auto duration = duration_cast<microseconds>(start-end);
cout<<(double)(duration.count())<<endl;//ms
```

## （5）**std::chrono::high_resolution_clock** 

`std::chrono::high_resolution_clock`C++11最高的精度的时钟，精确到ns

```cpp
auto now = high_resolution_clock::now();
auto duration = now.time_since_epoch();//time_since_epoch() 是 time_point 对象的一个方法，用于获取从“纪元”（epoch）到当前时间的时间间隔。
auto microSecs = duration_cast<microseconds>(duration).count();
```

## （6）使用示例

以下是一些常见的使用方式：

#### 获取当前时间点
```cpp
#include <iostream>
#include <chrono>
#include <ctime> // 用于将时间点转换为可读格式

int main() {
    // 获取当前时间点
    std::chrono::system_clock::time_point now = std::chrono::system_clock::now();

    // 将时间点转换为时间戳（自 1970 年以来的秒数）
    std::time_t now_c = std::chrono::system_clock::to_time_t(now);

    // 转换为可读格式
    std::cout << "Current time: " << std::ctime(&now_c);

    return 0;
}
```

#### 计算时间间隔
```cpp
#include <iostream>
#include <chrono>
#include <thread> // 用于模拟延时

int main() {
    // 获取开始时间点
    auto start = std::chrono::system_clock::now();

    // 模拟延时
    std::this_thread::sleep_for(std::chrono::seconds(2));

    // 获取结束时间点
    auto end = std::chrono::system_clock::now();

    // 计算时间间隔
    auto duration = std::chrono::duration_cast<std::chrono::seconds>(end - start);

    std::cout << "Elapsed time: " << duration.count() << " seconds" << std::endl;

    return 0;
}
```

#### 时间点的比较
```cpp
#include <iostream>
#include <chrono>

int main() {
    auto now = std::chrono::system_clock::now();
    auto future = now + std::chrono::seconds(10);

    if (future > now) {
        std::cout << "Future time is later than now." << std::endl;
    }

    return 0;
}
```

## （7）当地时间：std::localtime

> `std::localtime` 是 C++ 标准库中的一个函数，用于**将时间值（通常表示为自纪元（Epoch，即1970年1月1日00:00:00 UTC）以来的秒数）转换为当地时间**（`tm` 结构）。
>
> ```cpp
> #include <ctime>
> ```

```cpp
std::chrono::system_clock::time_point timePoint(ms);
std::time_t timeT = std::chrono::system_clock::to_time_t(timePoint);
```

```cpp
std::tm* std::localtime(const std::time_t* time_ptr);
//成功时，返回一个指向 std::tm 结构的指针，该结构表示当地时间。
```

```cpp
tm 结构

std::tm 结构用于表示分解的时间，包含以下成员：

tm_sec：秒（0-60，允许闰秒）
tm_min：分（0-59）
tm_hour：小时（0-23）
tm_mday：日期（1-31）
tm_mon：月份（0-11，0代表一月）
tm_year：年份，自1900年起的年数
tm_wday：星期几（0-6，0代表星期日）
tm_yday：一年中的第几天（0-365，0代表1月1日）
tm_isdst：夏令时标志（正值代表夏令时，0代表非夏令时，负值代表信息不可用）
```

## （8）时间戳转换

`std::chrono::system_clock` 提供了从时间点到时间戳（`std::time_t`）的转换，也可以通过 `std::ctime` 或 `std::put_time` 将时间点格式化为可读的字符串。

#### `std::chrono` -> `std::time_t`：

```cpp
auto now = std::chrono::system_clock::now();
std::time_t now_time = std::chrono::system_clock::to_time_t(now);
```

#### `std::time_t` -> `std::chrono`：

```cpp
std::time_t now_time = std::time(nullptr);
auto now = std::chrono::system_clock::from_time_t(now_time);
```

#### `std::chrono` -> `uint64_t`：

```cpp
auto now = std::chrono::system_clock::now();
uint64_t timestamp_sec = std::chrono::duration_cast<std::chrono::seconds>(now.time_since_epoch()).count();
```

#### `uint64_t` -> `std::chrono`：

```cpp
uint64_t timestamp_sec = ...;
auto now = std::chrono::system_clock::time_point(std::chrono::seconds(timestamp_sec));
```

```cpp
#include <iostream>
#include <chrono>
#include <iomanip> // 用于 std::put_time
#include <sstream>
#include <locale> // 用于设置本地化

int main() 
{
    auto now = std::chrono::system_clock::now();
    std::time_t now_c = std::chrono::system_clock::to_time_t(now);

    std::stringstream ss;
    ss << std::put_time(std::localtime(&now_c), "%Y-%m-%d %H:%M:%S");

    std::cout << "Formatted time: " << ss.str() << std::endl;

    return 0;
}
```

# 25. 使用filesystem读写文件
---
> * 使用了 C++ 的\<filesystem> 库（C++17 引入的）。
> * 使用 filesystem::path 替代 std::string 作为文件路径在C++编程中具有诸多优势，包括平台无关性、更丰富的功能、更好的类型安全、性能优化以及与 \<filesystem> 库中其他功能的紧密集成。这些优势使得 filesystem::path 成为处理文件路径的推荐选择
filesystem::path 提供了许多用于操作路径的函数，如提取父目录、文件名、扩展名，拼接路径，判断是否为绝对路径等。
* 异常
  ```cpp
    fs::filesystem_error 
  ```
* 获取文件名：
    ```cpp
    namespace fs = std::filesystem;
    fs::path filePath = "example/path.txt";
    string filename=filePath.filename().string();//获取文件名
    string parentPath=filePath.parent_path().string();//获取父目录路径
    ```
* 拼接路径：
  ```cpp
    fs::path path1 = "dir1/";  // 路径1
    fs::path path2 = "dir2";  // 路径2
    fs::path combinedPath = path1 / path2;  // 路径拼接
    combinePath /= "file.txt";  // 继续拼接路径
  
    std::cout << combinedPath.string() << std::endl;  // 输出: dir1/dir2/file.txt
  ```
* 判断路径是否存在
  ```cpp
    fs::path filePath = "example/path.txt";
    if (fs::exists(filePath)) {  // 检查文件是否存在
        std::cout << "File exists: " << filePath.string() << std::endl;  // 输出文件存在信息    
    }
  ```
* 创建文件
  ```cpp
    fs::path filePath = "example/newfile.txt";
    //fs::ofstream
    fs::ofstream file(filePath);  // 创建并打开文件
    if (file.is_open()) {
        file << "Hello, World!";  // 写入内容到文件
        file.close();  // 关闭文件
    }
  ```

* 删除文件：
  ```cpp
    fs::path filePath = "example/path.txt";
    if (fs::exists(filePath)) {  // 检查文件是否存在
        fs::remove(filePath);  // 删除文件
    }
  ```
* 创建目录：
  ```cpp
    fs::path dirPath = "new_directory";
    if (!fs::exists(dirPath)) {  // 检查目录是否存在
        fs::create_directories(dirPath);  // 创建新目录
        std::cout << "Directory created: " << dirPath.string() << std::endl;  // 输出创建的目录路径
    }
  ```




# 26. 使用 C++ 标准库的 \<fstream>读写文件
---
打开模式：

```cpp
std::ios::trunc	//截断：文件不存在则创建，文件存在则清空
std::ios::app	//追加
```

```cpp
if (!fs::exists(fileName) || !fs::is_regular_file(fileName)) {  
        //is_regular_file()检查是否为普通文件，而非目录等。如果路径不存在或不是一个常规文件，则返回false。
        return tl::make_unexpected("No such file or file type error: " +
                                   fileName.string());
      }

      std::ifstream ifs(fileName, std::ifstream::binary);//二进制模式打开文件(默认以文本模式打开)，以便读取原始数据。

      if (!ifs) {
        return tl::make_unexpected("Unable to open file: " + fileName.string());
      }

      DataFrame df;
      df.upHead = std::make_unique<char[]>(32);
      df.pHead_ = df.upHead.get();
      ifs.read((char *)df.pHead_, 32);  //读取32字节到df.pHead_指向的内存中。

```
```cpp
void DataFrame::writeToFile(string filePath) const {
    //std::ofstream::binary和std::ofstream::app等价于std::ios::binary和std::ios::app
  std::ofstream ofs(filePath.c_str(), std::ofstream::binary | std::ofstream::app);
  ofs.write(pHead_, 32);
  ofs.write(pData_, dataBytes());
  ofs.close();
}
```

# 27. 字节序转换
---
```cpp
be16toh/be32toh/be64toh
//把主机字节序转换为大端
htobe16/htobe32/htobe64
```

* 这几个函数比hton/ntoh新，也更通用。hton/ntoh主要用于网络编程。

 # 28. git分支合作流程

---

## （1）新入职程序员的Git流程分析与改进建议

#### **1. 拉取远程代码**
- **当前操作**：  
  ```bash
  git pull origin master
  ```
- **分析**：  
  - 此命令从远程`master`分支拉取最新代码到本地当前分支。  
  - 如果本地当前分支是`master`，则直接运行`git pull`更简洁（默认拉取当前分支的远程代码）。  
  - 如果本地有其他分支，建议先切换到`master`分支再拉取：  
    ```bash
    git checkout master
    git pull
    ```
- **建议**：  
  ```bash
  git checkout master      # 切换到master分支
  git pull origin master   # 明确拉取远程master分支的最新代码
  ```

---

#### **2. 创建开发分支`dev`**
- **当前操作**：  
  ```bash
  git checkout -b dev origin/master
  ```
  这条命令的作用是：

创建新分支：创建一个名为 dev 的新分支。

指定起点：新分支的起点是 origin/master（即远程仓库 origin 的 master 分支的最新提交）。

自动切换：创建后直接切换到 dev 分支。
- **分析**：  
  - 从`origin/master`创建并切换到`dev`分支，符合常见的开发分支策略（如Git Flow中的`develop`分支）。  
  - **优点**：隔离主分支（`master`），确保稳定代码不受开发影响。  
  - **注意事项**：需与团队约定分支命名规范（如使用`develop`而非`dev`）。

---

#### **3. 创建特性分支`dev-feature-xxx`**
- **当前操作**：  
  ```bash
  git checkout -b dev-feature-xxx
  git push origin dev-feature-xxx
  ```
- **分析**：  
  - 基于`dev`分支创建特性分支`dev-feature-xxx`，并在开发完成后推送至远程。  
  - **优点**：特性分支独立，避免污染主开发分支。  
  - **改进建议**：  
    - 明确指定基于`dev`分支创建，避免歧义：  
      ```bash
      git checkout -b dev-feature-xxx dev
      ```
    - 推送时建议设置上游分支：  
      ```bash
      git push -u origin dev-feature-xxx
      ```

---

#### **4. 合并特性分支到`dev`并清理**
- **当前操作**：  
  ```bash
  git checkout dev
  git merge dev-feature-xxx
  git branch -d dev-feature-xxx
  git push origin --delete dev-feature-xxx
  ```
- **分析**：  
  - 合并特性分支到`dev`分支，删除本地和远程的特性分支。  
  - **优点**：保持分支列表整洁，避免冗余。  
  - **改进建议**：  
    - 合并前建议使用`git merge --no-ff`保留合并记录：  
      ```bash
      git merge --no-ff dev-feature-xxx
      ```
    - 删除远程分支前，确保所有协作者已完成相关工作。  
    - 若使用GitLab/GitHub，可通过PR（Pull Request）合并，自动清理分支。

---

#### **5. 推送`dev`分支并等待PR**
- **当前操作**：  
  ```bash
  git push origin master
  ```
- **问题**：  
  - 此命令将`dev`分支直接推送到远程`master`分支，**绕过代码审查流程**，不符合协作规范。  
  - 正确流程应为：推送`dev`分支到远程，并创建从`dev`到`master`的PR。  
- **正确操作**：  
  ```bash
  git push origin dev       # 推送dev分支到远程
  # 在Git平台（如GitHub）创建PR：dev → master
  ```
- **注意事项**：  
  - PR需经过团队代码审查后，由负责人合并到`master`。  
  - 合并后，拉取最新的`master`代码到本地：  
    ```bash
    git checkout master
    git pull origin master
    ```

---

### **完整优化后的流程**
```bash
# 1. 更新本地master分支
git checkout master
git pull origin master

# 2. 创建开发分支dev（基于最新master）
git checkout -b dev origin/master

# 3. 创建特性分支
git checkout -b dev-feature-xxx dev
git push -u origin dev-feature-xxx

# ... 在dev-feature-xxx分支开发并提交 ...

# 4. 合并到dev分支并清理
git checkout dev
git merge --no-ff dev-feature-xxx
git branch -d dev-feature-xxx
git push origin --delete dev-feature-xxx

# 5. 推送dev分支并提交PR
git push origin dev
# 在Git平台创建PR：dev → master，等待审核合并
```

---

### **关键改进点总结**
1. **明确分支来源**：创建分支时指定基准分支（如`git checkout -b dev-feature-xxx dev`）。  
2. **保留合并记录**：使用`--no-ff`选项避免丢失分支历史。  
3. **代码审查流程**：通过PR合并而非直接推送至`master`。  
4. **分支命名规范**：与团队统一命名（如`feature/xxx`、`develop`）。  

通过上述优化，既能保证代码管理的规范性，又能提升团队协作效率。

# 29. 常用分支类型
---
* main（或 master）：通常是项目的主分支，包含稳定、可发布的代码。
* develop：用于日常开发的分支，开发者通常会将新功能合并到这个分支。
* feature/*：用于开发新功能的分支，例如 feature/login 用于开发登录功能。
* hotfix/*：用于修复紧急问题的分支，例如 hotfix/fix-bug。
* release/*：用于准备发布的分支，例如 release/1.0。

# 30. c++风格类型强转
---
```cpp
*(uint8_t *)(pHead_ + 16) =
          classId == 1 ? *(uint8_t *)(ptr + 16)
                       : uint8_t(be16toh(*(uint16_t *)(ptr + 17)));
```
类型转换方式：

* C风格的强制类型转换：(type)value
* C++风格的函数风格转换：type(value)
* 更安全的C++类型转换：static_cast<type>(value)、reinterpret_cast<type>(value)等
# 31. 操纵器混合使用不同进制（如二进制、八进制、十六进制）标准输出
* std::dec 设置十进制输出
* std::hex 设置十六进制输出
* std::oct 设置八进制输出
* std::bin 设置二进制输出（非标准，但某些编译器支持）
* std::fixed 设置固定宽度输出
* std::boolalpha 设置布尔值以文本形式输出（true/false）
* std::setprecision 设置浮点数输出的小数位数
```cpp
std::cout << std::hex << 255 << "\n"; // 输出十六进制：FF
std::cout << std::oct << 255 << "\n"; // 输出八进制：377
std::cout << std::dec << 255 << "\n"; // 输出十进制：255
std::cout << std::bin << 255 << "\n"; // 可能输出二进制：11111111（取决于编译器）
bool isTrue = true;
std::cout << std::boolalpha << isTrue << "\n"; // 输出：true
double pi = 3.14159;
std::cout << std::fixed << std::setprecision(2) << pi << "\n"; // 输出：3.14
```

# 32. 数据库操作
---
* 异常
sql_exception 异常类，用于处理SQL执行过程中出现的错误。

* 从连接池获取连接
  ```cpp
    auto conn = pool.getConnection();
  ```
* 事务
  ```cpp
    conn.beginTransaction();
  
    conn.commit();
  ```

* ?是参数占位符在SQL语句中使用占位符（如?），然后在执行语句之前通过相应的方法（如setInt、setString等）设置这些占位符的值。这有助于防止SQL注入攻击，因为参数值会被数据库自动转义。

* 预编译sql语句
  创建一个`PreparedStatement`对象封装语句。
  数据库可以预先编译这个语句，并且在之后的执行中只需要替换参数即可。以便之后可以多次执行这个语句，每次执行时可能使用不同的参数。
  通过`bind`绑定值到占位符，第一个参数为占位符编号，从1开始。
  ```cpp
    //封装sql
    PreparedStatement pps = conn.prepareStatement("INSERT INTO users (name, age) VALUES (?, ?)");
    //绑定参数
    pps.bind(1, "aaa");
    pps.bind(2, 10);
    //执行语句
    pps.execute();
  ```

* 执行SQL语句（非查询）
  ```cpp
    conn.execute("delete from e_data_frm_info where class_id=? and "
                 "abs(unix_timestamp(samp_time)*1000-?)<3",
                 classId(), sampTime());
  ```
* 执行查询（对查询封装）
    ```cpp
    template <typename... Args>
    tl::expected<ResultSet, std::string> queryDb(Connection &conn, const char *sql,Args... args)
    {
        try 
        {
            ResultSet result = conn.executeQuery(sql, args...);
            if (result.next()) 
            {
                return result;
            }    
            else 
            {
                return tl::make_unexpected("not found in query: " + std::string(sql));
            }
        } 
        catch (sql_exception &e)
        {
            return tl::make_unexpected("sql error for " + std::string(sql) + ": " +std::string(e.what()));
        } 
    }
    ```
    ```cpp
        queryDb(conn,
          "SELECT ch_pnts from e_data_frm_info WHERE class_id=? AND "
          "ROUND(UNIX_TIMESTAMP(samp_time),3)>= ? AND "
          "ROUND(UNIX_TIMESTAMP(samp_time),3) <= ?",
          type, from, to)
      .map([&](ResultSet rslt) {
        do {
          pnts += rslt.getInt("ch_pnts");
          frms++;
        } while (rslt.next());
        res.send(Http::Code::Ok,
                 (std::to_string(frms) + ":" + std::to_string(pnts)).c_str());
      })
      .map_error([&](auto err) {
        if (err.find("not found in query") != string::npos) {
          res.send(Http::Code::No_Content, "not found");
        } else {
          res.send(Http::Code::Bad_Request, "error");
        }
      });
    ```

* 获取查询结果

  `ResultSet`表示查询得到的集合。

  `setFetchSize(int n)`表示每次从数据库只获取n条记录。

  ```cpp
  ResultSet result =
          conn.executeQuery("SELECT MAX(cmd_seq) as maxsq FROM e_send_frm_log "
                            "WHERE  cmd_id=?", // addr=? AND
                            cmdId);            // addr,
                                               // conn.close();
      ;
  result.setFetchSize(1);	//查询只获取1条记录。在SQL内用LIMIT 1也可。
  if(result.next())
  {
      int max = result.getInt("maxsq");//只获取maxsq的1条记录
  }
  ```


* 函数

  ```sql
  FLOOR()	#<=括号中值
  FROM_UNIXTIME()	#用于将UNIX时间戳转换为日期时间字符串。
  ```

  

# 33. std::thread

---

* 不能复制，只能移动。提供了：

  ```cpp
  thread(thread&& Other) noexcept;
  thread& operator=(thread&& Other) noexcept;
  ```

  ```cpp
   std::thread receiverths[constSize];
  
    for (int i = 0; i < constSize; i++) 
    {
      //非匿名对象是std::thread th(startReceiver, port)
      //std::thread(startReceiver, port)创建的是一个匿名的临时thread对象，是右值，不用显式调用std::move
      //receiverths[i] = std::move(std::thread(startReceiver, port));
      receiverths[i] = std::thread(startReceiver, port);
      receiverths[i].detach();
    }
  ```

* 绑定函数

  ```cpp
  //构造绑定
  thread(func,args);
  //移动
  thread th = thread(func, args);
  ```

* 作为类成员的thread绑定函数

  ```cpp
  class AA
  {
  private:
      thread th;	//怎么绑定成员函数呢？
      void func1();//无参数
      void func2(args);//有参数
  }
  ```

  ```cpp
  //直接在 std::thread 的构造函数中绑定类的成员函数。
  #include <iostream>
  #include <thread>
  
  class MyClass 
  {
  public:
      int args;
      std::thread th1(&MyClass::func1, this); 
      
      MyClass
      {
          std::thread th2(&MyClass::func2, this, args);//传成员变量作为参数到func2
          th2.detach();
      }
  };
  
  int main() 
  {
      MyClass obj;
      return 0;
  }
  ```

  ```cpp
  //lambda调用成员函数
  #include <iostream>
  #include <thread>
  
  class MyClass 
  {
  public:
      int args;
      MyClass() 
      {
          std::thread worker1([this] {
              this->func1();
          });
          worker1.detach();  
          
          std::thread worker2([this,args] {
              this->func2(args);//传参到func2
          });
          worker2.detach();
      }
  };
  
  int main() {
      MyClass obj;
      return 0;
  }
  ```

  ```cpp
  //使用可调用对象绑定器 std::bind 将类的成员函数和 this 指针绑定，然后传递给 std::thread。
  #include <iostream>
  #include <thread>
  #include <functional>
  
  class MyClass {
  public:
      int args;
      MyClass() 
      {
          std::thread worker1(std::bind(&MyClass::func1, this));
          worker1.detach(); 
          
          std::thread worker2(std::bind(&MyClass::func2, this, args));
          worker2.detach(); 
      }
  };
  
  int main() {
      MyClass obj;
      return 0;
  }
  ```


# 34. git回滚

---

* 用暂存区回滚工作区

  ```git
  git checkout .
  git checkout filename
  ```

* 回滚暂存区，工作区不变

  ```cpp
  git reset HEAD filename
  ```

* 用本地仓库回滚暂存区+工作区

  ```cpp
  git reset --hard <commit>
  git reset --hard HEAD~0	#本次提交
  ```

# 35. 静态成员函数定义可写在头文件中

---

> static函数定义较短，逻辑简单时。
>
> 函数定义较长时，还是写在.cpp中。

* 直接写在类声明中。这是隐式内联的，无需显式inline。

  ```cpp
  // MyClass.h
  #ifndef MYCLASS_H
  #define MYCLASS_H
  
  #include <iostream>
  
  class MyClass {
  public:
      static void staticFunc() {
          std::cout << "Static function called!" << std::endl;
      }
  };
  
  #endif // MYCLASS_H
  ```

* 通过显式inline，可写在类声明外部，但也在头文件里。

  ```cpp
  // MyClass.h
  #ifndef MYCLASS_H
  #define MYCLASS_H
  
  #include <iostream>
  
  class MyClass {
  public:
      static void staticFunc();
  };
  
  //显式内联，写在类声明外部
  inline void MyClass::staticFunc() {
      std::cout << "Static function called!" << std::endl;
  }
  
  #endif // MYCLASS_H
  ```

# 36. socket

---

## （1）可选项

```cpp
int setsockopt(int sockfd, int level, int optname, const void *optval, socklen_t optlen);
sockfd：套接字描述符。
level：选项级别。通常为 SOL_SOCKET 或其他协议相关的级别（如 IPPROTO_TCP）。
optname：选项名称。
optval：选项值的指针。
optlen：选项值的长度。
```

* 端口被释放后立即可重用

  ```cpp
   callLibFunc(std::bind(setsockopt, sock, SOL_SOCKET, SO_REUSEADDR,
                                       &on, sizeof(on)),
                             "sock::create/setsockopt");//setsockopt设置socket可选项：服务端释放端口后端口立即可以被重用
  ```

* 启用或禁用 TCP 保活机制

  ```cpp
  //enableKeepAlive：通常是一个布尔值（int 类型），1 表示启用，0 表示禁用
  int enableKeepAlive = 1;
  setsockopt(sock, SOL_SOCKET, SO_KEEPALIVE, &enableKeepAlive, sizeof(enableKeepAlive));
  ```

* 设置空闲多久开始保活探测（以秒为单位）

  ```cpp
  int keepIdle = 60; // 60秒
  setsockopt(sock, IPPROTO_TCP, TCP_KEEPIDLE, &keepIdle, sizeof(keepIdle));
  ```

* 设置保活探测间隔（s为单位）

  ```cpp
  int keepInterval = 10; // 10秒
  setsockopt(sock, IPPROTO_TCP, TCP_KEEPINTVL, &keepInterval, sizeof(keepInterval));
  ```

* 在宣布对端无法连接前的保活探测最大尝试次数

  ```cpp
  int keepCount = 3;
  setsockopt(sock, IPPROTO_TCP, TCP_KEEPCNT, &keepCount, sizeof(keepCount));
  ```

* 示例

  ```cpp
  int enableKeepAlive = 1;
  int keepIdle = 1 * 60; 
  int keepInterval = 5;  
  int keepCount = 3;                
  
  setsockopt(sock, SOL_SOCKET, SO_KEEPALIVE, &enableKeepAlive, sizeof(enableKeepAlive));
  setsockopt(sock, IPPROTO_TCP, TCP_KEEPIDLE, &keepIdle, sizeof(keepIdle));
  setsockopt(sock, IPPROTO_TCP, TCP_KEEPINTVL, &keepInterval, sizeof(keepInterval));
  setsockopt(sock, IPPROTO_TCP, TCP_KEEPCNT, &keepCount, sizeof(keepCount));
  
  /*
  那么保活机制的流程如下：
  TCP 连接空闲 60 秒后，开始发送第一个保活探测。
  如果没有收到响应，每隔 10 秒发送一次保活探测，共发送 3 次。
  如果 3 次探测后仍未收到响应，TCP 连接将被关闭。
  */
  ```


# 37. 构造帧

---

* 内存地址指针用`void*`，只表示起始地址

* 使用具体长度时：

  ```cpp
  void* p;
  *(uint32_t)p=12;	//p的前4B存储12这个数字
  *(uint64_t)p=12;	//p的前8字节存储
  ```

  

# 38. 执行脚本文件

---

在Linux中，*system()*函数用于执行一个由字符串表示的命令。它通过调用*/bin/sh -c*来执行命令，并在命令执行完毕后返回。这个函数的定义如下：

```cpp
#include <stdlib.h>

int system(const char *command);
```

# 39. 查找文件

---

```shell
find ./ -type f -name "curl.h"#查找文件

find ./ -type d -name "curl"#查找目录
```

# 40. libcurl

---

> 使用方法：https://www.cnblogs.com/heluan/p/10177475.html
>
> 安装：
>
> ```bash
> sudo apt-get install libcurl4-openssl-dev
> curl --version	#查看版本
> ```

## （1）通过FTP向服务器上传数据文件

curl_easy_setopt设定选项：

* CURLOPT_URL

* CURLOPT_USERNAME
* CURLOPT_PASSWORD
* CURLOPT_UPLOAD
* CURLOPT_READFUNCTION
* CURLOPT_READDATA

### **服务器基础配置**

#### **(1) 安装 FTP 服务器**

- **Linux（vsftpd）**：

  bash

  复制

  ```
  sudo apt install vsftpd  # Debian/Ubuntu
  sudo systemctl start vsftpd
  sudo systemctl enable vsftpd
  ```

- **Windows**：使用 `FileZilla Server` 或 `IIS FTP`。

#### **(2) 配置 FTP 用户和权限**

- **创建专用用户**（推荐）：

  bash

  复制

  ```
  sudo useradd -m ftpuser  # 创建用户
  sudo passwd ftpuser      # 设置密码
  ```

- **设置用户目录权限**：

  bash

  复制

  ```
  sudo chown -R ftpuser:ftpuser /home/ftpuser  # 确保用户对目录有写权限
  ```

#### **(3) 允许写入操作**

- **修改 FTP 服务器配置**（以 `vsftpd` 为例，配置文件 `/etc/vsftpd.conf`）：

  ini

  复制

  ```
  write_enable=YES          # 允许写入
  local_enable=YES          # 允许本地用户登录
  anonymous_enable=NO       # 禁用匿名登录（按需）
  chroot_local_user=YES     # 限制用户在其主目录
  allow_writeable_chroot=YES
  ```

------

### **2. 防火墙和端口开放**

#### **(1) 开放 FTP 端口**

- **控制连接端口**：默认 `21/tcp`。
- **数据连接端口**：
  - **主动模式（PORT）**：服务器从 `20/tcp` 主动连接客户端。
  - **被动模式（PASV）**：客户端连接服务器的随机高端口（需配置范围）。

#### **(2) 配置被动模式端口范围**

- **修改 `vsftpd.conf`**：

  ini

  复制

  ```
  pasv_enable=YES
  pasv_min_port=50000      # 被动模式最小端口
  pasv_max_port=51000      # 被动模式最大端口
  pasv_address=YOUR_SERVER_PUBLIC_IP  # 服务器公网IP（若在NAT后）
  ```

- **开放防火墙端口**：

  bash

  复制

  ```
  sudo ufw allow 21/tcp
  sudo ufw allow 50000:51000/tcp
  ```

------

### **3. SSL/TLS 加密（可选）**

若需通过 **FTPS**（FTP over SSL）加密传输：

#### **(1) 生成 SSL 证书**

bash

复制

```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
    -keyout /etc/ssl/private/vsftpd.key \
    -out /etc/ssl/certs/vsftpd.crt
```

#### **(2) 启用 SSL 支持**

修改 `vsftpd.conf`：

ini

复制

```
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
rsa_cert_file=/etc/ssl/certs/vsftpd.crt
rsa_private_key_file=/etc/ssl/private/vsftpd.key
```

#### **(2) 关键选项说明**

- **被动模式**（默认启用）：

  c

  复制

  ```
  curl_easy_setopt(curl, CURLOPT_FTP_USE_EPSV, 1L);  // 推荐使用 EPSV
  ```

- **主动模式**（不推荐）：

  c

  复制

  ```
  curl_easy_setopt(curl, CURLOPT_FTPPORT, "-");  // 禁用客户端端口
  curl_easy_setopt(curl, CURLOPT_FTP_USE_EPRT, 0L);
  ```

- **SSL/TLS 加密**：

  c

  复制

  ```
  curl_easy_setopt(curl, CURLOPT_USE_SSL, CURLUSESSL_ALL);  // 强制 SSL
  curl_easy_setopt(curl, CURLOPT_CAINFO, "/path/to/cacert.pem");  // CA 证书
  ```

### **客户端上传文件代码**

```cpp
    CURL *curl;
    CURLcode res;

    std::string ftp_url = hzi::config.ftpUrl + fileName;
    std::string username = hzi::config.ftpUserName;
    std::string password = hzi::config.ftpPassWord;

    cout<<"ftp_url :"<<ftp_url<<" , fileName = "<<fileName<<" , dataFile = "<<dataFile<<endl;

    curl_global_init(CURL_GLOBAL_DEFAULT);
    curl = curl_easy_init();

    if (curl) {
      curl_easy_setopt(curl, CURLOPT_URL, ftp_url.c_str());// 设置FTP URL地址

      // Specify username and password
      curl_easy_setopt(curl, CURLOPT_USERNAME, username.c_str());
      curl_easy_setopt(curl, CURLOPT_PASSWORD, password.c_str());

      // Enable uploading
      curl_easy_setopt(curl, CURLOPT_UPLOAD, 1L);// 设置上传模式

      // Specify the read callback function
      curl_easy_setopt(curl, CURLOPT_READFUNCTION, read_callback);
      curl_easy_setopt(curl, CURLOPT_READDATA, &dataFile);//指定传入字符串dataFile。

      // Perform the request, res will get the return code
      res = curl_easy_perform(curl);

      // Check for errors
      if (res != CURLE_OK) {
        //  fprintf(stderr, "curl_easy_perform() failed: %s\n",
        //  curl_easy_strerror(res));
        std::cout << "curl_easy_perform() failed: %s\n";
      } else {
        std::cout << "File uploaded successfully!" << std::endl;
      }

      // Cleanup
      curl_easy_cleanup(curl);
    }

    curl_global_cleanup();
  }
```

```cpp
//回调函数，用于读取数据。除了根据curl_easy_setopt处实际传入类型强转void*指针，下面写法基本固定。
size_t read_callback(void *ptr, size_t size, size_t nmemb, void *stream) {
  std::string *data = static_cast<std::string *>(stream);//调用处stream传入字符串的指针
  size_t buffer_size = size * nmemb;//缓冲区：nmemb为单元数，size为单元大小

  if (data->size() < buffer_size) {
    buffer_size = data->size();
  }

  memcpy(ptr, data->c_str(), buffer_size);
  data->erase(0, buffer_size);

  return buffer_size;
}
```

# 41. unix信号定时器

----

> `struct itimerval` 是一个在 Unix-like 系统（如 Linux）中用于设置定时器时间间隔的结构体。该结构体通常与 `setitimer()` 系统调用配合使用，用于配置不同类型的定时器。
>
> 定时器类型：
>
> * 实时定时器 `ITIMER_REAL`：到期时发送 `SIGALRM` 信号
>
> * 虚拟定时器 `ITIMER_VIRTUAL`：进程在用户态下的执行时间累计达到设定值时发送 `SIGVTALRM`
>
> * 性能定时器 `ITIMER_PROF`：进程在用户态和内核态下的总执行时间达到设定值时发送 `SIGPROF` 信号

## （1）itimerval

```cpp
struct itimerval
{
    struct timeval it_interval; /* 定时器间隔时间 */
    struct timeval it_value;    /* 定时器初始到期时间 */
};

struct timeval
{
    int tv_sec;//s
    int tv_usec;//us
}
```

## （2）setitimer

```cpp
#include <sys/time.h>

int setitimer(int which, const struct itimerval *new_value, struct itimerval *old_value);

//which-定时器类型
//new_value-设置定时器的新值
//old_value-存储定时器之前的值。如果不需要旧值，可以传递 NULL
```

## （3）demo

```cpp
struct sigaction sa;
struct itimerval client_timer;
void start_timerOnce(int sec, int usec){
    // 设置定时器信号的处理函数
    sa.sa_handler = timer_handler_client;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;
    sigaction(SIGALRM, &sa, NULL);

    // 设置定时器的时间间隔
    client_timer.it_value.tv_sec = sec;//首次到期时间，单位为秒和微秒
    client_timer.it_value.tv_usec = usec;
    //定时器不重复
    client_timer.it_interval.tv_sec = 0;//重复间隔时间，单位为秒和微秒
    client_timer.it_interval.tv_usec = 0;

    // 开启定时器
    setitimer(ITIMER_REAL, &client_timer, NULL);
}
void stop_timer(){
    client_timer.it_value.tv_sec = 0;
    client_timer.it_value.tv_usec = 0;
    client_timer.it_interval.tv_sec = 0;
    client_timer.it_interval.tv_usec = 0;
    setitimer(ITIMER_REAL, &client_timer, NULL);
}
```

## 42. 结构体赋值也可初始化列表

---

```cpp
st.a=1;	//给成员赋值
```

```cpp
//列表初始化
 ChanelsConfig chCfg
 {
 	(uint8_t)rslt.getInt("chn_no"),   (uint8_t)rslt.getInt("type_id"),
 	(float)rslt.getDouble("loc_x"),   (float)rslt.getDouble("loc_y"),
 	(float)rslt.getDouble("loc_z"),   string(label),
 	(uint8_t)rslt.getInt("state_id"), (uint8_t)rslt.getInt("dev_id")
 };
```

# 43. make编译可指定Makefile

---

```cpp
//-f 选项：-f 选项用于指定 make 使用的 Makefile 文件名。默认情况下，make 会查找名为 Makefile 或 makefile 的文件。
make -f Makefile_gdb
```

