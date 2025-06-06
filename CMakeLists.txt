# Copyright (c) 2013-2018, EPFL/Blue Brain Project
#                          Raphael Dumusc <raphael.dumusc@epfl.ch>
#                          Daniel Nachbaur <daniel.nachbaur@epfl.ch>

cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
project(Deflect VERSION 1.0.2)
set(Deflect_VERSION_ABI 7)

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/CMake/common)
if(NOT EXISTS ${CMAKE_SOURCE_DIR}/CMake/common/Common.cmake)
  message(FATAL_ERROR "CMake/common missing, run: git submodule update --init")
endif()

set(DEFLECT_DESCRIPTION "A fast C++ library for streaming pixels and events")
set(DEFLECT_MAINTAINER "Blue Brain Project <bbp-open-source@googlegroups.com>")
set(DEFLECT_VENDOR "Blue Brain Project")
set(DEFLECT_LICENSE LGPL)
set(DEFLECT_DEB_DEPENDS freeglut3-dev libxi-dev libxmu-dev
  libjpeg-turbo8-dev libturbojpeg
  libboost-program-options-dev libboost-test-dev
  qtbase5-dev qtdeclarative5-dev
)
include(LSBInfo)
if(LSB_DISTRIBUTOR_ID STREQUAL "Ubuntu" AND LSB_RELEASE VERSION_GREATER 16.04)
  list(APPEND DEFLECT_DEB_DEPENDS libturbojpeg0-dev)
endif()
set(DEFLECT_PORT_DEPENDS boost freeglut qt5)
set(DEFLECT_BREW_DEPENDS boost freeglut jpeg-turbo qt5)

include(Common)

common_find_package(Boost COMPONENTS program_options unit_test_framework)
common_find_package(GLUT)
common_find_package(LibJpegTurbo 1.4)
if(NOT LibJpegTurbo_FOUND)
  common_find_package(LibJpegTurbo 1.2 REQUIRED)
  list(APPEND COMMON_FIND_PACKAGE_DEFINES DEFLECT_USE_LEGACY_LIBJPEGTURBO)
endif()
common_find_package(OpenGL)
common_find_package(OpenMP)
common_find_package(Qt5Concurrent REQUIRED SYSTEM)
common_find_package(Qt5Core REQUIRED)
if(APPLE)
  common_find_package(Qt5MacExtras)
endif()
common_find_package(Qt5Network REQUIRED)
common_find_package(Qt5Qml)
common_find_package(Qt5Quick)
common_find_package(Qt5OpenGL)
common_find_package(Qt5Widgets REQUIRED)
common_find_package_post()

if(NOT Qt5Quick_VERSION VERSION_LESS 5.5)
  option(DEFLECT_QMLSTREAMER_MULTITHREADED "Use multithreaded-rendering in QMLStreamer" ON)
endif()

set(LCOV_EXCLUDE "deflect/moodycamel/*")

add_subdirectory(deflect)
add_subdirectory(apps)
if(Boost_FOUND)
  add_subdirectory(tests)
endif()
add_subdirectory(doc)

# Export public package dependencies, used in CommonPackageConfig
if(TARGET DeflectQt)
  set(DEFLECT_DEPENDENT_LIBRARIES Qt5Quick)
endif()

# Name the package "desktopstreamer" instead of "deflect"
set(CPACK_PACKAGE_NAME "desktopstreamer")
set(DEFLECT_PACKAGE_DEB_DEPENDS qtbase5-dev libturbojpeg)
include(CommonCPack) # also includes CommonPackageConfig

set(COMMON_PROJECT_DOMAIN ch.epfl.bluebrain)
set(DOXYGEN_MAINPAGE_MD README.md)
set(DOXYGEN_EXTRA_INPUT ${PROJECT_SOURCE_DIR}/README.md)
include(DoxygenRule) # must be after all targets
