cmake_policy(SET CMP0048 NEW)
project(fast_double_parser LANGUAGES CXX VERSION 0.0.0.0)
cmake_minimum_required(VERSION 3.10)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
if (NOT CMAKE_BUILD_TYPE)
  message(STATUS "No build type selected, default to Release")
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build." FORCE)
endif()

option(FAST_DOUBLE_PARSER_SANITIZE "Sanitize addresses" OFF)

set(headers include/fast_double_parser.h)
set(unit_src tests/unit.cpp)
set(stats_src tests/stats.cpp)

set(benchmark_src benchmarks/benchmark.cpp)

add_executable(unit ${unit_src})
if(FAST_DOUBLE_PARSER_SANITIZE)
  target_compile_options(unit PUBLIC -fsanitize=address -fno-omit-frame-pointer -fsanitize=undefined -fno-sanitize-recover=all)
  target_link_options(unit PUBLIC -fsanitize=address -fno-omit-frame-pointer -fsanitize=undefined -fno-sanitize-recover=all)
  # Ubuntu bug for GCC 5.0+ (safe for all versions)
  if (CMAKE_COMPILER_IS_GNUCC)
    target_link_libraries(unit PUBLIC -fuse-ld=gold)
  endif()
endif()

target_include_directories(unit PUBLIC include)
enable_testing()
add_test(unit unit)

add_executable(stats ${stats_src})
target_include_directories(stats PUBLIC include)



function(initialize_submodule DIRECTORY)
  if(NOT EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/${DIRECTORY}/.git)
    find_package(Git QUIET REQUIRED)
    message(STATUS "${CMAKE_CURRENT_SOURCE_DIR}/${DIRECTORY}/.git does not exist. Initializing ${DIRECTORY} submodule ...")
    execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init ${CMAKE_CURRENT_SOURCE_DIR}/${DIRECTORY}
                    WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}
                    RESULT_VARIABLE GIT_EXIT_CODE)
    if(NOT GIT_EXIT_CODE EQUAL "0")
      message(FATAL_ERROR "${GIT_EXECUTABLE} submodule update --init dependencies/${DIRECTORY} failed with exit code ${GIT_EXIT_CODE}, please checkout submodules")
    endif()
  endif()
endfunction(initialize_submodule)

initialize_submodule(benchmarks/dependencies/abseil-cpp)
initialize_submodule(benchmarks/dependencies/double-conversion)

add_subdirectory(benchmarks/dependencies/abseil-cpp)
add_subdirectory(benchmarks/dependencies/double-conversion)



add_executable(benchmark ${benchmark_src})
target_link_libraries(benchmark PUBLIC double-conversion absl_strings)
target_include_directories(benchmark PUBLIC include)
