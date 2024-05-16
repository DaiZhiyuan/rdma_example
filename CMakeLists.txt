cmake_minimum_required (VERSION 2.8.12)

project (rdma-example)

set(PROJECT_SOURCE_DIR ${CMAKE_SOURCE_DIR}/src)
set(CMAKE_BINARY_DIR ${CMAKE_SOURCE_DIR}/bin)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR})

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(CMAKE_BUILD_TYPE Debug) # or debug if you need to use gdb on it.

set(CMAKE_CXX_COMPILER g++) # or clang++
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0 -ggdb") # get crazy here
#add_compile_options("-DRDMA_DEBUG")

find_library(IBVERBS_LIBRARY ibverbs)
find_library(RDMACM_LIBRARY rdmacm)

link_libraries(pthread ${IBVERBS_LIBRARY} ${RDMACM_LIBRARY})

include_directories("${PROJECT_SOURCE_DIR}")

add_executable(rdma_server ${PROJECT_SOURCE_DIR}/rdma_common.c ${PROJECT_SOURCE_DIR}/rdma_server.c)
add_executable(rdma_client ${PROJECT_SOURCE_DIR}/rdma_common.c ${PROJECT_SOURCE_DIR}/rdma_client.c)

