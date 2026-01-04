---
title: C&C++
tags: 
  - language
categories: 
  -	Memo
cover: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202509151730900.jpg
top_img: https://pricket.oss-cn-beijing.aliyuncs.com/D:%5CSoftware%5CPicGo%5Cpicture%5C202509151728823.jpg
abbrlink: 7
---

# C&C++

## STL

### Vector 连续动态数组

#### 特性

1. **动态大小**：vector 的大小可根据元素数量的增减自动调整，无需手动管理内存
2. **连续存储**：元素在内存中连续存储，类似于内置数组，因此支持通过索引进行快速随机访问
3. **自动内存管理**：vector 自动处理内存的分配和释放，减少了内存泄漏的风险
4. **类型安全**：作为模板类，vector 可以存储任意类型的元素（内置类型、对象、指针等），并在编译时进行类型检查

#### 初始化

| 初始化方式       | 示例代码                                   | 说明                           |
| ---------------- | ------------------------------------------ | ------------------------------ |
| 默认构造         | `std::vector<int> vec1;`                   | 创建空的 vector，动态添加      |
| 指定大小         | `std::vector<int> vec2(5);`                | 5 个元素，默认初始化为 0       |
| 指定大小和初始值 | `std::vector<int> vec3(5, 10);`            | 5 个元素，每个都初始化为 10    |
| 初始化列表       | `std::vector<int> vec4 = {1, 2, 3, 4, 5};` | 使用花括号初始化列表或表示列表 |
| 拷贝构造         | `std::vector<int> vec5(vec4);`             | 用另一个 vector 初始化         |

#### 元素访问

| 方法      | 示例代码                   | 特点                                                         |
| --------- | -------------------------- | ------------------------------------------------------------ |
| `at()`    | `int element = vec.at(1);` | **进行边界检查**，如果索引越界会抛出 `std::out_of_range`异常，更安全。 |
| `front()` | `int first = vec.front();` | 访问第一个元素。                                             |
| `back()`  | `int last = vec.back();`   | 访问最后一个元素。                                           |

#### 添加元素

| 方法             | 示例代码                           | 特点                                                         |
| ---------------- | ---------------------------------- | ------------------------------------------------------------ |
| `push_back()`    | `vec.push_back(10);`               | 将元素（通过拷贝或移动）添加到末尾。                         |
| `emplace_back()` | `vec.emplace_back(10);`            | **(C++11) 更高效**，直接在容器尾部构造元素，避免创建临时对象，节省拷贝开销。 |
| `insert()`       | `vec.insert(vec.begin() + 1, 20);` | 在指定迭代器位置插入元素。**注意**：在中间插入会导致元素移动，时间复杂度为 O(n)。 |

#### 删除元素

| 方法         | 示例代码                      | 特点                                                         |
| ------------ | ----------------------------- | ------------------------------------------------------------ |
| `pop_back()` | `vec.pop_back();`             | 删除末尾元素，时间复杂度 O(1)。                              |
| `erase()`    | `vec.erase(vec.begin() + 1);` | 删除指定位置的元素或范围。**注意**：在中间删除会导致元素移动，时间复杂度 O(n)。 |
| `clear()`    | `vec.clear();`                | 清空所有元素，**但不会释放容量**（`capacity`保持不变）。     |

#### 大小与容量

| 方法         | 示例代码                       | 说明                                                         |
| ------------ | ------------------------------ | ------------------------------------------------------------ |
| `size()`     | `size_t num = vec.size();`     | 返回当前**元素个数**。                                       |
| `capacity()` | `size_t cap = vec.capacity();` | 返回当前已分配的**内存空间**能容纳的元素数量（≥ `size`）。   |
| `empty()`    | `bool is_empty = vec.empty();` | 检查 vector 是否为空。                                       |
| `reserve(n)` | `vec.reserve(100);`            | **预分配内存**。提前分配至少能容纳 n 个元素的空间，避免多次扩容，提升性能。 |

#### 遍历方法

```c++
std::vector<int> vec = {1, 2, 3, 4, 5};

// 1. 普通迭代器
for (std::vector<int>::iterator it = vec.begin(); it != vec.end(); ++it) {
    std::cout << *it << " ";
}

// 2. 常量迭代器 (C++11, 防止修改)
for (auto cit = vec.cbegin(); cit != vec.cend(); ++cit) {
    std::cout << *cit << " ";
}

// 3. 反向迭代器
for (auto rit = vec.rbegin(); rit != vec.rend(); ++rit) {
    std::cout << *rit << " "; // 输出: 5 4 3 2 1
}

// 4. 基于范围的 for 循环 (C++11, 最简洁)
for (int num : vec) {
    std::cout << num << " ";
}
```

#### 与STL结合

```c++
#include <algorithm>
#include <numeric>

std::vector<int> vec = {5, 3, 8, 1, 4};

// 排序
std::sort(vec.begin(), vec.end()); // {1, 3, 4, 5, 8}

// 反转
std::reverse(vec.begin(), vec.end()); // {8, 5, 4, 3, 1}

// 查找 (返回迭代器)
auto it = std::find(vec.begin(), vec.end(), 5);
if (it != vec.end()) {
    std::cout << "Found at index: " << std::distance(vec.begin(), it);
}

// 删除特定值（remove-erase惯用法），remove方法将指定值移动至数组末尾并返回首个符合值的迭代器
vec.erase(std::remove(vec.begin(), vec.end(), 4), vec.end());

// 数值计算 + 求和
int sum = std::accumulate(vec.begin(), vec.end(), 0); 
```

### Stack

#### 初始化

| **操作**      | **说明**           | **示例代码**                      | **注意事项**                                     |
| ------------- | ------------------ | --------------------------------- | ------------------------------------------------ |
| `push(value)` | 将元素压入栈顶     | `myStack.push(10);`               |                                                  |
| `pop()`       | 移除栈顶元素       | `myStack.pop();`                  | **不返回被移除的元素值**，需先通过 `top()`获取。 |
| `top()`       | 返回栈顶元素的引用 | `int topElement = myStack.top();` | 在**空栈**上调用会导致未定义行为。               |
| `empty()`     | 检查栈是否为空     | `if (myStack.empty()) { ... }`    | 空返回 `true`，否则返回 `false`。                |
| `size()`      | 返回栈中元素个数   | `std::cout << myStack.size();`    |                                                  |

### Unordered_map

#### 初始化

```C++
std::unordered_map<std::string, int> ageMap;
```

#### 插入元素

```c++
// 方法1: 使用下标操作符[] (如果键已存在，则会更新值)
ageMap["Charlie"] = 35;

// 方法2：insert, 非覆盖，返回值类型pair<iter, bool>提供返回信息
auto result = umap.insert({3, "three"}); 
// std::pair<int, std::string>(1, "one") || std::make_pair(2, "two")
if (result.second) {
    // 插入成功
}

// 方法3：emplace，非覆盖，集合内构造元素，自定义类型更高效，返回值类型pair<iter, bool>提供返回信息
auto result = umap.emplace(6, "six");
if (result.second) {
    // 插入成功
}
```

#### 访问与查找元素

```c++
// 查找元素是否存在 (推荐安全查找方式)
auto it = ageMap.find("Frank");
if (it != ageMap.end()) {
    std::cout << "Found Frank, age: " << it->second << std::endl;
} else {
    std::cout << "Frank not found." << std::endl;
}

// 使用 count() 检查键是否存在 (返回0或1)
if (ageMap.count("Alice") > 0) {
    std::cout << "Alice exists." << std::endl;
}
```

#### 删除元素

```c++
// 通过键删除元素
ageMap.erase("Bob");
```

#### 遍历元素

```C++
// 方法1: 使用范围for循环 (C++11)
for (const auto& pair : ageMap) {
    std::cout << pair.first << ": " << pair.second << std::endl;
}

// 方法2: 使用迭代器
for (auto it = ageMap.begin(); it != ageMap.end(); ++it) {
    std::cout << it->first << ": " << it->second << std::endl;
}

// 方法3: 结构化绑定 (C++17)
for (const auto& [name, age] : ageMap) {
    std::cout << name << ": " << age << std::endl;
}
```

#### 其他操作

```C++
// 获取大小
std::cout << "Size: " << ageMap.size() << std::endl;

// 检查是否为空
if (ageMap.empty()) {
    std::cout << "The map is empty." << std::endl;
}
```

#### 数组类键

```C++
#include<bits/stdc++.h>
using namespace std;

int main() {
    auto arrayHash = [fn = hash<int>{}] (const array<int, 26>& arr) -> size_t {
        return accumulate(arr.begin(), arr.end(), [&](size_t acc, int num) {
         	return (acc << 1) ^ fn 
        });
    }
    
    return 0;
}
```

### Pair

#### 初始化

| 初始化方式           | 示例                                                         | 说明                               |
| -------------------- | ------------------------------------------------------------ | ---------------------------------- |
| **直接构造**         | `std::pair<int, std::string> p(42, "Hello");`                | 在构造函数中直接提供两个值。       |
| **使用 `make_pair`** | `auto p = std::make_pair(42, "Hello");`                      | 借助函数模板推导类型，通常更简洁。 |
| **列表初始化**       | `std::pair<int, std::string> p = {42, "Hello"};`             | 使用**花括号**（C++11起）。        |
| **默认构造后赋值**   | `std::pair<int, std::string> p; p.first = 42; p.second = "Hello";` | 先创建对象，再分别赋值。           |

### String

#### 特性

1. **自动内存管理**：无需手动分配和释放内存。
2. **动态大小调整**：可根据内容自动调整存储空间。
3. **丰富的成员函数**：支持查找、替换、插入、删除等多种操作。
4. **边界安全检查**：例如 `at()`方法会在越界时抛出异常。
5. **运算符重载**：支持使用 `+`(连接)、`==`(比较) 等直观运算符。
6. **兼容 C 风格字符串**：可通过 `c_str()`方法转换为 `const char*`。

#### 运算符重载 + 构造函数

```c++
#include <iostream>
#include <cstring>

class MyString {
private:
    char* str; // 存储字符串内容的指针
    size_t length; // 字符串长度

public:
    // 默认构造函数
    MyString() : str(nullptr), length(0) {
        str = new char[1];
        str[0] = '\0';
    }
    // 调用方式：
    // 显式调用：MyString str
    // 隐式调用：在动态分配数组或作为未显式初始化的成员变量时调用，例如 MyString* arr = new MyString[10];数组中的每个元素都会调用默认构造函数。

    // 从C风格字符串构造
    MyString(const char* s) {
        if (s) {
            length = strlen(s);
            str = new char[length + 1]; // 分配内存，+1用于'\0'
            strcpy(str, s);
        } else { // 处理空指针
            str = new char[1];
            str[0] = '\0';
            length = 0;
        }
    }
    // 调用方式：
    // 直接初始化：MyString str("Hello");或 MyString str = "Hello";
	// 转换构造函数：在需要类型转换的场合，编译器可能会自动调用，例如 void func(MyString s); func("Hello");。

    // 拷贝构造函数
    MyString(const MyString& other) {
        length = other.length;
        str = new char[length + 1];
        strcpy(str, other.str);
    }
    // 调用方式：
    // 显式初始化：MyString newStr(oldStr);或 MyString newStr = oldStr;
	// 函数值传递：void someFunction(MyString s); someFunction(existingStr); - 形参 s由 existingStr拷贝构造
	// 函数值返回：MyString createString() { MyString local; return local; } - 返回时可能调用拷贝构造（现代编译器通常会优化）

    // 析构函数
    ~MyString() {
        delete[] str;
    }

    // 重载运算符时，成员函数的左操作数为隐式的，故只需声明有操作数；非成员函数（eg：友元），没有隐式的this参数，第一个参数为左操作数，第二个为右操作数。
    // 重载 + 运算符，用于字符串连接
    MyString operator+(const MyString& other) const {
        size_t newLength = length + other.length;
        char* newStr = new char[newLength + 1];
        strcpy(newStr, str);
        strcat(newStr, other.str);
        MyString result(newStr);
        delete[] newStr;
        return result;
    }

    // 重载 == 运算符，用于字符串比较
    bool operator==(const MyString& other) const {
        if (length != other.length) {
            return false;
        }
        return strcmp(str, other.str) == 0;
    }

    // 重载 != 运算符
    bool operator!=(const MyString& other) const {
        return !(*this == other);
    }

    // 获取C风格字符串
    const char* c_str() const {
        return str;
    }

    // 获取字符串长度
    size_t size() const {
        return length;
    }

    // 重载输出运算符 << 
    // 友元函数允许直接访问类的私有成员，且为全局函数
    friend std::ostream& operator<<(std::ostream& os, const MyString& myStr) {
        os << myStr.str;
        return os;
    }
};

int main() {
    // 测试字符串连接和比较
    MyString s1("Hello");
    MyString s2("World");
    MyString s3 = s1 + " " + s2; // 使用 + 运算符连接字符串

    std::cout << "s1: " << s1 << std::endl;
    std::cout << "s2: " << s2 << std::endl;
    std::cout << "s3: " << s3 << std::endl;

    // 测试字符串比较
    MyString s4("Hello");
    std::cout << "s1 == s4: " << (s1 == s4 ? "true" : "false") << std::endl;
    std::cout << "s1 == s2: " << (s1 == s2 ? "true" : "false") << std::endl;
    std::cout << "s1 != s2: " << (s1 != s2 ? "true" : "false") << std::endl;

    return 0;
}
```

#### 初始化

| **初始化方式**                    | **说明**                      | **示例**                         |
| --------------------------------- | ----------------------------- | -------------------------------- |
| `string()`                        | 默认构造，创建空字符串 `""`   | `string s1;`                     |
| `string(const char* s)`           | 用 C 风格字符串构造           | `string s2("Hello");`            |
| `string(const string& str)`       | 拷贝构造函数                  | `string s3(s2);`                 |
| `string(size_t n, char c)`        | 用 `n`个字符 `c`构造          | `string s4(5, 'A'); // "AAAAA"`  |
| `string(const char* s, size_t n)` | 用 C 字符串的前 `n`个字符构造 | `string s5("Hello", 2); // "He"` |

#### 字符串连接

```c++
string s1 = "Hello";
string s2 = "World";

// 方法1: 使用 + 运算符
string s3 = s1 + ", " + s2; // "Hello, World"

// 方法2: 使用 append()
s1.append(" ").append(s2);   // s1 变为 "Hello World"
```

#### 长度

```c++
string str = "C++ String";
// 两者相同
cout << str.size();   // 输出 10
cout << str.length(); // 输出 10
```

#### 访问字符

```C++
string str = "ABCDE";
char c1 = str[0];    // 'A' (不检查越界，访问越界行为未定义)
char c2 = str.at(1); // 'B' (会检查越界，越界时抛出 std::out_of_range 异常)
```

#### 常用成员函数

| **函数声明**                                       | **功能说明**                                                 | **示例**                                       |
| -------------------------------------------------- | ------------------------------------------------------------ | ---------------------------------------------- |
| `bool empty() const`                               | 检查字符串是否为空（长度为0）                                | `if (str.empty()) { /* ... */ }`               |
| `void clear()`                                     | 清空字符串（长度变为0，但容量capacity可能不变）              | `str.clear();`                                 |
| `char& front()`                                    | 访问第一个字符                                               | `char first = str.front();`                    |
| `char& back()`                                     | 访问最后一个字符                                             | `char last = str.back();`                      |
| `void push_back(char c)`                           | 在字符串末尾追加**字符** `c`                                 | `str.push_back('!');`                          |
| `string& append(const string& str)`                | 在字符串末尾追加**另一个字符串或子串**                       | `str.append("!!!");`或 `str.append(s2, 1, 3);` |
| `string& insert(size_t pos, const string& str)`    | 在指定位置 `pos`插入**字符串**                               | `str.insert(5, " inserted ");`                 |
| `string& erase(size_t pos = 0, size_t len = npos)` | 从指定位置 `pos`开始删除 `len`个字符（若不指定 `len`或 `len`为 `npos`，则删除直到末尾） | `str.erase(5, 3);`// 从位置5开始删除3个字符    |
| `void swap(string& other)`                         | 交换两个字符串的内容                                         | `str1.swap(str2);`                             |

#### 字符串操作

**提取子串**

```c++
string str = "Hello, C++";
string sub1 = str.substr(7);    // "C++" (从索引7开始到末尾)
string sub2 = str.substr(0, 5); // "Hello" (从索引0开始，取5个字符)
```

**查找子串或字符**

```c++
string str = "C++ is powerful and C++ is fast";
size_t pos1 = str.find("C++");    // 返回首次出现的位置 (0)
size_t pos2 = str.find("C++", 1); // 从位置1开始查找，返回第二次出现的位置
size_t pos3 = str.find("Java");   // 找不到，返回 string::npos

// rfind() 从后向前查找
size_t last_pos = str.rfind("C++"); // 返回最后一次出现的位置

// 检查是否找到
if (pos != string::npos) {
    cout << "Found at: " << pos << endl;
} else {
    cout << "Not found!" << endl;
}
```

#### 数值转换

```c++
// 字符串 → 数值
string numStr = "12345";
int num = std::stoi(numStr); // 12345
double d = std::stod("3.14"); // 3.14

// 数值 → 字符串
int num = 100;
double pi = 3.14159;
string s1 = std::to_string(num); // "100"
string s2 = std::to_string(pi);  // "3.141590" (注意精度)
```

#### 与 C 风格字符串的转换

```C++
// C 风格字符串 → std::string
const char* cstr = "Hello";
string str(cstr); // 直接构造

// std::string → C 风格字符串 (只读)
const char* converted_cstr = str.c_str(); // 注意：返回的指针在 str 修改后可能失效

// 如果需要可修改的拷贝，需要手动复制
char buffer[20];
// 安全复制：使用 strncpy 并确保终止
strncpy(buffer, str.c_str(), sizeof(buffer) - 1);
buffer[sizeof(buffer) - 1] = '\0';
```

#### 遍历

```C++
string str = "Hello";

// 1. 使用下标遍历 (最常用)
for (size_t i = 0; i < str.size(); ++i) {
    cout << str[i] << " ";
}

// 2. 使用迭代器
for (string::iterator it = str.begin(); it != str.end(); ++it) {
    cout << *it << " ";
}

// 3. 使用常量迭代器 (确保不修改内容)
for (string::const_iterator it = str.begin(); it != str.end(); ++it) {
    cout << *it << " ";
}

// 4. 使用反向迭代器 (从后往前)
for (string::reverse_iterator it = str.rbegin(); it != str.rend(); ++it) {
    cout << *it << " ";
}

// 5. 使用 C++11 范围 for 循环 (推荐，简洁安全)
for (char ch : str) { // 如果需要修改，用 char& ch
    cout << ch << " ";
}
```

### 自定义比较函数

#### 函数指针

```c++
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

// 函数指针：按绝对值大小升序排序
bool compareByAbsoluteValue(int a, int b) {
    return abs(a) < abs(b);
}

int main() {
    vector<int> vec = {-3, 1, -4, 2, -1, 5};
    
    sort(vec.begin(), vec.end(), compareByAbsoluteValue); // 传入函数指针
    
    for (int num : vec) {
        std::cout << num << " ";
    }
    
    return 0;
}
```

#### Lambda

```C++
#include <iostream>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;

int main() {
    vector<std::string> words = {"apple", "banana", "cherry", "date"};
    
    // 使用Lambda按字符串长度降序排序
    sort(words.begin(), words.end(), 
              [](const string& a, const string& b) {
                  return a.size() > b.size(); // 注意是 > 表示降序
              });
    
    for (const auto& word : words) {
        std::cout << word << " ";
    }
    return 0;
}
```

#### 函数对象

拥有自己的成员变量，自定义比较方式或进行记录

```C++
#include <iostream>
#include <vector>
#include <algorithm>
#include <functional>
using namespace std;

// 函数对象：记录自己被调用的次数
class CallCounter {
private:
    int count; // 内部状态

public:
    CallCounter() : count(0) {}

    int operator()(int x) {
        ++count; // 修改状态
        return x;
    }

    int getCount() const {
        return count;
    }
};

int main() {
    std::vector<int> numbers = {1, 2, 3, 4, 5};
    CallCounter counter; 
    // 区分命名对象与临时对象
    // 前者CallCounter counter定义变量名，后者CallCounter()匿名无标识

    // 使用 ref 包装 counter，以传递引用, 避免值传递无法修改当前作用域下对象的count.
    std::for_each(numbers.begin(), numbers.end(), ref(counter));
    
    std::cout << "The function object was called " << counter.getCount() << " times.\n";

    return 0;
}
```

## 关键点

### 栈与堆

* ListNode node(1) VS ListNode *node = new ListNode(1)
* 栈空间与堆空间的分配问题

### 输入输出

1. cin：基本输入操作，它会自动跳过空白字符（如空格、制表符、换行符）
2. getline：读取包含空格的整行文本
3. 通过`cin.ignore()`忽略换行符

```c++
// 循环直到文件结束
while (cin >> num) {
    ...
}

string line;
while (getline(cin, line)) { // 读取整行
	istringstream iss(line); // 创建字符串流
	int num;
  	vector<int> numbers;
    
	while (iss >> num) { // 从字符串流中读取数字
		numbers.push_back(num);
	}
        
	// 处理读取到的数字
	cout << "本行数字个数: " << numbers.size() << endl;
}
```

