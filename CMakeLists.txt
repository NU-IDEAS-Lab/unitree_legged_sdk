cmake_minimum_required(VERSION 3.5)
project(unitree_legged_sdk)

# check arch and os
if("${CMAKE_SYSTEM_PROCESSOR}" MATCHES "x86_64.*")
  set(ARCH amd64)
endif()
if("${CMAKE_SYSTEM_PROCESSOR}" MATCHES "aarch64.*")
  set(ARCH arm64)
endif()

include_directories(include)
link_directories(lib/cpp/${ARCH})

option(PYTHON_BUILD "build python wrapper" OFF)
if(PYTHON_BUILD)
  add_subdirectory(python_wrapper)
endif()

set(EXTRA_LIBS -pthread)
set(CMAKE_CXX_FLAGS "-O3 -fPIC")
set(CMAKE_CXX_STANDARD 14)

set(unitree_legged_sdk_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/lib/cpp/${ARCH}/libunitree_legged_sdk.a)
add_library(unitree_legged_sdk IMPORTED STATIC GLOBAL)
set_target_properties(unitree_legged_sdk PROPERTIES
  IMPORTED_LOCATION ${unitree_legged_sdk_LOCATION}
)

# Declare the C++ executables
add_executable(example_position example/example_position.cpp)
target_link_libraries(example_position unitree_legged_sdk ${EXTRA_LIBS})

add_executable(example_velocity example/example_velocity.cpp)
target_link_libraries(example_velocity unitree_legged_sdk ${EXTRA_LIBS})

add_executable(example_torque example/example_torque.cpp)
target_link_libraries(example_torque unitree_legged_sdk ${EXTRA_LIBS})

add_executable(example_walk example/example_walk.cpp)
target_link_libraries(example_walk unitree_legged_sdk ${EXTRA_LIBS})

add_executable(example_joystick example/example_joystick.cpp)
target_link_libraries(example_joystick unitree_legged_sdk ${EXTRA_LIBS})

# Install targets
install(
  TARGETS example_position example_velocity example_torque example_walk example_joystick
  DESTINATION lib/${PROJECT_NAME}
)
# Unfortuantly an imported target is not copied to the devel/lib folder, so we do it manually
# so that installation of a catkin workspace is not required
# See http://answers.ros.org/question/223866/cmakecatkin-how-to-export-imported-target/
file(COPY ${unitree_legged_sdk_LOCATION}
  DESTINATION lib/${PROJECT_NAME}/libunitree_legged_sdk.a
)
# Mark imported library for installation using a workaround since imported
# targets do not have proper support for installation in CMakeLists
install(FILES ${unitree_legged_sdk_LOCATION} DESTINATION lib/${PROJECT_NAME})

install(DIRECTORY include/
  DESTINATION include/
)

# Include CMake utilities for ROS 2
find_package(ament_cmake QUIET)
if(${ament_cmake_FOUND})
  # Export the include directories for this package
  ament_export_include_directories(include)

  # Export the libraries for this package
  ament_export_libraries(unitree_legged_sdk)

  ament_package()
  message(STATUS "Packaged with ament_cmake.")
else()
  message(WARNING "Did not package with ament_cmake.")
endif()
