find_package(Doxygen)
if(DOXYGEN_FOUND)
  option(BUILD_DOCS "Build doxygen documentation" false)
  if(BUILD_DOCS)
    set(DOXYGEN_CFG_SRC "${PROJECT_SOURCE_DIR}/doc/doxygen.cfg.in")
    set(DOXYGEN_CFG "${PROJECT_BINARY_DIR}/doc/doxygen.cfg")
    configure_file(${DOXYGEN_CFG_SRC} ${DOXYGEN_CFG} @ONLY)
    add_custom_target(doc)
    add_custom_command(
      TARGET doc
      COMMAND ${DOXYGEN_EXECUTABLE} ${DOXYGEN_CFG}
      COMMENT "Building documentation..."
      )
  endif(BUILD_DOCS)
endif(DOXYGEN_FOUND)
