cmake_minimum_required(VERSION 3.10)
project(MainProject VERSION 1.0.0 LANGUAGES CXX)

add_subdirectory(MyLibrary)

add_subdirectory(Tools)


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

