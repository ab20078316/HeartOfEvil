cmake_minimum_required(VERSION 3.10)
project(HeartOfEvilAndroid LANGUAGES C CXX)

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

option(BUILD_CLIENT "Build client DLL (libclient.so)" ON)
option(BUILD_SERVER "Build server DLL (libserver.so)" ON)

file(GLOB_RECURSE DLL_SRCS "${CMAKE_SOURCE_DIR}/dlls/*.cpp" "${CMAKE_SOURCE_DIR}/dlls/*.c")
file(GLOB_RECURSE CL_SRCS "${CMAKE_SOURCE_DIR}/cl_dll/*.cpp" "${CMAKE_SOURCE_DIR}/cl_dll/*.c")
file(GLOB_RECURSE DEDICATED_SRCS "${CMAKE_SOURCE_DIR}/dedicated/*.cpp" "${CMAKE_SOURCE_DIR}/dedicated/*.c")

set(EXCLUDE_NAMES
  "mp3.cpp" "mp3.h"
  "fmod.h" "fmod_errors.h"
  "inputw32.cpp" "inputw32.h"
)

function(filter_sources out_var src_list)
  set(result "")
  foreach(f ${${src_list}})
    get_filename_component(fname ${f} NAME)
    set(skip FALSE)
    foreach(x ${EXCLUDE_NAMES})
      if("${fname}" STREQUAL "${x}")
        set(skip TRUE)
      endif()
    endforeach()
    if(NOT skip)
      list(APPEND result ${f})
    endif()
  endforeach()
  set(${out_var} "${result}" PARENT_SCOPE)
endfunction()

filter_sources(GAME_SRCS DLL_SRCS)
filter_sources(CLIENT_SRCS CL_SRCS)
filter_sources(SERVER_SRCS DEDICATED_SRCS)

set(ANDROID_STUBS
  "${CMAKE_SOURCE_DIR}/android_stubs/input_stub.cpp"
  "${CMAKE_SOURCE_DIR}/android_stubs/mp3_stub.cpp"
  "${CMAKE_SOURCE_DIR}/android_stubs/fmod_stub.cpp"
  "${CMAKE_SOURCE_DIR}/android_stubs/win32_stub.cpp"
)

add_library(game SHARED ${GAME_SRCS} ${ANDROID_STUBS})
target_include_directories(game PRIVATE
  "${CMAKE_SOURCE_DIR}"
  "${CMAKE_SOURCE_DIR}/common"
  "${CMAKE_SOURCE_DIR}/engine"
  "${CMAKE_SOURCE_DIR}/dlls"
  "${CMAKE_SOURCE_DIR}/pm_shared"
)
target_compile_definitions(game PRIVATE ANDROID)
target_compile_options(game PRIVATE -fPIC -O2 -Wall)
target_link_libraries(game PRIVATE m)
set_target_properties(game PROPERTIES OUTPUT_NAME "game")

if(BUILD_CLIENT)
  add_library(client SHARED ${CLIENT_SRCS} ${ANDROID_STUBS})
  target_include_directories(client PRIVATE
    "${CMAKE_SOURCE_DIR}"
    "${CMAKE_SOURCE_DIR}/common"
    "${CMAKE_SOURCE_DIR}/engine"
    "${CMAKE_SOURCE_DIR}/cl_dll"
  )
  target_compile_definitions(client PRIVATE ANDROID)
  target_compile_options(client PRIVATE -fPIC -O2 -Wall)
  target_link_libraries(client PRIVATE m)
  set_target_properties(client PROPERTIES OUTPUT_NAME "client")
endif()

if(BUILD_SERVER)
  add_library(server SHARED ${SERVER_SRCS} ${ANDROID_STUBS})
  target_include_directories(server PRIVATE
    "${CMAKE_SOURCE_DIR}"
    "${CMAKE_SOURCE_DIR}/common"
    "${CMAKE_SOURCE_DIR}/engine"
    "${CMAKE_SOURCE_DIR}/dedicated"
    "${CMAKE_SOURCE_DIR}/pm_shared"
  )
  target_compile_definitions(server PRIVATE ANDROID)
  target_compile_options(server PRIVATE -fPIC -O2 -Wall)
  target_link_libraries(server PRIVATE m)
  set_target_properties(server PROPERTIES OUTPUT_NAME "server")
endif()
