cmake_minimum_required(VERSION 3.10)
project(AnotherProject VERSION 1.0.0 LANGUAGES CXX)

#find_package(MyLibrary REQUIRED)
find_package(MyLibrary CONFIG)

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

add_executable(AnotherProject src/main.cpp)

target_link_libraries(AnotherProject MyLibrary::MyLibrary)

install(TARGETS AnotherProject
    EXPORT AnotherProjectConfig
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)

install(EXPORT AnotherProjectConfig NAMESPACE AnotherProject:: DESTINATION cmake)

