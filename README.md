- [CMake Tutorials](#cmake-tutorials)
  * [Project Structure](#project-structure)
  * [CMake Command Line Parameters](#cmake-command-line-parameters)
  * [CMake Generators](#cmake-generators)
  * [Setting Build Type with CMAKE_BUILD_TYPE and Ninja Multi-Config](#setting-build-type-with-cmake_build_type-and-ninja-multi-config)
  * [VS Code and Ninja Multi-Config](#vs-code-and-ninja-multi-config)
  * [CMakePresets.json and CMakeUserPresets.json](#cmakepresetsjson-and-cmakeuserpresetsjson)
  * [Building a Project](#building-a-project)
  * [Setting the Compiler](#setting-the-compiler)
    + [Check Compiler and Version](#check-compiler-and-version)
  * [Built-in Commands and Functions](#built-in-commands-and-functions)
    + [PUBLIC, PRIVATE, and INTERFACE](#public-private-and-interface)
  * [Variables, Cache Variables and Options](#variables-cache-variables-and-options)
    + [Variables](#variables)
    + [Cache Variables](#cache-variables)
    + [Options](#options)
  * [Setting Important Variables](#setting-important-variables)
  * [Properties](#properties)
  * [Scripting in CMake](#scripting-in-cmake)
  * [Connecting CMake with Your Code](#connecting-cmake-with-your-code)
    + [Reading from CMake into Your Files](#reading-from-cmake-into-your-files)
    + [CMake Reading from Your Files](#cmake-reading-from-your-files)
  * [Running a Command in CMake](#running-a-command-in-cmake)
    + [At Configure Time](#at-configure-time)
    + [At Build Time](#at-build-time)
  * [Creating and Installing a Library](#creating-and-installing-a-library)
    + [SOVERSION](#soversion)
    + [MyLibraryConfig](#mylibraryconfig)
  * [Exporting Your Project](#exporting-your-project)
    + [1. Adding a Subproject](#1-adding-a-subproject)
    + [2. Installing and Calling find_package()](#2-installing-and-calling-find_package)
    + [Explanation of MyProjectConfig.cmake.in](#explanation-of-myprojectconfigcmakein)
    + [Find\<package\>.cmake](#findpackagecmake)
    + [\<package\>Config.cmake](#packageconfigcmake)
  * [Finding CMake Packages from Arbitrary Locations](#finding-cmake-packages-from-arbitrary-locations)
  * [Using pkg-config (.pc files)](#using-pkg-config-pc-files)
  * [Testing with CMake](#testing-with-cmake)
    + [GoogleTest](#googletest)
    + [Submodule method](#submodule-method)
    + [FetchContent](#fetchcontent)
  * [Semantic Versioning](#semantic-versioning)
    + [GitHub Automatic Releases from Tags](#github-automatic-releases-from-tags)
  * [Diagnostics](#diagnostics)
    + [Visualising the Dependency Graph](#visualising-the-dependency-graph)
    + [Listing All Variables with Description](#listing-all-variables-with-description)

# CMake Tutorials

These notes target modern CMake. Examples use `cmake_minimum_required(VERSION 3.25)`, which is a sane floor for new projects and is fully supported by the CMake 4.x line (latest stable: **4.3.2**). Pick a higher minimum if you rely on newer features.

## Project Structure

```
project
├── .clang-format
├── .github
│   └── workflows
│       └── ci.yml
├── .gitignore
├── .gitmodules
├── CHANGELOG.md
├── CMakeLists.txt
├── CMakePresets.json
├── CONTRIBUTING.md
├── LICENSE
├── README.md
├── apps
│   ├── CMakeLists.txt
│   └── app.cpp
├── cmake
│   └── FindSomeLib.cmake
├── docs
│   ├── CMakeLists.txt
│   └── Doxyfile.in
├── extern
│   └── googletest
├── include
│   └── <project_name>
│       └── lib.hpp
├── scripts
│   └── helper.py
├── src
│   ├── CMakeLists.txt
│   ├── include
│   │   └── private_header.hpp
│   └── lib.cpp
└── tests
    ├── CMakeLists.txt
    └── testlib.cpp
```

A few notes:

- `include/<project_name>/` — the inner directory should match the library/target name so consumers write `#include <myproject/lib.hpp>` and headers don't collide on `-I` paths.
- `src/include/` holds private headers used inside the library but never installed.
- `extern/` (or `third_party/` / `vendor/`) holds vendored dependencies and submodules.
- `cmake/` holds `Find<Pkg>.cmake` modules and any reusable CMake snippets.

## CMake Command Line Parameters

```
-S <path to source directory>
-B <path to build directory>
-D <cache variable>=<value>
-G <generator-name>
```

Typical configure + build:

```bash
cmake -S . -B build -G "Ninja Multi-Config"
cmake --build build --config Release
```

## CMake Generators

A CMake Generator writes the input files for a native build system. Use `-G` to choose one.

1. **Makefile generators**

   ```bash
   cmake -S . -B build -G "Unix Makefiles"
   ```

2. **Ninja generators**: `Ninja` and `Ninja Multi-Config` (install with `sudo apt install ninja-build`).

   ```bash
   cmake -S . -B build -G Ninja
   ```

3. **IDE generators**, e.g. `Visual Studio 17 2022`:

   ```bash
   cmake -S . -B build -G "Visual Studio 17 2022"
   ```

You can read the active generator via `CMAKE_GENERATOR`:

```cmake
message(STATUS "CMAKE_GENERATOR: ${CMAKE_GENERATOR}")
```

The value of this variable should **never** be modified by project code. Always set it via `-G` on the command line.

## Setting Build Type with CMAKE_BUILD_TYPE and Ninja Multi-Config

The Ninja generator has two flavors. They handle build types differently:

1. **Ninja (single-config)**
   - Configure with `-DCMAKE_BUILD_TYPE=<type>`:
     ```bash
     cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
     ```
   - `cmake --build build` then builds the `Release` configuration. `--config` is ignored.

2. **Ninja Multi-Config** (stable since CMake 3.17)
   - Configure without a build type:
     ```bash
     cmake -S . -B build -G "Ninja Multi-Config"
     ```
   - Choose the configuration at build time:
     ```bash
     cmake --build build --config Release
     ```
   - You can switch between Debug, Release, etc. in the same build directory without reconfiguring.

## VS Code and Ninja Multi-Config

Add this to your **settings.json** (`~/.config/Code/User/settings.json`, or open via `Ctrl+Shift+P`):

```json
{
  "cmake.configureSettings": {
    "CMAKE_EXPORT_COMPILE_COMMANDS": "YES"
  },
  "cmake.generator": "Ninja Multi-Config"
}
```

## CMakePresets.json and CMakeUserPresets.json

Both files share a format. `CMakePresets.json` describes project-wide build details; `CMakeUserPresets.json` is for developer-local details. If the project is in Git, `CMakePresets.json` is tracked and `CMakeUserPresets.json` should be `.gitignore`d.

Example `CMakePresets.json`:

```json
{
  "version": 6,
  "cmakeMinimumRequired": {
    "major": 3,
    "minor": 25,
    "patch": 0
  },
  "configurePresets": [
    {
      "name": "ninja-multi",
      "displayName": "Ninja Multi-Config",
      "description": "Use Ninja with multiple configurations",
      "generator": "Ninja Multi-Config",
      "binaryDir": "${sourceDir}/out/build/${presetName}",
      "cacheVariables": {
        "CMAKE_CONFIGURATION_TYPES": "Debug;Release;RelWithDebInfo;MinSizeRel"
      }
    }
  ],
  "buildPresets": [
    { "name": "ninja-multi-debug",          "configurePreset": "ninja-multi", "configuration": "Debug" },
    { "name": "ninja-multi-release",        "configurePreset": "ninja-multi", "configuration": "Release" },
    { "name": "ninja-multi-relwithdebinfo", "configurePreset": "ninja-multi", "configuration": "RelWithDebInfo" },
    { "name": "ninja-multi-minsizerel",     "configurePreset": "ninja-multi", "configuration": "MinSizeRel" }
  ]
}
```

Configure once, then build any preset:

```bash
cmake --preset ninja-multi
cmake --build --preset ninja-multi-debug
cmake --build --preset ninja-multi-release
```

## Building a Project

Verbose, parallel build (`-v` for verbose, `-j N` for parallel on N cores):

```bash
cmake --build build -v -j 8
```

Build a specific target. Given:

```cmake
add_executable(filesystem src/filesystem.cpp)
```

Build it with:

```bash
cmake --build build --target filesystem
cmake --build build --target test
cmake --build build --target docs
```

Build and install (the install path is `CMAKE_INSTALL_PREFIX`):

```bash
cmake --build build --target install --config Release
```

For Ninja Multi-Config, choose the configuration at build time:

```bash
cmake --build build --config <Config>
```

Read more on Ninja Multi-Config [here](https://cmake.org/cmake/help/latest/generator/Ninja%20Multi-Config.html).

## Setting the Compiler

clang:

```bash
export CC=/usr/bin/clang
export CXX=/usr/bin/clang++
```

gcc:

```bash
export CC=/usr/bin/gcc
export CXX=/usr/bin/g++
```

To set the compiler from inside a `CMakeLists.txt` (this must come **before** `project()`):

```cmake
option(USE_CLANG "build with clang" OFF)

if(USE_CLANG)
    set(CMAKE_C_COMPILER   "/usr/bin/clang")
    set(CMAKE_CXX_COMPILER "/usr/bin/clang++")

    set(CMAKE_AR      "/usr/bin/llvm-ar")
    set(CMAKE_LINKER  "/usr/bin/llvm-ld")
    set(CMAKE_NM      "/usr/bin/llvm-nm")
    set(CMAKE_OBJDUMP "/usr/bin/llvm-objdump")
    set(CMAKE_RANLIB  "/usr/bin/llvm-ranlib")
endif()
```

Avoid hard-coding `CMAKE_CXX_FLAGS_*` for build types — the defaults are fine, and `-O3` is the highest valid optimization level (there is no `-O4`).

### Check Compiler and Version

```cmake
message(STATUS "compiler: ${CMAKE_CXX_COMPILER_ID}")
message(STATUS "compiler version: ${CMAKE_CXX_COMPILER_VERSION}")
```

## Built-in Commands and Functions

Always use lowercase function names. Upper case is reserved for variables. Supported languages: C, CXX, Fortran, ASM, CUDA, CSharp, SWIFT.

```cmake
cmake_minimum_required(VERSION 3.25)
project(my-cmake-project VERSION 1.2.3.4 DESCRIPTION "An example project with CMake" LANGUAGES CXX)
```

**`add_subdirectory()`** — add a subdirectory to the build. Its `CMakeLists.txt` is loaded and run.

```cmake
add_subdirectory(src)
add_subdirectory(src binary_dir)
```

**`include()`** — load and run CMake code from a file.

```cmake
include(someother.cmake)
```

**`include_directories()`** — tell CMake where to look for `*.h` files. Affects all targets in this directory and subdirectories added afterward. Prefer the target-scoped form below.

```cmake
include_directories(${PROJECT_SOURCE_DIR}/include)
```

**`target_include_directories()`** — add an include directory for a specific target.

```cmake
target_include_directories(my_target PUBLIC  ${PROJECT_SOURCE_DIR}/include)
target_include_directories(my_target PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/src/include)
```

The target must already have been created by `add_executable()` or `add_library()`. Public headers (your library's API) live under `<project_root>/include/<project_name>` and are exported with `PUBLIC`. Private headers live under your source directory and are added with `PRIVATE`. Private headers should not be installed.

#### PUBLIC, PRIVATE, and INTERFACE

Other targets may compile against yours and need to access headers you used. Declaring them as `PUBLIC` makes them available to consumers; `PRIVATE` keeps them target-local; `INTERFACE` exposes them only to consumers (header-only libraries). For example:

- `target_include_directories(A PRIVATE ${Boost_INCLUDE_DIRS})` — Boost headers are used only in A's `.cpp` files or private `.h` files.
- `target_include_directories(A PUBLIC  ${Boost_INCLUDE_DIRS})` — Boost headers appear in A's public headers, which are included by both A's sources and by clients of A.

**`add_library()`** — create a library:

```cmake
add_library(tools STATIC src/tools.cpp)   # or SHARED / MODULE
```

**`add_executable()`** — define an executable target:

```cmake
add_executable(main src/tools_main.cpp src/tools_lib.cpp)
```

**`target_link_libraries()`** — link targets together. Always pass a visibility specifier:

```cmake
target_link_libraries(main PRIVATE tools)
```

**`mark_as_advanced()`** — hide variables from the basic CMake GUI views; useful for keeping the cache tidy:

```cmake
mark_as_advanced(
    BUILD_GMOCK BUILD_GTEST BUILD_SHARED_LIBS
    gmock_build_tests gtest_build_samples gtest_build_tests
    gtest_disable_pthreads gtest_force_shared_crt gtest_hide_internal_symbols
)
```

**`install()`** — runs during `cmake --install <dir>` or `cmake --build <dir> --target install`. Export your targets so consumers can `find_package()` them:

```cmake
install(TARGETS Foo
    EXPORT FooTargets
    LIBRARY  DESTINATION lib
    ARCHIVE  DESTINATION lib
    RUNTIME  DESTINATION bin
    INCLUDES DESTINATION include
)

install(EXPORT FooTargets
    FILE FooTargets.cmake
    NAMESPACE Foo::
    DESTINATION lib/cmake/Foo
)
```

## Variables, Cache Variables and Options

### Variables

Setting:

```cmake
set(XXX value)
```

Reading:

```cmake
${XXX}
```

Environment variables:

```cmake
$ENV{HOME}
```

Paths may contain spaces, so always quote variables that hold paths: `"${VAR_PATH}"`, never bare `${VAR_PATH}`.

You can set a variable in the parent scope with `PARENT_SCOPE`. Given:

```
root
├── src
│   └── CMakeLists.txt
└── CMakeLists.txt
```

Root `CMakeLists.txt`:

```cmake
set(BAR "Bar from root.")
```

`src/CMakeLists.txt`:

```cmake
set(BAR "Bar from src.")              # set in this scope
set(BAR "${BAR}" PARENT_SCOPE)        # set in the parent scope too
```

Then in your root `CMakeLists.txt`, watch the value change:

```cmake
message(STATUS "root: ${BAR}")
add_subdirectory("${PROJECT_SOURCE_DIR}/src")
message(STATUS "root: ${BAR}")
```

### Cache Variables

If you want to set a variable from the command line, use the cache:

```cmake
set(MY_CACHE_VARIABLE "VALUE" CACHE STRING "Description")
```

Then override from the command line:

```bash
cmake -S . -B build -DMY_CACHE_VARIABLE=2.3.6.9
```

Use `FORCE` to prevent the user from overriding:

```cmake
set(MY_LIBRARY_VERSION_MAJOR 1 CACHE STRING "major version" FORCE)
```

Common cache variables every project should respect:

- `CMAKE_BUILD_TYPE` — pick from `Release`, `RelWithDebInfo`, `Debug`, sometimes more.
- `CMAKE_INSTALL_PREFIX` — installation root. Defaults to `/usr/local` on UNIX; `~/.local` is common for user installs.
- `BUILD_SHARED_LIBS` — `ON` or `OFF` to control the default for `add_library()` calls without an explicit type.
- `BUILD_TESTING` — common name for enabling tests (used by `include(CTest)`).

### Options

An `option` is a boolean cache variable:

```cmake
option(PACKAGE_TESTS "Build the tests" ON)
```

## Setting Important Variables

C++17:

```cmake
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
```

C++20:

```cmake
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
```

Position-independent code, globally:

```cmake
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
```

Or per target:

```cmake
set_target_properties(lib1 PROPERTIES POSITION_INDEPENDENT_CODE ON)
```

Warnings as errors / strict warnings (prefer `target_compile_options` over global flags):

```cmake
if(MSVC)
    target_compile_options(my_target PRIVATE /W4 /WX)
else()
    target_compile_options(my_target PRIVATE -Wall -Wextra -Wpedantic -Werror)
endif()
```

AddressSanitizer (compile **and** link with the same flag):

```cmake
target_compile_options(my_target PRIVATE -fsanitize=address -fno-omit-frame-pointer)
target_link_options(my_target    PRIVATE -fsanitize=address)
```

Suppress specific warnings on a target:

```cmake
target_compile_options(my_target PRIVATE -Wno-unused-variable -Wno-unused-private-field)
```

Set the build type (single-config generators only):

```cmake
set(CMAKE_BUILD_TYPE Release)   # or Debug / RelWithDebInfo / MinSizeRel
```

Useful variables to print while debugging your build:

```cmake
message(STATUS "PROJECT_NAME: ${PROJECT_NAME}")
message(STATUS "PROJECT_VERSION: ${PROJECT_VERSION}")
message(STATUS "BUILD_SHARED_LIBS: ${BUILD_SHARED_LIBS}")
message(STATUS "BUILD_TESTING: ${BUILD_TESTING}")
message(STATUS "CMAKE_C_COMPILER: ${CMAKE_C_COMPILER}")
message(STATUS "CMAKE_CXX_COMPILER: ${CMAKE_CXX_COMPILER}")
message(STATUS "CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
message(STATUS "CMAKE_INSTALL_PREFIX: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")
message(STATUS "CMAKE_SOURCE_DIR: ${CMAKE_SOURCE_DIR}")
message(STATUS "CMAKE_CURRENT_SOURCE_DIR: ${CMAKE_CURRENT_SOURCE_DIR}")
message(STATUS "PROJECT_BINARY_DIR: ${PROJECT_BINARY_DIR}")
message(STATUS "CMAKE_CURRENT_BINARY_DIR: ${CMAKE_CURRENT_BINARY_DIR}")
message(STATUS "CMAKE_BINARY_DIR: ${CMAKE_BINARY_DIR}")
message(STATUS "CMAKE_MODULE_PATH: ${CMAKE_MODULE_PATH}")
message(STATUS "CMAKE_INCLUDE_PATH: ${CMAKE_INCLUDE_PATH}")
message(STATUS "CMAKE_PREFIX_PATH: ${CMAKE_PREFIX_PATH}")
message(STATUS "CMAKE_LIBRARY_PATH: ${CMAKE_LIBRARY_PATH}")
message(STATUS "CMAKE_SYSTEM_LIBRARY_PATH: ${CMAKE_SYSTEM_LIBRARY_PATH}")
message(STATUS "CMAKE_CTEST_COMMAND: ${CMAKE_CTEST_COMMAND}")
message(STATUS "CMAKE_GENERATOR: ${CMAKE_GENERATOR}")
```

## Properties

CMake also stores information in **properties**, which attach to an item like a directory or a target. Two ways to set them:

The general form:

```cmake
set_property(TARGET TargetName PROPERTY CXX_STANDARD 17)
```

The shortcut for setting several properties on one target:

```cmake
set_target_properties(TargetName PROPERTIES CXX_STANDARD 17)
```

## Scripting in CMake

```cmake
if() ... else()/elseif() ... endif()

option(BUILD_TESTING "Enable testing" OFF)
if(BUILD_TESTING)
    add_subdirectory(tests)
endif()

if(IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/src/temp")
    add_subdirectory(src/temp)
endif()

foreach() ... endforeach()
```

Conditional checks against the compiler:

```cmake
if(CMAKE_CXX_COMPILER_ID STREQUAL "Clang" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 14)
    add_executable(foo src/foo.cpp)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 13)
    add_executable(foo src/foo.cpp)
elseif(CMAKE_CXX_COMPILER_ID STREQUAL "MSVC" AND CMAKE_CXX_COMPILER_VERSION VERSION_GREATER_EQUAL 1900)
    add_executable(foo src/foo.cpp)
endif()
```

Note: `VERSION_GREATER_EQUAL` does proper version-string comparison; plain `GREATER_EQUAL` does numeric comparison and can misbehave on multi-component versions.

Refs: [1](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html)

## Connecting CMake with Your Code

### Reading from CMake into Your Files

`configure_file` copies a template file and substitutes CMake variables. Use `@ONLY` to substitute only `@VAR@` (leaving `${...}` alone, which is helpful in shell or C++ files):

```cmake
configure_file(
    "${PROJECT_SOURCE_DIR}/include/project/Version.h.in"
    "${PROJECT_BINARY_DIR}/include/project/Version.h"
    @ONLY
)
```

Contents of `Version.h.in`:

```c
#pragma once

#define MY_VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define MY_VERSION_MINOR @PROJECT_VERSION_MINOR@
#define MY_VERSION_PATCH @PROJECT_VERSION_PATCH@
#define MY_VERSION_TWEAK @PROJECT_VERSION_TWEAK@
#define MY_VERSION       "@PROJECT_VERSION@"
```

Generated `Version.h`:

```c
#pragma once

#define MY_VERSION_MAJOR 1
#define MY_VERSION_MINOR 2
#define MY_VERSION_PATCH 3
#define MY_VERSION_TWEAK 4
#define MY_VERSION       "1.2.3.4"
```

### CMake Reading from Your Files

```cmake
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/project/Version.hpp" VERSION)

string(REGEX MATCH "VERSION_MAJOR ([0-9]+)" _ "${VERSION}")
set(VERSION_MAJOR ${CMAKE_MATCH_1})

string(REGEX MATCH "VERSION_MINOR ([0-9]+)" _ "${VERSION}")
set(VERSION_MINOR ${CMAKE_MATCH_1})

string(REGEX MATCH "VERSION_PATCH ([0-9]+)" _ "${VERSION}")
set(VERSION_PATCH ${CMAKE_MATCH_1})

message(STATUS "VERSION: ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")
```

## Running a Command in CMake

### At Configure Time

Use `${CMAKE_COMMAND}`, `find_package(Git)`, or `find_program` to locate a command. `RESULT_VARIABLE` captures the exit code; `OUTPUT_VARIABLE` captures stdout.

```cmake
find_package(Git QUIET)

if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
    execute_process(
        COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
        WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
        RESULT_VARIABLE GIT_SUBMOD_RESULT
    )
    if(NOT GIT_SUBMOD_RESULT EQUAL "0")
        message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
    endif()
endif()
```

### At Build Time

Use the modern `Python` find-module (the old `PythonInterp` is deprecated):

```cmake
find_package(Python REQUIRED COMPONENTS Interpreter)

add_custom_command(
    OUTPUT  "${CMAKE_CURRENT_BINARY_DIR}/include/Generated.hpp"
    COMMAND "${Python_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/scripts/GenerateHeader.py" --argument
    DEPENDS some_target
)

add_custom_target(generate_header ALL
    DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/include/Generated.hpp"
)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/include/Generated.hpp DESTINATION include)
```

## Creating and Installing a Library

Create the following structure:

```
MainProject/
├── CMakeLists.txt
├── MyLibrary
│   ├── CMakeLists.txt
│   ├── include
│   │   └── MyLibrary.hpp
│   └── src
│       └── MyLibrary.cpp
└── src
    └── main.cpp
```

`MainProject/MyLibrary/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.25)
project(MyLibrary VERSION 1.0.0 LANGUAGES CXX)

add_library(MyLibrary SHARED
    src/MyLibrary.cpp
)

target_include_directories(MyLibrary PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

set_target_properties(MyLibrary PROPERTIES
    VERSION       ${PROJECT_VERSION}
    SOVERSION     1
    PUBLIC_HEADER include/MyLibrary.hpp
)

install(TARGETS MyLibrary
    EXPORT MyLibraryConfig
    LIBRARY       DESTINATION lib
    ARCHIVE       DESTINATION lib
    RUNTIME       DESTINATION bin
    INCLUDES      DESTINATION include
    PUBLIC_HEADER DESTINATION include
)

install(EXPORT MyLibraryConfig NAMESPACE MyLibrary:: DESTINATION cmake)
```

### SOVERSION

`SOVERSION` sets the shared-object version of a shared library. On Unix-like systems, libraries are versioned so multiple versions can coexist and so binaries built against an older version keep working when ABI compatibility is preserved.

`SOVERSION` is independent of the project version from `project()`. It's encoded into the actual filename of the shared library — given `libMyLibrary.so` with `SOVERSION 1`, the on-disk file is `libMyLibrary.so.1`. Bump `SOVERSION` when you break ABI.

### MyLibraryConfig

`MyLibraryConfig` is the export configuration generated by `install(EXPORT ...)`. It exports the project's targets so other projects can consume them.

In the snippet above, `MyLibrary` is exported under the namespace `MyLibrary::`, so consumers reference it as `MyLibrary::MyLibrary`:

```cmake
install(EXPORT MyLibraryConfig NAMESPACE MyLibrary:: DESTINATION cmake)
```

This installs the export file under `cmake/` inside the install prefix. The export carries the target's include directories, compile options, and link dependencies. Consumers pick it up via `find_package(MyLibrary CONFIG)`.

`MainProject/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.25)
project(MainProject VERSION 1.0.0 LANGUAGES CXX)

add_subdirectory(MyLibrary)

add_executable(MainProject src/main.cpp)
target_link_libraries(MainProject PRIVATE MyLibrary)

install(TARGETS MainProject
    EXPORT MainProjectConfig
    RUNTIME  DESTINATION bin
    LIBRARY  DESTINATION lib
    ARCHIVE  DESTINATION lib
    INCLUDES DESTINATION include
)

install(EXPORT MainProjectConfig NAMESPACE MainProject:: DESTINATION cmake)
```

Configure, build, install:

```bash
cmake -S . -B build -G "Ninja Multi-Config" -DCMAKE_INSTALL_PREFIX=~/usr
cmake --build build --config Release
cmake --install build --config Release
```

This installs:

```
/home/behnam/usr/
├── bin
│   └── MainProject
├── cmake
│   ├── MainProjectConfig.cmake
│   ├── MainProjectConfig-release.cmake
│   ├── MyLibraryConfig.cmake
│   └── MyLibraryConfig-release.cmake
├── include
│   └── MyLibrary.hpp
└── lib
    ├── libMyLibrary.so     -> libMyLibrary.so.1
    ├── libMyLibrary.so.1   -> libMyLibrary.so.1.0.0
    └── libMyLibrary.so.1.0.0
```

To use `MyLibrary` from another project:

```
AnotherProject/
├── CMakeLists.txt
└── src
    └── main.cpp
```

`AnotherProject/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.25)
project(AnotherProject VERSION 1.0.0 LANGUAGES CXX)

find_package(MyLibrary REQUIRED)

add_executable(AnotherProject src/main.cpp)
target_link_libraries(AnotherProject PRIVATE MyLibrary::MyLibrary)
```

Optionally print everything CMake learned about the imported package:

```cmake
get_cmake_property(_variableNames VARIABLES)
foreach(_variableName ${_variableNames})
    if(_variableName MATCHES "^MyLibrary")
        message(STATUS "${_variableName}=${${_variableName}}")
    endif()
endforeach()

get_property(_targets GLOBAL PROPERTY IMPORTED_TARGETS)
foreach(_target ${_targets})
    if(_target MATCHES "^MyLibrary::")
        message(STATUS "Imported target: ${_target}")
    endif()
endforeach()
```

Build the consumer:

```bash
cmake -S . -B build -G "Ninja Multi-Config" -DCMAKE_PREFIX_PATH=~/usr -DCMAKE_INSTALL_PREFIX=~/usr
cmake --build build --config Debug
cmake --install build --config Debug
```

## Exporting Your Project

There are three ways to make your project consumable by another project.

### 1. Adding a Subproject

For small or header-only libraries, drop the source tree in and use `add_subdirectory()`. Use `CMAKE_CURRENT_SOURCE_DIR` instead of `PROJECT_SOURCE_DIR` so the project still works when included this way, and gate top-level-only behavior:

```cmake
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
    # tests, docs, install rules — anything that should only run
    # when this project is the top-level build.
endif()
```

### 2. Installing and Calling find_package()

Given:

```
Exporting/
├── CMakeLists.txt
├── include
│   └── my_header.h
├── MyProjectConfig.cmake.in
└── src
    ├── main.cpp
    └── my_source_file.cpp
```

`Exporting/MyProjectConfig.cmake.in`:

```cmake
@PACKAGE_INIT@

include("${CMAKE_CURRENT_LIST_DIR}/MyProjectTargets.cmake")

set_and_check(MY_PROJECT_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include")
```

`CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.25)
project(MyProject VERSION 1.0 LANGUAGES CXX)

set(SOURCES
    src/my_source_file.cpp
    src/main.cpp
)

set(HEADERS
    include/my_header.h
)

add_library(my_library ${SOURCES})

target_include_directories(my_library PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

install(TARGETS my_library
    EXPORT MyProjectTargets
    LIBRARY  DESTINATION lib
    ARCHIVE  DESTINATION lib
    RUNTIME  DESTINATION bin
    INCLUDES DESTINATION include
)
install(FILES ${HEADERS} DESTINATION include)

install(EXPORT MyProjectTargets
    FILE      MyProjectTargets.cmake
    NAMESPACE MyProject::
    DESTINATION lib/cmake/MyProject
)

include(CMakePackageConfigHelpers)
set(CONFIG_INSTALL_DIR lib/cmake/MyProject)

configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/MyProjectConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
    INSTALL_DESTINATION ${CONFIG_INSTALL_DIR}
)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
    VERSION       ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
    DESTINATION ${CONFIG_INSTALL_DIR}
)
```

In this `CMakeLists.txt`:

- `configure_package_config_file` generates the package config file `MyProjectConfig.cmake`.
- `write_basic_package_version_file` generates `MyProjectConfigVersion.cmake`.
- Both generated files are installed into `lib/cmake/MyProject`.
- `MyProjectConfig.cmake.in` is the template that says how the library should be configured when imported by other projects (typically just `include(MyProjectTargets)` to pull in the exported targets).

### Explanation of MyProjectConfig.cmake.in

`MyProjectConfig.cmake.in` is the template that becomes `MyProjectConfig.cmake` at install time:

1. `@PACKAGE_INIT@` — replaced with initialization code that sets up helper variables and macros.
2. `include("${CMAKE_CURRENT_LIST_DIR}/MyProjectTargets.cmake")` — pulls in the exported targets and their properties.
3. `set_and_check(MY_PROJECT_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include")` — sets a helper variable consumers can use, and asserts the path exists.

You can extend `MyProjectConfig.cmake.in` further:

- **Variables for consumers**:
  ```cmake
  set(MY_PROJECT_USE_FEATURE_XYZ ON)
  ```
- **Compiler flags specific to the project**:
  ```cmake
  if(MSVC)
      set(MY_PROJECT_MSVC_FLAGS "/W3 /EHsc")
  else()
      set(MY_PROJECT_GCC_FLAGS "-Wall -Wextra")
  endif()
  ```
- **Macros**:
  ```cmake
  target_compile_definitions(my_library PUBLIC MY_PROJECT_USE_SOME_FEATURE)
  ```
- **Dependency hints**:
  ```cmake
  set(MY_PROJECT_REQUIRED_LIBRARIES SomeOtherLibrary)
  ```

Once exported and installed, you can use `find_package`:

```
find_package(<package> [version] [EXACT] [QUIET] [MODULE] [REQUIRED] [[COMPONENTS] ...])
```

- `QUIET` suppresses messages on failure.
- `REQUIRED` fails configuration if the package is not found.
- `<package>_FOUND` is set to indicate whether the package was located.
- `version` requests a compatible version (`major[.minor[.patch[.tweak]]]`).

Configure, build, and install the consumer (see [ExportingConsumer](ExportingConsumer)):

```bash
cmake -S . -B build -G "Ninja Multi-Config" -DCMAKE_PREFIX_PATH=~/usr -DCMAKE_INSTALL_PREFIX=~/usr
cmake --build build --config Debug
cmake --install build --config Debug
```

Consumer `CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.25)
project(ConsumerProject)

find_package(MyProject REQUIRED)

set(SOURCES
    src/consumer_source.cpp
)

add_executable(consumer_executable ${SOURCES})
target_link_libraries(consumer_executable PRIVATE MyProject::my_library)
target_include_directories(consumer_executable PRIVATE ${MyProject_INCLUDE_DIRS})
```

`find_package` operates in two modes: **Module** mode and **Config** mode.

### Find\<package\>.cmake

Module mode looks for `Find<package>.cmake` on `CMAKE_MODULE_PATH`. Use it for libraries that don't ship their own CMake support. Drop your `Find<package>.cmake` under your project's `cmake/` directory, then point `CMAKE_MODULE_PATH` at it:

```cmake
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
```

### \<package\>Config.cmake

Config mode looks for `<name>Config.cmake` or `<lower-case-name>-config.cmake` under `<package>_DIR`. Set `<package>_DIR` first:

```cmake
cmake_minimum_required(VERSION 3.25)
set(MyPack_DIR "$ENV{HOME}/usr/lib/cmake/MyPack")
project(MyPack_user)
find_package(MyPack CONFIG REQUIRED)
message(STATUS "MyPack_FOUND: ${MyPack_FOUND}")
message(STATUS "MyPack_VERSION: ${MyPack_VERSION}")
```

## Finding CMake Packages from Arbitrary Locations

Common pinning patterns when packages are installed in non-standard prefixes:

- PCL:
  ```cmake
  set(PCL_DIR "$ENV{HOME}/usr/share/pcl-1.13/")
  ```
- OpenCV:
  ```cmake
  set(OpenCV_DIR "$ENV{HOME}/usr/share/OpenCV/")
  ```
- glog (from the command line):
  ```bash
  cmake -S . -B build -Dglog_DIR=~/usr/lib/cmake/glog/
  ```
- VTK:
  ```cmake
  set(VTK_DIR "$ENV{HOME}/usr/lib/cmake/vtk-9.2/")
  ```

You can also point `CMAKE_PREFIX_PATH` at an install root and let CMake search underneath it:

```bash
cmake -S . -B build -DCMAKE_PREFIX_PATH=~/usr
```

## Using pkg-config (.pc files)

A typical user-local install layout:

```
<home>/usr/lib/
<home>/usr/include/
<home>/usr/bin/
```

Make `pkg-config` look there:

```bash
export PKG_CONFIG_PATH=~/usr/lib/pkgconfig:${PKG_CONFIG_PATH}
printenv PKG_CONFIG_PATH

pkg-config --cflags    flann
pkg-config --libs      flann
pkg-config --modversion flann
```

- **Flann**:
  ```cmake
  find_package(PkgConfig REQUIRED)
  pkg_search_module(FLANN REQUIRED flann)
  if(FLANN_FOUND)
      message(STATUS "FLANN_VERSION:      ${FLANN_VERSION}")
      message(STATUS "FLANN_LIBRARIES:    ${FLANN_LIBRARIES}")
      message(STATUS "FLANN_INCLUDE_DIRS: ${FLANN_INCLUDEDIR}")
      message(STATUS "FLANN_LIBRARY_DIRS: ${FLANN_LIBDIR}")
  endif()
  ```

- **TinyXML2**:
  ```cmake
  set(ENV{PKG_CONFIG_PATH} "$ENV{HOME}/usr/lib/pkgconfig:$ENV{PKG_CONFIG_PATH}")
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(TINYXML2 REQUIRED tinyxml2)
  add_executable(tinyxml2_demo src/third_party_tools/xml/tinyxml2/tinyxml2_demo.cpp)
  target_include_directories(tinyxml2_demo PRIVATE ${TINYXML2_INCLUDE_DIRS})
  target_link_directories(tinyxml2_demo    PRIVATE ${TINYXML2_LIBRARY_DIRS})
  target_link_libraries(tinyxml2_demo      PRIVATE ${TINYXML2_LIBRARIES})
  ```

- **yaml-cpp**:
  ```cmake
  set(yaml-cpp_DIR "$ENV{HOME}/usr/share/cmake/yaml-cpp")
  find_package(yaml-cpp REQUIRED)
  add_executable(yaml-cpp_example src/third_party_tools/yaml/yaml-cpp/yaml-cpp_example.cpp)
  target_link_libraries(yaml-cpp_example PRIVATE yaml-cpp)
  ```

- **Google Benchmark**:
  ```cmake
  find_package(PkgConfig REQUIRED)
  pkg_check_modules(BENCHMARK REQUIRED benchmark)
  add_executable(benchmark_demo src/third_party_tools/benchmark/benchmark_demo.cpp)
  target_include_directories(benchmark_demo PRIVATE ${BENCHMARK_INCLUDE_DIRS})
  target_link_directories(benchmark_demo    PRIVATE ${BENCHMARK_LIBRARY_DIRS})
  target_link_libraries(benchmark_demo      PRIVATE ${BENCHMARK_LIBRARIES} pthread)
  ```

- **gRPC**:
  ```cmake
  find_package(PkgConfig REQUIRED)
  pkg_search_module(GRPC   REQUIRED grpc)    # C
  pkg_search_module(GRPCPP REQUIRED grpc++)  # C++

  add_executable(main src/main.cpp)
  target_include_directories(main PRIVATE ${GRPCPP_INCLUDEDIR})
  target_link_directories(main    PRIVATE ${GRPCPP_LIBDIR})
  target_link_libraries(main      PRIVATE ${GRPCPP_LIBRARIES})
  ```

## Testing with CMake

Use **CTest** to run unit tests. In your top-level `CMakeLists.txt`:

```cmake
option(BUILD_TESTING "Enable testing" ON)
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
    include(CTest)
    add_subdirectory(tests)
endif()
```

In `tests/CMakeLists.txt`:

```cmake
add_executable(test1 test1.cpp)
add_test(NAME mytester COMMAND test1)
```

`tests/test1.cpp`:

```cpp
int main(int argc, char ** argv)
{
    return 0;
}
```

Then, after building, run `ctest` from the build directory.

You can drive CTest from a script (e.g. `build.cmake`):

```cmake
set(CTEST_SOURCE_DIRECTORY "/source")
set(CTEST_BINARY_DIRECTORY "/binary")
set(ENV{CXXFLAGS} "--coverage")
set(CTEST_CMAKE_GENERATOR "Ninja")
set(CTEST_USE_LAUNCHERS 1)
set(CTEST_COVERAGE_COMMAND "gcov")
set(CTEST_MEMORYCHECK_COMMAND "valgrind")

ctest_start("Continuous")
ctest_configure()
ctest_build()
ctest_test()
ctest_coverage()
ctest_memcheck()
ctest_submit()
```

Run with:

```bash
ctest -S build.cmake
```

### GoogleTest

#### Submodule method

Add GoogleTest as a submodule:

```bash
git submodule add https://github.com/google/googletest.git extern/googletest
```

In your top-level `CMakeLists.txt`:

```cmake
option(PACKAGE_TESTS "Build the tests" ON)
if(PACKAGE_TESTS)
    enable_testing()
    include(GoogleTest)
    add_subdirectory(tests)
endif()
```

In `tests/CMakeLists.txt`:

```cmake
add_subdirectory("${PROJECT_SOURCE_DIR}/extern/googletest" "extern/googletest")
set(INSTALL_GTEST OFF)
add_executable(footest foo.cpp)
target_link_libraries(footest PRIVATE gtest_main)
gtest_discover_tests(footest)
```

`foo.cpp`:

```cpp
#include "gtest/gtest.h"

TEST(Foo, Sum) {
    EXPECT_EQ(2, 1 + 1);
}
```

Run with:

```bash
ctest --test-dir build
```

### FetchContent

`FetchContent_MakeAvailable` is the modern, supported entry point. The old `FetchContent_Populate(<name>)` form is deprecated in CMake 3.30+; don't use it.

```cmake
include(FetchContent)

FetchContent_Declare(
    googletest
    GIT_REPOSITORY https://github.com/google/googletest.git
    GIT_TAG        v1.14.0
    GIT_PROGRESS   TRUE
)

FetchContent_MakeAvailable(googletest)
```

## Semantic Versioning

The scheme is a three-part version number, `MAJOR.MINOR.PATCH`:

- **MAJOR** — incremented for changes that are not backward compatible.
- **MINOR** — incremented for new functionality added in a backward-compatible way.
- **PATCH** — incremented for backward-compatible bug fixes.

Each component is numeric and increases independently, e.g. `1.9.0` → `1.10.0` → `1.11.0`.

Pre-release identifiers can be appended, e.g. `1.3.0-alpha.1`.

Build metadata is appended after `+`, e.g. `1.2.3+20230507` or `2.0.0+25`.

Example progression: `1.0.0` → `1.0.1` → `1.0.2` → `2.0.0+25` → `2.0.0+32` → `2.0.0`.

Refs: [1](https://blogs.stonesteps.ca/1/p/80), [2](https://semver.org/)

To embed the version from a `git describe` tag into the binary:

```cmake
execute_process(
    COMMAND git describe
    OUTPUT_VARIABLE GIT_DESCRIBE_VERSION_TAG
    OUTPUT_STRIP_TRAILING_WHITESPACE
)

target_compile_definitions(main PRIVATE APP_VERSION="${GIT_DESCRIBE_VERSION_TAG}")
```

In your C++ code:

```cpp
std::string appVersion()
{
    return std::string(APP_VERSION);
}
```

### GitHub Automatic Releases from Tags

Refs: [1](https://github.com/marketplace/actions/automatic-releases), [2](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository)

## Diagnostics

### Visualising the Dependency Graph

```bash
cmake -S . -B build --graphviz=build/viz.dot --trace-source=CMakeLists.txt
dot -Tsvg build/viz.dot -o build/viz.svg
```

### Listing All Variables with Description

```bash
cmake -LAH -S . -B build
```

---

References:
[1](https://gist.github.com/mbinna/),
[2](https://cliutils.gitlab.io/modern-cmake/),
[3](https://stackoverflow.com/questions/20746936/what-use-is-find-package-if-you-need-to-specify-cmake-module-path-anyway),
[4](https://gitlab.kitware.com/cmake/community/-/wikis/doc/tutorials/How-To-Find-Libraries),
[5](https://github.com/forexample/package-example),
[6](https://gitlab.kitware.com/cmake/community/-/wikis/doc/tutorials/How-to-create-a-ProjectConfig.cmake-file),
[7](https://gitlab.kitware.com/cmake/community/-/wikis/doc/ctest/Scripting-Of-CTest),
[8](https://cmake.org/cmake/help/latest/command/find_package.html),
[9](https://foonathan.net/2016/03/cmake-install/),
[10](https://foonathan.net/2016/07/cmake-dependency-handling/),
[11](https://stackoverflow.com/questions/17511496/how-to-create-a-shared-library-with-cmake),
[12](https://stackoverflow.com/questions/31969547/what-is-the-difference-between-include-directories-and-target-include-directorie),
[13](https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/),
[14](https://www.youtube.com/watch?v=rLopVhns4Zs)

![alt text](https://img.shields.io/badge/license-BSD-blue.svg)
