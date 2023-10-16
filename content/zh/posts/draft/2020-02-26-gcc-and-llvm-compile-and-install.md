---
title: 编译安装gcc和llvm
author: starifly
date: 2020-02-26T10:25:37+08:00
lastmod: 2020-02-26T10:25:37+08:00
categories: [linux]
tags: [linux,gcc,llvm,clang,cmake]
draft: true
slug: gcc-and-llvm-compile-and-install
---

## 升级gcc

```shell
wget -c ftp://ftp.lip6.fr/pub/gcc/releases/gcc-9.2.0/gcc-9.2.0.tar.xz
tar xvf gcc-9.0.2.tar.xz
cd gcc-9.0.2
./contrib/download_prerequisites
mkdir build
cd build
../configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
make -j 4
sudo make install
```

安装完gcc后为防止找不到GLIBC，替换原有的库

```shell
sudo mv /usr/lib/libstdc++.so.6 /usr/lib/libstdc++.so.6.bk
sudo cp /usr/local/lib64/libstdc++.so.6 /usr/lib64/libstdc++.so.6
# 查看
strings /usr/lib64/libstdc++.so.6 | grep GLIBC
```

## 安装z3

```shell
wget https://codeload.github.com/Z3Prover/z3/tar.gz/z3-4.8.7
tar -zxvf z3-4.8.7.tar.gz
cd z3-4.8.7
python scripts/mk_make.py
cd build
make
sudo make install
```

## 安装cmake

```shell
wget https://cmake.org/files/v3.13/cmake-3.13.3.tar.gz
tar -zxvf cmake-3.13.3.tar.gz
cd cmake-3.13.3/
./bootstrap
gmake -j4
sudo gmake install
```

## 安装llvm

```shell
wget http://releases.llvm.org/7.1.0/llvm-7.1.0.src.tar.xz
wget http://releases.llvm.org/7.1.0/cfe-7.1.0.src.tar.xz
wget http://releases.llvm.org/7.1.0/compiler-rt-7.1.0.src.tar.xz
wget http://releases.llvm.org/7.1.0/clang-tools-extra-7.1.0.src.tar.xz
tar xvf llvm-7.1.0.src.tar.xz
tar xvf cfe-7.1.0.src.tar.xz
tar xvf compiler-rt-7.1.0.src.tar.xz
tar xvf clang-tools-extra-7.1.0.src.tar.xz

mv cfe-7.1.0.src clang
mv clang/ llvm-7.1.0.src/tools/

mv clang-tools-extra-7.1.0.src extra
mv extra/ llvm-7.1.0.src/tools/clang/

mv compiler-rt-7.1.0.src compiler-rt
mv compiler-rt llvm-7.1.0.src/projects/

# 指定编译器
vim llvm-7.1.0.src/CMakeLists.txt
# 添加如下：
SET(CMAKE_C_COMPILER “/usr/local/bin/gcc”)
SET(CMAKE_CXX_COMPILER “/usr/local/bin/g++”)

cd llvm-7.1.0.src
mkdir build
cd build
# 如下设置会大大减少编译空间和时间，如果在编译的时候出现链接错误，可以用gold这个linker代替ld：DLLVM_USER_LINKER=gold
cmake -G "Unix Makefiles" -DLLVM_TARGETS_TO_BUILD="host;AMDGPU" -DCMAKE_INSTALL_PREFIX=/usr/local/clang -DCMAKE_BUILD_TYPE=Release -DLLVM_ENABLE_ASSERTIONS=On DLLVM_EXPERIMENTAL_TARGETS_TO_BUILD=WebAssembly -DLLVM_INCLUDE_EXAMPLES=OFF -DLLVM_INCLUDE_TESTS=OFF -DCLANG_INCLUDE_TESTS=OFF ..
make -j 4
sudo make install
```

因为是指定的安装位置，所以要设置环境变量，最后重启机器。

> 注意的点：  
>
> - gcc 还可以通过 scl 源升级  
> - 查看哪个安装包包含该库：`yum provides libstdc++.so.6`  
> - 存在旧版本就升级下：`yum  update  libstdc++-4.4.7-3.el6.x86_64`  
> - 为了提高系统安全性，强烈建议不要直接安装在/usr目录中覆盖原版本。可以安装在/usr/local或/opt中，这样即使出错了，系统也会找到原版本不至于崩溃。另外建议最好提前用tar备份一下系统。  
> - 值得一说的就是 `make -j4` ，后面的参数如果不着急时间就不要加，有时候用多线程编译会出现找不到文件或正在使用中的意料之外的错误。make 的 -j 参数是个坑，虽然可以并行编译，但如果 Makefile 或依赖关系有问题，编译会出错。  
> - 如果命令执行时间过长，可以使用如下方式执行完命令后关机：`{ make && make install ;  } >/tmp/result_stdout.log 2>result_stderr.log ; shutdown -h now`  
> - 如果在编译的时候出现链接错误，可以用 gold 这个 linker 代替 ld：`DLLVM_USER_LINKER=gold`

