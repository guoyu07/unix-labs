cmake_minimum_required(VERSION 3.5)
project(lab1)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(SOURCE_FILES
    main.c timer.h timer.c my_fread.c my_fread.h)

add_executable(lab1 ${SOURCE_FILES})