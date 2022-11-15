project(libziparchive LANGUAGES C CXX)
cmake_minimum_required(VERSION 3.10)

set(CMAKE_VERBOSE_MAKEFILE ON)

# Require out-of-source builds
file(TO_CMAKE_PATH "${PROJECT_BINARY_DIR}/CMakeLists.txt" LOC_PATH)
if(EXISTS "${LOC_PATH}")
    message(FATAL_ERROR "You cannot build in a source directory (or any directory with a CMakeLists.txt file). Please make a build subdirectory. Feel free to remove CMakeCache.txt and CMakeFiles.")
endif()

set(LIBZIPARCHIVE_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/../)

if(NOT EXISTS "${CMAKE_BINARY_DIR}/conan.cmake")
    message(STATUS "Downloading conan.cmake from https://github.com/conan-io/cmake-conan")
    file(DOWNLOAD "https://raw.githubusercontent.com/conan-io/cmake-conan/0.18.1/conan.cmake"
            "${CMAKE_BINARY_DIR}/conan.cmake"
            TLS_VERIFY ON)
endif()
include(${CMAKE_BINARY_DIR}/conan.cmake)

conan_cmake_autodetect(settings)
conan_cmake_install(PATH_OR_REFERENCE ${LIBZIPARCHIVE_ROOT}/cmake
                    PROFILE_HOST ${LIBZIPARCHIVE_ROOT}/cmake/armv8.profile
                    PROFILE_BUILD ${LIBZIPARCHIVE_ROOT}/cmake/build.profile
                    BUILD missing)

# Set conan environment variables generated using the virtualenv generator
# This is necessary since for some reason Conan does not correctly set the variables found (e.g. the Android NDK compiler path)
# A particular attention should be paid to the PATH variable since it is not good to completely overwrite it
if(EXISTS "${CMAKE_BINARY_DIR}/environment.sh.env")
    FILE(READ "${CMAKE_BINARY_DIR}/environment.sh.env" _contents)
    STRING(REGEX REPLACE "\n" ";" _contents "${_contents}")
    foreach(_line ${_contents})
        string(REGEX MATCH "([^=]+)=\"(.*)\"" _match ${_line})
        if(NOT "${_match}" STREQUAL "" AND NOT "${CMAKE_MATCH_1}" STREQUAL "PATH")
            #message("in ${CMAKE_MATCH_1} = ${CMAKE_MATCH_2}")
            set(ENV{${CMAKE_MATCH_1}} "${CMAKE_MATCH_2}")
        endif()
    endforeach()
    unset(_contents)
    unset(_line)
    unset(_match)
else()
    message(FATAL_ERROR "Android NDK environment variables not found!")
endif()

set(CMAKE_C_COMPILER   $ENV{CC})
set(CMAKE_CXX_COMPILER $ENV{CXX})
set(CMAKE_CXX_COMPILER_AR $ENV{AR})
set(CMAKE_CXX_COMPILER_RANLIB $ENV{RANLIB})
set(CMAKE_AR           $ENV{AR})
set(CMAKE_AS           $ENV{AS})
set(CMAKE_RANLIB       $ENV{RANLIB})
set(CMAKE_STRIP        $ENV{STRIP})
set(CMAKE_ADDR2LINE    $ENV{ADDR2LINE})
set(CMAKE_NM           $ENV{NM})
set(CMAKE_OBJCOPY      $ENV{OBJCOPY})
set(CMAKE_OBJDUMP      $ENV{OBJDUMP})
set(CMAKE_READELF      $ENV{READELF})
set(CMAKE_LD           $ENV{LD})

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_BINARY_DIR})

set(ANDROID_SOURCES
    ${LIBZIPARCHIVE_ROOT}/android-base/errno_restorer.h
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

set(LIBZIPARCHIVE_SOURCES
    ${LIBZIPARCHIVE_ROOT}zip_archive.cc
    ${LIBZIPARCHIVE_ROOT}zip_archive_stream_entry.cc
    ${LIBZIPARCHIVE_ROOT}zip_cd_entry_map.cc
    ${LIBZIPARCHIVE_ROOT}zip_error.cpp
    ${LIBZIPARCHIVE_ROOT}zip_writer.cc
)

add_library(LIBZIPARCHIVE STATIC ${ANDROID_SOURCES} ${LIBZIPARCHIVE_SOURCES})
target_include_directories(LIBZIPARCHIVE PUBLIC ${LIBZIPARCHIVE_ROOT}/include ${LIBZIPARCHIVE_ROOT}/incfs_support/include)
target_include_directories(LIBZIPARCHIVE PRIVATE ${LIBZIPARCHIVE_ROOT})

set_property(TARGET LIBZIPARCHIVE PROPERTY CXX_STANDARD 20)

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(LIBZIPARCHIVE PRIVATE $<BUILD_INTERFACE:-Wno-unknown-attributes>)
endif()
