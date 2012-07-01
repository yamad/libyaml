cmake_minimum_required(VERSION 2.8.4) # >= 2.8.4 needed for CYGWIN behavior
project(yaml C)

set (YAML_VERSION_MAJOR 0)
set (YAML_VERSION_MINOR 2)
set (YAML_VERSION_PATCH 0)

set (YAML_VERSION_STRING "${YAML_VERSION_MAJOR}.${YAML_VERSION_MINOR}.${YAML_VERSION_PATCH}")
configure_file(                 # create configuration file
  "${PROJECT_SOURCE_DIR}/yaml_config.h.in"
  "${PROJECT_BINARY_DIR}/config.h"
  ESCAPE_QUOTES)
configure_file(                 # create pkg-config file
  "${PROJECT_SOURCE_DIR}/yaml.pc.in"
  "${PROJECT_BINARY_DIR}/yaml-${YAML_VERSION_MAJOR}.${YAML_VERSION_MINOR}.pc"
  @ONLY
  )
add_definitions(-DHAVE_CONFIG_H) # use configuration file

# build as a shared library or static library (default: shared)
option(BUILD_SHARED_LIBS "Library build type" true)
if(NOT BUILD_SHARED_LIBS)
  add_definitions(-DYAML_DECLARE_STATIC)
endif(NOT BUILD_SHARED_LIBS)

# include build directory to see generated files
include_directories("${PROJECT_BINARY_DIR}")

add_subdirectory(src)
add_subdirectory(tests)
add_subdirectory(doc)
