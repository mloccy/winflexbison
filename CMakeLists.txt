cmake_minimum_required(VERSION 3.5.0 FATAL_ERROR)

project (winflexbison)

if(NOT MSVC)
   message( WARNING "Visual Studio Build supported only" )
endif()

add_definitions(-D_CRT_SECURE_NO_WARNINGS)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
  add_definitions(-D_DEBUG)
endif()

# next line needed for compile in C (nor CPP) mode (ucrt headers bug)
add_definitions(-Dinline=__inline)
# next line needed for VS2017 only
add_definitions(-Drestrict=__restrict)

set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /W3 /MD /Od /Zi /EHsc")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /W3 /GL /Od /Oi /Gy /Zi /EHsc")

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    # Output Variables
    #------------------------------------------------------------------------
    # Static Windows Runtime
    #   Option to statically link to the Windows runtime. Maybe only
    #   applies to WIN32/MSVC.
    #------------------------------------------------------------------------
    if (MSVC)
        add_compile_definitions("__extension__")
        add_compile_options("/source-charset:utf-8")
        option( USE_STATIC_RUNTIME "Set ON to change /MD(DLL) to /MT(static)" OFF )
        if (USE_STATIC_RUNTIME)
            set(CompilerFlags
                CMAKE_CXX_FLAGS
                CMAKE_CXX_FLAGS_DEBUG
                CMAKE_CXX_FLAGS_RELEASE
                CMAKE_C_FLAGS
                CMAKE_C_FLAGS_DEBUG
                CMAKE_C_FLAGS_RELEASE
                )
            foreach(CompilerFlag ${CompilerFlags})
                string(REPLACE "/MD" "/MT" ${CompilerFlag} "${${CompilerFlag}}")
            endforeach()
            message(STATUS "Using /MT STATIC runtime")
        endif ()
    endif ()
endif ()



add_subdirectory(common)
add_subdirectory(flex)
add_subdirectory(bison)


include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
install(TARGETS flex bison
        EXPORT FlexTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
install(EXPORT FlexTargets NAMESPACE flex:: DESTINATION "${CMAKE_INSTALL_LIBDIR}/cmake/flex")
install(FILES "flex/src/FlexLexer.h" DESTINATION "include")
install(DIRECTORY "bison/data/" DESTINATION "share/bison")

write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/FlexConfigVersion.cmake"
  VERSION "master"
  COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_SOURCE_DIR}/cmake/Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/FlexConfig.cmake"
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/flex
)

install(FILES
          "${CMAKE_CURRENT_BINARY_DIR}/FlexConfig.cmake"
          "${CMAKE_CURRENT_BINARY_DIR}/FlexConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/flex
)
set(PACKAGE_GENERATORS_DEFAULT ZIP)

set(CPACK_GENERATOR ${PACKAGE_GENERATORS_DEFAULT} CACHE STRING "List of CPack Generators which will be created by the 'PACKAGE' target. Default: ZIP")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Winflexbison - Flex and Bison for Windows")

set(ENV_VERSION "$ENV{WINFLEXBISON_VERSION}")
if(NOT ENV_VERSION)
  set(CPACK_PACKAGE_VERSION "master" CACHE STRING "Complete version of the created package.")
else()
  set(CPACK_PACKAGE_VERSION "${ENV_VERSION}" CACHE STRING "Complete version of the created package.")
endif()

set(CPACK_INCLUDE_TOPLEVEL_DIRECTORY OFF)
set(CPACK_ARCHIVE_COMPONENT_INSTALL OFF)
set(CPACK_MONOLITHIC_INSTALL ON)

set(CPACK_PACKAGE_FILE_NAME "win_flex_bison-${CPACK_PACKAGE_VERSION}")

include(CPack)
