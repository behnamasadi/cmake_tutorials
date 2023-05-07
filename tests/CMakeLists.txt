###
# project
###
project(${PROJECT}_tests DESCRIPTION "An example of GTest" LANGUAGES CXX)


###
# includes
###
include_directories(${EXTERN_INCLUDES} ${MY_PROJECT_INCLUDES})

###
# libraries
###
link_directories(${EXTERN_LIBRARIES})


###
# executables
###
set(MAIN "src/main.cpp")
set(DIRS "src/spec" "src/spec/**")
foreach(DIR ${DIRS})
  file(GLOB s_${DIR} "${DIR}/*.cpp")
  foreach(SOURCE ${s_${DIR}})
    get_filename_component(TEST_NAME ${SOURCE} NAME_WE)
    add_executable(${TEST_NAME} ${MAIN} ${SOURCE})
    target_link_libraries(${TEST_NAME}  gtest pthread)
    add_test(NAME ${TEST_NAME}_tests COMMAND ${TEST_NAME})
  endforeach()
endforeach()
