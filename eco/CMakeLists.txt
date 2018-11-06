cmake_minimum_required(VERSION 2.8)
set(EXECUTABLE_OUTPUT_PATH ${CMAKE_CURRENT_LIST_DIR})
include_directories(${CMAKE_CURRENT_LIST_DIR})

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
add_executable(eco_demo ${eco_demo_srcs})
target_link_libraries(eco_demo eco m ${OpenCV_LIBS})



