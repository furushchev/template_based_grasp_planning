cmake_minimum_required(VERSION 2.8.3)
project(grasp_template_planning)

find_package(catkin REQUIRED COMPONENTS
  geometry_msgs
  grasp_template
  image_geometry
  pcl_ros
  pcl_conversions
  polled_camera
  rosbag
  roscpp
  sensor_msgs
  std_msgs
  tabletop_segmenter
  tf
  visualization_msgs
  message_generation
)

find_package(Boost REQUIRED COMPONENTS system)

find_package(Eigen REQUIRED)
include_directories(${EIGEN_INCLUDE_DIRS})
add_definitions(${EIGEN_DEFINITIONS})

find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})

find_package(OpenMP REQUIRED)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} ${OpenMP_EXE_LINKER_FLAGS}")

add_message_files(
  FILES
  DoubleVector.msg
  GraspAnalysis.msg
  GraspLog.msg
  MatchAnalysis.msg
  SimpleLabel.msg
)

add_service_files(
  FILES
  TabletopObject.srv
  )

generate_messages(
  DEPENDENCIES
  geometry_msgs
  grasp_template
  sensor_msgs
  std_msgs
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS geometry_msgs grasp_template image_geometry pcl_ros pcl_conversions polled_camera rosbag roscpp sensor_msgs std_msgs tabletop_segmenter tf visualization_msgs message_runtime
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${boost_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME}
  src/demonstration_parser.cpp
  src/grasp_planning_params.cpp
  src/grasp_demo_library.cpp
  src/planning_pipeline.cpp
  src/visualization.cpp
  src/grasp_pool.cpp
  src/demo_writer.cpp
  src/template_matching.cpp
  src/object_detection_listener.cpp
  src/image_listener.cpp
  src/pca_grasp_container.cpp
  src/pca_planning_pipe.cpp)

set(DEPEND_LIBRARIES
  ${catkin_LIBRARIES}
  ${Boost_LIBRARIES}
  ${OpenCV_LIBRARIES}
  ${Eigen_LIBRARIES})

target_link_libraries(${PROJECT_NAME}
  ${DEPEND_LIBRARIES}
  )

add_executable(user_demonstration_recorder
  src/user_demonstration_recorder.cpp)

target_link_libraries(user_demonstration_recorder
  ${PROJECT_NAME}
  ${DEPEND_LIBRARIES})

add_executable(palm_marker
  config/palm_marker.cpp)

target_link_libraries(palm_marker
  ${PROJECT_NAME}
  ${DEPEND_LIBRARIES})

add_executable(generate_grasp_library
  src/generate_grasp_library.cpp)

target_link_libraries(generate_grasp_library
  ${PROJECT_NAME}
  ${DEPEND_LIBRARIES})

install(TARGETS ${PROJECT_NAME} palm_marker generate_grasp_library
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  PATTERN ".svn" EXCLUDE)
