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



# Print include directories
message(STATUS "MyProject include directories: ${MyProject_INCLUDE_DIRS}")

# Print libraries
message(STATUS "MyProject libraries: ${MyProject_LIBRARIES}")



# Print variables related to foo
get_cmake_property(_variableNames VARIABLES)
foreach (_variableName ${_variableNames})
  if (_variableName MATCHES "^MyProject")
    message(STATUS "${_variableName}=${${_variableName}}")
  endif()
endforeach()

# Print imported targets
get_property(_targets GLOBAL PROPERTY IMPORTED_TARGETS)
foreach(_target ${_targets})
  if(_target MATCHES "^MyProject::")
    message(STATUS "Imported target: ${_target}")
  endif()
endforeach()
