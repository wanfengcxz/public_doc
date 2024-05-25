## 在windows上使用cuda开发程序

cuda安装成功：

```bash
C:\Users\wanfengcxz>nvcc -V
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2022 NVIDIA Corporation
Built on Tue_May__3_19:00:59_Pacific_Daylight_Time_2022
Cuda compilation tools, release 11.7, V11.7.64
Build cuda_11.7.r11.7/compiler.31294372_0
```

然后CMake项目加载失败，提示找不到cl.exe和rc.exe

```bash
"D:\CLion 2022.3.3\bin\cmake\win\x64\bin\cmake.exe" -DCMAKE_BUILD_TYPE=Debug "-DCMAKE_MAKE_PROGRAM=D:/CLion 2022.3.3/bin/ninja/win/x64/ninja.exe" -G Ninja -S F:\project\cuda\cuda_learn -B F:\project\cuda\cuda_learn\cmake-build-debug
-- The CUDA compiler identification is NVIDIA 11.7.64
-- The CXX compiler identification is GNU 11.2.0
-- Detecting CUDA compiler ABI info
-- Detecting CUDA compiler ABI info - failed
-- Check for working CUDA compiler: G:/cuda_dev/bin/nvcc.exe
-- Check for working CUDA compiler: G:/cuda_dev/bin/nvcc.exe - broken
CMake Error at D:/CLion 2022.3.3/bin/cmake/win/x64/share/cmake-3.24/Modules/CMakeTestCUDACompiler.cmake:102 (message):
  The CUDA compiler

    "G:/cuda_dev/bin/nvcc.exe"

  is not able to compile a simple test program.

  It fails with the following output:

    Change Dir: F:/project/cuda/cuda_learn/cmake-build-debug/CMakeFiles/CMakeTmp
    
    Run Build Command(s):D:/CLion 2022.3.3/bin/ninja/win/x64/ninja.exe cmTC_37ffa && [1/2] Building CUDA object CMakeFiles\cmTC_37ffa.dir\main.cu.obj
    main.cu
    [2/2] Linking CUDA executable cmTC_37ffa.exe
    FAILED: cmTC_37ffa.exe 
    cmd.exe /C "cd . && "D:\CLion 2022.3.3\bin\cmake\win\x64\bin\cmake.exe" -E vs_link_exe --intdir=CMakeFiles\cmTC_37ffa.dir --rc=rc --mt=CMAKE_MT-NOTFOUND --manifests  -- C:\PROGRA~1\MICROS~3\2022\COMMUN~1\VC\Tools\MSVC\1438~1.331\bin\Hostx86\x64\link.exe /nologo CMakeFiles\cmTC_37ffa.dir\main.cu.obj  /out:cmTC_37ffa.exe /implib:cmTC_37ffa.lib /pdb:cmTC_37ffa.pdb /version:0.0 /debug /INCREMENTAL  cudadevrt.lib  cudart_static.lib  kernel32.lib user32.lib gdi32.lib winspool.lib shell32.lib ole32.lib oleaut32.lib uuid.lib comdlg32.lib advapi32.lib -LIBPATH:"G:/cuda_dev/lib/x64"  && cd ."
    RC Pass 1: command "rc /fo CMakeFiles\cmTC_37ffa.dir/manifest.res CMakeFiles\cmTC_37ffa.dir/manifest.rc" failed (exit code 0) with the following output:
    系统找不到指定的文件。
    ninja: build stopped: subcommand failed.
    
    

  

  CMake will not be able to correctly generate this project.
Call Stack (most recent call first):
  CMakeLists.txt:4 (project)


-- Configuring incomplete, errors occurred!
See also "F:/project/cuda/cuda_learn/cmake-build-debug/CMakeFiles/CMakeOutput.log".
See also "F:/project/cuda/cuda_learn/cmake-build-debug/CMakeFiles/CMakeError.log".

[Previous CMake output restored: 2024/4/23 11:36]

```

于是将下面两个路径（即两个exe的路径）加入环境变量

```bash
C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.38.33130\bin\Hostx64\x64
C:\Program Files (x86)\Windows Kits\10\bin\10.0.22621.0\x64
```

此时编译器使用的是migw，会报错

```bash
"D:\CLion 2022.3.3\bin\cmake\win\x64\bin\cmake.exe" -DCMAKE_BUILD_TYPE=Debug "-DCMAKE_MAKE_PROGRAM=D:/CLion 2022.3.3/bin/ninja/win/x64/ninja.exe" -G Ninja -S F:\project\cuda\cuda_learn -B F:\project\cuda\cuda_learn\cmake-build-debug
-- The CUDA compiler identification is NVIDIA 11.7.64
-- The CXX compiler identification is GNU 11.2.0
-- Detecting CUDA compiler ABI info
-- Detecting CUDA compiler ABI info - failed
-- Check for working CUDA compiler: G:/cuda_dev/bin/nvcc.exe
-- Check for working CUDA compiler: G:/cuda_dev/bin/nvcc.exe - broken
CMake Error at D:/CLion 2022.3.3/bin/cmake/win/x64/share/cmake-3.24/Modules/CMakeTestCUDACompiler.cmake:102 (message):
  The CUDA compiler

    "G:/cuda_dev/bin/nvcc.exe"

  is not able to compile a simple test program.

  It fails with the following output:

    Change Dir: F:/project/cuda/cuda_learn/cmake-build-debug/CMakeFiles/CMakeTmp
    
    Run Build Command(s):D:/CLion 2022.3.3/bin/ninja/win/x64/ninja.exe cmTC_4678a && [1/2] Building CUDA object CMakeFiles\cmTC_4678a.dir\main.cu.obj
    main.cu
    [2/2] Linking CUDA executable cmTC_4678a.exe
    FAILED: cmTC_4678a.exe 
    cmd.exe /C "cd . && "D:\CLion 2022.3.3\bin\cmake\win\x64\bin\cmake.exe" -E vs_link_exe --intdir=CMakeFiles\cmTC_4678a.dir --rc=C:\PROGRA~2\WI3CF2~1\10\bin\100226~1.0\x64\rc.exe --mt=C:\PROGRA~2\WI3CF2~1\10\bin\100226~1.0\x64\mt.exe --manifests  -- C:\PROGRA~1\MICROS~3\2022\COMMUN~1\VC\Tools\MSVC\1438~1.331\bin\Hostx64\x64\link.exe /nologo CMakeFiles\cmTC_4678a.dir\main.cu.obj  /out:cmTC_4678a.exe /implib:cmTC_4678a.lib /pdb:cmTC_4678a.pdb /version:0.0 /debug /INCREMENTAL  cudadevrt.lib  cudart_static.lib  kernel32.lib user32.lib gdi32.lib winspool.lib shell32.lib ole32.lib oleaut32.lib uuid.lib comdlg32.lib advapi32.lib -LIBPATH:"G:/cuda_dev/lib/x64"  && cd ."
    LINK Pass 1: command "C:\PROGRA~1\MICROS~3\2022\COMMUN~1\VC\Tools\MSVC\1438~1.331\bin\Hostx64\x64\link.exe /nologo CMakeFiles\cmTC_4678a.dir\main.cu.obj /out:cmTC_4678a.exe /implib:cmTC_4678a.lib /pdb:cmTC_4678a.pdb /version:0.0 /debug /INCREMENTAL cudadevrt.lib cudart_static.lib kernel32.lib user32.lib gdi32.lib winspool.lib shell32.lib ole32.lib oleaut32.lib uuid.lib comdlg32.lib advapi32.lib -LIBPATH:G:/cuda_dev/lib/x64 /MANIFEST /MANIFESTFILE:CMakeFiles\cmTC_4678a.dir/intermediate.manifest CMakeFiles\cmTC_4678a.dir/manifest.res" failed (exit code 1104) with the following output:
    LINK : fatal error LNK1104: 无法打开文件“kernel32.lib”
    ninja: build stopped: subcommand failed.
```

选择了mingw64也还是报错，猜测是mingw缺少某些库，于是选择msvc，并且架构选择x86_amd64，随即加载项目成功。

```bash
"D:\CLion 2022.3.3\bin\cmake\win\x64\bin\cmake.exe" -DCMAKE_BUILD_TYPE=Debug "-DCMAKE_MAKE_PROGRAM=D:/CLion 2022.3.3/bin/ninja/win/x64/ninja.exe" -G Ninja -S F:\project\cuda\cuda_learn -B F:\project\cuda\cuda_learn\cmake-build-debug
-- The CUDA compiler identification is NVIDIA 11.7.64
-- The CXX compiler identification is MSVC 19.38.33133.0
-- Detecting CUDA compiler ABI info
-- Detecting CUDA compiler ABI info - done
-- Check for working CUDA compiler: G:/cuda_dev/bin/nvcc.exe - skipped
-- Detecting CUDA compile features
-- Detecting CUDA compile features - done
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Check for working CXX compiler: C:/Program Files/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.38.33130/bin/Hostx86/x64/cl.exe - skipped
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: F:/project/cuda/cuda_learn/cmake-build-debug

[Finished]
```

