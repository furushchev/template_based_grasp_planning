cmake_minimum_required(VERSION 2.8.3)
project(grasp_template)

find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  pcl_ros
  roscpp
  std_msgs
  sensor_msgs
  tf
  visualization_msgs
)

find_package(Boost REQUIRED COMPONENTS system)

find_package(Eigen REQUIRED)
include_directories(${EIGEN_INCLUDE_DIRS})
add_definitions(${EIGEN_DEFINITIONS})

add_message_files(
  FILES
  Heightmap.msg
)

generate_messages(
  DEPENDENCIES
  std_msgs
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS geometry_msgs pcl_ros roscpp sensor_msgs tf visualization_msgs
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME}
  src/grasp_template.cpp
  src/template_heightmap.cpp
  src/height_value_extractor.cpp
  src/heightmap_sampling.cpp
  src/heightmap_difference.cpp
  src/dismatch_measure.cpp
  src/grasp_template_params.cpp
)

target_link_libraries(${PROJECT_NAME}
  ${catkin_LIBRARIES}
)

install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
  )

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  PATTERN ".svn" EXCLUDE)
