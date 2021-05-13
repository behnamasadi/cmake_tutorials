###
# core
###
add_library(core core/lib.cpp)


install(TARGETS core
        EXPORT coreTargets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)



include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    coreConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY AnyNewerVersion
    )
    
    
###
# aux
###
    
    
add_library(aux core/lib.cpp)


install(TARGETS aux
        EXPORT auxTargets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)



include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    auxConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY AnyNewerVersion
    )    
    
    
###
# misc
###
    
add_library(misc misc/lib.cpp)

install(TARGETS misc
        EXPORT miscTargets
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
        INCLUDES DESTINATION include)



include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    miscConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY AnyNewerVersion
    )    
