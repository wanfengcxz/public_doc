# Linux进程间通信

每个进程的用户地址空间都是独立的，一般而言是不能互相访问的，但内核空间是每个进程都共享的，所以进程之间要通信必须通过内核。

Linux 内核提供了不少进程间通信的机制：管道（包括匿名管道和命名管道）、消息队列、信号量、共享内存、Socket（套接字）等。

其中 Socket和支持不同主机上的两个进程通信。

## 管道

管道是单向传输（半双工），只能用于具有亲缘关系的进程之间的通信（父子进程/兄弟进程），是一种特殊的文件（存在于内存中，不属于任何文件系统）。其实，所谓的管道，就是内核里面的一串缓存。

```bash
# 匿名管道
ps auxf | grep mysql

# 创建命名管道 一些皆文件
mkfifo myPipe
ls -l

# 将数据写进管道 停住了 ...
echo "hello" > myPipe  

# 这是因为管道里的内容没有被读取，只有当管道里的数据被读完后，命令才可以正常退出。
# 读取管道里的数据
cat < myPipe
hello
```

创建管道

```c
#include <stdio.h>
int pipe(int fd[2]);  // 返回值：若成功返回0，失败返回-1；
```

fd[0]为读而打开，fd[1]为写而打开。

![image-20240502215208583](linux进程通信.assets/image-20240502215208583.png)

单个进程中的管道几乎没有任何用处。所以，通常调用 pipe 的进程接着调用 fork，这样就创建了父进程与子进程之间的通信通道。若要数据流从父进程流向子进程，则关闭父进程的读端（fd[0]）与子进程的写端（fd[1]）；反之，则可以使数据流从子进程流向父进程（不关闭会造成混乱）。所以说如果需要双向通信，则应该创建两个管道。

![image-20240503150712282](linux进程通信.assets/image-20240503150712282.png)

在shell中，执行 A | B命令的时候，A 进程和 B 进程都是 shell 创建出来的子进程，A 和 B 之间不存在父子关系，它俩的父进程都是 shell。

![image-20240503151606061](linux进程通信.assets/image-20240503151606061.png)

对于`匿名管道，它的通信范围是存在父子关系的进程`。

对于`命名管道，它可以在不相关的进程间也能相互通信`。因为命令管道，提前创建了一个类型为管道的设备文件，在进程里只要使用这个设备文件，就可以相互通信。

## 消息队列

消息队列生命周期随内核，如果没有释放消息队列或者没有关闭操作系统，消息队列会一直存在，而前面提到的匿名管道的生命周期，是随进程的创建而建立，随进程的结束而销毁。

消息队列不适合比较大数据的传输

## 信号量

信号量用于进程间同步，若要在进程间传递数据需要结合共享内存。信号量基于操作系统的 PV 操作，程序对信号量的操作都是原子操作。



## 共享内存

共享内存的机制，就是拿出一块虚拟地址空间来，映射到相同的物理内存中。两个或多个进程共享一个给定的存储区。

- 进程直接对内存进行存取，非常快
- 多个进程可以同时操作，所以需要进行同步

所以信号量+共享内存通常结合在一起使用，信号量用来同步对共享内存的访问。

![image-20240503153432428](linux进程通信.assets/image-20240503153432428.png)

在Linux中提供了一组函数接口用于使用共享内存，而且使用共享共存的接口还与信号量的非常相似，而且比使用信号量的接口来得简单。

```c
#include <sys/shm.h>
int shmget(key_t key, size_t size, int shmflg);
```

- 第一个参数，程序需要提供一个参数key（非0整数），它有效地为共享内存段命名，shmget()函数成功时返回一个与key相关的共享内存标识符（非负整数），用于后续的共享内存函数。调用失败返回-1。

  不相关的进程可以通过该函数的返回值访问同一共享内存，它代表程序可能要使用的某个资源，程序对所有共享内存的访问都是间接的，程序先通过调用shmget()函数并提供一个键，再由系统生成一个相应的共享内存标识符（shmget()函数的返回值），只有shmget()函数才直接使用信号量键，所有其他的信号量函数使用由semget函数返回的信号量标识符。



例子

```c
#include <iostream>
#include <sys/shm.h>
#include <sys/stat.h>
#include <unistd.h>
#include <cstring>

struct SharedMemory {
    int data;
    bool dataAvailable;
};

int main() {
    // 创建共享内存
    int shmId = shmget(IPC_PRIVATE, sizeof(SharedMemory), IPC_CREAT | 0660);
    if (shmId == -1) {
        std::cerr << "Failed to create shared memory" << std::endl;
        return 1;
    }

    // 将共享内存映射到进程地址空间
    SharedMemory* sharedMemory = static_cast<SharedMemory*>(shmat(shmId, nullptr, 0));
    if (sharedMemory == reinterpret_cast<SharedMemory*>(-1)) {
        std::cerr << "Failed to attach shared memory" << std::endl;
        shmctl(shmId, IPC_RMID, nullptr);
        return 1;
    }

    // 生产者往共享内存写入数据
    sharedMemory->data = 42;
    sharedMemory->dataAvailable = true;
    std::cout << "Producer wrote data: " << sharedMemory->data << std::endl;

    // 等待消费者读取数据
    while (sharedMemory->dataAvailable) {
        sleep(1);
    }

    // 分离共享内存
    shmdt(sharedMemory);

    // 删除共享内存
    shmctl(shmId, IPC_RMID, nullptr);

    return 0;
}
```

## 对于大量的访问还使用互斥锁和信号量还可以吗

对于大量访问共享内存的情况,使用互斥锁和信号量可能会成为性能瓶颈。在这种情况下,可以考虑使用更高级的同步机制,如无锁编程技术。

一种常见的无锁编程技术是使用原子操作。在 C++ 中,可以使用 `std::atomic` 类型来实现无锁访问共享内存。`std::atomic` 提供了一系列原子操作,如 `load()`, `store()`, `exchange()` 等,可以确保多个线程/进程之间的安全访问。

下面是一个使用 `std::atomic` 的例子:

```c++
#include <iostream>
#include <atomic>
#include <thread>

struct SharedData {
    std::atomic<int> data;
    std::atomic<bool> dataAvailable;
};

void producer(SharedData* sharedData) {
    sharedData->data.store(42, std::memory_order_release);
    sharedData->dataAvailable.store(true, std::memory_order_release);
    std::cout << "Producer wrote data: " << sharedData->data.load(std::memory_order_acquire) << std::endl;
}

void consumer(SharedData* sharedData) {
    while (!sharedData->dataAvailable.load(std::memory_order_acquire)) {
        // 等待数据可用
    }
    int data = sharedData->data.load(std::memory_order_acquire);
    std::cout << "Consumer read data: " << data << std::endl;
    sharedData->dataAvailable.store(false, std::memory_order_release);
}

int main() {
    SharedData sharedData;

    std::thread producerThread(producer, &sharedData);
    std::thread consumerThread(consumer, &sharedData);

    producerThread.join();
    consumerThread.join();

    return 0;
}
```

在这个例子中,我们使用 `std::atomic` 类型来代替之前的手动锁定机制。`store()` 和 `load()` 函数分别用于原子写入和读取共享内存中的数据。通过指定 `memory_order` 参数,可以控制 CPU 对内存访问的重排序,从而确保进程间的同步。

## socket