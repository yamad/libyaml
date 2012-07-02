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
macro(auto_add_exec _src_file _target_list)
  string(REPLACE ".c" "" _name ${_src_file})
  add_executable(${_name} EXCLUDE_FROM_ALL ${_src_file})
  if(EXAMPLES_STATIC)
    set_target_properties(${_name} PROPERTIES
      COMPILE_FLAGS -DYAML_DECLARE_STATIC)
  endif(EXAMPLES_STATIC)
  target_link_libraries(${_name} ${ARGN})
  list(APPEND ${_target_list} ${_name})
endmacro()

# only build tests or examples if a library is being built
if(BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS)

if(BUILD_TESTING)
  enable_testing()              # use CTest
  set(TEST_TARGETS)
  foreach(_testfile ${TEST_SRCS})
    string(REPLACE ".c" "" _name ${_testfile})
    auto_add_exec(${_testfile} TEST_TARGETS ${EXAMPLE_DEPS})
    add_test(NAME ${_name}
      COMMAND ${_name})
  endforeach()
  # `check` target builds tests before running if necessary
  # CTest `test` target doesn't automatically build tests
  add_custom_target(check
    COMMAND ${CMAKE_CTEST_COMMAND}
    DEPENDS ${TEST_TARGETS}
    COMMENT "Building and running tests...")
endif(BUILD_TESTING)

if(BUILD_EXAMPLES)
  set(EXAMPLE_TARGETS)
  foreach(_exfile ${EXAMPLE_SRCS})
    auto_add_exec(${_exfile} EXAMPLE_TARGETS ${EXAMPLE_DEPS})
  endforeach()
  add_custom_target(examples DEPENDS ${EXAMPLE_TARGETS})
endif(BUILD_EXAMPLES)

endif(BUILD_SHARED_LIBS OR BUILD_STATIC_LIBS)