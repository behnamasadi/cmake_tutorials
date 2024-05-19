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

# Print MyProject version
message(STATUS "MyProject version: ${MyProject_VERSION}")
