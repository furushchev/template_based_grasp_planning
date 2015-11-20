cmake_minimum_required(VERSION 2.8.3)
project(pr2_template_based_grasping)

find_package(catkin REQUIRED COMPONENTS
  actionlib
  arm_navigation_msgs
  grasp_template
  grasp_template_planning
  interactive_markers
  object_manipulation_msgs
  pcl_ros
  pcl_conversions
#  pr2_arm_navigation_tutorials
  pr2_controllers_msgs
  roscpp
#  tabletop_collision_map_processing
  tabletop_segmenter
  visualization_msgs
  message_generation
)

find_package(Boost REQUIRED COMPONENTS system)

add_service_files(
  FILES
  PlanningFeedback.srv
  PlanningSummary.srv
  PlanningVisualization.srv
  )

generate_messages(
  DEPENDENCIES
  object_manipulation_msgs
  visualization_msgs
  grasp_template_planning
  geometry_msgs
  sensor_msgs
)

catkin_package(
  INCLUDE_DIRS include
  LIBRARIES ${PROJECT_NAME}
  CATKIN_DEPENDS actionlib arm_navigation_msgs grasp_template grasp_template_planning interactive_markers object_manipulation_msgs pcl_ros pcl_conversions pr2_arm_navigation_tutorials pr2_controllers_msgs roscpp tabletop_collision_map_processing tabletop_segmenter visualization_msgs
)

include_directories(
  include
  ${catkin_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
)

set(DEPEND_LIBRARIES
  ${catkin_LIBRARIES}
  ${Boost_LIBRARIES})

add_library(${PROJECT_NAME}
  src/arm_movement_handler.cpp
  src/head_movement_handler.cpp
  src/grasp_planning_server.cpp
  src/interactive_candidate_filter.cpp)
target_link_libraries(${PROJECT_NAME}
  ${DEPEND_LIBRARIES})

add_executable(template_grasp_planning_server
  src/template_grasp_planning_server.cpp
)
target_link_libraries(template_grasp_planning_server
  ${PROJECT_NAME})

# add_executable(template_grasp_feedback_ui
#   src/template_grasp_feedback_ui.cpp
# )
# target_link_libraries(template_grasp_feedback_ui ${PROJECT_NAME})

# add_executable(template_grasp_execution_ui
#   src/template_grasp_execution_ui.cpp
# )
# target_link_libraries(template_grasp_execution_ui ${PROJECT_NAME})

install(TARGETS ${PROJECT_NAME} # template_grasp_feedback_ui template_grasp_execution_ui
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
  PATTERN ".svn" EXCLUDE)
