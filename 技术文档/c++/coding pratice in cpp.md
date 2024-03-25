# 委托构造函数

使用列表初始化 `ByteStream` 类的对象，下面实现不正确

```c++
class ByteStream {
  private:
    std::vector<uint8_t> _vec;
    size_t _start = 0, _end = 0;
    size_t _used_cap = 0;
    size_t _capacity;
    size_t _total_written = 0;
    size_t _total_read = 0;
    bool _end_input{};
    bool _error{};
    
  public:
    ByteStream(const size_t capacity):_capacity(capacity),_vec(std::vector<uint8_t>(capacity)) {}
    
};
```

应该采用委托构造函数的方式

```c++
class ByteStream {
private:
    std::vector<uint8_t> _vec;
    size_t _start = 0, _end = 0;
    size_t _used_cap = 0;
    size_t _capacity;
    size_t _total_written = 0;
    size_t _total_read = 0;
    bool _end_input{};
    bool _error{};

public:
    ByteStream(const size_t capacity) : ByteStream(std::vector<uint8_t>(capacity), capacity) {}

    ByteStream(std::vector<uint8_t> vec, const size_t capacity)
        : _vec(std::move(vec)), _capacity(capacity) {}
};
```

使用列表初始化有以下几个好处：

1. 明确的初始化语法：列表初始化提供了一种明确的语法来初始化对象，使代码更加清晰易读。通过使用大括号 `{}`，你可以直接指定初始化值，而无需依赖于构造函数的参数列表。
2. 避免狭窄化转换：列表初始化对于类型转换更加严格，可以帮助你避免一些不明确或潜在的狭窄化转换。狭窄化转换是指将一个值赋给一个较小的类型，导致精度丢失或溢出的情况。使用列表初始化可以帮助你在编译时捕捉到这些问题，并提供更严格的类型检查。
3. 初始化复杂对象：列表初始化在初始化复杂对象时非常有用。通过使用嵌套的大括号，你可以初始化对象中的成员变量，甚至是多维数组。这样可以更方便地初始化复杂的数据结构。
4. 初始化容器对象：列表初始化对于初始化容器对象（如 `std::vector`、`std::map` 等）特别方便。你可以直接提供一个值的列表，用于初始化容器的元素，而无需显式地调用容器的成员函数。
5. 移动语义支持：列表初始化对于利用移动语义进行性能优化非常有帮助。当使用列表初始化时，编译器可以更容易地进行优化，利用移动构造函数或移动赋值运算符来避免不必要的数据复制。





In Lab 1, you’ll implement a stream reassembler—a module that stitches small pieces
of the byte stream (known as substrings, or segments) back into a contiguous stream
of bytes in the correct sequence.
2. In Lab 2, you’ll implement the part of TCP that handles the inbound byte-stream: the
TCPReceiver. This involves thinking about how TCP will represent each byte’s place
in the stream—known as a “sequence number.” The TCPReceiver is responsible for
telling the sender (a) how much of the inbound byte stream it’s been able to assemble
successfully (this is called “acknowledgment”) and (b) how many more bytes the sender
is allowed to send right now (“flow control”).
3. In Lab 3, you’ll implement the part of TCP that handles the outbound byte-stream: the
TCPSender. How should the sender react when it suspects that a segment it transmitted
was lost along the way and never made it to the receiver? When should it try again
and re-transmit a lost segment?
4. In Lab 4, you’ll combine your work from the previous to labs to create a working
TCP implementation: a TCPConnection that contains a TCPSender and TCPReceiver.
You’ll use this to talk to real servers around the world.