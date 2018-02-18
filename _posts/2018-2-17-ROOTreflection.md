---
layout: post
title: "ROOT 反射特性的应用"
date: 2018-2-17 19:47
categories: software
tags: ROOT Geant4
excerpt: 利用 ROOT 的反射特性实现自定义类的存储和读取。
author: zhiy
mathjax: true
---

* content
{:toc}

在 ROOT 里， 我们可以定义自己的数据类型，并将该类型的实例保存到硬盘。在 ROOT 脚本中可以很容易实现这一目的：  
```cpp
obj = new MyClass;
tree->Branch("Branch Name", "MyClass", &obj)
```
读取数据的过程与普通数据类型相同：
```cpp
obj = new MyClass;
branch->SetAddress(&obj);
```
这样做的好处有：1、不同类型数据的 I/O 操作简化一个自定义类型（类，结构体等）的 I/O 操作。2、可以给数据类型定义常用的方法，方便数据分析。




## 生成动态链接库
定义名为 &quot;MyClass&quot; 的类，将头文件 &quot;MyClass.hh&quot; 放置在 /include 文件夹内，源文件 &quot;MyClass.cc&quot; 放在 /src 文件夹内。在 /include 文件夹内编辑 LinkDef.hh 文件(也可以将 LinkDef.hh 的内容写在 MyClass.hh 的末尾)：
```java
#ifdef __MAKECINT__
#pragma link C++ class MyClass+;
#endif
``` 
LinkDef.hh 文件用于指定希望保存到硬盘上的数据类型， 如果成员变量存在模板类，需要在 LinkDef.hh 中明确指出，例如：
```java
#pragma link C++ class std::vector<double>+;
#pragma link C++ class std::vector<TString>+;
```
首先利用 LinkDef.hh 和 MyClass.hh 生成字典（dictionary），然后将 MyClass.cc 和生成的字典文件编译生成动态链接库，最后用 rootcint 生成 rootmap 文件。在 /include 目录下运行 ROOT 命令：
```shell
rootcint -f MyClass_dict.cc -c -p MyClass.hh LinkDef.hh
`root-config --cxx --cflags` -fPIC -c ../src/MyClass.cc
`root-config --cxx --cflags` -fPIC -c MyClass_dict.cc
`root-config --cxx --cflags` -fPIC -shared -o libMyClass.so MyClass_dict.o MyClass.o
rootcint -f MyClass_dict.cc -rmf libMyClass.rootmap -rml libMyClass.so -c -p MyClass.hh LinkDef.hh 
``` 
最后将生成的 .so 文件， .pcm 文件以及 rootmap 文件移动到 /lib 文件夹，接下来就可以通过适当的命令加载动态链接库对 MyClass 进行 I/O 操作了。
## 使用动态链接库
如果想在 ROOT 脚本中使用对 MyClass 类进行存储操作，可以用`R__LOAD_LIBRARY($LIBRARY_PATH/libMyClass.so)`宏控制动态链接库的加载（在第一次使用库的定义时加载即可）。如果想在比较大的工程（如 Geant4 应用）中使用 libMyClass.so, 可以用如下 cmake 命令：
```cmake
find_package(ROOT)
include_directories(${ROOT_INCLUDE_DIR})
include_directories("$INCLUDE_PATH")
set(MyClass_LIBRARIES "$LIBRARY_PATH/libMyClass.so")
target_link_libraries(PROJECt_NAME ${ROOT_LIBRARIES})
target_link_libraries(PROJECT_NAME ${MyCLass_LIBRARIES})
```

