cmake_minimum_required(VERSION 3.4)
project(solver)

set(PRINT_VERSION_MAJOR 0)
set(PRINT_VERSION_MINOR 1)
set(PRINT_VERSION_PATCH 0)
set(PRINT_VERSION_TWEAK 0)
set(PRINT_VERSION "${PRINT_VERSION_MAJOR}.${PRINT_VERSION_MINOR}.${PRINT_VERSION_PATCH}.${PRINT_VERSION_TWEAK}")
set(PRINT_VERSION_STRING "v${PRINT_VERSION}")

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(../formatter_ex_lib ${CMAKE_CURRENT_BINARY_DIR}/formatter_ex_lib_build)
add_subdirectory(../solver_lib ${CMAKE_CURRENT_BINARY_DIR}/solver_lib_build)

add_executable(solver equation.cpp)
target_link_libraries(solver PRIVATE formatter_ex solver_lib)

include(CPackConfig.cmake)
