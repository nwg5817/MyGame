cmake_minimum_required(VERSION 3.10)

include(CMake/out_of_tree.cmake)
include(CMake/genex.cmake)

project(MyGame)

list(APPEND CMAKE_MODULE_PATH "${MyGame_SOURCE_DIR}/CMake")

  if (NOT SDL2_FOUND)
    find_package(SDL2 REQUIRED)
  endif()

  if (NOT SDL2_IMAGE_FOUND)
    find_package(SDL2_image REQUIRED)
  endif()

set(MyGame_SRCS
  src/main.cpp)

set(BIN_TARGET MyGame)
add_executable(${BIN_TARGET} WIN32 ${MyGame_SRCS})

# Use file GENERATE instead of configure_file because configure_file
# does not support generator expressions.
get_property(is_multi_config GLOBAL PROPERTY GENERATOR_IS_MULTI_CONFIG)
if(is_multi_config)
  set(CONFIG_PATH $<CONFIG>/config.h)
  target_include_directories(${BIN_TARGET} PRIVATE ${CMAKE_BINARY_DIR}/$<CONFIG>)
else()
  set(CONFIG_PATH config.h)
endif()
file(GENERATE OUTPUT ${CONFIG_PATH} CONTENT
"#pragma once
#define PROJECT_NAME \"${PROJECT_NAME}\"
#define PROJECT_VERSION \"${PROJECT_VERSION}${VERSION_SUFFIX}\"
")
  
  set(TARGET_PLATFORM host CACHE STRING "Target platform")
set_property(CACHE TARGET_PLATFORM PROPERTY STRINGS host)

genex_for_option(ASAN)
genex_for_option(UBSAN)
foreach(target MyGame ${BIN_TARGET})
  target_compile_options(${target} PUBLIC "$<${ASAN_GENEX}:-fsanitize=address;-fsanitize-recover=address>")
  target_link_libraries(${target} PUBLIC "$<${ASAN_GENEX}:-fsanitize=address;-fsanitize-recover=address>")
  target_compile_options(${target} PUBLIC $<${UBSAN_GENEX}:-fsanitize=undefined>)
  target_link_libraries(${target} PUBLIC $<${UBSAN_GENEX}:-fsanitize=undefined>)

    target_link_libraries(${target} PRIVATE
		SDL2::SDL2main
		SDL2::SDL2_image)		

  target_compile_definitions(${target} PRIVATE ${def_list})
endforeach(target)