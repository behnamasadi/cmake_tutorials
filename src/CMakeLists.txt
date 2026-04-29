include(CMakePackageConfigHelpers)

foreach(LIB IN ITEMS core auxiliary misc)
    if(LIB STREQUAL "auxiliary")
        set(LIB_SOURCE core/lib.cpp)
    else()
        set(LIB_SOURCE ${LIB}/lib.cpp)
    endif()

    add_library(${LIB} ${LIB_SOURCE})

    target_include_directories(${LIB} PUBLIC
        $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
    )

    install(TARGETS ${LIB}
        EXPORT   ${LIB}Targets
        LIBRARY  DESTINATION lib
        ARCHIVE  DESTINATION lib
        RUNTIME  DESTINATION bin
        INCLUDES DESTINATION include
    )

    write_basic_package_version_file(
        ${LIB}ConfigVersion.cmake
        VERSION       ${PROJECT_VERSION}
        COMPATIBILITY AnyNewerVersion
    )
endforeach()
