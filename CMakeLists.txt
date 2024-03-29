cmake_minimum_required(VERSION 3.2)

message(STATUS " CMakeLists: Zep")

# Global Options
option(BUILD_QT "Make Qt Library" OFF)
option(BUILD_IMGUI "Make Imgui Library" ON)
option(BUILD_DEMOS "Make the demo app" ON)
option(BUILD_TESTS "Make the tests" ON)
option(ZEP_FEATURE_CPP_FILE_SYSTEM "Default File system enabled" ON)

# Global Settings
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set_property(GLOBAL PROPERTY USE_FOLDERS ON)

if (CMAKE_BUILD_TYPE MATCHES Coverage)
    set(CMAKE_BUILD_TYPE "Debug")
    set(PROJECT_COVERAGE ON)
endif()

set(CMAKE_DEBUG_POSTFIX "-debug")

project(Zep
    LANGUAGES CXX C
    VERSION 0.5.0
)

# Used Packages
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# Can I remove these now?
if (ZEP_FEATURE_CPP_FILE_SYSTEM)
add_definitions(-DZEP_FEATURE_CPP_FILE_SYSTEM)
endif()
add_definitions(-DZEP_USE_SDL)

# config_app.h checks
# This makes a config_shared.h file which can be included for system settings
#  Must be after setting up the platforms
set (ZEP_ROOT ${CMAKE_CURRENT_LIST_DIR})
set (M3RDPARTY_DIR ${CMAKE_CURRENT_LIST_DIR}/m3rdparty)

configure_file(${ZEP_ROOT}/cmake/config_app.h.cmake ${CMAKE_BINARY_DIR}/config_app.h)

# Set all compiler flags 
include(cmake/all.cmake)

# Functions for file copying
include(m3rdparty/cmake/copy_files.cmake)

# The main library
include(src/CMakeLists.txt)

include (tests/CMakeLists.txt)

# Demos require example interpreter and other 3rdparty
IF (BUILD_DEMOS)
include(${M3RDPARTY_DIR}/list.cmake)
include(demo_imgui/CMakeLists.txt)
include(demo_qt/CMakeLists.txt)
ENDIF()

# Make the CMake bits that ensure find_package does the right thing
install(EXPORT zep-targets
    FILE zep-targets.cmake
    NAMESPACE Zep::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zep
)

configure_package_config_file(
    ${ZEP_ROOT}/cmake/zep-config.cmake.in
    ${CMAKE_BINARY_DIR}/cmake/zep-config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zep
)

write_basic_package_version_file(
    ${CMAKE_BINARY_DIR}/cmake/zep-config-version.cmake
    VERSION 1.0
    COMPATIBILITY AnyNewerVersion
)

install(
    FILES
        ${CMAKE_BINARY_DIR}/cmake/zep-config.cmake
        ${CMAKE_BINARY_DIR}/cmake/zep-config-version.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/zep
)

