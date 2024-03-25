# RVO

```c++
#include <iostream>
#include <string>

std::string createString() {
    std::string str = "Hello";
    return std::move(str); // 移动本地对象
}

int main() {
    std::string result = createString();
    std::cout << result << std::endl;
    return 0;
}
```

对于字符串类型（`std::string`），返回语句中的移动操作实际上是多余的。这是因为 C++ 的标准库已经对字符串类型的返回进行了优化，即使没有移动操作，也会进行返回值优化（RVO，Return Value Optimization）。RVO 允许编译器在返回时直接构造结果对象，而不进行复制或移动操作。

因此，我们直接返回 `str`，并且没有使用 `std::move`。编译器会自动应用返回值优化，避免不必要的复制操作。

不光是`std::string`，STL中的一些组件类都有。例如`std::vecotr/std::map/std::set`



```c++
std::unique_ptr<T> unique_ptr1 = std::make_unique<T>(); // 假设这是一个 unique_ptr 对象
std::unique_ptr<T> unique_ptr2 = std::make_unique<T>();
unique_ptr2 = std::move(unique_ptr1);

```

第三行赋值之后，unique_ptr2所指向的对象会自动释放吗

