cmake_minimum_required(VERSION 2.8)
project(eco)

set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_LIST_DIR})
include_directories(${CMAKE_CURRENT_LIST_DIR})
set( LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib )
link_directories( ${PROJECT_BINARY_DIR}/lib )

set(CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS ON)
option(GPU         		  "use GPU"                 OFF)
option(OPENCV      		  "use OPENCV"              ON)
option(OPENMP      		  "use OPENMP"              OFF)
option(CUDNN       		  "use CUDNN"               OFF)
option(MULTI_THREAD       "use MULTI_THREAD"        ON)
option(USE_FFTW		      "use fftw"		        OFF)

if(USE_FFTW)
	message(STATUS "use fftw to compile")
	add_definitions(-DUSE_FFTW)
	set(LIBS ${LIBS} -lfftw3)
endif()

#0:no-simd;1:sse-Intel;2:neon-TX2;
set(SIMD 1)
if(SIMD EQUAL 1)
	message(STATUS "use sse-Intel to compile")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse4")
	add_definitions(-DUSE_SIMD)
elseif(SIMD EQUAL 2)
	message(STATUS "use neon-TX2 to compile")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ffast-math -flto -march=armv8-a+crypto -mcpu=cortex-a57+crypto")
	add_definitions(-DUSE_SIMD)
	add_definitions(-DUSE_NEON)
elseif(SIMD EQUAL 0)
	message(STATUS "no-simd to compile")
endif()

if(MULTI_THREAD)
	add_definitions(-DUSE_MULTI_THREAD)
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -pthread")
endif()

if(NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE DEBUG)
endif()
message(STATUS "compile type：" ${CMAKE_BUILD_TYPE})

if(GPU)
	message(STATUS "use gpu to compile")
	find_package( CUDA QUIET REQUIRED )
	include(FindCUDA)
	add_definitions(-DGPU)
	set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS}
	-gencode arch=compute_50,code=sm_50
	-gencode arch=compute_52,code=sm_52
	-gencode arch=compute_53,code=sm_53
	-gencode arch=compute_61,code=sm_61
	-gencode arch=compute_62,code=sm_62)
	set(CUDA_NVCC_FLGAS_DEBUG "-g -G -Og")
	set(CUDA_NVCC_FLGAS_RELEASE "-Ofast -lineinfo")
	include_directories( ${CUDA_INCLUDE_DIRS} )
	link_directories( /usr/local/cuda/lib64 )
	set(LIBS ${LIBS} "-lcuda -lcudart -lcublas -lcurand -lstdc++")
	if(CUDNN)
		message(STATUS "use cudnn to compile")
		add_definitions(-DCUDNN)
		set(LIBS ${LIBS} "-lcudnn")
	endif()
endif()

if(OPENCV)
	message(STATUS "use opencv to compile")	
	find_package( OpenCV REQUIRED )
	add_definitions(-DOPENCV)
	include_directories(${OpenCV_INCLUDE_DIRS})
	link_directories ( ${OpenCV_LIB_DIR})
endif()
if(OPENMP)
	find_package(OpenMP REQUIRED)
	SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OPTIMAZE_FLAG} -Wall -Wno-unknown-pragmas -Wfatal-errors -fPIC")
set(CMAKE_C_FLAGS_DEBUG "-O0 -g")
set(CMAKE_C_FLAGS_RELEASE "-O3")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -lm -pthread -Wall")
set(CMAKE_CXX_FLAGS_DEBUG "-O0 -g")
set(CMAKE_CXX_FLAGS_RELEASE "-O3")

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 11)



file(GLOB eco_srcs
	${CMAKE_CURRENT_LIST_DIR}/*.hpp
	${CMAKE_CURRENT_LIST_DIR}/*.h
	${CMAKE_CURRENT_LIST_DIR}/eco.cc
	${CMAKE_CURRENT_LIST_DIR}/feature_extractor.cc
	${CMAKE_CURRENT_LIST_DIR}/feature_operator.cc
	${CMAKE_CURRENT_LIST_DIR}/ffttools.cc
	${CMAKE_CURRENT_LIST_DIR}/fhog.cc
	${CMAKE_CURRENT_LIST_DIR}/interpolator.cc
	${CMAKE_CURRENT_LIST_DIR}/metrics.cc
	${CMAKE_CURRENT_LIST_DIR}/optimize_scores.cc
	${CMAKE_CURRENT_LIST_DIR}/regularization_filter.cc
	${CMAKE_CURRENT_LIST_DIR}/sample_update.cc
	${CMAKE_CURRENT_LIST_DIR}/scale_filter.cc
	${CMAKE_CURRENT_LIST_DIR}/training.cc)

if(SIMD GREATER 0)
	set(eco_srcs ${eco_srcs} ${CMAKE_CURRENT_LIST_DIR}/gradient.cpp)
endif()

set(eco_demo_srcs ${CMAKE_CURRENT_LIST_DIR}/runecotracker.cc )

#compile eco library
add_library(eco SHARED ${eco_srcs})
target_link_libraries(eco ${LIBS})
#compile eco demo
add_executable(eco_demo.bin ${eco_demo_srcs})
target_link_libraries(eco_demo.bin eco m ${OpenCV_LIBS})



