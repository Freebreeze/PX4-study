# CMake 最低版本号要求
cmake_minimum_required (VERSION 2.8)
# 项目信息
project (Demo1)
# 指定生成目标
add_executable(Demo main.cc)

set(PX4_BOARD_DIR ${CMAKE_CURRENT_LIST_DIR} CACHE STRING "PX4 board directory" FORCE)

