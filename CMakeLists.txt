cmake_minimum_required(VERSION 3.0.0)
project(nanovis VERSION 0.1.0 LANGUAGES CXX)

find_package(Eigen3 REQUIRED NO_MODULE)
find_package(OpenCV REQUIRED)

# configuring nanogui
if (NOT IS_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/ext/nanogui")
  message(FATAL_ERROR "NanoGUI submodule is missing! "
    "You probably did not clone the project with --recursive. It is possible to recover "
    "by calling \"git submodule update --init --recursive\"")
endif()

# Disable unnecessary parts of nanogui
set(NANOGUI_BUILD_EXAMPLE OFF CACHE BOOL " " FORCE)
set(NANOGUI_BUILD_PYTHON  OFF CACHE BOOL " " FORCE)
set(NANOGUI_INSTALL       OFF CACHE BOOL " " FORCE)

# Use system or user defined Eigen
set(NANOGUI_EIGEN_INCLUDE_DIR ${EIGEN3_INCLUDE_DIR} CACHE BOOL " " FORCE)
message("NanoGUI will use Eigen located at \"${NANOGUI_EIGEN_INCLUDE_DIR}\".")

add_subdirectory(ext/nanogui)

set_property(TARGET glfw glfw_objects nanogui PROPERTY FOLDER "dependencies")
add_definitions(${NANOGUI_EXTRA_DEFS})

add_library(nanovis
    src/nanovis.cpp
    src/screen.cpp
    src/window.cpp
)

target_compile_features(
    nanovis
    PUBLIC
        cxx_std_14
)

target_include_directories(
    nanovis
    PUBLIC
        $<INSTALL_INTERFACE:${CMAKE_INSTALL_PREFIX}/include>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    PRIVATE
        ${NANOGUI_EXTRA_INCS}
        ${CMAKE_CURRENT_SOURCE_DIR}/ext/nanogui/include
        ${CMAKE_CURRENT_SOURCE_DIR}/src
)

target_link_libraries(
    nanovis
    PUBLIC
        Eigen3::Eigen
        ${OpenCV_LIBS}
        nanogui
        ${NANOGUI_EXTRA_LIBS}
)

add_executable(
    example
    example/example.cpp
)

target_link_libraries(
    example
    PRIVATE
        nanovis
)