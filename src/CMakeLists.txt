include_directories(${PROJECT_SOURCE_DIR}/includes)
set(SRCS yaml_private.h api.c reader.c scanner.c parser.c loader.c writer.c emitter.c dumper.c)

set(YAML_TARGETS)
option(BUILD_SHARED_LIBS "Build shared library" true)
if(BUILD_SHARED_LIBS)
  add_library(yaml_shared SHARED ${SRCS})
  set_target_properties(yaml_shared PROPERTIES
    OUTPUT_NAME yaml
    VERSION ${YAML_SO_VERSION}
    SOVERSION ${YAML_SO_RELEASE}
    )
  list(APPEND YAML_TARGETS yaml_shared)
  install(TARGETS yaml_shared DESTINATION lib)
endif(BUILD_SHARED_LIBS)

option(BUILD_STATIC_LIBS "Build static library" true)
if(BUILD_STATIC_LIBS)
  add_library(yaml_static STATIC ${SRCS})
  set_target_properties(yaml_static PROPERTIES
    OUTPUT_NAME yaml
    COMPILE_FLAGS -DYAML_DECLARE_STATIC)
  list(APPEND YAML_TARGETS yaml_static)
  install(TARGETS yaml_static DESTINATION lib)
endif(BUILD_STATIC_LIBS)

add_custom_target(yaml ${YAML_TARGETS})
