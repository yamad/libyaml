include_directories(${PROJECT_SOURCE_DIR}/includes)
set(SRCS yaml_private.h api.c reader.c scanner.c parser.c loader.c writer.c emitter.c dumper.c)
add_library(yaml ${SRCS})
install(TARGETS yaml DESTINATION lib)