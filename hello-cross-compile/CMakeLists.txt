cmake_minimum_required(VERSION 3.29)

if (NOT CMAKE_TOOLCHAIN_FILE)
    # ツールチェイン定義を読み込む
    include(toolchain.cmake)
endif ()

project(raspberry_pi C CXX ASM)

set(CMAKE_C_STANDARD 11)
set(CMAKE_CXX_STANDARD 17)

find_package(PkgConfig REQUIRED)
pkg_check_modules(OpenCV REQUIRED opencv4)
pkg_check_modules(Charls REQUIRED charls)

add_executable(raspberry_pi main.cpp)
target_include_directories(raspberry_pi PRIVATE ${OpenCV_INCLUDE_DIRS} ${Charls_INCLUDE_DIRS})
target_link_directories(raspberry_pi PRIVATE ${OpenCV_LIBRARY_DIRS} ${Charls_LIBRARY_DIRS})
target_link_libraries(raspberry_pi PRIVATE ${OpenCV_LIBRARIES} ${Charls_LIBRARIES})

message("OpenCV_INCLUDE_DIRS: ${OpenCV_INCLUDE_DIRS}")
message("OpenCV_LIBRARIES: ${OpenCV_LIBRARIES}")
message("OpenCV_LIBRARY_DIRS: ${OpenCV_LIBRARY_DIRS}")
