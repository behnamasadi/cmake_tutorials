cmake_minimum_required(VERSION 3.10)

project(Tools)


add_executable(Tools1     src/main.cpp)
target_link_libraries(Tools1     MyLibrary)
