cmake_minimum_required(VERSION 3.10)
project(mxnet-build)
include(ExternalProject)

list(APPEND CMAKE_PREFIX_PATH ${CMAKE_BINARY_DIR}/install_root)

# Use ccache when available
find_program(CCACHE_FOUND ccache)
if(CCACHE_FOUND)
  set(CMAKE_C_COMPILER_LAUNCHER ${CCACHE_FOUND})
  set(CMAKE_CXX_COMPILER_LAUNCHER ${CCACHE_FOUND})
endif()

ExternalProject_Add(
  openblas
  GIT_REPOSITORY https://github.com/OpenMathLib/OpenBLAS.git
  GIT_TAG v0.3.29
  PREFIX ${CMAKE_BINARY_DIR}/openblas
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/install_root
    -DBUILD_STATIC_LIBS=OFF
    -DDYNAMIC_ARCH=ON
    -DDYNAMIC_OLDER=ON
)

ExternalProject_Add(
  opencv
  GIT_REPOSITORY https://github.com/opencv/opencv.git
  GIT_TAG 4.11.0
  PREFIX ${CMAKE_BINARY_DIR}/opencv
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/install_root
    -DBUILD_LIST=core,highgui,imgproc,imgcodecs  # Build only required modules
    -DBUILD_JAVA=OFF
    -DBUILD_SHARED_LIBS=OFF
    -DWITH_OPENEXR=OFF
    -DWITH_PROTOBUF=OFF
    -DWITH_ADE=OFF
  DEPENDS openblas
)

ExternalProject_Add(
  mxnet
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/mxnet
  PATCH_COMMAND
    COMMAND ${CMAKE_COMMAND} -E chdir <SOURCE_DIR> git reset --hard
    COMMAND ${CMAKE_COMMAND} -E chdir <SOURCE_DIR> patch -p1 < ${CMAKE_CURRENT_SOURCE_DIR}/mxnet.patch
  PREFIX ${CMAKE_BINARY_DIR}/mxnet
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/install_root
    -DCMAKE_INSTALL_INCLUDEDIR=${CMAKE_BINARY_DIR}/install_root/include
    -DCMAKE_BUILD_TYPE=Distribution
    -DUSE_BLAS=open
    -DUSE_CUDA=OFF
    -DUSE_OPENMP=OFF
    ${mxnet_CMAKE_ARGS}
  DEPENDS opencv openblas
)
