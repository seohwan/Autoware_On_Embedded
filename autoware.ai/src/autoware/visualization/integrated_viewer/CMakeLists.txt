cmake_minimum_required(VERSION 2.8.7)
project(integrated_viewer)

find_package(OpenCV REQUIRED)

find_package(autoware_build_flags REQUIRED)
find_package(catkin REQUIRED COMPONENTS
  autoware_msgs
  cv_bridge
  image_transport
  roscpp
  rviz
  sensor_msgs
  std_msgs
)

set(CMAKE_CXX_FLAGS "-Wall ${CMAKE_CXX_FLAGS}")

catkin_package(
  DEPENDS rviz
)

if(rviz_QT_VERSION VERSION_LESS "5")
  find_package(Qt4 ${rviz_QT_VERSION} EXACT REQUIRED QtCore QtGui)
  include(${QT_USE_FILE})
  # Qt's user-interface compiler.
  qt4_wrap_ui(UI_HEADERS
    node/traffic_light_plugin/form.ui
    node/image_viewer_plugin/image_viewer_form.ui
    node/data_rate_checker_plugin/data_rate_checker_form.ui
  )

  # Here we specify which header files need to be run through "moc",
  # Qt's meta-object compiler.
  qt4_wrap_cpp(MOC_FILES
    node/traffic_light_plugin/traffic_light_plugin.h
    node/image_viewer_plugin/image_viewer_plugin.h
    node/data_rate_checker_plugin/data_rate_checker_plugin.h
  )
else()
  find_package(Qt5 ${rviz_QT_VERSION} EXACT REQUIRED Core Widgets Quick)
  set(QT_LIBRARIES Qt5::Widgets)
  # Qt's user-interface compiler.
  qt5_wrap_ui(UI_HEADERS
    node/traffic_light_plugin/form.ui
    node/image_viewer_plugin/image_viewer_form.ui
    node/data_rate_checker_plugin/data_rate_checker_form.ui
  )

  # Here we specify which header files need to be run through "moc",
  # Qt's meta-object compiler.
  qt5_wrap_cpp(MOC_FILES
    node/traffic_light_plugin/traffic_light_plugin.h
    node/image_viewer_plugin/image_viewer_plugin.h
    node/data_rate_checker_plugin/data_rate_checker_plugin.h
  )
endif()

include_directories(
  ${CMAKE_CURRENT_BINARY_DIR}
  ${CMAKE_CURRENT_SOURCE_DIR}
  ${catkin_INCLUDE_DIRS}
  include
  lib/
)

link_directories(${catkin_LIBRARY_DIRS})

# I prefer the Qt signals and slots to avoid defining "emit", "slots",
# etc because they can conflict with boost signals, so define QT_NO_KEYWORDS here.
add_definitions(-DQT_NO_KEYWORDS)


# Here we specify the list of source files, including the output of
# the previous command which is stored in ``${MOC_FILES}`` and ``${UI_HEADERS}``.
set(SOURCE_FILES
  lib/convert_image.h
  node/traffic_light_plugin/traffic_light_plugin.cpp
  node/image_viewer_plugin/image_viewer_plugin.cpp
  node/image_viewer_plugin/draw_rects.cpp
  node/image_viewer_plugin/draw_points.cpp
  node/image_viewer_plugin/draw_lane.cpp
  node/data_rate_checker_plugin/data_rate_checker_plugin.cpp
  ${MOC_FILES}
  ${UI_HEADERS}
)

# An rviz plugin is just a shared library, so here we declare the
# library to be called ``${PROJECT_NAME}`` (which is
# "rviz_plugin_tutorials", or whatever your version of this project
# is called) and specify the list of source files we collected above
# in ``${SOURCE_FILES}``.
#
add_library(${PROJECT_NAME}
  ${SOURCE_FILES}
)

set_target_properties(${PROJECT_NAME}
  PROPERTIES
  COMPILE_FLAGS
  "-DIMAGE_VIEWER_DEFAULT_PATH=${CMAKE_CURRENT_SOURCE_DIR}/node/image_viewer_plugin/"
)

## Specify libraries to link a library or executable target against
target_link_libraries(${PROJECT_NAME}
  ${QT_LIBRARIES}
  ${catkin_LIBRARIES}
  ${OpenCV_LIBS}
)

add_dependencies(${PROJECT_NAME}
  ${catkin_EXPORTED_TARGETS}
)

install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

install(FILES rviz_plugin_description.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(
  FILES
    node/image_viewer_plugin/autoware_logo.png
    node/image_viewer_plugin/car.png
    node/image_viewer_plugin/pedestrian.png
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}/node/image_viewer_plugin/
)
