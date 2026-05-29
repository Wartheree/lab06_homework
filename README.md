# Отчёт к homework из lab03
## Ход работы:
1) Были скопированы все необходимые директории из `lab03`:
```bash
$ git clone https://github.com/tp-labs/lab03.git projects/temp
Клонирование в «projects/temp»...
remote: Enumerating objects: 91, done.
remote: Counting objects: 100% (30/30), done.
remote: Compressing objects: 100% (9/9), done.
remote: Total 91 (delta 23), reused 21 (delta 21), pack-reused 61 (from 1)
Получение объектов: 100% (91/91), 1.02 МиБ | 1.24 МиБ/с, готово.
Определение изменений: 100% (41/41), готово.

$ cp -r projects/temp/formatter_lib projects/lab03_homework
$ cp -r projects/temp/formatter_ex_lib projects/lab03_homework
$ cp -r temp/solver_lib lab03_homework/
$ cp -r temp/solver_application lab03_homework/
$ cp -r temp/hello_world_application/ lab03_homework/

$ rm -rf projects/temp/
```
2) Написан `CMakeLists.txt` для `formatter_lib`:
```CMake
cmake_minimum_required(VERSION 3.4)
project(formatter_lib)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(formatter STATIC formatter.cpp)
target_include_directories(formatter PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```
3) Написан `CMakeLists.txt` для `formatter_ex_lib`:
```CMake
cmake_minimum_required(VERSION 3.4)
project(formatter_ex_lib)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(../formatter_lib ${CMAKE_CURRENT_BINARY_DIR}/formatter_lib_build)
add_library(formatter_ex STATIC formatter_ex.cpp)
target_include_directories(formatter_ex PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_link_libraries(formatter_ex PUBLIC formatter)
```
4) В директории `solver_lib` отсутствовал `CMakeLists.txt`, поэтому мной был написан м он
```CMake
cmake_minimum_required(VERSION 3.4)
project(solver_lib)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_library(solver_lib STATIC solver.cpp)
target_include_directories(solver_lib PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
```
5) Написан `CMakeLists.txt` для `hello_world`, которое использует библиотеку `formatter_ex`:
```CMake
cmake_minimum_required(VERSION 3.4)
project(hello_world)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(../formatter_ex_lib ${CMAKE_CURRENT_BINARY_DIR}/formatter_ex_lib_build)
add_executable(hello_world hello_world.cpp)
target_link_libraries(hello_world PRIVATE formatter_ex)
```
6) Написан `CMakeLists.txt` для `solver`, которое испольует статические библиотеки `formatter_ex` и `solver_lib`:
```CMake
cmake_minimum_required(VERSION 3.4)
project(solver)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_subdirectory(../formatter_ex_lib ${CMAKE_CURRENT_BINARY_DIR}/formatter_ex_lib_build)
add_subdirectory(../solver_lib ${CMAKE_CURRENT_BINARY_DIR}/solver_lib_build)

add_executable(solver equation.cpp)
target_link_libraries(solver PRIVATE formatter_ex solver_lib)
```
7) Произведена сборка `hello_world` и проиведен тест:
```bash
$ cd hello_world_application/
$ mkdir build
$ cmake ..
-- Configuring done (1.9s)
-- Generating done (0.0s)
-- Build files have been written to: /home/danila/Wartheree/workspace/projects/lab03_homework/hello_world_application/build
$ make
[ 16%] Building CXX object formatter_ex_lib_build/formatter_lib_build/CMakeFiles/formatter.dir/formatter.cpp.o
[ 33%] Linking CXX static library libformatter.a
[ 33%] Built target formatter
[ 50%] Building CXX object formatter_ex_lib_build/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o
[ 66%] Linking CXX static library libformatter_ex.a
[ 66%] Built target formatter_ex
[ 83%] Building CXX object CMakeFiles/hello_world.dir/hello_world.cpp.o
[100%] Linking CXX executable hello_world
[100%] Built target hello_world
$ ./hello_world
-------------------------
hello, world!
-------------------------
```
8) В процессе сборки `solver` была обнаружена ошибка в `solve.cpp`, которая была мною исправлена:
```cpp
#include <cmath>
#include "solver.h"

#include <stdexcept>

void solve(float a, float b, float c, float& x1, float& x2)
{
    float d = (b * b) - (4 * a * c);

    if (d < 0)
    {
        throw std::logic_error{"error: discriminant < 0"};
    }

    x1 = (-b - sqrtf(d)) / (2 * a);
    x2 = (-b + sqrtf(d)) / (2 * a);
}
```
9) Произведена сборка `solver` и проиведен тест:
```bash
$ cd solver_application/
$ mkdir build
$ cd build/
$ cmake ..
-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/danila/Wartheree/workspace/projects/lab03_homework/solver_application/build
$ make
[ 12%] Building CXX object solver_lib_build/CMakeFiles/solver_lib.dir/solver.cpp.o
[ 25%] Linking CXX static library libsolver_lib.a
[ 25%] Built target solver_lib
[ 37%] Building CXX object formatter_ex_lib_build/formatter_lib_build/CMakeFiles/formatter.dir/formatter.cpp.o
[ 50%] Linking CXX static library libformatter.a
[ 50%] Built target formatter
[ 62%] Building CXX object formatter_ex_lib_build/CMakeFiles/formatter_ex.dir/formatter_ex.cpp.o
[ 75%] Linking CXX static library libformatter_ex.a
[ 75%] Built target formatter_ex
[ 87%] Building CXX object CMakeFiles/solver.dir/equation.cpp.o
[100%] Linking CXX executable solver
[100%] Built target solver
$ ./solver
1
2
1
-------------------------
x1 = -1.000000
-------------------------
-------------------------
x2 = -1.000000
-------------------------
```
10) Все было запушено на данный репозиторий (по ошибке были запушены сборки, позже они были удалены)
