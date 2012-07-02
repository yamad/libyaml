# test source files
set(TEST_SRCS
  test-reader.c
  test-version.c
)

# example source files
set(EXAMPLE_SRCS
  example-deconstructor-alt.c
  example-deconstructor.c
  example-reformatter-alt.c
  example-reformatter.c
  run-dumper.c
  run-emitter.c
  run-loader.c
  run-parser.c
  run-scanner.c
)

# BUILD_TESTING
# build target "test", which builds and runs all tests in TEST_SRCS
cmake_dependent_option(BUILD_TESTING "Build and run test files" true
  "BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS" true)

# BUILD_EXAMPLES
# build target "examples", which builds all examples in EXAMPLE_SRCS
cmake_dependent_option(BUILD_EXAMPLES "Build example files" false
  "BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS" true)


# target dependencies for tests/examples
if(BUILD_SHARED_LIBS)
  set(EXAMPLE_DEPS yaml_shared)
elseif(BUILD_STATIC_LIBS)
  set(EXAMPLE_DEPS yaml_static)
  set(EXAMPLES_STATIC true)
endif(BUILD_SHARED_LIBS)

include_directories(${PROJECT_BINARY_DIR}/src)
include_directories(${PROJECT_BINARY_DIR}/include)

# auto_add_exec -- adds an executable target from a <name>.c file and
#adds to the target list
macro(auto_add_exec _name _src_file _target_list)
  add_executable(${_name} EXCLUDE_FROM_ALL ${_src_file})
  if(EXAMPLES_STATIC)
    set_target_properties(${_name} PROPERTIES
      COMPILE_FLAGS -DYAML_DECLARE_STATIC)
  endif(EXAMPLES_STATIC)
  target_link_libraries(${_name} ${ARGN})
  list(APPEND ${_target_list} ${_name})
endmacro()

# .dll needed in the tests directory on Windows
set(CTEST_WORKDIR "${CMAKE_CURRENT_BINARY_DIR}/${CMAKE_CFG_INTDIR}")
macro(copy_shared_lib_command _target)
  if(WIN32 AND BUILD_SHARED_LIBS)
    add_custom_command(TARGET ${_target}
      PRE_BUILD
      COMMAND ${CMAKE_COMMAND} -E copy $<TARGET_FILE:yaml_shared> ${CTEST_WORKDIR}
      COMMENT "Move yaml_shared file to ${CTEST_WORKDIR}...")
  endif(WIN32 AND BUILD_SHARED_LIBS)
endmacro()

# only build tests or examples if a library is being built
if(BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS)

if(BUILD_TESTING)
  if(WIN32)                     # ctest needs -C <config> option
    set(CTEST_BUILD_CONFIGURATION "$(Configuration)")
  else(WIN32)
    set(CTEST_BUILD_CONFIGURATION "${CMAKE_BUILD_TYPE}")
  endif(WIN32)

  set(TEST_TARGETS)
  foreach(_testfile ${TEST_SRCS})
    string(REPLACE ".c" "" _name ${_testfile})
    auto_add_exec(${_name} ${_testfile} TEST_TARGETS ${EXAMPLE_DEPS})
    add_test(${_name} ${_name})
  endforeach()

  # default CTest target `test` does not automatically build tests
  # `check` target will build tests if necessary. prefer `check`
  add_custom_target(check
    COMMAND ${CMAKE_CTEST_COMMAND} -C ${CTEST_BUILD_CONFIGURATION}
    DEPENDS ${TEST_TARGETS}
    COMMENT "Building and running tests...")
  copy_shared_lib_command(check)
endif(BUILD_TESTING)

if(BUILD_EXAMPLES)
  set(EXAMPLE_TARGETS)
  foreach(_exfile ${EXAMPLE_SRCS})
    string(REPLACE ".c" "" _name ${_exfile})
    auto_add_exec(${_name} ${_exfile} EXAMPLE_TARGETS ${EXAMPLE_DEPS})
    copy_shared_lib_command(${_name})
  endforeach()
  add_custom_target(examples DEPENDS ${EXAMPLE_TARGETS})
endif(BUILD_EXAMPLES)

endif(BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS)