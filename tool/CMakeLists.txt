cmake_minimum_required(VERSION 3.8.11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 14)
PROJECT(SDKTool)
set(VERSION_MAJOR 2)
set(VERSION_MINOR 0)
set(VERSION_TINY  8)
configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/Version.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/Version.h"
)

cmake_policy(SET CMP0053 NEW)
set(CMAKE_BUILD_TYPE ON)
set(CMAKE_BUILD_TYPE Release)
add_definitions(-O3)
add_compile_options(-Wall)
# add_definitions(-DJT128_256)

# 设置生成平台为 x64
if(MSVC)
set(CMAKE_GENERATOR_PLATFORM x64)
set(OPENSSL_ROOT_DIR "C:/Program Files/OpenSSL-Win64")
set(OPENSSL_INCLUDE_DIR ${OPENSSL_ROOT_DIR}/include)
endif()


include_directories("${PROJECT_BINARY_DIR}")
find_package(OpenSSL REQUIRED)
add_definitions(-DQT_MESSAGELOGCONTEXT)
add_subdirectory(../libhesai lbhesai.out)


include_directories(
  ../libhesai
  ../libhesai/Lidar
  ../libhesai/UdpParser
  ../libhesai/UdpParser/include
  ../libhesai/UdpParser/src
  ../libhesai/UdpProtocol
  ../libhesai/Source/include
  ../libhesai/Container/include
  ../libhesai/Container/src
  ../libhesai/UdpParserGpu
  ../libhesai/UdpParserGpu/include
  ../libhesai/UdpParserGpu/src
  ../libhesai/PtcClient/include
  ../libhesai/SerialClient/include
  ../libhesai/PtcParser
  ../libhesai/PtcParser/src
  ../libhesai/PtcParser/include
  ../libhesai/Logger/include
  ../libhesai/include
  ../driver
  ../third_party
  ./
  ${OPENSSL_INCLUDE_DIR}
)
link_directories(../libhesai/PtcClient/lib)


add_executable(packet_loss_tool
  ./packet_loss_tool.cc
)

target_link_libraries(packet_loss_tool
  source_lib
  container_lib
  ptcClient_lib
  serialClient_lib
  log_lib 
  ptcParser_lib
  ${OPENSSL_LIBRARIES}
)

find_package(PCL COMPONENTS common visualization io QUIET REQUIRED)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})

add_executable(pcl_tool
  ./pcl_tool.cc
)

target_link_libraries(pcl_tool
  source_lib
  container_lib
  ptcClient_lib
  serialClient_lib
  ptcParser_lib
  log_lib
  ${PCL_LIBRARIES}  
  ${OPENSSL_LIBRARIES}
)

find_package(LASlib)
if(${LASlib_FOUND})
    include_directories(${LASlib_INCLUDE_DIRS})
    link_directories(${LASlib_INCLUDE_DIRS})
    add_executable(las_tool
    ./las_tool.cc
    )

    target_link_libraries(las_tool
    source_lib
    container_lib
    ptcClient_lib
    ptcParser_lib
    log_lib
    LASlib
    ${PCL_LIBRARIES}  
    ${OPENSSL_LIBRARIES}
    )
else(${LASlib_FOUND})
    message("-- LASlib not found")
endif(${LASlib_FOUND})

add_executable(ptc_tool
  ./ptc_tool.cc
)

target_link_libraries(ptc_tool
  source_lib
  container_lib
  ptcClient_lib
  serialClient_lib
  ptcParser_lib
  log_lib
  ${PCL_LIBRARIES}  
  ${OPENSSL_LIBRARIES}
)

# No support cuda and pcl compile, the error case will occur
# error: unterminated argument list invoking macro "VTK_AUTOINIT1" 
# find_package(CUDA )
if(${CUDA_FOUND})
	set(CUDA_SOURCE_PROPERTY_FORMAT OBJ)
	set(CUDA_SEPARABLE_COMPILATION ON)
	include_directories(${CUDA_INCLUDE_DIRS})
	set(CUDA_PROPAGATE_HOST_FLAGS OFF)
	set(CUDA_NVCC_FLAGS -arch=sm_61;-O3;-G;-g;-std=c++14)#根据具体GPU性能更改算力参数
	link_directories($ENV{CUDA_PATH}/lib/x64)
  file(GLOB_RECURSE CURRENT_HEADERS  ../*.h ../*.hpp ../*.cuh)
  file(GLOB CURRENT_SOURCES  ../*.cpp ../*.cu)
  source_group("Include" FILES ${CURRENT_HEADERS}) 
  source_group("Source" FILES ${CURRENT_SOURCES}) 
  CUDA_ADD_EXECUTABLE(pcl_tool_gpu ${CURRENT_HEADERS} ${CURRENT_SOURCES} 
  ./pcl_tool.cu
  ../libhesai/UdpParserGpu/src/buffer.cu
  )
  target_link_libraries(pcl_tool_gpu 
  source_lib
  container_lib
  ptcClient_lib
  serialClient_lib
  log_lib
  ptcParser_lib
  ${OPENSSL_LIBRARIES}
  ${PCL_LIBRARIES} 
)
else(${CUDA_FOUND})
	MESSAGE(STATUS "cuda not found!")
endif(${CUDA_FOUND})

