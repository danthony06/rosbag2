cmake_minimum_required(VERSION 3.5)
project(rosbag2_tests)

add_definitions(-D_SRC_RESOURCES_DIR_PATH="${CMAKE_CURRENT_SOURCE_DIR}/resources")

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

# Windows supplies macros for min and max by default. We should only use min and max from stl
if(WIN32)
  add_definitions(-DNOMINMAX)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(ament_index_cpp REQUIRED)

if(BUILD_TESTING)
  find_package(ament_cmake_gmock REQUIRED)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()

  find_package(rclcpp REQUIRED)
  find_package(rcpputils REQUIRED)
  find_package(rosbag2_compression REQUIRED)
  find_package(rosbag2_cpp REQUIRED)
  find_package(rosbag2_storage REQUIRED)
  find_package(rosbag2_storage_default_plugins REQUIRED)
  find_package(rosbag2_test_common REQUIRED)
  find_package(std_msgs REQUIRED)
  find_package(test_msgs REQUIRED)

  ament_add_gmock(test_rosbag2_record_end_to_end
    test/rosbag2_tests/test_rosbag2_record_end_to_end.cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
    TIMEOUT 120)
  if(TARGET test_rosbag2_record_end_to_end)
    ament_target_dependencies(test_rosbag2_record_end_to_end
      rclcpp
      rosbag2_compression
      rosbag2_compression_zstd
      rosbag2_storage
      rosbag2_storage_default_plugins
      rosbag2_test_common
      test_msgs)
    ament_add_test_label(test_rosbag2_record_end_to_end xfail)
  endif()

  ament_add_gmock(test_rosbag2_play_end_to_end
    test/rosbag2_tests/test_rosbag2_play_end_to_end.cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  if(TARGET test_rosbag2_play_end_to_end)
    ament_target_dependencies(test_rosbag2_play_end_to_end
      rclcpp
      rosbag2_storage
      rosbag2_storage_default_plugins
      rosbag2_test_common
      test_msgs)
    ament_add_test_label(test_rosbag2_play_end_to_end xfail)
  endif()

  ament_add_gmock(test_rosbag2_info_end_to_end
    test/rosbag2_tests/test_rosbag2_info_end_to_end.cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  if(TARGET test_rosbag2_info_end_to_end)
    ament_target_dependencies(test_rosbag2_info_end_to_end
      rosbag2_storage
      rosbag2_test_common)
    ament_add_test_label(test_rosbag2_info_end_to_end xfail)
  endif()

  ament_add_gmock(test_converter
    test/rosbag2_tests/test_converter.cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  if(TARGET test_converter)
    ament_target_dependencies(test_converter
      rosbag2_cpp
      rosbag2_test_common
      test_msgs)
  endif()

  ament_add_gmock(test_reindex
    test/rosbag2_tests/test_reindexer.cpp)
  if(TARGET test_reindex)
    ament_target_dependencies(test_reindex
      rclcpp
      rosbag2_cpp
      rosbag2_storage
      rosbag2_storage_default_plugins
      rosbag2_test_common
      std_msgs)
  endif()


  ament_add_gmock(test_rosbag2_cpp_api
    test/rosbag2_tests/test_rosbag2_cpp_api.cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})
  if(TARGET test_rosbag2_cpp_api)
    ament_target_dependencies(test_rosbag2_cpp_api
      rclcpp
      rosbag2_cpp
      rosbag2_storage
      rosbag2_storage_default_plugins
      rosbag2_test_common
      test_msgs)
  endif()
endif()

ament_package()
