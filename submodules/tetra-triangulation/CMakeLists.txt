# Written By Yu Ji

cmake_minimum_required(VERSION 3.25)
set(CMAKE_CUDA_COMPILER "C:/Program Files/NVIDIA GPU Computing Toolkit/CUDA/v12.3/bin/nvcc.exe")
project(fast_traversal LANGUAGES CXX CUDA)
if(POLICY CMP0148)
  cmake_policy(SET CMP0148 NEW)
endif()
enable_language(CUDA)

if (MSVC AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
    set(VCPKG_ROOT $ENV{VCPKG_ROOT})
    if (NOT VCPKG_ROOT)
        # set the vcpkg path here
        message(WARNING "Please set the VCPKG_ROOT environment variable")
        set(PATH_TO_VCPKG "D:/vcpkg")
        if (EXISTS ${PATH_TO_VCPKG})
            set(VCPKG_ROOT ${PATH_TO_VCPKG})
        else()
            message(FATAL_ERROR "Please set the VCPKG_ROOT environment variable")
        endif()
    endif()    
    set(CMAKE_TOOLCHAIN_FILE "${VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake" CACHE STRING "")
    set(CMAKE_PREFIX_PATH "${VCPKG_ROOT}/installed/x64-windows/share")    
endif()

include(FetchContent)
# Add paths to our CMake code to the module path, so they can be found automatically by
# CMake.
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})
set(CMAKE_BUILD_TYPE "Release")
set(CMAKE_CUDA_ARCHITECTURES "native")
set(CGAL_DIR "F:/gitclone/gaussian-opacity-fields/submodules/tetra-triangulation/CGAL-5.6.1")

# Find necessary packages
find_package(Torch REQUIRED)
find_package(CGAL CONFIG REQUIRED)
find_package(GMP REQUIRED)
find_package(MPFR REQUIRED)

# Fetch pybind11
FetchContent_Declare(
    pybind11
    GIT_REPOSITORY https://github.com/pybind/pybind11
    GIT_TAG        v2.9.2
)
FetchContent_GetProperties(pybind11)
if(NOT pybind11_POPULATED)
    FetchContent_Populate(pybind11)
    add_subdirectory(${pybind11_SOURCE_DIR} ${pybind11_BINARY_DIR})
endif()

# Set output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin")
set(BUILD_DIRECTORY "${CMAKE_BINARY_DIR}/build")
set(CUDA_GENERATED_OUTPUT_DIR "${BUILD_DIRECTORY}")

# Compiler-specific flags
if (MSVC)
    add_compile_options(/W3 /D_CRT_SECURE_NO_WARNINGS /wd4996)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /std:c++17")
else()
    add_compile_options(-Wno-deprecated-declarations -std=c++17)
endif()

# Main build configuration
add_link_options(-flto=auto)  # disables some warning
pybind11_add_module(tetranerf_cpp_extension 
    src/triangulation.cpp
    src/py_binding.cpp)

# Ensure the correct C++ standard is set
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED YES)
set(CMAKE_CXX_EXTENSIONS NO)

# Link libraries
target_link_libraries(tetranerf_cpp_extension PRIVATE ${CUDA_LIBRARIES} ${TORCH_LIBRARIES})
target_link_libraries(tetranerf_cpp_extension PRIVATE CGAL::CGAL)
set_target_properties(tetranerf_cpp_extension
    PROPERTIES
    ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/tetranerf/utils/extension"
    RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
)
