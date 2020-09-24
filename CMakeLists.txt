cmake_minimum_required(VERSION 2.6.0)

project(STREAMRIPPER)

# Tell CMake to be quiet
cmake_policy(SET CMP0003 NEW)

# Default to release
if (NOT CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE Release)
endif (NOT CMAKE_BUILD_TYPE)

include(CheckFunctionExists)
include(CheckLibraryExists)

# http://www.cmake.org/pipermail/cmake/2008-December/025886.html
include(InstallRequiredSystemLibraries)

if (UNIX)
set(MATH_LIB -lm)
endif (UNIX)

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})

find_package(PkgConfig REQUIRED)
pkg_check_modules(GLIB REQUIRED glib-2.0)
link_directories(${GLIB_LIBRARY_DIRS})

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/ThirdParty/mad)
set(MAD_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/ThirdParty/mad/include)
set(MAD_LIBRARIES mad)

add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/ThirdParty/ogg)
set(OGG_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/ThirdParty/ogg/include)
set(OGG_LIBRARIES ogg)

find_package(VORBIS)

include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/SR_FindSockets.cmake)

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/sr_config.h.cmake 
  ${CMAKE_CURRENT_BINARY_DIR}/sr_config.h)

file(GLOB STREAMRIPPER_LIB_SRC "src/*.c")

ADD_LIBRARY (streamripper1 ${STREAMRIPPER_LIB_SRC})
target_include_directories(streamripper1 PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_include_directories(streamripper1 PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(streamripper1 PRIVATE ${OGG_INCLUDE_DIRS})
target_include_directories(streamripper1 PRIVATE ${MAD_INCLUDE_DIRS})
target_include_directories(streamripper1 PRIVATE ${GLIB_INCLUDE_DIRS})

add_executable(streamripper "src/console/streamripper.c")
target_include_directories(streamripper PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_include_directories(streamripper PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_include_directories(streamripper PRIVATE ${GLIB_INCLUDE_DIRS})
set(STREAMRIPPER_LIBS ${MAD_LIBRARIES} ${GLIB_LIBRARIES} ${MATH_LIB} ${OGG_LIBRARIES} ${VORBIS_LIBRARIES})
if (WIN32)
set(STREAMRIPPER_LIBS ${STREAMRIPPER_LIBS} unicows ws2_32 vorbis zdll)
else ()
set(STREAMRIPPER_LIBS ${STREAMRIPPER_LIBS} rt pthread)
endif (WIN32)
if (SEMAPHORE_LIBRARIES)
SET (STREAMRIPPER_LIBS ${STREAMRIPPER_LIBS} ${SEMAPHORE_LIBRARIES})
endif (SEMAPHORE_LIBRARIES)
target_link_libraries(streamripper streamripper1 ${STREAMRIPPER_LIBS})

if (WIN32)
file(GLOB WSTREAMRIPPER_SRC "winamp_plugin/*.c")
add_executable(wstreamripper WIN32 ${WSTREAMRIPPER_SRC})
target_link_libraries(wstreamripper streamripper1 ${STREAMRIPPER_LIBS} comctl32.lib)
endif (WIN32)

install(TARGETS streamripper1 DESTINATION lib)
install(TARGETS streamripper DESTINATION bin)
if (WIN32)
install(TARGETS wstreamripper DESTINATION bin)
endif (WIN32)
