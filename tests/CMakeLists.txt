include_directories(${PROJECT_BINARY_DIR}/src)
include_directories(${PROJECT_BINARY_DIR}/include)

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

# target dependencies for tests/examples
set(EXAMPLE_DEPS yaml)


# auto_add_exec -- adds an executable target from a <name>.c file and
#adds to the target list
macro(auto_add_exec _src_file _target_list)
  string(REPLACE ".c" "" _name ${_src_file})
  add_executable(${_name} EXCLUDE_FROM_ALL ${_src_file})
  target_link_libraries(${_name} ${ARGN})
  list(APPEND ${_target_list} ${_name})
endmacro()

# add_test_exec -- adds an test target from a <name>.c file and adds
#to the target list
macro(add_test_exec _test_file)
  string(REPLACE ".c" "" _name ${_test_file})
  add_custom_command(
    TARGET ${_name}
    POST_BUILD
    COMMAND ./${CMAKE_CURRENT_BUILD_DIR}/${_name}
    COMMENT "Running ${_name} test..."
    )
endmacro()

# BUILD_TESTING
# build target "test", which builds and runs all tests in TEST_SRCS
option(BUILD_TESTING "Build and run test files" true)
set(TEST_TARGETS)
foreach(_testfile ${TEST_SRCS})
  auto_add_exec(${_testfile} TEST_TARGETS ${EXAMPLE_DEPS})
  add_test_exec(${_testfile})
endforeach()

if(BUILD_TESTING)
  add_custom_target(test ALL DEPENDS ${TEST_TARGETS})
else(BUILD_TESTING)
  add_custom_target(test DEPENDS ${TEST_TARGETS})
endif(BUILD_TESTING)

# build target "examples", which builds all examples in EXAMPLE_SRCS
option(BUILD_EXAMPLES "Build example files" false)
if(BUILD_EXAMPLES)
  set(EXAMPLE_TARGETS)
  foreach(_exfile ${EXAMPLE_SRCS})
    auto_add_exec(${_exfile} EXAMPLE_TARGETS ${EXAMPLE_DEPS})
  endforeach()
  add_custom_target(examples DEPENDS ${EXAMPLE_TARGETS})
endif(BUILD_EXAMPLES)