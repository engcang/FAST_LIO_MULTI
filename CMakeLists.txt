cmake_minimum_required(VERSION 2.8.3)
project(fast_lio_multi)

add_definitions(-DROOT_DIR=\"${CMAKE_CURRENT_SOURCE_DIR}/\")
ADD_COMPILE_OPTIONS(-std=c++14 )
set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_FLAGS "-std=c++14 -O3 -g -pthread -fexceptions" )
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS}")

find_package(OpenMP QUIET)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}   ${OpenMP_C_FLAGS}")

message("Current CPU archtecture: ${CMAKE_SYSTEM_PROCESSOR}")
if(CMAKE_SYSTEM_PROCESSOR MATCHES "(x86)|(X86)|(amd64)|(AMD64)" )
  include(ProcessorCount)
  ProcessorCount(N)
  message("Processer number:  ${N}")
  if(N GREATER 12)
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM=6)
    message("core for MP: 6")
  elseif(N GREATER 6)
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM=4)
    message("core for MP: 4")
  elseif(N GREATER 4)
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM=3)
    message("core for MP: 3")
  elseif(N GREATER 3)
    add_definitions(-DMP_EN)
    add_definitions(-DMP_PROC_NUM=2)
    message("core for MP: 2")
  else()
    add_definitions(-DMP_PROC_NUM=1)
  endif()
else()
  add_definitions(-DMP_PROC_NUM=1)
endif()


find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  nav_msgs
  sensor_msgs
  roscpp
  rospy
  std_msgs
  pcl_ros
  tf
  livox_ros_driver
  eigen_conversions
)
find_package(Eigen3 REQUIRED)
find_package(PCL 1.8 REQUIRED)
message(Eigen: ${EIGEN3_INCLUDE_DIR})

include_directories(
	${catkin_INCLUDE_DIRS} 
  ${EIGEN3_INCLUDE_DIR}
  ${PCL_INCLUDE_DIRS}
  include)

catkin_package(
  CATKIN_DEPENDS geometry_msgs nav_msgs roscpp std_msgs
  DEPENDS EIGEN3 PCL
  INCLUDE_DIRS
)


#### Common library
add_library(${PROJECT_NAME}_ikd_preprocess include/ikd-Tree/ikd_Tree.cpp src/preprocess.cpp)
target_link_libraries(${PROJECT_NAME}_ikd_preprocess ${catkin_LIBRARIES} ${PCL_LIBRARIES})
#### Bundle
add_executable(${PROJECT_NAME}_bundle src/laserMapping_bundle.cpp)
target_link_libraries(${PROJECT_NAME}_bundle ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${PROJECT_NAME}_ikd_preprocess)
add_executable(${PROJECT_NAME}_wrong_bundle src/laserMapping_wrong_bundle.cpp)
target_link_libraries(${PROJECT_NAME}_wrong_bundle ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${PROJECT_NAME}_ikd_preprocess)
#### Async
add_executable(${PROJECT_NAME}_async src/laserMapping_async.cpp)
target_link_libraries(${PROJECT_NAME}_async ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${PROJECT_NAME}_ikd_preprocess)
#### Adaptively mixed
add_executable(${PROJECT_NAME}_adaptive src/laserMapping_adaptive.cpp)
target_link_libraries(${PROJECT_NAME}_adaptive ${catkin_LIBRARIES} ${PCL_LIBRARIES} ${PROJECT_NAME}_ikd_preprocess)