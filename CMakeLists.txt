cmake_minimum_required(VERSION 3.25)

project(wrapper)
include(ExternalProject)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 11)

set(ANDROID_NDK_PATH "$ENV{HOME}/android-ndk-r23b")
set(TOOLCHAIN "${ANDROID_NDK_PATH}/toolchains/llvm/prebuilt/linux-x86_64")

set(CMAKE_C_COMPILER "${TOOLCHAIN}/bin/x86_64-linux-android22-clang")
set(CMAKE_CXX_COMPILER "${TOOLCHAIN}/bin/x86_64-linux-android22-clang++")
set(C_COMPILER "${TOOLCHAIN}/bin/clang")

set(CMAKE_C_FLAGS "-Wall -Werror -O3")
set(CMAKE_CXX_FLAGS "-Wall -Werror -O3")

set(CMDLINE_SOURCE cmdline.c)
set(HANDLE_SOURCE main.cpp)
set(MAIN_SOURCE main.c)
set(WRAPPER_SOURCE wrapper.c)

add_library(cmdline_object OBJECT ${CMDLINE_SOURCE})
add_library(handle_object OBJECT ${HANDLE_SOURCE})

add_executable(main ${MAIN_SOURCE} $<TARGET_OBJECTS:cmdline_object> $<TARGET_OBJECTS:handle_object>)

set_target_properties(main PROPERTIES 
    COMPILE_DEFINITIONS "MyRelease"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/rootfs/system/bin"
)

find_library(ANDROIDAPPMUSIC_LIB androidappmusic PATHS ${CMAKE_SOURCE_DIR}/rootfs/system/lib64)
find_library(STORESERVICESCORE_LIB storeservicescore PATHS ${CMAKE_SOURCE_DIR}/rootfs/system/lib64)
find_library(MEDIAPLATFORM_LIB mediaplatform PATHS ${CMAKE_SOURCE_DIR}/rootfs/system/lib64)
find_library(CXX_SHARED_LIB c++_shared PATHS ${CMAKE_SOURCE_DIR}/rootfs/system/lib64)

# Link libraries
target_link_libraries(main
    ${CXX_SHARED_LIB}
    ${ANDROIDAPPMUSIC_LIB}
    ${STORESERVICESCORE_LIB}
    ${MEDIAPLATFORM_LIB}
)

link_directories(${CMAKE_SOURCE_DIR}/rootfs/system/lib64)

ExternalProject_Add(
    wrapper
    PREFIX ${CMAKE_BINARY_DIR}
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}
    CONFIGURE_COMMAND ""
    BUILD_COMMAND ${C_COMPILER} -O3 -Wall -o wrapper ${WRAPPER_SOURCE} ${CMAKE_CURRENT_SOURCE_DIR}/cmdline.c
    BUILD_IN_SOURCE 1
    INSTALL_COMMAND ""
    DEPENDS main
)