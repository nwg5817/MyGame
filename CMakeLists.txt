cmake_minimum_required(VERSION 3.10)

include(CMake/out_of_tree.cmake)
include(CMake/genex.cmake)

project(MyGame)

list(APPEND CMAKE_MODULE_PATH "${MyGame_SOURCE_DIR}/CMake")

if(NOT VERSION_NUM)
  include(CMake/git.cmake)
  get_git_tag(VERSION_NUM)
  get_git_commit_hash(GIT_COMMIT_HASH)
  if(NOT VERSION_SUFFIX)
    set(VERSION_SUFFIX "$<$<NOT:$<CONFIG:Release>>:-${GIT_COMMIT_HASH}>")
  endif()
endif()

find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
  set_property(GLOBAL PROPERTY RULE_LAUNCH_COMPILE "${CCACHE_PROGRAM}")
endif()

if(VERSION_NUM MATCHES untagged)
  project(MyGame
    LANGUAGES C CXX)
else()
  project(MyGame
    VERSION ${VERSION_NUM}
    LANGUAGES C CXX)
endif()

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_THREAD_PREFER_PTHREAD ON)
set(THREADS_PREFER_PTHREAD_FLAG ON)

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

genex_for_option(DEBUG)
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

# Create the source groups for source tree with root at CMAKE_CURRENT_SOURCE_DIR.
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${MyGame_SRCS})