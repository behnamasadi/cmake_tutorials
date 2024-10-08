- [CMake Tutorials](#cmake-tutorials)
  * [Project Structure](#project-structure)
  * [Setting The Compiler](#setting-the-compiler)
    + [Check Compiler And Version](#check-compiler-and-version)
  * [CMake Command Line Parameters](#cmake-command-line-parameters)
  * [CMake Variables, Cache Variables and Options](#cmake-variables--cache-variables-and-options)
    + [Variables](#variables)
    + [Cache Variables](#cache-variables)
    + [Options](#options)
  * [Setting Important Variables](#setting-important-variables)
  * [Properties](#properties)
  * [CMake Generators](#cmake-generators)
  * [setting build type with DCMAKE_BUILD_TYPE and Ninja Multi-Config](#setting-build-type-with-dcmake-build-type-and-ninja-multi-config)
  * [VScode and Ninja Multi-Config](#vscode-and-ninja-multi-config)
  * [CMakePresets.json and CMakeUserPresets.json](#cmakepresetsjson-and-cmakeuserpresetsjson)
  * [Building Project](#building-project)
  * [Visualising Dependency Graph:](#visualising-dependency-graph-)
  * [Listing All Variables With Description:](#listing-all-variables-with-description-)
  * [Watch a Variable](#watch-a-variable)
  * [Colorized Message](#colorized-message)
  * [Scripting in CMake](#scripting-in-cmake)
  * [Built-in Commands And Functions in CMake](#built-in-commands-and-functions-in-cmake)
      - [PUBLIC, PRIVATE, and INTERFACE](#public--private--and-interface)
  * [Semantic Versioning](#semantic-versioning)
    + [GitHub Automatic Releases From tags and  Release Management](#github-automatic-releases-from-tags-and--release-management)
  * [Connecting CMake With Your Code](#connecting-cmake-with-your-code)
    + [Reading From CMake Into Your Files](#reading-from-cmake-into-your-files)
    + [CMake Reading From Your Files](#cmake-reading-from-your-files)
  * [Testing with CMake](#testing-with-cmake)
    + [GoogleTest](#googletest)
    + [Download method](#download-method)
    + [FetchContent](#fetchcontent)
  * [Exporting Your Project](#exporting-your-project)
    + [1. Adding Subproject](#1-adding-subproject)
    + [2. Exporting Build Directory of Your Project](#2-exporting-build-directory-of-your-project)
    + [3. Installing Your Project And Calling find_package()](#3-installing-your-project-and-calling-find-package--)
    + [Explanation of MyProjectConfig.cmake.in](#explanation-of-myprojectconfigcmakein)
    + [Find\<package\>.cmake](#find--package--cmake)
    + [\<package\>Config.cmake](#--package--configcmake)
  * [How to find CMake from arbitrary installed locations](#how-to-find-cmake-from-arbitrary-installed-locations)
  * [Using pkgconfig (.pc files)](#using-pkgconfig--pc-files-)
  * [Running a Command in CMake](#running-a-command-in-cmake)
    + [At Configure Time](#at-configure-time)
    + [At Build Time](#at-build-time)
  * [Creating and Installing a library](#creating-and-installing-a-library)
    + [SOVERSION](#soversion)
    + [MyLibraryConfig](#mylibraryconfig)

# CMake Tutorials

##  Project Structure

```
project  
├──.gitignore  
├──README.md  
├──LICENCE.md  
├──CMakeLists.txt  
├──cmake  
│    └──FindSomeLib.cmake  
├──include  
│    └──poject  
│        └── lib.hpp  
├──src  
│    ├──CMakeLists.txt  
│    ├──lib.cpp  
│    └──include  
│        └──private_header.hpp  
├──apps  
│    ├──CMakeLists.txt  
│    └──app.cpp  
├──tests  
│    ├──CMakeLists.txt  
│    └──testlib.cpp  
├──docs  
│    └── CMakeLists.txt  
├──extern  
|    └──googletest  
└──scripts  
     └──helper.py  
```
## Setting The Compiler
clang

```
export CC=/usr/bin/clang
export CXX=/usr/bin/clang++
```
gcc

```
export CC=/usr/bin/gcc
export CXX=/usr/bin/g++
```

to use it inside of your CMakeLists.txt

```
option(USE_CLANG "build application with clang" OFF)

if(USE_CLANG)
        set (CMAKE_C_COMPILER             "/usr/bin/clang")
        #set (CMAKE_C_FLAGS                "-Wall -std=c99")
        set (CMAKE_C_FLAGS_DEBUG          "-g")
        set (CMAKE_C_FLAGS_MINSIZEREL     "-Os -DNDEBUG")
        set (CMAKE_C_FLAGS_RELEASE        "-O4 -DNDEBUG")
        set (CMAKE_C_FLAGS_RELWITHDEBINFO "-O2 -g")

        set (CMAKE_CXX_COMPILER             "/usr/bin/clang++")
        set (CMAKE_CXX_FLAGS                "-Wall")
        set (CMAKE_CXX_FLAGS_DEBUG          "-g")
        set (CMAKE_CXX_FLAGS_MINSIZEREL     "-Os -DNDEBUG")
        set (CMAKE_CXX_FLAGS_RELEASE        "-O4 -DNDEBUG")
        set (CMAKE_CXX_FLAGS_RELWITHDEBINFO "-O2 -g")

        set (CMAKE_AR      "/usr/bin/llvm-ar")
        set (CMAKE_LINKER  "/usr/bin/llvm-ld")
        set (CMAKE_NM      "/usr/bin/llvm-nm")
        set (CMAKE_OBJDUMP "/usr/bin/llvm-objdump")
        set (CMAKE_RANLIB  "/usr/bin/llvm-ranlib")
endif()
```

### Check Compiler And Version

```
message("compiler is:" ${CMAKE_CXX_COMPILER_ID})
message("compiler version:" ${CMAKE_CXX_COMPILER_VERSION})
```

## CMake Command Line Parameters

-S `<path to source directory>`   
-B `<path to build directory>`  
-D `<cache variable>=<value>`  
-G `<generator-name>`   

## CMake Variables, Cache Variables and Options

### Variables
Setting variables:
```
SET(xxx value)
```
Reading variables:
```
${XXX}
```
Environment variables:
```
$ENV{HOME}  
```
Paths may contain a space at any time and should always be quoted when they are a variable (never write ${VAR_PATH}, always should be "${VAR_PATH}").
```
$ENV{PATH}  
```

You can set a variable in the scope immediately above your current one with PARENT_SCOPE at the end. Let say you have the followings:
```
root
├── src
│   └── CMakeLists.txt
└── CMakeLists.txt
```
in your root CMake 
```
set( BAR "Bar from root." )
```
In your src CMake
```
set( BAR "Bar from src." ) #<-- set in this scope
set( BAR ${BAR} PARENT_SCOPE ) #<-- set in the parent scope too
```
Then in your root CMake, put this and observe the changes in the value of  `BAR`:
```
MESSAGE( STATUS "root: " ${BAR} )
add_subdirectory("${PROJECT_SOURCE_DIR}/src")
MESSAGE( STATUS "root: " ${BAR} )  
```


### Cache Variables
If you want to set a variable from the command line, CMake offers a variable cache. For example you can define `MY_LIBRARY_VERSION`. The syntax for declaring a variable and setting it if it is not already set is:
```
set(MY_CACHE_VARIABLE "VALUE" CACHE STRING "Description")
```
Then you can sent the value via command line:
```
cmake -DMY_CACHE_VARIABLE=2.3.6.9
```
If you use keyword force, user can not override your value:
```
set(MY_LIBRARY_VERSION_MAJOR 1 CACHE STRING "major version" FORCE)
```
These are common CMake options to most packages:

`-DCMAKE_BUILD_TYPE` Pick from Release, RelWithDebInfo, Debug, or sometimes more.  
`-DCMAKE_INSTALL_PREFIX` The location to install to. System install on UNIX would often be `/usr/local` (the default), user directories are often `~/.local`, or you can pick a folder.  
`-DBUILD_SHARED_LIBS` You can set this ON or OFF to control the default for shared libraries (the author can pick one vs. the other explicitly instead of using the default, though)  
`-DBUILD_TESTING` This is a common name for enabling tests, not all packages use it, though, sometimes with good reason.  


### Options
Options provide an option for the user to select as ON or OFF.
```
option(PACKAGE_TESTS "Build the tests" ON)
```
## Setting Important Variables
- C++17 support
```
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
```

-  C++20 support
```
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_BUILD_TYPE debug)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++2a")
```


-  Position independent code

Globally:
```
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
```
Explicitly turn it ON (or OFF) for a target:

```
set_target_properties(lib1 PROPERTIES POSITION_INDEPENDENT_CODE ON)
```

-  Turning warning into errors
```
if(MSVC)
  # Force to always compile with W4
  if(CMAKE_CXX_FLAGS MATCHES "/W[0-4]")
    string(REGEX REPLACE "/W[0-4]" "/W4" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
  else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
  endif()
elseif(CMAKE_COMPILER_IS_GNUCC OR CMAKE_COMPILER_IS_GNUCXX)
  # Update if necessary
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-long-long -pedantic")
endif()
```

-  Finding Memory leaking, Stack and Heap overflow
```
set(CMAKE_CXX_FLAGS "-fsanitize=address ${CMAKE_CXX_FLAGS}")
set(CMAKE_CXX_FLAGS "-fno-omit-frame-pointer ${CMAKE_CXX_FLAGS}")
```

-  Turning off the warning from unused variables
```
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-variable")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-unused-private-field")
```

-  Setting build type
```
set(CMAKE_BUILD_TYPE DEBUG|RELEASE)
```


-  List Of Important Variables

```
MESSAGE( STATUS "PROJECT_NAME: " ${PROJECT_NAME} )  
MESSAGE( STATUS "PROJECT_VERSION: " ${PROJECT_VERSION} )  
MESSAGE( STATUS "BUILD_SHARED_LIBS: " ${BUILD_SHARED_LIBS} )  
MESSAGE( STATUS "BUILD_TESTING: " ${BUILD_TESTING} )  
MESSAGE( STATUS "CMAKE_C_COMPILER: " ${CMAKE_C_COMPILER} )  
MESSAGE( STATUS "CMAKE_CXX_COMPILER: " ${CMAKE_CXX_COMPILER} )  
MESSAGE( STATUS "CMAKE_BUILD_TYPE: " ${CMAKE_BUILD_TYPE} )
MESSAGE( STATUS "CMAKE_INSTALL_PREFIX: " ${CMAKE_INSTALL_PREFIX} )
MESSAGE( STATUS "PROJECT_SOURCE_DIR: " ${PROJECT_SOURCE_DIR} )
MESSAGE( STATUS "CMAKE_SOURCE_DIR: " ${CMAKE_SOURCE_DIR} )
MESSAGE( STATUS "CMAKE_CURRENT_SOURCE_DIR: " ${CMAKE_CURRENT_SOURCE_DIR} )
MESSAGE( STATUS "PROJECT_BINARY_DIR: " ${PROJECT_BINARY_DIR} )
MESSAGE( STATUS "CMAKE_CURRENT_BINARY_DIR: " ${CMAKE_CURRENT_BINARY_DIR} )
MESSAGE( STATUS "CMAKE_BINARY_DIR: " ${CMAKE_BINARY_DIR} )
MESSAGE( STATUS "EXECUTABLE_OUTPUT_PATH: " "${EXECUTABLE_OUTPUT_PATH}" )
MESSAGE( STATUS "LIBRARY_OUTPUT_PATH:     " "${LIBRARY_OUTPUT_PATH}" )
MESSAGE( STATUS "CMAKE_MODULE_PATH: " "${CMAKE_MODULE_PATH}" )
MESSAGE( STATUS "CMAKE_INCLUDE_PATH: " "${CMAKE_INCLUDE_PATH}" )
MESSAGE( STATUS "CMAKE_PREFIX_PATH: " "${CMAKE_PREFIX_PATH}" )
MESSAGE( STATUS "CMAKE_LIBRARY_PATH: " "${CMAKE_LIBRARY_PATH}" )
MESSAGE( STATUS "CMAKE_SYSTEM_LIBRARY_PATH: " "${CMAKE_SYSTEM_LIBRARY_PATH}" )
MESSAGE( STATUS "CMAKE_CTEST_COMMAND: " ${CMAKE_CTEST_COMMAND} )  
MESSAGE( STATUS "CMAKE_GENERATOR: " ${CMAKE_GENERATOR} )  
```

## Properties
The other way CMake stores information is in properties. This is like a variable, but it is attached to some other item, like a directory or a target. There are two ways to set properties:
The first form is more general, 
```
set_property(TARGET TargetName PROPERTY CXX_STANDARD 11)
```

The second is a shortcut for setting several properties on one target
`set_target_properties(TargetName PROPERTIES CXX_STANDARD 11)`


## CMake Generators
A CMake Generator is responsible for writing the input files for a native build system. Use `-G` option to specify the generator for a new build tree. 

1. Makefile Generators

For instance: `cmake -G"Unix Makefiles" -S ..`

2. Ninja Generators: `Ninja` and `Ninja Multi-Config` (First install the packages: `sudo apt install ninja-build`)

For instance: `cmake -G Ninja -S ..`

3. IDE Build Tool Generators: for instance `Visual Studio 17 2022` or `Visual Studio 16 2019`

Visual Studio 2017

For instance: `cmake -G"Visual Studio 15" ..`

You can check the generator used to build the project via `CMAKE_GENERATOR`.

```
MESSAGE( STATUS "CMAKE_GENERATOR: " ${CMAKE_GENERATOR} )
```
The value of this variable should **never** be modified by project code. on use `-G` option to set it.

## setting build type with DCMAKE_BUILD_TYPE and Ninja Multi-Config

The Ninja build system in relation to CMake's `-DCMAKE_BUILD_TYPE` and `--config` options has a slightly complex story due to the introduction of the Ninja Multi-Config generator in newer CMake versions. Let's break it down:

1. **Traditional Ninja (Single-Config)**:
    - **Configuration Phase**: You use `-DCMAKE_BUILD_TYPE` to specify the build type (e.g., Debug, Release).
      ```bash
      cmake -G "Ninja" -DCMAKE_BUILD_TYPE=Release -S [path to source] -B build
      ```
    - **Build Phase**: When you run `ninja`, it will build the `Release` configuration (based on the earlier example).
    - `--config` is not relevant for this variant of Ninja because it's a single-config generator. There's only one build type configured per build directory.

2. **Ninja Multi-Config (Experimental as of CMake 3.17 and later)**:
    - **Configuration Phase**: You don't specify a build type with `-DCMAKE_BUILD_TYPE` during configuration. Instead, you'd just configure with:
      ```bash
      cmake -G "Ninja Multi-Config" -S [path to source] -B build
      ```
    - **Build Phase**: You choose the build configuration when invoking the build. This can be done in a couple of ways:
      - Using the `cmake --build` command and the `--config` option:
        ```bash
        cmake --build build --config Release
        ```

    - With Ninja Multi-Config, you can switch between different configurations (like Debug, Release) in the same build directory without reconfiguring.

## VScode and Ninja Multi-Config

Add the following settings to your **settings.json** located at `/home/$USER/.config/Code/User` or press `ctrl+shift+p` to open it

```
{
  "cmake.configureSettings": {
    "CMAKE_EXPORT_COMPILE_COMMANDS": "YES"
  },
  "cmake.generator": "Ninja Multi-Config"
}
```


## CMakePresets.json and CMakeUserPresets.json

`CMakePresets.json` and `CMakeUserPresets.json` both have exactly the same format, `CMakePresets.json` is meant to specify project-wide build details, while `CMakeUserPresets.json` is meant for developers to specify their own local build details. meaning, if a project is using Git, `CMakePresets.json` may be tracked, and `CMakeUserPresets.json` should be added to the `.gitignore`.


Here is a simple example of a `CMakePresets.json` file:
```
{
  "version": 3,
  "cmakeMinimumRequired": {
    "major": 3,
    "minor": 19,
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
        "CMAKE_POLICY_DEFAULT_CMP0048": "NEW",
        "CMAKE_CONFIGURATION_TYPES": "Debug;Release;RelWithDebInfo;MinSizeRel"
      }
    }
  ],
  "buildPresets": [
    {
      "name": "ninja-multi-debug",
      "configurePreset": "ninja-multi",
      "configuration": "Debug"
    },
    {
      "name": "ninja-multi-release",
      "configurePreset": "ninja-multi",
      "configuration": "Release"
    },
    {
      "name": "ninja-multi-relwithdebinfo",
      "configurePreset": "ninja-multi",
      "configuration": "RelWithDebInfo"
    },
    {
      "name": "ninja-multi-minsizerel",
      "configurePreset": "ninja-multi",
      "configuration": "MinSizeRel"
    }
  ]
}
```


To configure and build a specific configuration using these presets, you would use commands similar to the following, replacing `<preset-name>` with the desired build preset name (e.g., ninja-multi-debug):

```
cmake --preset ninja-multi
cmake --build --preset <preset-name>
```

If you prefer `preset` use:

```
cmake --preset ninja-multi
```
and 

```
cmake --build --preset ninja-multi-debug
```
or 
```
cmake --build --preset ninja-multi-release
```

## Building Project

to build (you can use `-v` for verbose builds and `-j N` for parallel builds on N cores) :

```
cmake --build . -v -j 8
```

build an specific target from your CMakeLists, i.e.
```
add_executable(filesystem src/filesystem.cpp)
target_link_libraries(filesystem)
```
can be build by:
```
cmake --build . --target filesystem
```
```
cmake --build . --target test
```
build docs
```
cmake --build . --target docs
```
build and install (the installation path would be `CMAKE_INSTALL_PREFIX`)
```
cmake --build . --target all install --config Release
```

to build Ninja Multi-Config:
```
cmake --build . --config <Config>
```


read more [here](https://cmake.org/cmake/help/latest/generator/Ninja%20Multi-Config.html)


`cmake -S .. -B. && cmake  --build .  --parallel` or `cmake -S .. -B. && cmake  --build . -j`


## Visualising Dependency Graph:

--graphviz=[dependency graph outfile]  
--trace-source=CMakeLists.txt  

```
cmake   --graphviz=viz.dot  --trace-source=CMakeLists.txt
dot -Tsvg viz.dot -o viz.svg
```
## Listing All Variables With Description:
```
cmake -LAH   ../
```
## Watch a Variable
In CMake, `variable_watch` is a feature that allows you to register a callback function that is called whenever the value of a CMake variable changes. This can be useful for performing certain actions whenever a variable is modified, such as updating other variables or files.

Here's an example of how to use variable_watch in CMake:

```
# Define a variable "MY_VAR" with an initial value of "foo"
set(MY_VAR "foo")

# Define a function to be called when the variable changes
function(on_my_var_change varname varvalue)
    message("The value of variable ${varname} has changed to ${varvalue}")
endfunction()

# Register the callback function to be called whenever "MY_VAR" changes
variable_watch(MY_VAR on_my_var_change)

# Change the value of "MY_VAR" to "bar"
set(MY_VAR "bar")
```

## Colorized Message
```
if(NOT WIN32)
  string(ASCII 27 Esc)
  set(ColourReset "${Esc}[m")
  set(ColourBold  "${Esc}[1m")
  set(Red         "${Esc}[31m")
  set(Green       "${Esc}[32m")
  set(Yellow      "${Esc}[33m")
  set(Blue        "${Esc}[34m")
  set(Magenta     "${Esc}[35m")
  set(Cyan        "${Esc}[36m")
  set(White       "${Esc}[37m")
  set(BoldRed     "${Esc}[1;31m")
  set(BoldGreen   "${Esc}[1;32m")
  set(BoldYellow  "${Esc}[1;33m")
  set(BoldBlue    "${Esc}[1;34m")
  set(BoldMagenta "${Esc}[1;35m")
  set(BoldCyan    "${Esc}[1;36m")
  set(BoldWhite   "${Esc}[1;37m")
endif()

message("This is normal")
message("${Red}This is Red${ColourReset}")
message("${Green}This is Green${ColourReset}")
message("${Yellow}This is Yellow${ColourReset}")
message("${Blue}This is Blue${ColourReset}")
message("${Magenta}This is Magenta${ColourReset}")
message("${Cyan}This is Cyan${ColourReset}")
message("${White}This is White${ColourReset}")
message("${BoldRed}This is BoldRed${ColourReset}")
message("${BoldGreen}This is BoldGreen${ColourReset}")
message("${BoldYellow}This is BoldYellow${ColourReset}")
message("${BoldBlue}This is BoldBlue${ColourReset}")
message("${BoldMagenta}This is BoldMagenta${ColourReset}")
message("${BoldCyan}This is BoldCyan${ColourReset}")
message("${BoldWhite}This is BoldWhite\n\n${ColourReset}")
```



Refs: [1](https://stackoverflow.com/questions/18968979/how-to-make-colorized-message-with-cmake)

## Scripting in CMake

```
if() ... else()/elseif() ... endif()

option(TESTING "Enable testing" OFF)
if(testing_enabled)
	add_subdirectory(tests)
endif()


if(IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/src/temp")
	add_subdirectory(src/temp)
endif()

foreach() ... endforeach()
```

writing conditions, checking string, values and 

```
if (CMAKE_CXX_COMPILER_ID STREQUAL "Clang" AND ${CMAKE_CXX_COMPILER_VERSION} GREATER_EQUAL 14 )
	add_executable(foo src/foo.cpp)
	target_link_libraries(foo)
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "GNU" AND ${CMAKE_CXX_COMPILER_VERSION} GREATER_EQUAL 13)
	add_executable(foo src/foo.cpp)
	target_link_libraries(foo)
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC" AND ${CMAKE_CXX_COMPILER_VERSION} GREATER_EQUAL 1900)
	add_executable(foo src/foo.cpp)
	target_link_libraries(foo)
endif()
```

Refs: [1](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html)

## Built-in Commands And Functions in CMake

Always use lowercase function names. Always user lower case. Upper case is for variables.
The languages are C, CXX, Fortran, ASM, CUDA (CMake 3.8+), CSharp (3.8+), and SWIFT.

```
cmake_minimum_required(VERSION 3.1)  
project(my-cmake-project VERSION 1.2.3.4 DESCRIPTION "An example project with CMake" LANGUAGES CXX)  
```

**add_subdirectory()**

Add a subdirectory to the build. The binary_dir specifies the directory in which to place the output files. The CMakeLists.txt in the added subdirectory will be called and executed.
```
add_subdirectory(src)
add_subdirectory(src binary_dir)
```
**include()**
Load and run CMake code from the file given.
```
include(someother.cmake)
```
**include_directories()**
tell cmake where to look for *.h files
```
include_directories(${PROJECT_SOURCE_DIR}/include)
```

**target_include_directories()**

`include_directories(given_include_dir_path)` affects all the targets in its CMakeLists, as well as those in all sub directories added after the point of its call. They would have access to "given_include_dir_path" for including headers.

`target_include_directories(target <INTERFACE|PUBLIC|PRIVATE> target_include_directory_path)` would add an include directory for a specific target. 
The target must have been created by a command such as `add_executable()` or `add_library()`. The reason that we might use both of them is the following:
You should declare your public API of your library for the third-party applications, so you place them under `<project_root/include/project_name>`. You might have some headers that are being used only by your application and you don't need (or want) to give them to the public, so you place them under your source directory and use target_include_directories() to make them accessible by your target. Notice that, private headers should not be installed.

#### PUBLIC, PRIVATE, and INTERFACE
Other target could be compiled against your targets and they might need to access the headers that you have 
used in your target, by declaring them as `PUBLIC` other targets can have access to those include directories that you added
to your target and by using `PRIVATE` those include directories are nly available for your target. For example:

`target_include_directories(A PRIVATE ${Boost_INCLUDE_DIRS})` if you only use those Boost headers inside your source files (.cpp) or private header files (.h).

`target_include_directories(A PUBLIC ${Boost_INCLUDE_DIRS})` if you use those Boost headers in your public header files, which are included BOTH in some of A's source files and might also be included in any other client of your A library.

**add_library()**
create library "libtools"
```
add_library(tools STATIC|SHARED|MODULE src/tools.cpp)
```
[How to add static/ shared library](Creating_Librares)

[How to link your static/ shared library](Linking_to_Libraries)

**add_executable()**
Adds an executable target called <name> to be built from the source files listed. 

```
add_executable(main [WIN32]  src/tools_main.cpp src/tools_lib.cpp)
```
**target_link_libraries()**
Tell the linker to bind these objects together.
```
target_link_libraries(main tools)
```

**mark_as_advanced()**
Mark the named cached variables as advanced. An advanced variable will not be displayed in any of the cmake GUIs unless the show advanced option is on, for instance to keep **CACHE**  clean:
```
mark_as_advanced(
    BUILD_GMOCK BUILD_GTEST BUILD_SHARED_LIBS
    gmock_build_tests gtest_build_samples gtest_build_tests
    gtest_disable_pthreads gtest_force_shared_crt gtest_hide_internal_symbols
)
```

**install()**

When you call the `make install`  or `cmake --build . --target install` this command will be executed.
In the following example, we export the properties of our TARGETS into an EXPORT callled FooTargets, so FooTargets know where we 
put the lib, dll, share dobject etc (On windows the dll will be in bin directory and .lib in the lib directory, On linux .a and .so will be in lib directory).

```
install( TARGETS Foo EXPORT FooTargets
LIBRARY DESTINATION lib
ARCHIVE DESTINATION lib
RUNTIME DESTINATION bin
INCLUDES DESTINATION include)
```
But this only stores this information in an object named "FooTargets". The following will write the data that our EXPORT (FooTargets) contains into disk:
```
install(EXPORT FooTargets
FILE FooTargets.cmake
NAMESPACE FOO::
DESTINATION lib/cmake/Foo
)
```

## Semantic Versioning

The scheme is based on a three-part version number, in the format of `MAJOR.MINOR.PATCH`:

- MAJOR version number is incremented when there are significant changes to the software that are not backward compatible with previous versions. 

- MINOR version number is incremented when new features or functionality are added to the software in a backward-compatible manner. 

- PATCH version number is incremented when bug fixes or small changes are made to the software, also in a backward-compatible manner.

Each element MUST increase numerically, For instance: `1.9.0` -> `1.10.0` -> `1.11.0`.

If we release a pre-release version of the software before the final release, we can add a pre-release identifier to the version number, such as 1.3.0-alpha.1, to indicate that it is not the final release and may contain bugs.

If we need to add build metadata, such as the build date or build number, we can add it as a build metadata identifier after the version number, such as `1.2.3+20230507`.


For instance: `1.0.0` -> `1.0.1` -> `1.0.2`-> `2.0.0+25` -> `2.0.0+32` -> `2.0.0`.

Refs: [1](https://blogs.stonesteps.ca/1/p/80), [2](https://semver.org/)


so to set the version from git tags into the software:

```
execute_process(COMMAND "git" "describe" 
                OUTPUT_VARIABLE GIT_DESCRIBE_VERSION_TAG
                OUTPUT_STRIP_TRAILING_WHITESPACE)

target_compile_definitions(main PRIVATE APP_VERSION="${GIT_DESCRIBE_VERSION_TAG}")
```
now in the cpp file:

```cpp
std::string appVersion()
{
    return std::string(APP_VERSION);
}
```
### GitHub Automatic Releases From tags and  Release Management 

Refs: [1](https://github.com/marketplace/actions/automatic-releases), [2](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository)

## Connecting CMake With Your Code

### Reading From CMake Into Your Files
`configure_file` command copies the content of the first parameter (Version.h.in) to second parameter (Version) and substitute all CMake variables it finds. If you want to avoid replacing existing `${}` syntax in your input file, use the `@ONLY` keyword. Passing `@ONLY` option to configure_file forces CMake to not touch `${...}` expressions but substitute only `@VAR@ `ones.

```
configure_file(<input> <output> [@ONLY])
configure_file ( "${PROJECT_SOURCE_DIR}/include/project/Version.h.in"  "${PROJECT_BINARY_DIR}/include/project/Version.h")
```
Content of Version.h.in
```
#pragma once

#define MY_VERSION_MAJOR @PROJECT_VERSION_MAJOR@
#define MY_VERSION_MINOR @PROJECT_VERSION_MINOR@
#define MY_VERSION_PATCH @PROJECT_VERSION_PATCH@
#define MY_VERSION_TWEAK @PROJECT_VERSION_TWEAK@
#define MY_VERSION "@PROJECT_VERSION@"
```
The content of "Version.h" will be 

```
#pragma once

#define MY_VERSION_MAJOR 1
#define MY_VERSION_MINOR 2
#define MY_VERSION_PATCH 3
#define MY_VERSION_TWEAK 4
#define MY_VERSION "1.2.3.4"
```

### CMake Reading From Your Files

```
file(READ "${CMAKE_CURRENT_SOURCE_DIR}/include/project/Version.hpp" VERSION)

string(REGEX MATCH "VERSION_MAJOR ([0-9]*)" _ ${VERSION})
set(VERSION_MAJOR ${CMAKE_MATCH_1})

string(REGEX MATCH "VERSION_MINOR ([0-9]*)" _ ${VERSION})
set(VERSION_MINOR ${CMAKE_MATCH_1})

string(REGEX MATCH "VERSION_PATCH ([0-9]*)" _ ${VERSION})
set(VERSION_PATCH ${CMAKE_MATCH_1})

message(STATUS "VERSION: ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH}")

```



## Testing with CMake
You can use **ctest** to test your **unittests**. In your main CMakelist.txt:

```
option(BUILD_TESTING "this will automatically enable testing" ON)
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME AND BUILD_TESTING)
    include(CTest)
    add_subdirectory(tests)
endif()

```
in your **tests** directory:
```
add_executable(test1 test1.cpp)
target_link_libraries(test1)
add_test(mytester test1)
```
The content of test1.cpp
```
int main(int argc, char ** argv)
{
    return 0;
}
```

Then in the build directory after building call **ctest**.

you can store your test configuration in file like "build.cmake" 
```
set(CTEST_SOURCE_DIRECTORY "/source")
set(CTEST_BINARY_DIRECTORY "/binary")
set(ENV{CXXFLAGS "--coverage"})
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
Execute it by:
```
ctest -S build.cmake
```
And then send the results to a dashboard server.

### GoogleTest
If you want to use google test for performing your unit test, first let's add it as submodule into your extern/googletest directory in your project:

```
git submodule add  https://github.com/google/googletest.git extern/googletest
```
Then, in your main CMakeLists.txt:
```
option(PACKAGE_TESTS "Build the tests" ON)
if(PACKAGE_TESTS)
    enable_testing()
    include(GoogleTest)
    add_subdirectory(tests)
endif()
```
Now, in the CMakeLists of your tests directory:
```
add_subdirectory("${PROJECT_SOURCE_DIR}/extern/googletest" "extern/googletest")
set(INSTALL_GTEST OFF)
add_executable(footest foo.cpp)
target_link_libraries(footest gtest_main)
gtest_discover_tests(footest)
```
The extra path here is needed to correct the build path because we are calling it from a subdirectory. 
Then copy this under `foo.cpp`:
```
#include "gtest/gtest.h"
TEST(Foo, Sum)
{
  EXPECT_EQ(2, 1 + 1);
}
```
And run it with:
```
ctest
```
### Download method
```
cmake_minimum_required(VERSION 3.10)
project(MyProject CXX)
list(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

enable_testing() # Must be in main file

include(AddGoogleTest) # Could be in /tests/CMakeLists.txt
add_executable(SimpleTest SimpleTest.cu)
add_gtest(SimpleTest)

target_link_libraries(SimpleTest gtest gmock gtest_main)
add_test(SimpleTest SimpleTest)
```
### FetchContent

```
include(FetchContent)

FetchContent_Declare(
  googletest
  GIT_REPOSITORY https://github.com/google/googletest.git
  GIT_TAG        release-1.8.0
  GIT_PROGRESS TRUE
)

FetchContent_MakeAvailable(googletest)


FetchContent_GetProperties(googletest)
if(NOT googletest_POPULATED)
  FetchContent_Populate(googletest)
  add_subdirectory(${googletest_SOURCE_DIR} ${googletest_BINARY_DIR})
endif()
```

## Exporting Your Project

There are 3 ways to access your project from another project:  
### 1. Adding Subproject
Adding your project with add_subdirectory().  For small and header only libraries, you can just use add_subdirectory() and include the entire  porject.
You can use **CMAKE_CURRENT_SOURCE_DIR** instead of **PROJECT_SOURCE_DIR** and 
```
if(CMAKE_PROJECT_NAME STREQUAL PROJECT_NAME)
...
endif()
```

### 2. Exporting Build Directory of Your Project  
To use the build directory of one project in another project, you will need to export targets.
```
export(TARGETS taget1 target2  FILE MyLibTargets.cmake)
```

This puts the targets you have listed into a file in the build directory. Now, to allow CMake to find this package, export the package into the` $HOME/.cmake/packages` folder:
```
set(CMAKE_EXPORT_PACKAGE_REGISTRY ON)
export(PACKAGE MyLib)
```
Now, if you find_package(MyLib), CMake can find the build folder.


### 3. Installing Your Project And Calling find_package()
Let say you have the following [project](Exporting):
```
Exporting/
├── CMakeLists.txt
├── include
│   └── my_header.h
├── MyProjectConfig.cmake.in
└── src
    ├── main.cpp
    └── my_source_file.cpp
```

The content of `Exporting/MyProjectConfig.cmake.in`:

```
# MyProjectConfig.cmake.in

@PACKAGE_INIT@

include("${CMAKE_CURRENT_LIST_DIR}/MyProjectTargets.cmake")

set_and_check(MY_PROJECT_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include")
```

The content of `CMakeLists.txt`:


```cmake
cmake_minimum_required(VERSION 3.0)
project(MyProject VERSION 1.0)

# Add source files
set(SOURCES
    src/my_source_file.cpp
    src/main.cpp
)

# Add header files
set(HEADERS
    include/my_header.h
)

# Define the library
add_library(my_library ${SOURCES})

# Specify include directories
target_include_directories(my_library
    PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

# Install targets and files
install(TARGETS my_library
    EXPORT MyProjectTargets
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    RUNTIME DESTINATION bin
    INCLUDES DESTINATION include
)
install(FILES ${HEADERS} DESTINATION include)

# Export targets
install(EXPORT MyProjectTargets
    FILE MyProjectTargets.cmake
    NAMESPACE MyProject::
    DESTINATION lib/cmake/MyProject
)

# Configure package config file
include(CMakePackageConfigHelpers)
set(CONFIG_INSTALL_DIR lib/cmake/MyProject)
configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/MyProjectConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
    INSTALL_DESTINATION ${CONFIG_INSTALL_DIR}
)
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

# Install package config files
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
    DESTINATION ${CONFIG_INSTALL_DIR}
)

```

In this CMakeLists.txt file:

- The `configure_package_config_file` command generates the package configuration file `MyProjectConfig.cmake`.
- The `write_basic_package_version_file` command generates the version file `MyProjectConfigVersion.cmake`.
- Both generated files are installed into the appropriate location (`lib/cmake/MyProject`).
- Ensure that you have a template file `MyProjectConfig.cmake.in` which specifies how your library should be configured when imported by other projects. This file typically includes commands like `include(Targets)` to include the exported targets from `MyProjectTargets.cmake`.



### Explanation of MyProjectConfig.cmake.in

The `MyProjectConfig.cmake.in` file is a template used to generate the final `MyProjectConfig.cmake` file during the installation process. It typically includes commands and variables needed to configure the package for use in other CMake projects.

Here's a breakdown of the content of `MyProjectConfig.cmake.in`:

1. `@PACKAGE_INIT@`: This is a placeholder that will be replaced with initialization code when the final `MyProjectConfig.cmake` file is generated. The initialization code typically sets up variables and macros needed for the package.

2. `include("${CMAKE_CURRENT_LIST_DIR}/MyProjectTargets.cmake")`: This line includes the `MyProjectTargets.cmake` file, which defines the targets (e.g., libraries, executables) provided by `MyProject` and their properties.

3. `set_and_check(MY_PROJECT_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include")`: This line sets the `MY_PROJECT_INCLUDE_DIRS` variable to the path where the header files of `MyProject` are installed. `${CMAKE_INSTALL_PREFIX}` is a CMake variable representing the installation prefix specified during installation. This line ensures that consumers of `MyProject` can include its header files using this variable.

Here are a few examples of what else you can include in `MyProjectConfig.cmake.in`:

- **Setting Variables**: You can set variables to provide configuration options for consumers. For example:

  ```cmake
  set(MY_PROJECT_USE_FEATURE_XYZ ON)
  ```

- **Configuring Compiler Flags**: You can configure compiler flags or options that are specific to `MyProject`. For example:

  ```cmake
  if(MSVC)
      set(MY_PROJECT_MSVC_FLAGS "/W3 /EHsc")
  else()
      set(MY_PROJECT_GCC_FLAGS "-Wall -Wextra")
  endif()
  ```

- **Defining Macros**: You can define macros that can be used by consumers or within the project itself. For example:

  ```cmake
  add_definitions(-DMY_PROJECT_USE_SOME_FEATURE)
  ```

- **Setting Dependencies**: If `MyProject` depends on other libraries, you can set variables or provide information about these dependencies. For example:

  ```cmake
  set(MY_PROJECT_REQUIRED_LIBRARIES SomeOtherLibrary)
  ```

These examples demonstrate how you can customize `MyProjectConfig.cmake.in` to provide additional information and configurations for consumers of `MyProject`. When consumers use `find_package(MyProject)`, the generated `MyProjectConfig.cmake` file will contain this information, making it easier for them to use `MyProject` in their projects.






Once you exported and installed and your project, you can call `find_package`. find_package() has the following parameter:  
`find_package(<package> [version] [EXACT] [QUIET] [MODULE] [REQUIRED] [[COMPONENTS] )` finds and loads settings from an external project.

The `QUIET` option disables messages if the package cannot be found.  
The `REQUIRED` option stops processing with an error message if the package cannot be found.  
`<package>_FOUND`  will be set to indicate whether the package was found.  
The `version` argument requests a version with which the package found should be compatible (format is major[.minor[.patch[.tweak]]]). 	



Now run:

```
cmake -G "Ninja Multi-Config" -S . -B build -DCMAKE_PREFIX_PATH=~/usr -DCMAKE_INSTALL_PREFIX=~/usr
cmake --build build --config Debug
cmake --install build --config Debug
```

After that you can use in [Consumer](ExportingConsumer)
```
cmake_minimum_required(VERSION 3.0)
project(ConsumerProject)

# Find MyProject
find_package(MyProject REQUIRED)

# Add source files
set(SOURCES
    src/consumer_source.cpp
)

# Add executable
add_executable(consumer_executable ${SOURCES})

# Link with MyProject library
target_link_libraries(consumer_executable PRIVATE MyProject::my_library)

# Specify include directories
target_include_directories(consumer_executable PRIVATE ${MyProject_INCLUDE_DIRS})
```




Command `find_package` has two modes: `Module` mode and `Config` mode. 

### Find\<package\>.cmake
Module mode will look for `Find<package>.cmake`in `CMAKE_MODULE_PATH`. This should be used when a project has no CMake support. Usually you create **Find<package>.cmake** for a library and put under **cmake** directory in your porject. Then you should set the set the **CMAKE_MODULE_PATH** pointing to that.
```
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_LIST_DIR}/cmake")
```

### \<package\>Config.cmake
In config mode it will look for `<name>Config.cmake` or `<lower-case-name>-config.cmake` in `<package>_DIR`. First set the `<package>_DIR`, for example:

```
cmake_minimum_required(VERSION 3.1)
set(MyPack_DIR "$ENV{HOME}/usr/lib/cmake/MyPack")
project(MyPack_user)
set(MyPack_DIR "X:/install/lib/cmake/MyPack")
find_package(MyPack CONFIG REQUIRED )
message("MyPack_FOUND: " ${MyPack_FOUND} )
message("MyPack_VERSION: " ${MyPack_VERSION} )
```

## How to find CMake from arbitrary installed locations

Here I gathered several examples:
- PCL point cloud
```
set(PCL_DIR "$ENV{HOME}/usr/share/pcl-1.8/")
```

- OpenCV
```
set(OpenCV_DIR "$ENV{HOME}/usr/share/OpenCV/")
```

- glog
```
cmake  -Dglog_DIR=~/usr/lib/cmake/glog/
```

- VTK
```
set(VTK_DIR "$ENV{HOME}/usr/lib/cmake/vtk-9.2/")
```


## Using pkgconfig (.pc files)
I usually install my program in my home directory therefore everything goes into
```
<home>/usr/lib/
<home>/usr/include
<home>/usr/bin
```
So first let's check the pkg-config in terminal:

```
PKG_CONFIG_PATH=~/usr/lib/pkgconfig:${PKG_CONFIG_PATH}
export PKG_CONFIG_PATH
printenv PKG_CONFIG_PATH

pkg-config --cflags flann
pkg-config --libs flann
pkg-config --modversion flann
```

- Flann
```
pkg_search_module(FLANN REQUIRED flann)
if(${FLANN_FOUND})
    MESSAGE("FLANN_FOUND:" ${FLANN_FOUND})
    MESSAGE("FLANN_VERSION:" ${FLANN_VERSION})
    MESSAGE("FLANN_LIBRARIES:" ${FLANN_LIBRARIES})
    MESSAGE("FLANN_INCLUDE_DIRS:" ${FLANN_INCLUDEDIR})
    MESSAGE("FLANN_LIBRARY_DIRS:" ${FLANN_LIBDIR})
    INCLUDE_DIRECTORIES(${FLANN_INCLUDEDIR})
    LINK_DIRECTORIES(${FLANN_LIBDIR})
    #ADD_EXECUTABLE(main src/main.cpp)
    #TARGET_LINK_LIBRARIES(main ${FLANN_LIBRARIES})
endif()
```

- TinyXML2
```
SET(ENV{PKG_CONFIG_PATH} "$ENV{HOME}/usr/lib/pkgconfig:" $ENV{PKG_CONFIG_PATH})
MESSAGE("PKG_CONFIG_PATH:" $ENV{PKG_CONFIG_PATH})
find_package(PkgConfig)
pkg_check_modules(TINYXML2 tinyxml2)
if(${TINYXML2_FOUND})
    MESSAGE("TINYXML2_FOUND:" ${TINYXML2_FOUND})
    MESSAGE("TINYXML2_VERSION:" ${TINYXML2_VERSION})
    MESSAGE("TINYXML2_LIBRARIES:" ${TINYXML2_LIBRARIES})
    MESSAGE("TINYXML2_INCLUDE_DIRS:" ${TINYXML2_INCLUDE_DIRS})
    MESSAGE("TINYXML2_LIBRARY_DIRS:" ${TINYXML2_LIBRARY_DIRS})
    INCLUDE_DIRECTORIES(${TINYXML2_INCLUDE_DIRS})
    LINK_DIRECTORIES(${TINYXML2_LIBRARY_DIRS})
    ADD_EXECUTABLE(tinyxml2_demo src/third_party_tools/xml/tinyxml2/tinyxml2_demo.cpp)
    TARGET_LINK_LIBRARIES(tinyxml2_demo ${TINYXML2_LIBRARIES})
endif()
```
- yaml-cpp
```
SET(yaml-cpp_DIR "$ENV{HOME}/usr/share/cmake/yaml-cpp")  
FIND_PACKAGE(yaml-cpp)  
IF(${yaml-cpp_FOUND})  
    MESSAGE("yaml-cpp_FOUND:" ${yaml-cpp_FOUND})  
    MESSAGE("yaml-cpp_VERSION:" ${yaml-cpp_VERSION})  
    ADD_EXECUTABLE(yaml-cpp_example src/third_party_tools/yaml/yaml-cpp/yaml-cpp_example.cpp )  
    TARGET_LINK_LIBRARIES(yaml-cpp_example yaml-cpp)  
ENDIF()
```

- Google Benchmark
```
pkg_check_modules(BENCHMARK benchmark)
if(${BENCHMARK_FOUND})
    MESSAGE("BENCHMARK_FOUND:" ${BENCHMARK_FOUND})
    MESSAGE("BENCHMARK_VERSION:" ${BENCHMARK_VERSION})
    MESSAGE("BENCHMARK_LIBRARIES:" ${BENCHMARK_LIBRARIES})
    MESSAGE("BENCHMARK_INCLUDE_DIRS:" ${BENCHMARK_INCLUDE_DIRS})
    MESSAGE("BENCHMARK_LIBRARY_DIRS:" ${BENCHMARK_LIBRARY_DIRS})
    INCLUDE_DIRECTORIES(${TINYXML2_INCLUDE_DIRS})
    LINK_DIRECTORIES(${TINYXML2_LIBRARY_DIRS})
    ADD_EXECUTABLE(benchmark_demo src/third_party_tools/benchmark/benchmark_demo.cpp)
    TARGET_LINK_LIBRARIES(benchmark_demo ${BENCHMARK_LIBRARIES} pthread)
endif()
```

- gRPC
```
find_package(PkgConfig REQUIRED)
#for c
pkg_search_module(GRPC REQUIRED grpc)

#for c++
pkg_search_module(GRPCPP REQUIRED grpc++)

if(${GRPCPP_FOUND})
    MESSAGE("GRPCPP_FOUND:" ${GRPCPP_FOUND})
    MESSAGE("GRPCPP_VERSION:" ${GRPCPP_VERSION})
    MESSAGE("GRPCPP_LIBRARIES:" ${GRPCPP_LIBRARIES})
    MESSAGE("GRPCPP_INCLUDE_DIRS:" ${GRPCPP_INCLUDEDIR})
    MESSAGE("GRPCPP_LIBRARY_DIRS:" ${GRPCPP_LIBDIR})
    INCLUDE_DIRECTORIES(${GRPCPP_INCLUDEDIR})
    LINK_DIRECTORIES(${GRPCPP_LIBDIR})
    ADD_EXECUTABLE(main src/main.cpp)
    TARGET_LINK_LIBRARIES(main ${GRPCPP_LIBRARIES})
endif()
```



## Running a Command in CMake

### At Configure Time
```
find_package(Git QUIET)

 you can use ${CMAKE_COMMAND}, find_package(Git), or find_program to get access to a command to run. 
 Use RESULT_VARIABLE to check the return code and OUTPUT_VARIABLE to get the output.

if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init --recursive
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
                    RESULT_VARIABLE GIT_SUBMOD_RESULT)
    if(NOT GIT_SUBMOD_RESULT EQUAL "0")
        message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
    endif()
endif()
```

### At Build Time
```
find_package(PythonInterp REQUIRED)
add_custom_command(OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/include/Generated.hpp"
    COMMAND "${PYTHON_EXECUTABLE}" "${CMAKE_CURRENT_SOURCE_DIR}/scripts/GenerateHeader.py" --argument
    DEPENDS some_target)

add_custom_target(generate_header ALL
    DEPENDS "${CMAKE_CURRENT_BINARY_DIR}/include/Generated.hpp")

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/include/Generated.hpp DESTINATION include)
```



## Creating and Installing a library 

Create the following structure:

```
MainProject/
├── CMakeLists.txt
├── CMakeLists.txt.user
├── MyLibrary
│   ├── CMakeLists.txt
│   ├── include
│   │   └── MyLibrary.hpp
│   └── src
│       └── MyLibrary.cpp
└── src
    └── main.cpp
```
Now inside of `MainProject/MyLibrary/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.10)  # Minimum required CMake version

project(MyLibrary VERSION 1.0.0 LANGUAGES CXX)  # Project name and version with C++ as the language

# Add the source files to create the library
add_library(MyLibrary SHARED
    src/MyLibrary.cpp
)

# Specify the include directories for users of the library
target_include_directories(MyLibrary PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>  # Include directory during build
    $<INSTALL_INTERFACE:include>  # Include directory after installation
)

# Set properties for the library such as version and public header
set_target_properties(MyLibrary PROPERTIES
    VERSION ${PROJECT_VERSION}  # Version of the library
    SOVERSION 1  # Shared object version
    PUBLIC_HEADER include/MyLibrary.hpp  # Header file to be installed and used by users
)

# Specify where to install the built library and its associated files
install(TARGETS MyLibrary
    EXPORT MyLibraryConfig  # Export library configuration for use in other projects
    LIBRARY DESTINATION lib  # Destination for shared libraries
    ARCHIVE DESTINATION lib  # Destination for static libraries
    RUNTIME DESTINATION bin  # Destination for executables
    INCLUDES DESTINATION include  # Destination for include files
    PUBLIC_HEADER DESTINATION include  # Destination for public header file
)

# Install the export configuration for the library
install(EXPORT MyLibraryConfig NAMESPACE MyLibrary:: DESTINATION cmake)
```

### SOVERSION
The line `SOVERSION 1` in CMake sets the version number of the shared object (shared library) being built. 

In Unix-like operating systems, shared libraries are typically versioned to allow multiple versions of the same library to coexist on a system. This versioning is helpful for backward compatibility and ensures that programs compiled against an older version of the library will continue to work with newer versions, provided backward compatibility is maintained.

The `SOVERSION` property sets the version number of the shared object file itself. It's separate from the project version specified in the `project()` command. It's particularly important when generating shared libraries because it's used to construct the actual filename of the shared library. 

For example, if your library is named `libMyLibrary.so` and you set `SOVERSION 1`, then the actual filename of the shared library would be `libMyLibrary.so.1`. This version number is embedded into the filename to distinguish between different versions of the library.

In CMake, setting `SOVERSION` to `1` in your `CMakeLists.txt` file would result in the generated shared library having a version number of `1`. If you were to release a major update to your library, you might increment this version number to `2`, and so on, to distinguish between different versions of your library's shared object files.





### MyLibraryConfig

The `MyLibraryConfig` export configuration is generated using the `install(EXPORT ...)` command. This command exports targets defined in the project for use by other projects. 

In your `CMakeLists.txt`, you're exporting the target `MyLibrary` with the namespace `MyLibrary::`. This means that when other projects use `MyLibraryConfig`, they can reference the target `MyLibrary` as `MyLibrary::MyLibrary`.

Here's how `MyLibraryConfig` is generated in your `CMakeLists.txt`:

```cmake
# Install the export configuration for the library
install(EXPORT MyLibraryConfig NAMESPACE MyLibrary:: DESTINATION cmake)
```

This command tells CMake to install the export configuration named `MyLibraryConfig` with the namespace `MyLibrary::` into the `cmake` directory of the installation prefix. This export configuration will contain information about the target `MyLibrary` and its properties, such as include directories, compile options, and dependencies. Other projects can then import this configuration using `find_package(MyLibrary CONFIG)`, which will make the target `MyLibrary::MyLibrary` available for use in their CMakeLists.txt.




Now in the `MainProject/CMakeLists.txt`:

```cmake
cmake_minimum_required(VERSION 3.10)
project(MainProject VERSION 1.0.0 LANGUAGES CXX)

add_subdirectory(MyLibrary)

add_executable(MainProject src/main.cpp)

target_link_libraries(MainProject MyLibrary)

install(TARGETS MainProject
    EXPORT MainProjectConfig
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)

install(EXPORT MainProjectConfig NAMESPACE MainProject:: DESTINATION cmake)
```
configure it:


```
cmake -G "Ninja Multi-Config" -S . -B build -DCMAKE_INSTALL_PREFIX=~/usr
```

build it:

```
cmake --build build --config Release
```


```
cmake --install build --config Release
```
This will install the followings:


```
/home/behnam/usr/
├── bin
│   └── MainProject
├── cmake
│   ├── MainProjectConfig.cmake
│   ├── MainProjectConfig-release.cmake
│   ├── MyLibraryConfig.cmake
│   └── MyLibraryConfig-release.cmake
├── include
│   └── MyLibrary.hpp
└── lib
    ├── libMyLibrary.so -> libMyLibrary.so.1
    ├── libMyLibrary.so.1 -> libMyLibrary.so.1.0.0
    └── libMyLibrary.so.1.0.0
```

Now let say we want to use the `MyLibrary` in another project, 

```
AnotherProject/
├── CMakeLists.txt
└── src
    └── main.cpp
```

In the `AnotherProject/CMakeLists.txt`:


```cmake
cmake_minimum_required(VERSION 3.10)
project(AnotherProject VERSION 1.0.0 LANGUAGES CXX)

find_package(MyLibrary REQUIRED)

add_executable(AnotherProject src/main.cpp)

target_link_libraries(AnotherProject MyLibrary::MyLibrary)
```

you can optionally add the followings to all related variables to `MyLibrary`:

```cmake
# Print variables related to foo
get_cmake_property(_variableNames VARIABLES)
foreach (_variableName ${_variableNames})
  if (_variableName MATCHES "^MyLibrary")
    message(STATUS "${_variableName}=${${_variableName}}")
  endif()
endforeach()

# Print imported targets
get_property(_targets GLOBAL PROPERTY IMPORTED_TARGETS)
foreach(_target ${_targets})
  if(_target MATCHES "^MyLibrary::")
    message(STATUS "Imported target: ${_target}")
  endif()
endforeach()
```

Now run this:

```
cmake -G "Ninja Multi-Config" -S . -B build -DCMAKE_PREFIX_PATH=~/usr -DCMAKE_INSTALL_PREFIX=~/usr
cmake --build build --config Debug
cmake --install build --config Debug
```





References:[1](https://gist.github.com/mbinna/), 
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
[13](https://foonathan.net/2016/03/cmake-install/), 
[14](https://pabloariasal.github.io/2018/02/19/its-time-to-do-cmake-right/),
[15](https://www.youtube.com/watch?v=rLopVhns4Zs)



[![Build Status](https://travis-ci.com/behnamasadi/cmake_tutorials.svg?branch=master)](https://travis-ci.com/behnamasadi/cmake_tutorials)
![alt text](https://img.shields.io/badge/license-BSD-blue.svg)  



