cmake_minimum_required(VERSION 3.15)
project(mxnet-build)
include(ExternalProject)

set(INSTALL_ROOT_PATH ${CMAKE_BINARY_DIR}/install_root)
list(APPEND CMAKE_PREFIX_PATH ${INSTALL_ROOT_PATH})
set(COMMON_CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${INSTALL_ROOT_PATH})

set(openblas_CMAKE_ARGS
  ${COMMON_CMAKE_ARGS}
  -DCMAKE_BUILD_TYPE=Release
  -DBUILD_STATIC_LIBS=ON
  -DDYNAMIC_ARCH=ON
  -DNOFORTRAN=1
)
if(WIN32)
  list(PREPEND openblas_CMAKE_ARGS
    -DCMAKE_C_COMPILER=clang-cl
    -DCMAKE_CXX_COMPILER=clang-cl
    -DCMAKE_MSVC_RUNTIME_LIBRARY=MultiThreadedDLL
  )
endif()
ExternalProject_Add(
  openblas
  GIT_REPOSITORY https://github.com/OpenMathLib/OpenBLAS.git
  GIT_TAG v0.3.29
  GIT_SHALLOW TRUE
  PREFIX ${CMAKE_BINARY_DIR}/openblas
  CMAKE_ARGS ${openblas_CMAKE_ARGS}
)

ExternalProject_Add(
  opencv
  GIT_REPOSITORY https://github.com/opencv/opencv.git
  GIT_TAG 4.11.0
  GIT_SHALLOW TRUE
  PREFIX ${CMAKE_BINARY_DIR}/opencv
  CMAKE_ARGS
    ${COMMON_CMAKE_ARGS}
    -DCMAKE_BUILD_TYPE=Release
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
    COMMAND ${CMAKE_COMMAND} -E chdir <SOURCE_DIR> git apply < ${CMAKE_CURRENT_SOURCE_DIR}/mxnet.patch
  PREFIX ${CMAKE_BINARY_DIR}/mxnet
  CMAKE_ARGS
    ${COMMON_CMAKE_ARGS}
    -DCMAKE_INSTALL_INCLUDEDIR=${INSTALL_ROOT_PATH}/include  # Fix install issue for 3rdparty/mkldnn
    -DCMAKE_BUILD_TYPE=Distribution
    -DUSE_BLAS=open
    -DUSE_CUDA=OFF
    -DUSE_OPENMP=OFF
    -DOpenCV_STATIC=ON
    ${mxnet_CMAKE_ARGS}
  DEPENDS opencv openblas
)
