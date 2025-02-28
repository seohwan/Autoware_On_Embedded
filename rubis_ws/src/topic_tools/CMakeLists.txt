cmake_minimum_required(VERSION 3.0.2)
project(topic_tools)

if(NOT WIN32)
  set_directory_properties(PROPERTIES COMPILE_OPTIONS "-Wall;-Wextra")
endif()

find_package(catkin COMPONENTS cpp_common message_generation rosconsole roscpp rostime std_msgs xmlrpcpp rubis_lib)
catkin_python_setup()

include_directories(include)
include_directories(${catkin_INCLUDE_DIRS})

add_service_files(DIRECTORY srv
  FILES
  MuxAdd.srv
  MuxDelete.srv
  MuxList.srv
  MuxSelect.srv
  DemuxAdd.srv
  DemuxDelete.srv
  DemuxList.srv
  DemuxSelect.srv)

generate_messages(DEPENDENCIES std_msgs)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES topic_tools
  CATKIN_DEPENDS message_runtime rosconsole roscpp std_msgs xmlrpcpp
 )

catkin_add_env_hooks(20.transform SHELLS bash DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/env-hooks)

add_library(topic_tools src/shape_shifter.cpp src/parse.cpp)
target_link_libraries(topic_tools ${catkin_LIBRARIES})

add_executable(switch_mux src/switch_mux.cpp)
target_link_libraries(switch_mux topic_tools ${catkin_LIBRARIES})

add_executable(mux src/mux.cpp)
target_link_libraries(mux topic_tools ${catkin_LIBRARIES})
add_dependencies(topic_tools ${${PROJECT_NAME}_EXPORTED_TARGETS})

add_executable(demux src/demux.cpp)
target_link_libraries(demux topic_tools ${catkin_LIBRARIES})
add_dependencies(topic_tools ${${PROJECT_NAME}_EXPORTED_TARGETS})

add_executable(relay src/relay.cpp)
target_link_libraries(relay topic_tools ${catkin_LIBRARIES})

add_executable(drop src/drop.cpp)
target_link_libraries(drop topic_tools ${catkin_LIBRARIES})

add_executable(throttle src/throttle.cpp)
target_link_libraries(throttle topic_tools ${catkin_LIBRARIES})

install(TARGETS topic_tools
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})

install(TARGETS switch_mux mux demux relay drop throttle
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  FILES_MATCHING PATTERN "*.h")

catkin_install_python(PROGRAMS
  scripts/mux_add
  scripts/mux_delete
  scripts/mux_list
  scripts/mux_select
  scripts/relay_field
  scripts/transform
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

#Testing
if(CATKIN_ENABLE_TESTING)
  find_package(rostest)
  find_package(rosunit)
  catkin_add_gtest(${PROJECT_NAME}-utest test/utest.cpp)
  if(TARGET ${PROJECT_NAME}-utest)
    target_link_libraries(${PROJECT_NAME}-utest topic_tools)
  endif()

  if(GTEST_FOUND)
    include_directories(${GTEST_INCLUDE_DIRS})
    add_executable(${PROJECT_NAME}-test_shapeshifter EXCLUDE_FROM_ALL test/test_shapeshifter.cpp)
    target_link_libraries(${PROJECT_NAME}-test_shapeshifter ${GTEST_LIBRARIES} topic_tools)
  endif()
  if(TARGET tests)
    add_dependencies(tests ${PROJECT_NAME}-test_shapeshifter)
  endif()

  add_rostest(test/shapeshifter.test)
  add_rostest(test/throttle.test)
  add_rostest(test/throttle_simtime.test)
  add_rostest(test/throttle_simtime_loop.test)
  add_rostest(test/drop.test)
  add_rostest(test/relay.test)
  add_rostest(test/relay_stealth.test)
  add_rostest(test/lazy_transport.test)
  add_rostest(test/mux_initial_none.test)
  add_rostest(test/mux_initial_other.test)
  add_rostest(test/transform.test)
  ## Latched test disabled until underlying issue in roscpp is resolved,
  ## #3385, #3434.
  #rosbuild_add_rostest(test/relay_latched.test)
  add_rostest(test/mux.test)
  add_rostest(test/switch_mux.test)
  add_rostest(test/switch_mux_leading_slash.test)
  add_rostest(test/switch_mux_none.test)
  #rosbuild_add_rostest(test/mux_add.test)

  catkin_add_nosetests(test/args.py)
endif()
