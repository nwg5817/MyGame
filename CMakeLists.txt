cmake_minimum_required(VERSION 3.10)

include(CMake/out_of_tree.cmake)
include(CMake/genex.cmake)

option(USE_SDL1 "Use SDL1.2 instead of SDL2" OFF)
project(MyGame)

set(MyGame_SRCS
  src/main.cpp)

set(BIN_TARGET MyGame)
add_executable(${BIN_TARGET} WIN32 MACOSX_BUNDLE ${MyGame_SRCS})

  list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/CMake")  
  
  if (NOT SDL2_FOUND)
    find_package(SDL2 REQUIRED)
  endif()  
    find_package(SDL2_image REQUIRED)
  
  set(TARGET_PLATFORM host CACHE STRING "Target platform")
set_property(CACHE TARGET_PLATFORM PROPERTY STRINGS host)

genex_for_option(ASAN)
genex_for_option(UBSAN)
foreach(target MyGame ${BIN_TARGET})
  target_compile_options(${target} PUBLIC "$<${ASAN_GENEX}:-fsanitize=address;-fsanitize-recover=address>")
  target_link_libraries(${target} PUBLIC "$<${ASAN_GENEX}:-fsanitize=address;-fsanitize-recover=address>")
  target_compile_options(${target} PUBLIC $<${UBSAN_GENEX}:-fsanitize=undefined>)
  target_link_libraries(${target} PUBLIC $<${UBSAN_GENEX}:-fsanitize=undefined>)

  if(USE_SDL1)
    target_link_libraries(${target} PRIVATE
      ${SDL_TTF_LIBRARY}
      ${SDL_MIXER_LIBRARY}
      ${SDL_LIBRARY})
    target_compile_definitions(${target} PRIVATE USE_SDL1)
  else()
    target_link_libraries(${target} PRIVATE
		SDL2::SDL2main)
  endif()

  target_compile_definitions(${target} PRIVATE ${def_list})
endforeach(target)