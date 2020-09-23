cmake_minimum_required(VERSION 3.1)

project(mad)

file(GLOB SRCS "src/*.c")

add_library(${PROJECT_NAME} SHARED ${SRCS})
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/data)
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_compile_definitions(${PROJECT_NAME} PRIVATE HAVE_CONFIG_H FPM_INTEL ASO_ZEROCHECK)
