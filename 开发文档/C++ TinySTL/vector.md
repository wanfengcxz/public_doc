## 检测vector内存是否有泄漏

```c++
root@ba0a3e4c230d:/home/me/MyCPPSTL/build# valgrind --leak-check=yes ./test_vector 
==4382== Memcheck, a memory error detector
==4382== Copyright (C) 2002-2017, and GNU GPL'd, by Julian Seward et al.
==4382== Using Valgrind-3.15.0 and LibVEX; rerun with -h for copyright info
==4382== Command: ./test_vector
==4382== 
[==========] Running 24 tests from 5 test suites.
[----------] Global test environment set-up.
[----------] 9 tests from StlVectorIntTest
[ RUN      ] StlVectorIntTest.iterator
[       OK ] StlVectorIntTest.iterator (12 ms)
[ RUN      ] StlVectorIntTest.init
[       OK ] StlVectorIntTest.init (19 ms)
[ RUN      ] StlVectorIntTest.assign
[       OK ] StlVectorIntTest.assign (4 ms)
[ RUN      ] StlVectorIntTest.emplace
[       OK ] StlVectorIntTest.emplace (6 ms)
[ RUN      ] StlVectorIntTest.insert
[       OK ] StlVectorIntTest.insert (8 ms)
[ RUN      ] StlVectorIntTest.erase
[       OK ] StlVectorIntTest.erase (3 ms)
[ RUN      ] StlVectorIntTest.reverse
[       OK ] StlVectorIntTest.reverse (2 ms)
[ RUN      ] StlVectorIntTest.swap
[       OK ] StlVectorIntTest.swap (3 ms)
[ RUN      ] StlVectorIntTest.shrink_to_fit
[       OK ] StlVectorIntTest.shrink_to_fit (1 ms)
[----------] 9 tests from StlVectorIntTest (68 ms total)

[----------] 3 tests from StlVectorClassATest
[ RUN      ] StlVectorClassATest.init
[       OK ] StlVectorClassATest.init (19 ms)
[ RUN      ] StlVectorClassATest.assign
[       OK ] StlVectorClassATest.assign (7 ms)
[ RUN      ] StlVectorClassATest.emplace
[       OK ] StlVectorClassATest.emplace (7 ms)
[----------] 3 tests from StlVectorClassATest (34 ms total)

[----------] 8 tests from StlVectorStringTest
[ RUN      ] StlVectorStringTest.init
[       OK ] StlVectorStringTest.init (26 ms)
[ RUN      ] StlVectorStringTest.assign
[       OK ] StlVectorStringTest.assign (6 ms)
[ RUN      ] StlVectorStringTest.emplace
[       OK ] StlVectorStringTest.emplace (8 ms)
[ RUN      ] StlVectorStringTest.insert
[       OK ] StlVectorStringTest.insert (9 ms)
[ RUN      ] StlVectorStringTest.erase
[       OK ] StlVectorStringTest.erase (4 ms)
[ RUN      ] StlVectorStringTest.reverse
[       OK ] StlVectorStringTest.reverse (2 ms)
[ RUN      ] StlVectorStringTest.swap
[       OK ] StlVectorStringTest.swap (4 ms)
[ RUN      ] StlVectorStringTest.shrink_to_fit
[       OK ] StlVectorStringTest.shrink_to_fit (1 ms)
[----------] 8 tests from StlVectorStringTest (65 ms total)

[----------] 1 test from vector
[ RUN      ] vector.const_iterator
[       OK ] vector.const_iterator (4 ms)
[----------] 1 test from vector (4 ms total)

[----------] 3 tests from other
[ RUN      ] other.const_ref
[       OK ] other.const_ref (0 ms)
[ RUN      ] other.const_cast_
[       OK ] other.const_cast_ (0 ms)
[ RUN      ] other.int64_max
[       OK ] other.int64_max (0 ms)
[----------] 3 tests from other (2 ms total)

[----------] Global test environment tear-down
[==========] 24 tests from 5 test suites ran. (206 ms total)
[  PASSED  ] 24 tests.
==4382== 
==4382== HEAP SUMMARY:
==4382==     in use at exit: 0 bytes in 0 blocks
==4382==   total heap usage: 697 allocs, 697 frees, 171,803 bytes allocated
==4382== 
==4382== All heap blocks were freed -- no leaks are possible
==4382== 
==4382== For lists of detected and suppressed errors, rerun with: -s
==4382== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)
```

