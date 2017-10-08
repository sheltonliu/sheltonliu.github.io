---
title: cmake 简介
date: 2017-10-08 12:46:13
categories: "cmake简介"
tags:
	- "cmake"
---
# cmake简介
<font size=4>

以下是CMakeLists.txt的内容

	cmake_minimum_required(VERSION 2.6)

	#myapp:main.o 定义工程名称为myapp 
	PROJECT(myapp)

	#导入头文件
	INCLUDE_DIRECTORIES(
	include
	)

	#指定源文件的目录
	AUX_SOURCE_DIRECTORY(src DIR_SRCS)

	#把可执行的文件放大哪个目录下  SET  方法设置变量
	SET(TEST_MATH ${DIR_SRCS})
	
	#定义了这个工程会生成一个文件名为 myapp 的可执行文件,相关的源文件是 TEST_MATH 中定义的源文件列表
	ADD_EXECUTABLE(${PROJECT_NAME} ${TEST_MATH})

测试的目录解构：

build文件夹(自己新建的)

CMakeLists.txt

include文件夹----->add.h

src文件夹--------->add.c  main.c

1.然后进入build目录(cd build) 然后执行cmake .. 其实就是执行上层目录下CMakeLists.txt中的命令

2 在build目录中会看到 Makefile文件， 然后执行make命令，会生成myapp的可执行文件

3 执行./myapp展现出程序的结果



最后：相关目录解构可以参考：
[目录结构参考](https://github.com/sheltonliu/Unix_Code_Project/tree/master/cmake/cmake)

关于cmake更详细的内容可以参见[cmake实战](https://github.com/sheltonliu/Unix_Code_Project/tree/master/cmake)



