1.cmake_minimum_required (<obj>):指明CMake最低版本
	cmake_minimum_required (VERSION 2.8)
	CMake最低版本是2.8
2.project (<obj>):项目名称
	project (Demo1)
	项目名称是Demo1，**注意这里只是项目名称，并不是最后可执行文件的名字**
3.add_executable(<name> <source>):编译可执行文件，<name>是最后可执行文件的名字，<source>是编译的目标文件
	add_executable(Demo main.cc)
	将名为 main.cc 的源文件编译成一个名称为 Demo 的可执行文件
4.aux_source_directory(<dir> <variable>)：此命令会查找指定目录下的所有源文件，然后将结果存进指定变量名。<dir>是指定目录，<variable>是变量名
	aux_source_directory(. DIR_SRCS)
	将当前目录所有源文件的文件名赋值给变量 DIR_SRCS
5.add_subdirectory(<name>):指明本项目包含一个子目录，<name>是子目录名字
	add_subdirectory(math)
	本项目包含一个名为math的子目录
6.target_link_libraries(<name> <obj>):指明可执行文件需要的静态链接库。<name>是可执行文件名，<obj>是静态链接库名字
	target_link_libraries(Demo MathFunctions)
	指明可执行文件 main 需要连接一个名为 MathFunctions 的链接库 。
7.add_library (<obj> <source>):将目录中的源文件编译为静态链接库,<obj>是静态链接库名字,<source>是编译的目标文件
	add_library (MathFunctions ${DIR_LIB_SRCS})
	该文件中使用命令 add_library 将 src 目录中的源文件编译为静态链接库。



