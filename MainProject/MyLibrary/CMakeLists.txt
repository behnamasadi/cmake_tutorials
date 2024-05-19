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
