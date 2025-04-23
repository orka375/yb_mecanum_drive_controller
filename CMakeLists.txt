cmake_minimum_required(VERSION 3.16)
project(this_mecanum_drive_controller)

find_package(ros2_control_cmake REQUIRED)
set_compiler_options()
export_windows_symbols()

# Dependencies
set(THIS_PACKAGE_INCLUDE_DEPENDS
  controller_interface
  hardware_interface
  generate_parameter_library
  nav_msgs
  pluginlib
  rclcpp
  rclcpp_lifecycle
  realtime_tools
  std_srvs
  tf2
  tf2_geometry_msgs
  tf2_msgs
)

find_package(ament_cmake REQUIRED)
find_package(backward_ros REQUIRED)
foreach(Dependency IN ITEMS ${THIS_PACKAGE_INCLUDE_DEPENDS})
  find_package(${Dependency} REQUIRED)
endforeach()

# Parameter generation
generate_parameter_library(this_mecanum_drive_controller_parameters
  src/this_mecanum_drive_controller.yaml
)

# Library
add_library(
  this_mecanum_drive_controller
  SHARED
  src/mecanum_drive_controller.cpp
  src/odometry.cpp
)
target_compile_features(this_mecanum_drive_controller PUBLIC cxx_std_17)
target_include_directories(this_mecanum_drive_controller PUBLIC
  "$<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>"
  "$<INSTALL_INTERFACE:include/${PROJECT_NAME}>")
target_link_libraries(this_mecanum_drive_controller PUBLIC
  this_mecanum_drive_controller_parameters)
ament_target_dependencies(this_mecanum_drive_controller PUBLIC ${THIS_PACKAGE_INCLUDE_DEPENDS})

# Plugin export
pluginlib_export_plugin_description_file(
  controller_interface this_mecanum_drive_controller.xml)

# Tests (optional - leave as-is unless you're renaming the test files too)
if(BUILD_TESTING)
  find_package(ament_cmake_gmock REQUIRED)
  find_package(controller_manager REQUIRED)
  find_package(hardware_interface REQUIRED)
  find_package(ros2_control_test_assets REQUIRED)

  add_definitions(-DTEST_FILES_DIRECTORY="${CMAKE_CURRENT_SOURCE_DIR}/test")

  ament_add_gmock(test_load_this_mecanum_drive_controller test/test_load_mecanum_drive_controller.cpp)
  ament_target_dependencies(test_load_this_mecanum_drive_controller
    controller_manager
    hardware_interface
    ros2_control_test_assets
  )

  add_rostest_with_parameters_gmock(
    test_this_mecanum_drive_controller test/test_mecanum_drive_controller.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/test/mecanum_drive_controller_params.yaml)
  target_include_directories(test_this_mecanum_drive_controller PRIVATE include)
  target_link_libraries(test_this_mecanum_drive_controller this_mecanum_drive_controller)
  ament_target_dependencies(
    test_this_mecanum_drive_controller
    controller_interface
    hardware_interface
  )

  add_rostest_with_parameters_gmock(
    test_this_mecanum_drive_controller_preceeding test/test_mecanum_drive_controller_preceeding.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/test/mecanum_drive_controller_preceeding_params.yaml)
  target_include_directories(test_this_mecanum_drive_controller_preceeding PRIVATE include)
  target_link_libraries(test_this_mecanum_drive_controller_preceeding this_mecanum_drive_controller)
  ament_target_dependencies(
    test_this_mecanum_drive_controller_preceeding
    controller_interface
    hardware_interface
  )
endif()

# Install headers
install(
  DIRECTORY include/
  DESTINATION include/this_mecanum_drive_controller
)

# Install library and parameters
install(
  TARGETS this_mecanum_drive_controller this_mecanum_drive_controller_parameters
  EXPORT export_this_mecanum_drive_controller
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
)

# Export targets
ament_export_targets(export_this_mecanum_drive_controller HAS_LIBRARY_TARGET)
ament_export_dependencies(${THIS_PACKAGE_INCLUDE_DEPENDS})
ament_package()
