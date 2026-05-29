# Отчёт к homework из lab06
## Ход работы:
1) Были скопирован репозиторий из `lab03_homework`:
```bash
$ git clone https://github.com/${GITHUB_USERNAME}/lab03_homework projects/lab06_homework
Клонирование в «projects/lab06_homework»...
remote: Enumerating objects: 163, done.
remote: Counting objects: 100% (163/163), done.
remote: Compressing objects: 100% (86/86), done.
remote: Total 163 (delta 69), reused 153 (delta 66), pack-reused 0 (from 0)
Получение объектов: 100% (163/163), 76.14 КиБ | 666.00 КиБ/с, готово.
Определение изменений: 100% (69/69), готово.
$ cd projects/lab06_homework/
$ git remote remove origin
$ git remote add origin https://github.com/${GITHUB_USERNAME}/lab06_homework
```
2) Написаны `DESCRIPTION` и `ChangeLog.md`:
```bash
$ cd solver_application/
$ echo "solver library" > DESCRIPTION
$ touch ChangeLog.md
$ export DATE="`LANG=en_US date +'%a %b %d %Y'`"
$ cat > ChangeLog.md <<EOF
* ${DATE} ${GITHUB_USERNAME} <${GITHUB_EMAIL}> 0.1.0.0
- Initial RPM release
EOF
```
3) Написан `CPackConfig.cmake`:
```CMake
include(InstallRequiredSystemLibraries)
set(CPACK_PACKAGE_CONTACT "danila.obidovskiy@gmail.com")

set(CPACK_PACKAGE_VERSION_MAJOR ${PRINT_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${PRINT_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${PRINT_VERSION_PATCH})
set(CPACK_PACKAGE_VERSION_TWEAK ${PRINT_VERSION_TWEAK})
set(CPACK_PACKAGE_VERSION ${PRINT_VERSION})

set(CPACK_PACKAGE_DESCRIPTION_FILE ${CMAKE_CURRENT_SOURCE_DIR}/DESCRIPTION)
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "static C++ library for solving quadratic equations")
set(CPACK_RESOURCE_FILE_README ${CMAKE_CURRENT_SOURCE_DIR}/../README.md)

set(CPACK_RPM_PACKAGE_NAME "solver")
set(CPACK_RPM_CHANGELOG_FILE ${CMAKE_CURRENT_SOURCE_DIR}/ChangeLog.md)
set(CPACK_RPM_PACKAGE_RELEASE 1)
set(CPACK_RPM_PACKAGE_LICENSE "MIT")
set(CPACK_RPM_PACKAGE_GROUP "Development/Tools")

set(CPACK_DEBIAN_PACKAGE_RELEASE 1)
set(CPACK_DEBIAN_PACKAGE_NAME "solver")
set(CPACK_DEBIAN_PACKAGE_PREDEPENDS "cmake >= 3.0")

include(CPack)
```
4) Отредактирован `CMakeLists.txt`
```CMake
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
```
5) Создан `.github/workflows/cpack.yml`:
```yaml
name: CMake CI with CPack

on:
  push:
    branches: [ main ]
    tags:
      - 'v*'
  pull_request:
    branches: [ main ]

permissions:
  contents: write
  packages: write

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Install dependencies
      run: |
        sudo apt-get update
        sudo apt-get install -y cmake rpm

    - name: Configure CMake
      run: |
        cd solver_application
        cmake -H. -B_build

    - name: Build
      run: |
        cd solver_application
        cmake --build _build

    - name: Package with CPack
      if: startsWith(github.ref, 'refs/tags/')
      run: |
        cd solver_application/_build
        cpack -G "TGZ"
        cpack -G "DEB"
        cpack -G "RPM"

    - name: Upload to GitHub Release
      if: startsWith(github.ref, 'refs/tags/')
      uses: softprops/action-gh-release@v1
      with:
        files: |
          solver_application/_build/*.tar.gz
          solver_application/_build/*.deb
          solver_application/_build/*.rpm
        generate_release_notes: true
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
6) После многочисленных исправлений ошибок были запушены все изменения, а также был создан и запушен тег v0.1.0.2, после чего сработал GitHub Actions workflow, который автоматически собрал пакеты и загрузил их в релиз:
https://github.com/Wartheree/lab06_homework/releases/tag/v0.1.0.2
