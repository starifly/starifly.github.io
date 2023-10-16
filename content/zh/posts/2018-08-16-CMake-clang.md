---
title: 使用 CMake 不用路径地调用 libclang
author: ice1000
date: 2018-08-16T10:36:02+08:00
categories: [cmake]
tags: [cmake, clang]
slug: CMake-clang
---

作为一个有高尚的情操的程序员，应该学会在写非练手项目的时候构建流程不使用与本机路径相关的依赖，这样把代码往 CI 上部署、在公司与个人电脑间传输、或者给别人用的时候都方便的多。

如果有 gradle, pip, gem, maven, sbt 这类基于包的构建工具的话，这根本就不是问题。别人写的依赖可以在远端调用，自己写的依赖也可以本地调用。  
如果有 npm, go get 这类基于版本控制的构建工具的话，这当作也只有一个小问题，就是项目维护者脑抽了一下，可能就会误伤在此期间构建程序的程序员。

不过至少他们都能自主寻找包并下载安装，只要包是正确的就能成功安装，安装的配置是由包的开发者提供的。

CMake 作为全宇宙 `用户量 / 构建工具本身的质量` 最高的构建工具，要做到这一点是很困难的 —— CMake 只有在『没有需要导入的依赖』或者『所有依赖都是系统自带的（比如在 Windows 上调用 DirectX）』情况下能做到『无痛使用』。
一旦需要导入一些外部依赖（比如 `glfw3`，比如 `jni`），就需要用到 `find_package` 这个函数，which 只有满足这两个条件的时候才能成功执行：

+ CMake 能找到 `[package名]Config.cmake` 或者 `[package名]-config.cmake` 中的一个文件
+ CMake 能找到 `Find[package名].cmake` 这个文件，并且这个文件里的代码能 find 到这个 package

之前在写 JNI 代码的时候需要使用 CMake 寻找 `jni.h` 等一系列文件，于是只需要这么一小段代码就可以把他们加进 CMake 编译时的 classpath:

```cmake
find_package(Java REQUIRED)
find_package(JNI REQUIRED)
```

然后我们可以在找到之后输出一下找到的 JDK 的路径：

```kotlin
if (JNI_FOUND)
	message(STATUS "JNI_INCLUDE_DIRS=${JNI_INCLUDE_DIRS}")
	message(STATUS "JNI_LIBRARIES=${JNI_LIBRARIES}")
endif ()
```

但如果我们要调用 `libclang` 的时候:

```cmake
find_package(LibClang REQUIRED)
```

它就会说找不到 `LibClangConfig.cmake`, `LibClang-config.cmake`, `FindLibClang.cmake` 中的一个。

首先确保已经安装 clang:

```shell
$ sudo apt install clang-dev
```

在这个时候我已经上 Google/StackOverflow 找了一遍了，由于 `clang` 这个名词的特殊性（不仅是个包，还是个 C++ 编译器），搜索 `clang + cmake` 都是教你怎么配置 CMake 使 CMake 调用 clang 而不是 gcc 的，搜索 `libclang + cmake` 得到的很多结果需要配置环境变量，把 clang/llvm 的安装路径指定好。
而我的 clang 明明就在 `PATH` 里，凭什么需要另外配置呢。

那么我就来自己想办法。

前两者应该是安装包的时候就自带了的，既然 clang 没有，那么只能去找个 `FindLibClang.cmake` 啦。

搜索 `FindLibClang.cmake` ，找到了一个 Emacs 插件 `rtags` 的一个 [CMake 模块](https://github.com/Andersbakken/rtags/blob/master/cmake/FindLibClang.cmake)（注意这个代码是 GPLv3 的哦）：

```shell
if (NOT LIBCLANG_ROOT_DIR)
    set(LIBCLANG_ROOT_DIR $ENV{LIBCLANG_ROOT_DIR})
endif ()

if (NOT LIBCLANG_LLVM_CONFIG_EXECUTABLE)
    set(LIBCLANG_LLVM_CONFIG_EXECUTABLE $ENV{LIBCLANG_LLVM_CONFIG_EXECUTABLE})
    if (NOT LIBCLANG_LLVM_CONFIG_EXECUTABLE)
        if (APPLE)
            foreach(major RANGE 9 3)
                foreach(minor RANGE 9 0)
                    foreach(patch RANGE 9 0)
                        message(STATUS "trying llvm-config llvm-config${major}${minor} in /usr/local/Cellar/llvm/${major}.${minor}.${patch}/bin")
                        find_program(LIBCLANG_LLVM_CONFIG_EXECUTABLE NAMES llvm-config llvm-config${major}${minor} llvm-config-${major}${minor} llvm-config-${major} llvm-config${major} PATHS /usr/local/Cellar/llvm/${major}.${minor}.${patch}/bin)
                        if (LIBCLANG_LLVM_CONFIG_EXECUTABLE)
                            break()
                        endif ()
                    endforeach ()
                    if (LIBCLANG_LLVM_CONFIG_EXECUTABLE)
                        break()
                    endif ()
                endforeach ()
                if (LIBCLANG_LLVM_CONFIG_EXECUTABLE)
                    break()
                endif ()
            endforeach ()
        else ()
            set(llvm_config_names llvm-config)
            foreach(major RANGE 9 3)
                list(APPEND llvm_config_names "llvm-config${major}" "llvm-config-${major}")
                foreach(minor RANGE 9 0)
                    list(APPEND llvm_config_names "llvm-config${major}${minor}" "llvm-config-${major}.${minor}" "llvm-config-mp-${major}.${minor}")
                endforeach ()
            endforeach ()
            find_program(LIBCLANG_LLVM_CONFIG_EXECUTABLE NAMES ${llvm_config_names})
        endif ()
    endif ()
    if (LIBCLANG_LLVM_CONFIG_EXECUTABLE)
        message(STATUS "llvm-config executable found: ${LIBCLANG_LLVM_CONFIG_EXECUTABLE}")
    endif ()
endif ()

if (NOT LIBCLANG_CXXFLAGS)
    if (NOT LIBCLANG_LLVM_CONFIG_EXECUTABLE)
        message(FATAL_ERROR "Could NOT find llvm-config executable and LIBCLANG_CXXFLAGS is not set ")
    endif ()
    execute_process(COMMAND ${LIBCLANG_LLVM_CONFIG_EXECUTABLE} --cxxflags OUTPUT_VARIABLE LIBCLANG_CXXFLAGS OUTPUT_STRIP_TRAILING_WHITESPACE)
    if (NOT LIBCLANG_CXXFLAGS)
        find_path(LIBCLANG_CXXFLAGS_HACK_CMAKECACHE_DOT_TEXT_BULLSHIT clang-c/Index.h HINTS ${LIBCLANG_ROOT_DIR}/include NO_DEFAULT_PATH)
        if (NOT EXISTS ${LIBCLANG_CXXFLAGS_HACK_CMAKECACHE_DOT_TEXT_BULLSHIT})
            find_path(LIBCLANG_CXXFLAGS clang-c/Index.h)
            if (NOT EXISTS ${LIBCLANG_CXXFLAGS})
                message(FATAL_ERROR "Could NOT find clang include path. You can fix this by setting LIBCLANG_CXXFLAGS in your shell or as a cmake variable.")
            endif ()
        else ()
            set(LIBCLANG_CXXFLAGS ${LIBCLANG_CXXFLAGS_HACK_CMAKECACHE_DOT_TEXT_BULLSHIT})
        endif ()
        set(LIBCLANG_CXXFLAGS "-I${LIBCLANG_CXXFLAGS}")
    endif ()
    string(REGEX MATCHALL "-(D__?[a-zA-Z_]*|I([^\" ]+|\"[^\"]+\"))" LIBCLANG_CXXFLAGS "${LIBCLANG_CXXFLAGS}")
    string(REGEX REPLACE ";" " " LIBCLANG_CXXFLAGS "${LIBCLANG_CXXFLAGS}")
    set(LIBCLANG_CXXFLAGS ${LIBCLANG_CXXFLAGS} CACHE STRING "The LLVM C++ compiler flags needed to compile LLVM based applications.")
    unset(LIBCLANG_CXXFLAGS_HACK_CMAKECACHE_DOT_TEXT_BULLSHIT CACHE)
endif ()

if (NOT EXISTS ${LIBCLANG_LIBDIR})
    if (NOT LIBCLANG_LLVM_CONFIG_EXECUTABLE)
        message(FATAL_ERROR "Could NOT find llvm-config executable and LIBCLANG_LIBDIR is not set ")
    endif ()
    execute_process(COMMAND ${LIBCLANG_LLVM_CONFIG_EXECUTABLE} --libdir OUTPUT_VARIABLE LIBCLANG_LIBDIR OUTPUT_STRIP_TRAILING_WHITESPACE)
    if (NOT EXISTS ${LIBCLANG_LIBDIR})
        message(FATAL_ERROR "Could NOT find clang libdir. You can fix this by setting LIBCLANG_LIBDIR in your shell or as a cmake variable.")
    endif ()
    set(LIBCLANG_LIBDIR ${LIBCLANG_LIBDIR} CACHE STRING "Path to the clang library.")
endif ()

if (NOT LIBCLANG_LIBRARIES)
    find_library(LIBCLANG_LIB_HACK_CMAKECACHE_DOT_TEXT_BULLSHIT NAMES clang libclang HINTS ${LIBCLANG_LIBDIR} ${LIBCLANG_ROOT_DIR}/lib NO_DEFAULT_PATH)
    if (LIBCLANG_LIB_HACK_CMAKECACHE_DOT_TEXT_BULLSHIT)
        set(LIBCLANG_LIBRARIES "${LIBCLANG_LIB_HACK_CMAKECACHE_DOT_TEXT_BULLSHIT}")
    else ()
        find_library(LIBCLANG_LIBRARIES NAMES clang libclang)
        if (NOT EXISTS ${LIBCLANG_LIBRARIES})
            set (LIBCLANG_LIBRARIES "-L${LIBCLANG_LIBDIR}" "-lclang" "-Wl,-rpath,${LIBCLANG_LIBDIR}")
        endif ()
    endif ()
    unset(LIBCLANG_LIB_HACK_CMAKECACHE_DOT_TEXT_BULLSHIT CACHE)
endif ()

set(LIBCLANG_LIBRARY ${LIBCLANG_LIBRARIES} CACHE FILEPATH "Path to the libclang library")

if (LIBCLANG_LLVM_CONFIG_EXECUTABLE)
    execute_process(COMMAND ${LIBCLANG_LLVM_CONFIG_EXECUTABLE} --version OUTPUT_VARIABLE LIBCLANG_VERSION_STRING OUTPUT_STRIP_TRAILING_WHITESPACE)
else ()
    set(LIBCLANG_VERSION_STRING "Unknown")
endif ()
message("-- Using Clang version ${LIBCLANG_VERSION_STRING} from ${LIBCLANG_LIBDIR} with CXXFLAGS ${LIBCLANG_CXXFLAGS}")

# Handly the QUIETLY and REQUIRED arguments and set LIBCLANG_FOUND to TRUE if all listed variables are TRUE
include(FindPackageHandleStandardArgs)
find_package_handle_standard_args(LibClang DEFAULT_MSG LIBCLANG_LIBRARY LIBCLANG_CXXFLAGS LIBCLANG_LIBDIR)
mark_as_advanced(LIBCLANG_CXXFLAGS LIBCLANG_LIBRARY LIBCLANG_LLVM_CONFIG_EXECUTABLE LIBCLANG_LIBDIR)
```

其实我中间找到过两三个类似的，但那几个 `FindLibClang` 都写的不稳，在我的电脑上都不 work ，需要环境变量。
而看这个实现，它是借助 `llvm-config` 这个程序（就是配置 llvm 环境的时候的标配啦，一般是 `c++ $(llvm-config --cxxflags) main.cpp` 这样用的）查找的 clang 配置，稳如老狗。

根据它在注释里写的：

```ruby
# FindLibClang
#
# This module searches libclang and llvm-config, the llvm-config tool is used to
# get information about the installed llvm/clang package to compile LLVM based
# programs.
#
# It defines the following variables
#
# ``LIBCLANG_LLVM_CONFIG_EXECUTABLE``
#   the llvm-config tool to get various information.
# ``LIBCLANG_LIBRARIES``
#   the clang libraries to link against to use Clang/LLVM.
# ``LIBCLANG_LIBDIR``
#   the directory where the clang libraries are located.
# ``LIBCLANG_FOUND``
#   true if libclang was found
# ``LIBCLANG_VERSION_STRING``
#   version number as a string
# ``LIBCLANG_CXXFLAGS``
#   the compiler flags for files that include LLVM headers
```

生成的变量都写清楚了，所以我们就可以很容易地导入它了。
在我们自己项目里，先把上面的 `FindLibClang.cmake` 放进 `项目根目录/cmake-modules/` 目录里（其实就是随便一个目录），然后在 `CMakeLists.txt` 里写：

```cmake
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake-modules/")
find_package(LibClang REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${LIBCLANG_CXXFLAGS}")
```

就可以让 CMake 找到 clang 的头文件啦。

然后就可以在 CLion 里面写这样的代码了：

```cpp
#include <clang/AST/ASTConsumer.h>
#include <clang/AST/RecursiveASTVisitor.h>
#include <clang/Frontend/CompilerInstance.h>
#include <clang/Frontend/FrontendAction.h>
#include <clang/Tooling/Tooling.h>
```

可以正确地 navigate 到对应的文件，自动生成 `override` 之类的特性都可以用了。

```cpp
class FindNamedClassAction : public clang::ASTFrontendAction {
protected:
		std::unique_ptr<clang::ASTConsumer> CreateASTConsumer(clang::CompilerInstance &CI, StringRef InFile) override {
			return std::unique_ptr<clang::ASTConsumer>(new FindNamedClassASTConsumer(CI.getASTContext()));
		}
};

auto main(int argc, const char **const argv) -> int {
	if (argc > 1) clang::tooling::runToolOnCode(new FindNamedClassAction, argv[1]);
	return 0;
}
```

目前我还没找到如何正确编译和链接程序的方法，这个代码会导致 unresolved reference:

```cmake
target_link_libraries(clang_test ${LIBCLANG_LIBRARIES})
```

网上也找不到解决方案，只能自己脑补程序运行的样子。
