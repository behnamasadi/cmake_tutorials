# CMakeLists.txt

cmake_minimum_required(VERSION 3.0)
project(MyProject)

# Add source files
set(SOURCES
    src/my_source_file.cpp
    src/main.cpp
)

# Add header files
set(HEADERS
    include/my_header.h
)

# Define the library and executable
add_library(my_library ${SOURCES})
add_executable(my_executable ${SOURCES})

# Specify include directories
target_include_directories(my_library PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(my_executable PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)

# Install targets and files
install(TARGETS my_library my_executable
        DESTINATION bin)
install(FILES ${HEADERS}
        DESTINATION include)

# Export targets
export(TARGETS my_library my_executable
       FILE MyProjectTargets.cmake)  # Ensure export file name matches

# Create configuration files
include(CMakePackageConfigHelpers)
set(CONFIG_INSTALL_DIR lib/cmake/MyProject)  # Specify the installation directory for CMake files
configure_package_config_file(MyProjectConfig.cmake.in
                              ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
                              INSTALL_DESTINATION ${CONFIG_INSTALL_DIR})
write_basic_package_version_file(${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
                                 VERSION 1.0
                                 COMPATIBILITY SameMajorVersion)
install(FILES
        ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfig.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/MyProjectConfigVersion.cmake
        DESTINATION ${CONFIG_INSTALL_DIR})

