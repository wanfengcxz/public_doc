# Tmux

```bash
# 新建会话
tmux new -s my_session

# 划分左右两个窗格
tmux split-window -h

# ctrl+b o 切换窗口
```

在 Tmux 中开启和管理多个窗口非常简单，以下是详细操作指南：

### 1. 启动 Tmux

```
tmux
```

或指定会话名：

```
tmux new -s mysession
```

### 2. 窗口管理快捷键

所有快捷键都需要先按 `Ctrl+b`（前缀键），然后按后续键：

- **新建窗口**：`c`
  （会在底部状态栏看到新增的窗口编号）
- **切换窗口**：
  - `0-9`：切换到指定编号窗口
  - `n`：下一个窗口（next）
  - `p`：上一个窗口（previous）
  - `w`：窗口列表（可视化选择）
- **重命名当前窗口**：`,`
  （方便识别窗口用途）
- **关闭当前窗口**：`&`

### 4. 高级技巧

- **分屏**：`Ctrl+b "`（水平分屏），`Ctrl+b %`（垂直分屏）
- **会话分离**：`Ctrl+b d`（后台运行所有窗口）
- **重新连接**：`tmux attach -t mysession`

# rsync同步文件

```bash
rsync -n -a -v -c -e "ssh -p 20002" /var/data/* root@101.230.144.220:/root/workspace/data/
```

# 检测C++程序内存泄漏

使用valgrind来检测内存泄漏

```bash
apt-get install valgrind

valgrind --leak-check=yes ./your_program
valgrind --tool=memcheck your_program

root@ba0a3e4c230d:/home/me/memory_leak/demo1# valgrind --leak-check=yes ./a.out 
==27275== Memcheck, a memory error detector
==27275== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==27275== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==27275== Command: ./a.out
==27275== 
==27275== 
==27275== HEAP SUMMARY:
==27275==     in use at exit: 8 bytes in 2 blocks
==27275==   total heap usage: 3 allocs, 1 frees, 72,712 bytes allocated
==27275== 
==27275== 4 bytes in 1 blocks are definitely lost in loss record 1 of 2
==27275==    at 0x483BE63: operator new(unsigned long) (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
==27275==    by 0x1091BB: fun2() (main.cpp:9)
==27275==    by 0x1091D3: main (main.cpp:13)
==27275== 
==27275== 4 bytes in 1 blocks are definitely lost in loss record 2 of 2
==27275==    at 0x483BE63: operator new(unsigned long) (in /usr/lib/x86_64-linux-gnu/valgrind/vgpreload_memcheck-amd64-linux.so)
==27275==    by 0x1091DD: main (main.cpp:14)
==27275== 
==27275== LEAK SUMMARY:
==27275==    definitely lost: 8 bytes in 2 blocks
==27275==    indirectly lost: 0 bytes in 0 blocks
==27275==      possibly lost: 0 bytes in 0 blocks
==27275==    still reachable: 0 bytes in 0 blocks
==27275==         suppressed: 0 bytes in 0 blocks
==27275== 
==27275== For lists of detected and suppressed errors, rerun with: -s
==27275== ERROR SUMMARY: 2 errors from 2 contexts (suppressed: 0 from 0)
```

