cmake_minimum_required(VERSION 2.8.3)
project(points_downsampler)

if("${CMAKE_SYSTEM_PROCESSOR}" STREQUAL "aarch64")
  add_definitions(-D__aarch64__)
endif()

find_package(autoware_build_flags REQUIRED)

find_package(catkin REQUIRED COMPONENTS
  autoware_config_msgs
  message_generation
  pcl_conversions
  pcl_ros
  roscpp
  sensor_msgs
  velodyne_pointcloud
  rubis_lib
  rubis_msgs
)

add_message_files(
  FILES
  PointsDownsamplerInfo.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(
  CATKIN_DEPENDS
  message_runtime
  pcl_conversions
  pcl_ros
  sensor_msgs
  rubis_lib
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)
SET(CMAKE_CXX_FLAGS "-O2 -g -Wall ${CMAKE_CXX_FLAGS}")

add_executable(voxel_grid_filter nodes/voxel_grid_filter/voxel_grid_filter.cpp)
add_executable(ring_filter nodes/ring_filter/ring_filter.cpp)
add_executable(distance_filter nodes/distance_filter/distance_filter.cpp)
add_executable(random_filter nodes/random_filter/random_filter.cpp)

add_dependencies(voxel_grid_filter ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
add_dependencies(ring_filter ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
add_dependencies(distance_filter ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})
add_dependencies(random_filter ${${PROJECT_NAME}_EXPORTED_TARGETS} ${catkin_EXPORTED_TARGETS})

target_link_libraries(voxel_grid_filter ${catkin_LIBRARIES})
target_link_libraries(ring_filter ${catkin_LIBRARIES})
target_link_libraries(distance_filter ${catkin_LIBRARIES})
target_link_libraries(random_filter ${catkin_LIBRARIES})

install(
  TARGETS
    voxel_grid_filter
    ring_filter
    distance_filter
    random_filter
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY launch/
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/launch
  PATTERN ".svn" EXCLUDE
)
