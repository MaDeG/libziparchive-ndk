#TODO: Switch to the more Conan 2.0 compliant Python way of operating it

cmake_minimum_required(VERSION 3.10)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    include("./CMakeListsConan.txt")
endif()

project(libziparchive LANGUAGES CXX)

include(FetchContent)

FetchContent_Declare(libbase-ndk
                     GIT_REPOSITORY git@github.com:MaDeG/libbase-ndk.git
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

add_library(libziparchive STATIC ${ANDROID_SOURCES} ${LIBZIPARCHIVE_SOURCES})
target_include_directories(libziparchive PUBLIC ${LIBZIPARCHIVE_ROOT}/include ${LIBZIPARCHIVE_ROOT}/incfs_support/include)
target_include_directories(libziparchive PRIVATE ${LIBZIPARCHIVE_ROOT} ${libbase-ndk_SOURCE_DIR}/include)
target_link_libraries(libziparchive log libbase)

set_property(TARGET libziparchive PROPERTY CXX_STANDARD 20)

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_options(libziparchive PRIVATE $<BUILD_INTERFACE:-Wno-unknown-attributes>)
endif()