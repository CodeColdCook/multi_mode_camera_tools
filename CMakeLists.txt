cmake_minimum_required(VERSION 3.10)
project(camodocal 
  VERSION 1.1
  LANGUAGES CXX)

#准备的版本设置
SET(PROJECT_VERSION_MAJOR 1)
SET(PROJECT_VERSION_MINOR 0)
SET(PROJECT_VERSION_PATCH 1)

# =============#
# Dependencies #
# =============#

# ----------------------#
# Third-party libraries #
# ----------------------#


# useful macros
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/pmtd_cmake/util.cmake)

find_package(OpenCV REQUIRED)
find_package(Boost REQUIRED COMPONENTS filesystem program_options system)
find_package(Ceres REQUIRED)
find_package(GTest REQUIRED) 
 
# =========#
# Settings # 
# =========#  

# shared libraries
if(NOT DEFINED BUILD_SHARED_LIBS) 
  message(STATUS "${PROJECT_NAME}: Building dynamically-linked binaries")
  option(BUILD_SHARED_LIBS "Build dynamically-linked binaries" ON)
  set(BUILD_SHARED_LIBS ON)
endif()

# build type
if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  message(STATUS "${PROJECT_NAME}: Defaulting build type to RelWithDebInfo")
  set(CMAKE_BUILD_TYPE RelWithDebInfo)
endif()

# global compilations
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
add_definitions(-O2)

# ##############################################################################
# Build #
# ##############################################################################

# ==============#
# Build targets #
# ==============#

include(GenerateExportHeader)
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)

# ---------------#
# Library target #
# ---------------#

# find all source files
file(GLOB_RECURSE src_files ${PROJECT_SOURCE_DIR}/src/*.cc
     ${PROJECT_SOURCE_DIR}/src/*/*.cc)

# install path setting
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX "/opt/pmtd")
endif()

# common library and install settings
pmtd_deb_library(
  NAME
  ${PROJECT_NAME}
  SRCS
  ${src_files}
  INCLUDES
  ${GTEST_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${CERES_INCLUDE_DIRS}
  ${EIGEN3_INCLUDE_DIR}
  DEPS
  ${GTEST_BOTH_LIBRARIES}
  pthread
  ${Boost_LIBRARIES} 
  ${OpenCV_LIBS} 
  ${CERES_LIBRARIES}
  )

add_simple_apps()  

# unit test
# tests
option(BUILD_WITH_TESTS "Build project with tests" OFF)
if (BUILD_WITH_TESTS)
   add_tests_in_dir(common)
endif (BUILD_WITH_TESTS)

# install config DIRECTORY or not
option(INSTALL_CONFIG "Install the package config to ${CONFIG_DIR}" OFF)
pmtd_library_settings()

# make deb pkg settings
option(ARCHITECTURE_ARM "CPACK_DEBIAN_PACKAGE_ARCHITECTURE default false(amd) " OFF)
set(description "camera model package for undistort and space to plane")
if (${ARCHITECTURE_ARM})
  pmtd_deb_settings("arm64" ${description})
else(${ARCHITECTURE_ARM})
  pmtd_deb_settings("amd64" ${description})
endif (${ARCHITECTURE_ARM})
