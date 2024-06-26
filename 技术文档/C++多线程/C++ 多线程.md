# 多线程基础

## 创建线程

```c++
#include <iostream>
#include <thread> // ①

using namespace std; // ②

void hello() { // ③
  cout << "Hello World from new thread." << endl;
}

int main() {
  thread t(hello); // ④
  t.join(); // ⑤

  return 0;
}
```

## join和detach

一旦启动线程之后，我们必须决定是要等待直到它结束（join），还是让他独立运行（detach）。

如果在thread对象销毁时还没有做出决定（即例子中推出main函数时），则thread对象在析构函数中将调用std::terminate()从而导致我们的进程异常退出。

需要注意的是：在我们做决定的时候，很可能线程已经执行完了（例如上面的示例中线程的逻辑仅仅是一句打印，执行时间会很短）。新的线程创建之后，究竟是新的线程先执行，还是当前线程的下一条语句先执行这是不确定的，因为这是由操作系统的调度策略决定的。不过这不要紧，我们只要在`thread`对象销毁前做决定即可。

- `join`：调用此接口时，当前线程会一直阻塞，直到目标线程执行完成（当然，很可能目标线程在此处调用之前就已经执行完成了，不过这不要紧）。因此，如果目标线程的任务非常耗时，你就要考虑好是否需要在主线程上等待它了，因此这很可能会导致主线程卡住。
- `detach`：`detach`是让目标线程成为守护线程（daemon threads）。一旦`detach`之后，目标线程将独立执行，即便其对应的`thread`对象销毁也不影响线程的执行。并且，你无法再与之通信。





















































