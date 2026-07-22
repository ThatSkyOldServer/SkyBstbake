cmake_minimum_required(VERSION 3.15)

project(SkyBstbakeMeshopt LANGUAGES CXX)

set(MESHOPT_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/_meshopt/meshoptimizer")
if(NOT EXISTS "${MESHOPT_SOURCE_DIR}/CMakeLists.txt")
    message(FATAL_ERROR
        "meshoptimizer submodule is missing. Run: "
        "git submodule update --init --recursive")
endif()

set(MESHOPT_BUILD_DEMO OFF CACHE BOOL "" FORCE)
set(MESHOPT_BUILD_GLTFPACK OFF CACHE BOOL "" FORCE)
set(MESHOPT_BUILD_SHARED_LIBS ON CACHE BOOL "" FORCE)
set(MESHOPT_STABLE_EXPORTS ON CACHE BOOL "" FORCE)
set(MESHOPT_INSTALL OFF CACHE BOOL "" FORCE)

# Keep the upstream target in the default build graph so that a plain
# `cmake --build` always produces the library consumed by Sky-Bstbake.py.
add_subdirectory("${MESHOPT_SOURCE_DIR}" "${CMAKE_BINARY_DIR}/meshoptimizer")

set(MESHOPT_OUTPUT_DIR "${CMAKE_CURRENT_SOURCE_DIR}/_meshopt/lib")
set_target_properties(meshoptimizer PROPERTIES
    OUTPUT_NAME "meshoptimizer"
    ARCHIVE_OUTPUT_DIRECTORY "${MESHOPT_OUTPUT_DIR}"
    LIBRARY_OUTPUT_DIRECTORY "${MESHOPT_OUTPUT_DIR}"
    RUNTIME_OUTPUT_DIRECTORY "${MESHOPT_OUTPUT_DIR}"
)

# Multi-config generators (Visual Studio, Xcode) otherwise append the
# configuration name to output directories. Keep one predictable path for
# the Python loader on every platform.
foreach(CONFIG DEBUG RELEASE RELWITHDEBINFO MINSIZEREL)
    set_target_properties(meshoptimizer PROPERTIES
        ARCHIVE_OUTPUT_DIRECTORY_${CONFIG} "${MESHOPT_OUTPUT_DIR}"
        LIBRARY_OUTPUT_DIRECTORY_${CONFIG} "${MESHOPT_OUTPUT_DIR}"
        RUNTIME_OUTPUT_DIRECTORY_${CONFIG} "${MESHOPT_OUTPUT_DIR}"
    )
endforeach()
