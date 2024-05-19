cmake_minimum_required(VERSION 3.0)
project(MyProject VERSION 3.5.1)

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

# Generate and install package config files
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
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
    DESTINATION ${CONFIG_INSTALL_DIR}
)

