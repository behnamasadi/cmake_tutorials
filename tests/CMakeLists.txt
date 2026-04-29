include(GoogleTest)

set(MAIN_SOURCE "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp")

file(GLOB SPEC_SOURCES CONFIGURE_DEPENDS
    "${CMAKE_CURRENT_SOURCE_DIR}/src/spec/*.cpp"
)

foreach(SOURCE IN LISTS SPEC_SOURCES)
    get_filename_component(TEST_NAME "${SOURCE}" NAME_WE)
    add_executable(${TEST_NAME} "${MAIN_SOURCE}" "${SOURCE}")
    target_include_directories(${TEST_NAME} PRIVATE
        "${PROJECT_SOURCE_DIR}/include"
    )
    target_link_libraries(${TEST_NAME} PRIVATE GTest::gtest GTest::gmock)
    gtest_discover_tests(${TEST_NAME})
endforeach()
