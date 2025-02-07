cmake_minimum_required(VERSION 3.8)
project(shm_vision_demo)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# find dependencies
find_package(ament_cmake_auto REQUIRED)
ament_auto_find_build_dependencies()

find_package(OpenCV REQUIRED)
find_package(iceoryx_posh CONFIG REQUIRED)

include_directories(include)
include_directories(${OpenCV_INCLUDE_DIRS})

set(msg_files
  "msg/Image.msg"
)

rosidl_generate_interfaces(${PROJECT_NAME}
  ${msg_files}
)

ament_export_dependencies(rosidl_default_runtime)

rosidl_get_typesupport_target(typesupport_target
  ${PROJECT_NAME} "rosidl_typesupport_cpp")

add_executable(camera src/talker.cpp)
target_link_libraries(camera ${typesupport_target} ${OpenCV_LIBS})
ament_target_dependencies(camera rclcpp)

add_executable(display src/listener.cpp)
target_link_libraries(display ${typesupport_target} ${OpenCV_LIBS})
ament_target_dependencies(display rclcpp)

add_executable(filter src/filter_node.cpp)
target_link_libraries(filter ${typesupport_target} ${OpenCV_LIBS})
ament_target_dependencies(filter rclcpp)

add_executable(edge_detector src/edge_detector_node.cpp)
target_link_libraries(edge_detector ${typesupport_target} ${OpenCV_LIBS})
ament_target_dependencies(edge_detector rclcpp)

add_executable(optical_flow src/optical_flow_node.cpp)
target_link_libraries(optical_flow ${typesupport_target} ${OpenCV_LIBS})
ament_target_dependencies(optical_flow rclcpp)

add_executable(object_detector src/object_detector_node.cpp)
target_link_libraries(object_detector ${typesupport_target} ${OpenCV_LIBS})
ament_target_dependencies(object_detector rclcpp)

install(TARGETS camera display filter edge_detector optical_flow object_detector
  DESTINATION lib/${PROJECT_NAME})

# download yolov3 weights to shm_vision_demo/config/
add_custom_command(TARGET object_detector
  PRE_BUILD
  COMMAND sh scripts/download_yolo_weights.sh yolov3.weights
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  COMMENT "YOLOv3 weights")
# download yolov3 cfg to shm_vision_demo/config/
add_custom_command(TARGET object_detector
  PRE_BUILD
  COMMAND sh scripts/download_yolo_cfg.sh yolov3.cfg
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  COMMENT "YOLOv3 cfg")

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_auto_package(
  INSTALL_TO_SHARE config scripts
)
