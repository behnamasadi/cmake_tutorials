cmake_minimum_required(VERSION 3.25)

project(my_project
    VERSION     1.2.3.4
    DESCRIPTION "An example project with CMake"
    LANGUAGES   CXX
)

set(CMAKE_CXX_STANDARD          17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS        OFF)

if(NOT CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
    # Honor the user's -DCMAKE_INSTALL_PREFIX=...
else()
    set(CMAKE_INSTALL_PREFIX "$ENV{HOME}/usr" CACHE PATH "Install prefix" FORCE)
endif()

# Nicer IDE folder grouping
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

option(BUILD_EXAMPLES "Build the examples"      ON)
option(BUILD_TESTING  "Build and run the tests" ON)
option(BUILD_DOCS     "Build the documentation" ON)

add_subdirectory(src)
add_subdirectory(apps)

if(BUILD_EXAMPLES)
    add_subdirectory(examples)
endif()

if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
    include(CTest)

    include(FetchContent)
    FetchContent_Declare(
        googletest
        GIT_REPOSITORY https://github.com/google/googletest.git
        GIT_TAG        v1.14.0
        GIT_PROGRESS   TRUE
    )
    set(INSTALL_GTEST OFF CACHE BOOL "" FORCE)
    FetchContent_MakeAvailable(googletest)

    add_subdirectory(tests)
endif()

if(BUILD_DOCS)
    find_package(Doxygen)
    if(Doxygen_FOUND)
        add_subdirectory(docs)
    else()
        message(STATUS "Doxygen not found, not building docs")
    endif()
endif()

message(STATUS "PROJECT_NAME:              ${PROJECT_NAME}")
message(STATUS "PROJECT_VERSION:           ${PROJECT_VERSION}")
message(STATUS "BUILD_SHARED_LIBS:         ${BUILD_SHARED_LIBS}")
message(STATUS "BUILD_TESTING:             ${BUILD_TESTING}")
message(STATUS "CMAKE_CXX_COMPILER:        ${CMAKE_CXX_COMPILER}")
message(STATUS "CMAKE_BUILD_TYPE:          ${CMAKE_BUILD_TYPE}")
message(STATUS "CMAKE_INSTALL_PREFIX:      ${CMAKE_INSTALL_PREFIX}")
message(STATUS "CMAKE_GENERATOR:           ${CMAKE_GENERATOR}")
message(STATUS "PROJECT_SOURCE_DIR:        ${PROJECT_SOURCE_DIR}")
message(STATUS "CMAKE_CURRENT_BINARY_DIR:  ${CMAKE_CURRENT_BINARY_DIR}")
