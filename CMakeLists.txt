cmake_minimum_required(VERSION 3.5)
project(cm-lirv)

set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS_RELEASE "-O3 -Wall -g -Wno-sign-compare -Wno-unused -Wno-comment -pthread")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -msse4.2")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(NOT DEFINED CMAKE_SUPPRESS_DEVELOPER_WARNINGS)
  set(CMAKE_SUPPRESS_DEVELOPER_WARNINGS 1 CACHE INTERNAL "No dev warnings")
endif()

find_package(OpenMP)
if (OPENMP_FOUND)
    set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
    set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

# FIND_PACKAGE(Boost REQUIRED COMPONENTS thread)
FIND_PACKAGE(Boost REQUIRED COMPONENTS filesystem iostreams program_options system serialization thread)
if(Boost_FOUND)
	INCLUDE_DIRECTORIES(${Boost_INCLUDE_DIRS})
	LINK_DIRECTORIES(${Boost_LIBRARY_DIRS})
endif()

find_package(Eigen3 REQUIRED)
find_package(Ceres REQUIRED)
# find_package(Sophus REQUIRED)
find_package(OpenCV REQUIRED)
find_package(PCL REQUIRED)
# find_package(PCL 1.13.0 REQUIRED)

find_package(yaml-cpp REQUIRED)
find_package(PkgConfig REQUIRED)
find_package(glog REQUIRED)

aux_source_directory(${CMAKE_CURRENT_SOURCE_DIR}/src/spline/core SPLINE_CORE_SRC_FILES)

add_library(
  ${PROJECT_NAME} SHARED
  # source files
  ${SPLINE_CORE_SRC_FILES}
)

target_include_directories(
  ${PROJECT_NAME} PUBLIC
  # include
  ${CMAKE_CURRENT_SOURCE_DIR}/include
  ${YAML_INCLUDE_DIRS}
  ${CERES_INCLUDE_DIRS}
  ${Sophus_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  ${PCL_INCLUDE_DIRS}
)

target_link_libraries(
  ${PROJECT_NAME} PUBLIC
  ${YAML_CPP_LIBRARIES}
  ${Boost_LIBRARIES}
  ${OpenCV_LIBS}
  ${CERES_LIBRARIES}
  ${PCL_LIBRARIES}
  ${Boost_LIBRARIES}
  ${Boost_FILESYSTEM_LIBRARY}
  ${Boost_SERIALIZATION_LIBRARY} # serialization
  glog::glog
)

add_executable(spline_prog tool/spline_main.cpp)
target_link_libraries(
  spline_prog
  ${PROJECT_NAME}
)