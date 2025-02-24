cmake_minimum_required(VERSION 3.8)
project(locobot_control)

set (CMAKE_CXX_STANDARD 17)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(nav2_bringup REQUIRED)
find_package(nav2_util REQUIRED)

# source file dependencies
find_package(nav2_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(rclcpp_action REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(rclcpp REQUIRED)
find_package(tf2_ros REQUIRED)
find_package(locobot_control_interfaces REQUIRED)


#moveit dependencies
find_package(moveit_core REQUIRED)
find_package(moveit_ros_planning_interface REQUIRED)
find_package(moveit_msgs REQUIRED)
find_package(control_msgs REQUIRED)

# uncomment the following section in order to fill in
# further dependencies manually.
# find_package(<dependency> REQUIRED)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  # the following line skips the linter which checks for copyrights
  # comment the line when a copyright and license is added to all source files
  set(ament_cmake_copyright_FOUND TRUE)
  # the following line skips cpplint (only works in a git repo)
  # comment the line when this package is in a git repo and when
  # a copyright and license is added to all source files
  set(ament_cmake_cpplint_FOUND TRUE)
  ament_lint_auto_find_test_dependencies()
endif()


# State Machine
add_library( state_machine_component SHARED
  src/state_machine/StateMachine.cpp src/LocobotControl.cpp)

target_include_directories(state_machine_component PRIVATE 
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

target_compile_definitions(state_machine_component PRIVATE "STATE_MACHINE_BUILDING_DLL")
ament_target_dependencies(state_machine_component  
                              "rclcpp"
                              "rclcpp_components"
                              "geometry_msgs"
                              "nav2_msgs"
                              "moveit_core"
                              "moveit_ros_planning_interface"
                              "moveit_msgs"
                              "locobot_control_interfaces")

rclcpp_components_register_node(state_machine_component PLUGIN "StateMachine" EXECUTABLE state_machine)

install(TARGETS state_machine_component
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

# Arm sleep position
add_executable(arm_sleep_position src/ArmSleepPosition.cpp)
ament_target_dependencies(arm_sleep_position rclcpp
                               geometry_msgs 
                               nav2_msgs 
                               rclcpp_action 
                               rclcpp_components
                               moveit_core
                               moveit_ros_planning_interface
                               moveit_msgs)
target_include_directories(arm_sleep_position PUBLIC include)

# install targets
install(TARGETS
  arm_sleep_position
  state_machine
  DESTINATION lib/${PROJECT_NAME})

# install directories
install(DIRECTORY
    launch
    config
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
