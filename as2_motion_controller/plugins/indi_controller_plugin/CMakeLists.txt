cmake_minimum_required(VERSION 3.5)
set(PLUGIN_NAME indi_controller)

# Default to C++17
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()

# set Release as default
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# set fPIC to ON by default
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# find dependencies
set(PLUGIN_DEPENDENCIES
  ament_cmake
  rclcpp
  pluginlib
  as2_core
  as2_msgs
  as2_motion_reference_handlers
  geometry_msgs
  Eigen3
)

foreach(DEPENDENCY ${PLUGIN_DEPENDENCIES})
  find_package(${DEPENDENCY} REQUIRED)
endforeach()

# Add project submodules
if(${multirotor_simulator_FOUND})
  message(STATUS "multirotor_simulator found")
else()
  message(STATUS "multirotor_simulator not found")
  include(FetchContent)
  fetchcontent_declare(
    multirotor_simulator
    GIT_REPOSITORY https://github.com/RPS98/multirotor_simulator.git
    GIT_TAG v2.0
  )
  fetchcontent_makeavailable(multirotor_simulator)
endif()

include_directories(
  include
  include/${PLUGIN_NAME}

  ${EIGEN3_INCLUDE_DIRS}

  # Incluye multirotor_simulator
  ${CMAKE_BINARY_DIR}/_deps/multirotor_simulator-src/include
)

set(SOURCE_CPP_FILES
  src/${PLUGIN_NAME}.cpp
)

# Library
add_library(${PLUGIN_NAME} SHARED ${SOURCE_CPP_FILES})
target_link_libraries(${PLUGIN_NAME} ${PROJECT_NAME} ${PROJECT_NAME}_plugin_base multirotor simulator)
ament_target_dependencies(${PLUGIN_NAME} ${PLUGIN_DEPENDENCIES})

install(
  DIRECTORY include/
  DESTINATION include
)

ament_export_include_directories(
  include
)
ament_export_libraries(
  ${PLUGIN_NAME}
)
ament_export_targets(
  export_${PLUGIN_NAME}
)

install(
  TARGETS ${PLUGIN_NAME}
  EXPORT export_${PLUGIN_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
  
  add_subdirectory(tests)
endif()
