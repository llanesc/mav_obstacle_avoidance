cmake_minimum_required(VERSION 3.5)
project(mav_obstacle_avoidance)

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()


if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()
set(LIBRARY_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/lib)


add_compile_definitions(EIGEN_USE_BLAS EIGEN_USE_LAPACKE )
# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(px4_msgs REQUIRED)
find_package(mav_asif_msgs REQUIRED)
find_package(px4_ros_com REQUIRED)
find_package(OpenMP REQUIRED)
find_package(Eigen3 3.4 REQUIRED NO_MODULE)
find_package(LAPACK REQUIRED)
find_package(BLAS REQUIRED)
find_package(OsqpEigen REQUIRED)
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")
set(OMP_NUM_THREADS 16)

option(RUN_SITL "Target build is simulation" OFF)## RUN IN SIMULATION

if(RUN_SITL)
  message(STATUS "RUN_SITL ACTIVE")
  add_definitions(-DRUN_SITL=1)
endif()

add_library(mav_asif_util SHARED src/lib/mav_asif_util.cpp)
ament_target_dependencies(mav_asif_util Eigen3 rclcpp px4_msgs mav_asif_msgs px4_ros_com)

target_include_directories(mav_asif_util PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/${PROJECT_NAME}>
        $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
        ${Eigen3_INCLUDE_DIRS}
        )
target_link_libraries(mav_asif_util Eigen3::Eigen OsqpEigen::OsqpEigen ${rclcpp_LIBRARIES} ${LAPACK_LIBRARIES} ${BLAS_LIBRARIES})

add_library(mav_avoidance_control_router SHARED src/lib/mav_avoidance_control_router.cpp)
ament_target_dependencies(mav_avoidance_control_router Eigen3 rclcpp px4_msgs mav_asif_msgs px4_ros_com)
target_include_directories(mav_avoidance_control_router PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/${PROJECT_NAME}>
        $<INSTALL_INTERFACE:include/${PROJECT_NAME}>
        ${Eigen3_INCLUDE_DIRS}
        )
target_link_libraries(mav_avoidance_control_router Eigen3::Eigen OsqpEigen::OsqpEigen ${rclcpp_LIBRARIES} mav_asif_util ${LAPACK_LIBRARIES} ${BLAS_LIBRARIES})

add_executable(mav_obstacle_avoidance_node src/mav_obstacle_avoidance_node.cpp)
target_link_libraries(mav_obstacle_avoidance_node mav_avoidance_control_router)

install(TARGETS mav_obstacle_avoidance_node
        DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY
        config
        DESTINATION share/${PROJECT_NAME}
        )

install(DIRECTORY
        launch
        DESTINATION share/${PROJECT_NAME}
        )

## INSTALL ##
#ament_export_dependencies(ament_cmake rclcpp rosidl_default_runtime eigen3_cmake_module Eigen3 px4_msgs)
#ament_export_targets(export_mav_asif_util HAS_LIBRARY_TARGET)
#ament_export_libraries(mav_asif_util)
#
#install(TARGETS mav_asif_util
#        EXPORT export_mav_asif_util
#        ARCHIVE DESTINATION lib
#        LIBRARY DESTINATION lib
#        RUNTIME DESTINATION bin
#        INCLUDES DESTINATION include
#        )
#
## Install header files
#install(DIRECTORY include/${PROJECT_NAME}/
#        DESTINATION include/${PROJECT_NAME}
#)
#
## Install artifacts
#install(DIRECTORY cmake
#        DESTINATION share/${PROJECT_NAME}
#)

ament_package()
