# FAQ-cmake

众多周知 cmake 的功能是很强大的，而且逐渐成为了 C/C++ 语言系的标准构建工具，整个生态相对比较成熟。然而，cmake 也存在语法奇葩、资料贫弱混乱、学习成本高等各种问题。鄙人在学习 cmake 过程中反复学习多次，每隔一段事件就发现前面研究的一些问题都忘记了。痛定思痛，鄙人决定将在工作和学习 cmake 过程中遇到的一些问题以及自己找到的解决办法整理成 FAQ，以省下一些工作和生活的时间。

## FAQ-001 如何从文件中读取版本号？

假设在 CMakeLists.txt 目录下有个文件 VERSION，里面的内容为：

```txt
1.1.2
```

可以，使用 `file` 命令将 VERSION 文件中的版本号信息，读入到变量 `MY_VERSION` 中去：

```
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/VERSION" MY_VERSION)
string(STRIP "${MY_VERSION}" MY_VERSION)
```

**需要注意**：`file(READ xxx)` 命令似乎有bug，它总是会多读一个换行，所以，我们可以再使用 `string(STRIP xxx)` 将 MY_VERSION 变量前后的空行去掉。

## FAQ-002 如何编译成动态库？

可以使用 `add_library` 命令定义一个 library，如果指定 `SHARED` 属性，就会自动生成动态库：

```
add_library(mylib SHARED xxx.cpp yyy.cpp)
```

## FAQ-003 如何指定头文件搜索路径？

`target_include_directories` 专门用来完成这个事情：

```
target_include_directories(mylib PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
```

## FAQ-004 当前这个 CMakeLists.txt 所在的目录是什么变量？

`CMAKE_CURRENT_SOURCE_DIR`

## FAQ-005 如何为动态库指定 x.y.z 形式的版本号？

一种简单的拌饭是先指定 project 的版本号，然后通过 `CMAKE_PROJECT_VERSION` `CMAKE_PROJECT_VERSION_MAJOR` 等变量去灵活引用版本号

下面有个示例：

```

project(libproperties VERSION ${MY_VERSION} LANGUAGES C)

...

set_target_properties(properties PROPERTIES VERSION     ${CMAKE_PROJECT_VERSION})
set_target_properties(properties PROPERTIES SOVERSION   ${CMAKE_PROJECT_VERSION_MAJOR} )
```

编译出来效果如下：

```
$ ll
总用量 24
lrwxrwxrwx. 1 liugang liugang    18 12月 13 20:04 libproperties.so -> libproperties.so.1
lrwxrwxrwx. 1 liugang liugang    22 12月 13 20:04 libproperties.so.1 -> libproperties.so.1.1.2
-rwxrwxr-x. 1 liugang liugang 22576 12月 13 20:04 libproperties.so.1.1.2
```

`CMAKE_PROJECT_VERSION` 还有几个相关的变量可用，可以自行查阅 cmake 手册。


## FAQ-006 如何指定最终文件输出路径？

这个要分情况来处理，不同的类型，是有不同变量控制的：

* 如果是可执行文件： `CMAKE_RUNTIME_OUTPUT_DIRECTORY`
* 如果是静态库： `CMAKE_ARCHIVE_OUTPUT_DIRECTORY`
* 如果是动态库： `CMAKE_LIBRARY_OUTPUT_DIRECTORY`

示例：

```
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lib)
```

## FAQ-007 如何指定使用 C 编译器编译所有源代码？

可以在定义 project 时，指定 project 的 `LANGUAGES` 属性，如下示例：

```
project(libproperties LANGUAGES C)
```


