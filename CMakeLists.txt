cmake_minimum_required(VERSION 3.1)

project(SDFilter)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_VERBOSE_MAKEFILE ON)


# Additional compiler flags
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
	message("Clang compiler found.")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native -mtune=native")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "AppleClang")
	message("AppleClang compiler found.")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native -mtune=native")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	message("GNU compiler found.")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")
	set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native -mtune=native")
elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
    message("MSVC compiler found.")
    add_definitions(/DUSE_MSVC)
    add_definitions(/D_USE_MATH_DEFINES)
endif()

# Detect OpenMP environment
set(OPENMP ON CACHE BOOL "OpenMP")
if(OPENMP)
  find_package(OpenMP QUIET)
  if(OPENMP_FOUND)
      message("OpenMP found. OpenMP activated in release.")
      add_definitions(-DUSE_OPENMP)

  else()
      message("OpenMP not found.")
  endif()
endif()


# Detect Eigen3
if(NOT EIGEN3_FOUND)
  find_package(Eigen3 REQUIRED)
  if(EIGEN3_FOUND)
    message("Found external Eigen. include: ${EIGEN3_INCLUDE_DIR}, version: ${EIGEN3_VERSION_STRING}.")
  endif()
endif()


# OpenMesh library
set(OPENMESH_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/external/OpenMesh")
file(GLOB_RECURSE OPENMESH_HEADERS "${OPENMESH_SOURCE_DIR}/Core/*.hh" "${OPENMESH_SOURCE_DIR}/Core/*.h" "${OPENMESH_SOURCE_DIR}/Tools/*.hh" "${OPENMESH_SOURCE_DIR}/Tools/*.h")
file(GLOB_RECURSE OPENMESH_SOURCES "${OPENMESH_SOURCE_DIR}/Core/*.cc" "${OPENMESH_SOURCE_DIR}/Tools/*.cc")
add_library(OpenMesh STATIC ${OPENMESH_HEADERS} ${OPENMESH_SOURCES})
target_include_directories(OpenMesh PUBLIC "${CMAKE_CURRENT_SOURCE_DIR}/external")

target_compile_definitions(OpenMesh PUBLIC OM_STATIC_BUILD)

add_library(MeshSDLibrary INTERFACE)
target_include_directories(MeshSDLibrary INTERFACE
                           ${CMAKE_CURRENT_SOURCE_DIR}
)
target_link_libraries(MeshSDLibrary
    INTERFACE
      OpenMesh
      Eigen3::Eigen
)


# Executable for filtering
add_executable(MeshSDFilter
  EigenTypes.h
  MeshTypes.h
  SDFilter.h
  MeshNormalFilter.h
  MeshSDFilter.cpp
)
target_link_libraries(MeshSDFilter MeshSDLibrary)


# Executable for denoising
add_executable(MeshDenoiser
  EigenTypes.h
  MeshTypes.h
  SDFilter.h
  MeshNormalFilter.h
  MeshNormalDenoising.h
  MeshDenoiser.cpp
)
target_link_libraries(MeshDenoiser MeshSDLibrary)


if(OPENMP_FOUND)
  #target_compile_options(MeshSDLibrary PUBLIC "$<$<CONFIG:RELEASE>:${OpenMP_CXX_FLAGS}>")
  #target_compile_definitions(MeshSDLibrary PUBLIC "$<$<CONFIG:RELEASE>:USE_OPENMP>")
  #target_link_libraries(MeshSDLibrary "$<$<CONFIG:RELEASE>:${OpenMP_CXX_FLAGS}>")

  target_compile_options(MeshSDFilter PUBLIC "$<$<CONFIG:RELEASE>:${OpenMP_CXX_FLAGS}>")
  target_compile_definitions(MeshSDFilter PUBLIC "$<$<CONFIG:RELEASE>:USE_OPENMP>")
  target_link_libraries(MeshSDFilter "$<$<CONFIG:RELEASE>:${OpenMP_CXX_FLAGS}>")

  target_compile_options(MeshDenoiser PUBLIC "$<$<CONFIG:RELEASE>:${OpenMP_CXX_FLAGS}>")
  target_compile_definitions(MeshDenoiser PUBLIC "$<$<CONFIG:RELEASE>:USE_OPENMP>")
  target_link_libraries(MeshDenoiser "$<$<CONFIG:RELEASE>:${OpenMP_CXX_FLAGS}>")
endif()
