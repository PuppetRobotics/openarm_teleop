# Copyright 2025 Enactic, Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 3.22)
project(openarm_teleop VERSION 0.1.0)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Install/layout helpers and package config helpers
include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

# -----------------------------
# Find external packages
# -----------------------------
find_package(OpenArmCAN REQUIRED)
find_package(urdf REQUIRED)
find_package(orocos_kdl REQUIRED)
find_package(kdl_parser REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(urdfdom REQUIRED)
find_package(urdfdom_headers REQUIRED)
find_package(yaml-cpp REQUIRED)


# -----------------------------
# Include directories
# -----------------------------
include_directories(
  include
)

# -----------------------------
# Create static library
# -----------------------------
add_library(openarm_teleop_lib STATIC
    src/dynamics.cpp
    src/control.cpp
)

target_include_directories(openarm_teleop_lib PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
    ${EIGEN3_INCLUDE_DIRS}
)

target_link_libraries(openarm_teleop_lib PUBLIC
    OpenArmCAN::openarm_can
    ${EIGEN3_LIBRARIES}
    ${orocos_kdl_LIBRARIES}
    kdl_parser::kdl_parser
    urdfdom_model
    yaml-cpp
)

# -----------------------------
# Executables
# -----------------------------
add_executable(gravity_comp control/gravity_compasation_main.cpp)
add_executable(comm_test control/openarm_communication_test.cpp)
add_executable(unilateral_control control/openarm_unilateral_control.cpp)

target_link_libraries(gravity_comp openarm_teleop_lib)
target_link_libraries(comm_test openarm_teleop_lib)
target_link_libraries(unilateral_control openarm_teleop_lib)


target_include_directories(gravity_comp PRIVATE ${EIGEN3_INCLUDE_DIRS})
target_include_directories(unilateral_control PRIVATE ${EIGEN3_INCLUDE_DIRS})


# -----------------------------
# CMake Package
# -----------------------------
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/openarm_teleop)

# Export and install the library target
install(TARGETS openarm_teleop_lib
    EXPORT openarm_teleopTargets
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Install public headers
install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# Generate and install the CMake package config files
configure_package_config_file(
    cmake/openarm_teleopConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/openarm_teleopConfig.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/openarm_teleopConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/openarm_teleopConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/openarm_teleopConfigVersion.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)

# Install the export set for use with the install-tree
install(EXPORT openarm_teleopTargets
    NAMESPACE openarm_teleop::
    DESTINATION ${INSTALL_CONFIGDIR}
)