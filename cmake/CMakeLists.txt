if (POLICY CMP0048)
    cmake_policy(SET CMP0048 NEW)
endif (POLICY CMP0048)

cmake_minimum_required(VERSION 3.20)

set(CMAKE_VERBOSE_MAKEFILE ON)

# Require out-of-source builds
file(TO_CMAKE_PATH "${PROJECT_BINARY_DIR}/CMakeLists.txt" LOC_PATH)
if(EXISTS "${LOC_PATH}")
    message(FATAL_ERROR "You cannot build in a source directory (or any directory with a CMakeLists.txt file). Please make a build subdirectory. Feel free to remove CMakeCache.txt and CMakeFiles.")
endif()

# Set a default build type if none was specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to 'Release' as none was specified.")
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

project(libziparchive LANGUAGES CXX)

include(FetchContent)

FetchContent_Declare(libbase-ndk
                     GIT_REPOSITORY https://github.com/MaDeG/libbase-ndk.git
                     GIT_TAG master
                     GIT_SUBMODULES ""
                     SOURCE_SUBDIR cmake)
FetchContent_MakeAvailable(libbase-ndk)
FetchContent_GetProperties(libbase-ndk)

set(LIBZIPARCHIVE_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../)

set(ANDROID_SOURCES ${LIBZIPARCHIVE_ROOT}/android-base/errno_restorer.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/file.cpp
                    ${LIBZIPARCHIVE_ROOT}/android-base/file.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/log_main.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/logging.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/macros.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/off64_t.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/parseint.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/stringprintf.cpp
                    ${LIBZIPARCHIVE_ROOT}/android-base/stringprintf.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/strings.cpp
                    ${LIBZIPARCHIVE_ROOT}/android-base/strings.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/threads.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/unique_fd.h
                    ${LIBZIPARCHIVE_ROOT}/android-base/utf8.h
)

set(LIBZIPARCHIVE_SOURCES ${LIBZIPARCHIVE_ROOT}/zip_archive.cc
                          ${LIBZIPARCHIVE_ROOT}/zip_archive_stream_entry.cc
                          ${LIBZIPARCHIVE_ROOT}/zip_cd_entry_map.cc
                          ${LIBZIPARCHIVE_ROOT}/zip_error.cpp
                          ${LIBZIPARCHIVE_ROOT}/zip_writer.cc
                          ${LIBZIPARCHIVE_ROOT}/incfs_support/signal_handling.cpp
)

add_library(ziparchive STATIC ${ANDROID_SOURCES} ${LIBZIPARCHIVE_SOURCES})
target_include_directories(ziparchive PUBLIC ${LIBZIPARCHIVE_ROOT}/include ${LIBZIPARCHIVE_ROOT}/incfs_support/include)
target_include_directories(ziparchive PRIVATE ${LIBZIPARCHIVE_ROOT} ${libbase-ndk_SOURCE_DIR}/include)
target_link_libraries(ziparchive base)

set_property(TARGET ziparchive PROPERTY CXX_STANDARD 20)

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(ziparchive PRIVATE $<BUILD_INTERFACE:-Wno-unknown-attributes>)
endif()