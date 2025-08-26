cmake_minimum_required(VERSION 4.0)
project(tricore-toolchain)

include(ExternalProject)
include(FetchContent)

FetchContent_Declare(llvm
    GIT_REPOSITORY https://github.com/tricore-oss/llvm-project.git
    GIT_TAG tricore
    GIT_SHALLOW true
    SOURCE_SUBDIR llvm/
)

set(LLVM_TARGETS_TO_BUILD "" CACHE STRING "")
set(LLVM_EXPERIMENTAL_TARGETS_TO_BUILD "Tricore" CACHE STRING "")
set(LLVM_USE_LINKER lld CACHE STRING "")
set(LLVM_ENABLE_PROJECTS "clang;lld" CACHE STRING "")
set(LLVM_DEFAULT_TARGET_TRIPLE "tricore-elf" CACHE STRING "")

FetchContent_MakeAvailable(llvm)

FetchContent_Declare(picolibc
    GIT_REPOSITORY https://github.com/tricore-oss/picolibc.git
    GIT_TAG tricore
    SOURCE_SUBDIR x/
)
FetchContent_MakeAvailable(picolibc)

set(MULTILIB_C_FLAGS "-march=tc162 -fno-rtti -fno-exceptions -nostdlib")
set(MULTILIB_CXX_FLAGS "-march=tc162 -fno-rtti -fno-exceptions -nostdlib")

ExternalProject_Add(compiler-rt-tc16
    SOURCE_DIR ${llvm_SOURCE_DIR}/compiler-rt/
    DEPENDS clang lld
    CMAKE_ARGS
        -DCMAKE_SYSTEM_NAME=Generic
        -DCMAKE_SYSTEM_PROCESSOR=tricore
        -DCMAKE_C_COMPILER=$<TARGET_FILE_DIR:clang>/clang$<TARGET_FILE_SUFFIX:clang>
        -DCMAKE_CXX_COMPILER=$<TARGET_FILE_DIR:clang>/clang++$<TARGET_FILE_SUFFIX:clang>
        -DCMAKE_C_FLAGS_INIT=${MULTILIB_C_FLAGS}
        -DCMAKE_CXX_FLAGS_INIT=${MULTILIB_CXX_FLAGS}
        -DCMAKE_EXE_LINKER_FLAGS=-fuse-ld=lld
        -DCMAKE_C_COMPILER_TARGET=tricore-unknown-unknown-elf
        -DCOMPILER_RT_BUILD_BUILTINS=ON 
        -DCOMPILER_RT_BUILD_LIBFUZZER=OFF 
        -DCOMPILER_RT_BUILD_MEMPROF=OFF 
        -DCOMPILER_RT_BUILD_PROFILE=OFF
        -DCOMPILER_RT_BUILD_CTX_PROFILE=OFF 
        -DCOMPILER_RT_BUILD_SANITIZERS=OFF 
        -DCOMPILER_RT_BUILD_XRAY=OFF 
        -DCOMPILER_RT_BUILD_ORC=OFF 
        -DCOMPILER_RT_BUILD_CRT=OFF 
        -DCOMPILER_RT_DEFAULT_TARGET_ONLY=ON 
        -DCOMPILER_RT_BAREMETAL_BUILD=ON
)

set(PICOLIBC_C_FLAGS "-march=tc162 -fno-rtti -fno-exceptions -nostdlib")
set(PICOLIBC_CXX_FLAGS "-march=tc162 -fno-rtti -fno-exceptions -nostdlib")
ExternalProject_Add(picolibc-tc16
    SOURCE_DIR ${Picolibc_SOURCE_DIR}
    DEPENDS clang lld
    CMAKE_ARGS
        -DCMAKE_SYSTEM_NAME=Generic
        -DCMAKE_SYSTEM_PROCESSOR=tricore
        -DCMAKE_C_COMPILER=$<TARGET_FILE_DIR:clang>/clang$<TARGET_FILE_SUFFIX:clang>
        -DCMAKE_CXX_COMPILER=$<TARGET_FILE_DIR:clang>/clang++$<TARGET_FILE_SUFFIX:clang>
        -DCMAKE_C_FLAGS_INIT=${PICOLIBC_C_FLAGS}
        -DCMAKE_CXX_FLAGS_INIT=${PICOLIBC_CXX_FLAGS}
        -D__THREAD_LOCAL_STORAGE=Off
)
