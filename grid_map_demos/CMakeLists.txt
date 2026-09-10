cmake_minimum_required(VERSION 3.5.0)
project(grid_map_demos)

## Find ament_cmake macros and libraries
find_package(ament_cmake REQUIRED)
# Provides ament_auto_depend_on_packages(), which replaces
# ament_target_dependencies(), removed in the ament_cmake ROS 2 Lyrical ships
find_package(ament_cmake_auto REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(filters REQUIRED)
find_package(grid_map_cmake_helpers REQUIRED)
find_package(grid_map_core REQUIRED)
find_package(grid_map_cv REQUIRED)
find_package(grid_map_msgs REQUIRED)
find_package(grid_map_octomap REQUIRED)
find_package(grid_map_ros REQUIRED)
find_package(OCTOMAP REQUIRED)
find_package(octomap_msgs REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)

find_package(OpenCV REQUIRED
  COMPONENTS
  opencv_highgui
  CONFIG
)

grid_map_package()

set(dependencies
  grid_map_msgs
  grid_map_ros
  rclcpp
)

###########
## Build ##
###########

## Specify additional locations of header files
## Your package locations should be listed before other locations
include_directories(
  include
  SYSTEM
    ${EIGEN3_INCLUDE_DIR}
)

## Declare a cpp executable
add_executable(
  simple_demo
  src/simple_demo_node.cpp
)

add_executable(
  tutorial_demo
  src/tutorial_demo_node.cpp
)

add_executable(
  iterators_demo
  src/iterators_demo_node.cpp
  src/IteratorsDemo.cpp
)

add_executable(
  image_to_gridmap_demo
  src/image_to_gridmap_demo_node.cpp
  src/ImageToGridmapDemo.cpp
)

add_executable(
  octomap_to_gridmap_demo
  src/octomap_to_gridmap_demo_node.cpp
  src/OctomapToGridmapDemo.cpp
)

add_executable(
  move_demo
  src/move_demo_node.cpp
)

add_executable(
  iterator_benchmark
  src/iterator_benchmark.cpp
)

add_executable(
  opencv_demo
  src/opencv_demo_node.cpp
)

add_executable(
  resolution_change_demo
  src/resolution_change_demo_node.cpp
)

add_executable(
  filters_demo
  src/filters_demo_node.cpp
)

add_library(filters_demo_lib SHARED
  src/FiltersDemo.cpp
)

add_executable(
  normal_filter_comparison_demo
  src/normal_filter_comparison_node.cpp
)

add_executable(
  interpolation_demo
  src/interpolation_demo_node.cpp
  src/InterpolationDemo.cpp
)

## Specify libraries to link a library or executable target against
ament_auto_depend_on_packages(simple_demo SYSTEM
  PACKAGES
    ${dependencies}
)

ament_auto_depend_on_packages(tutorial_demo SYSTEM
  PACKAGES
    ${dependencies}
)

ament_auto_depend_on_packages(iterators_demo SYSTEM
  PACKAGES
    ${dependencies}
)

ament_auto_depend_on_packages(image_to_gridmap_demo SYSTEM
  PACKAGES
    ${dependencies}
)

ament_auto_depend_on_packages(octomap_to_gridmap_demo SYSTEM
  PACKAGES
    ${dependencies}
    grid_map_octomap
    octomap_msgs
)

ament_auto_depend_on_packages(move_demo SYSTEM
  PACKAGES
    ${dependencies}
)

ament_auto_depend_on_packages(iterator_benchmark SYSTEM
  PACKAGES
    grid_map_core
)

ament_auto_depend_on_packages(opencv_demo SYSTEM
  PACKAGES
    ${dependencies}
    grid_map_cv
)

ament_auto_depend_on_packages(resolution_change_demo SYSTEM
  PACKAGES
    ${dependencies}
)

target_link_libraries(filters_demo filters_demo_lib)

ament_auto_depend_on_packages(filters_demo_lib SYSTEM
  PACKAGES
    ${dependencies}
    filters
)

ament_auto_depend_on_packages(filters_demo SYSTEM
  PACKAGES
    ${dependencies}
    filters
)

ament_auto_depend_on_packages(normal_filter_comparison_demo SYSTEM
  PACKAGES
    ${dependencies}
    filters
)

ament_auto_depend_on_packages(interpolation_demo SYSTEM
  PACKAGES
    ${dependencies}
)

set(targets_list
  filters_demo
  filters_demo_lib
  image_to_gridmap_demo
  interpolation_demo
  iterator_benchmark
  iterators_demo
  move_demo
  normal_filter_comparison_demo
  octomap_to_gridmap_demo
  opencv_demo
  resolution_change_demo
  simple_demo
  tutorial_demo
)

foreach(target ${targets_list})
  if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_compile_options(${target} PRIVATE "SHELL:--param ggc-min-expand=1")
    target_compile_options(${target} PRIVATE "SHELL:--param ggc-min-heapsize=32768")
  endif()
endforeach()

#############
## Install ##
#############

# Mark executables and/or libraries for installation
install(
  TARGETS ${targets_list}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

# Mark other files for installation
install(
  DIRECTORY config data doc launch rviz
  DESTINATION share/${PROJECT_NAME}
)

# Install python scripts
install(
  PROGRAMS scripts/image_publisher.py
  DESTINATION lib/${PROJECT_NAME}
)

#############
## Testing ##
#############

if(BUILD_TESTING)
  # Linting is setup this way to add a filter
  # to ament_cpplint to ignore the lack of
  # copyright messages at the top of files.
  # Copyright messages are being checked for by both
  # ament_cmake_cpplint & ament_cmake_copyright.

  find_package(ament_lint_auto REQUIRED)
  find_package(ament_lint_auto QUIET)
  if(ament_lint_auto_FOUND)
    # exclude copyright checks
    list(APPEND AMENT_LINT_AUTO_EXCLUDE
      ament_cmake_cpplint
      ament_cmake_copyright
    )
    ament_lint_auto_find_test_dependencies()

    # run cpplint without copyright filter
    find_package(ament_cmake_cpplint)
    ament_cpplint(
      FILTERS -legal/copyright -build/include_order
    )
  endif()
  ament_lint_auto_find_test_dependencies()
endif()

ament_export_include_directories(include ${EIGEN3_INCLUDE_DIR})
ament_package()
