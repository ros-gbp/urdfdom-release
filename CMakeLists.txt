cmake_minimum_required( VERSION 2.8 FATAL_ERROR )
project (urdfdom CXX C)

#set (URDF_MAJOR_VERSION 0)
#set (URDF_MINOR_VERSION 2)
#set (URDF_PATCH_VERSION 4)

set (URDF_VERSION ${URDF_MAJOR_VERSION}.${URDF_MINOR_VERSION}.${URDF_PATCH_VERSION})

message (STATUS "${PROJECT_NAME} version ${URDF_VERSION}")

# set the default build type
if (NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

# This shouldn't be necessary, but there has been trouble
# with MSVC being set off, but MSVCXX ON.
if(MSVC OR MSVC90 OR MSVC10)
  set(MSVC ON)
endif (MSVC OR MSVC90 OR MSVC10)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

include(SearchForTinyXML)
include_directories(${tinyxml_include_dirs})
link_directories(${tinyxml_library_dirs})
add_definitions(${tinyxml_cflags})

find_package(urdfdom_headers REQUIRED)
include_directories(SYSTEM ${urdfdom_headers_INCLUDE_DIRS})

find_package(console_bridge REQUIRED)
include_directories(SYSTEM ${console_bridge_INCLUDE_DIRS})
link_directories(${console_bridge_LIBRARY_DIRS})

find_package(Boost REQUIRED thread)
include_directories(${Boost_INCLUDE_DIR})
link_directories(${Boost_LIBRARY_DIRS})

set(${PROJECT_NAME}_inc_dirs urdf_parser/include)
find_package(catkin REQUIRED)
catkin_package(
  INCLUDE_DIRS ${${PROJECT_NAME}_inc_dirs}
  LIBRARIES urdfdom_model urdfdom_model_state urdfdom_sensor urdfdom_world
  DEPENDS console_bridge urdfdom_headers
  DEPENDS Boost COMPONENTS thread
)
include_directories(${${PROJECT_NAME}_inc_dirs})

# split version in parts and pass to preprocessor
string(REPLACE "." ";" ${PROJECT_NAME}_VERSION_LIST "${${PROJECT_NAME}_VERSION}")
list(LENGTH ${PROJECT_NAME}_VERSION_LIST _count)
if(NOT _count EQUAL 3)
  message(FATAL_ERROR "${PROJECT_NAME} version '${${PROJECT_NAME}_VERSION}' does not match 'MAJOR.MINOR.PATCH' pattern")
endif()
list(GET ${PROJECT_NAME}_VERSION_LIST 0 URDF_MAJOR_VERSION)
list(GET ${PROJECT_NAME}_VERSION_LIST 1 URDF_MINOR_VERSION)
list(GET ${PROJECT_NAME}_VERSION_LIST 2 URDF_PATCH_VERSION)

add_subdirectory(urdf_parser)

set(PKG_NAME ${PROJECT_NAME})
set(PKG_LIBRARIES urdfdom_sensor urdfdom_model_state urdfdom_model urdfdom_world)
set(PKG_DEPENDS urdfdom_headers console_bridge)
set(cmake_conf_file "${CMAKE_CURRENT_SOURCE_DIR}/cmake/urdfdom-config.cmake")
#configure_file("${cmake_conf_file}.in" "${cmake_conf_file}" @ONLY)
#install(FILES ${cmake_conf_file} DESTINATION share/${PROJECT_NAME}/cmake/ COMPONENT cmake)

# Make the package config file
if (NOT MSVC)
  set(PKG_DESC "Unified Robot Description Format")
  set(PKG_DEPENDS "urdfdom_headers console_bridge") # make the list separated by spaces instead of ;
  set(PKG_URDF_LIBS "-lurdfdom_sensor -lurdfdom_model_state -lurdfdom_model -lurdfdom_world")
  set(pkg_conf_file "${CMAKE_CURRENT_SOURCE_DIR}/cmake/pkgconfig/urdfdom.pc")
  #configure_file("${pkg_conf_file}.in" "${pkg_conf_file}" @ONLY)
  #install(FILES ${pkg_conf_file} DESTINATION lib/pkgconfig/ COMPONENT pkgconfig)
endif()


#message(STATUS "Configuration successful. Type make to compile urdfdom")
