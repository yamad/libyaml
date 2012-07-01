cmake_minimum_required(VERSION 2.8.4) # >= 2.8.4 needed for CYGWIN behavior
project(yaml C)

include(CMakeDependentOption)

# package version numbers
set(YAML_VERSION_MAJOR 0)
set(YAML_VERSION_MINOR 1)
set(YAML_VERSION_PATCH 4)

# shared library version numbers
set(YAML_SO_RELEASE 0)
set(YAML_SO_VERSION "${YAML_VERSION_MAJOR}.${YAML_VERSION_MINOR}")

set(YAML_VERSION_STRING "${YAML_VERSION_MAJOR}.${YAML_VERSION_MINOR}.${YAML_VERSION_PATCH}")
configure_file(                 # create configuration file
  "${PROJECT_SOURCE_DIR}/yaml_config.h.in"
  "${PROJECT_BINARY_DIR}/config.h"
  ESCAPE_QUOTES)
add_definitions(-DHAVE_CONFIG_H) # use configuration file

# create pkg-config file
option(MAKE_PKGCONFIG "Install pkg-config support files" true)
if(MAKE_PKGCONFIG)
  set(YAML_PKGCONFIG_SRC "${PROJECT_SOURCE_DIR}/yaml.pc.in")
  set(YAML_PKGCONFIG "${PROJECT_BINARY_DIR}/yaml-${YAML_VERSION_MAJOR}.${YAML_VERSION_MINOR}.pc")
  configure_file(
    ${YAML_PKGCONFIG_SRC}
    ${YAML_PKGCONFIG}
    @ONLY
    )
  install(FILES ${YAML_PKGCONFIG} DESTINATION lib/pkgconfig)
endif(MAKE_PKGCONFIG)

# include build directory to see generated files
include_directories("${PROJECT_BINARY_DIR}")

add_subdirectory(src)
add_subdirectory(tests)
add_subdirectory(doc)

install(FILES "include/yaml.h" DESTINATION include)
