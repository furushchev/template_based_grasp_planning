cmake_minimum_required(VERSION 2.8.3)
project(tabletop_segmenter)

find_package(catkin REQUIRED COMPONENTS
  pcl_ros
  pcl_conversions
  cv_bridge
  geometry_msgs
  image_geometry
  roscpp
  sensor_msgs
  tf
  visualization_msgs
  message_generation
)

find_package(Boost REQUIRED COMPONENTS system)

find_package(OpenCV REQUIRED)

add_message_files(
  FILES
  Table.msg
)

add_service_files(
  FILES
  TabletopSegmentation.srv
)

generate_messages(
  DEPENDENCIES
  geometry_msgs
  sensor_msgs
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES marker_generator
  CATKIN_DEPENDS cv_bridge geometry_msgs image_geometry roscpp sensor_msgs tf visualization_msgs pcl_ros pcl_conversions
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)

set(DEPEND_LIBRARIES
  ${catkin_LIBRARIES}
  ${Boost_LIBRARIES}
  ${OpenCV_LIBRARIES})

add_library(marker_generator
  src/marker_generator.cpp
)

target_link_libraries(marker_generator
  ${DEPEND_LIBRARIES})

add_executable(tabletop_segmentation
  src/tabletop_segmentation.cpp
  src/utilities.cpp
)

add_dependencies(tabletop_segmentation
  tabletop_segmenter_generate_messages_cpp
  marker_generator)

target_link_libraries(tabletop_segmentation
  ${DEPEND_LIBRARIES}
  marker_generator
)

add_executable(ping_tabletop_node
  src/ping_tabletop_node.cpp
)

target_link_libraries(ping_tabletop_node
  ${DEPEND_LIBRARIES}
)

add_dependencies(ping_tabletop_node ${PROJECT_NAME}_gencpp)

install(TARGETS marker_generator tabletop_segmentation ping_tabletop_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  PATTERN ".svn" EXCLUDE)
