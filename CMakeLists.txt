cmake_minimum_required(VERSION 3.10)
project(mxnet-build)
include(ExternalProject)

list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR}/install_root)

# Help find issues
set(CMAKE_FIND_DEBUG_MODE TRUE)

# Use ccache when available
find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
    set(CMAKE_C_COMPILER_LAUNCHER ${CCACHE_FOUND})
    set(CMAKE_CXX_COMPILER_LAUNCHER ${CCACHE_FOUND})
endif()

ExternalProject_Add(
    opencv
    GIT_REPOSITORY https://github.com/opencv/opencv.git
    GIT_TAG 4.11.0
    PREFIX ${CMAKE_BINARY_DIR}/opencv
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/install_root
    CMAKE_CACHE_ARGS
        "-DBUILD_JAVA:BOOL=OFF"
        "-DBUILD_SHARED_LIBS:BOOL=OFF"
        "-DWITH_OPENEXR:BOOL=OFF"
)

ExternalProject_Add(
    openblas
    GIT_REPOSITORY https://github.com/OpenMathLib/OpenBLAS.git
    GIT_TAG v0.3.29
    PREFIX ${CMAKE_BINARY_DIR}/openblas
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/install_root
    CMAKE_CACHE_ARGS
        "-DBUILD_STATIC_LIBS:BOOL=ON"
)

ExternalProject_Add(
    mxnet
    SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/mxnet
    PATCH_COMMAND ${CMAKE_COMMAND} -E chdir <SOURCE_DIR> patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/mxnet.patch
    PREFIX ${CMAKE_BINARY_DIR}/mxnet
    CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/install_root
    CMAKE_CACHE_ARGS
        "-DBLAS:STRING=Open"
        "-DUSE_CUDA:BOOL=OFF"
        "-DUSE_OPENMP:BOOL=OFF"
    DEPENDS opencv openblas
)
