cmake_minimum_required(VERSION 3.14)

project(lua_cmake_wrapper VERSION 5.4.3 LANGUAGES C)

set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# grab the lua project, version 5.4.3
include (FetchContent)
FetchContent_Declare(lua_base
  GIT_REPOSITORY     https://github.com/lua/lua.git
  GIT_TAG            v5.4.3)
FetchContent_MakeAvailable(lua_base)
FetchContent_GetProperties(lua_base SOURCE_DIR lua_base_SRCDIR)

# create a cmake target library for the fetched project
add_library(lua_lib STATIC
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lapi.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lcode.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lctype.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/ldebug.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/ldo.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/ldump.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lfunc.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lgc.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/llex.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lmem.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lobject.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lopcodes.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lparser.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lstate.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lstring.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/ltable.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/ltm.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lundump.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lvm.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lzio.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/ltests.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lauxlib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lbaselib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/ldblib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/liolib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lmathlib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/loslib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/ltablib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lstrlib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lutf8lib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/loadlib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/lcorolib.c
  $<BUILD_INTERFACE:${lua_base_SRCDIR}>/linit.c)

target_include_directories(lua_lib PUBLIC $<BUILD_INTERFACE:${lua_SRCDIR}>)
#target_compile_features(lua_lib PUBLIC c_std_99)
target_link_libraries(lua_lib m dl)
add_library(lua_cmake_wrapper::lua ALIAS lua_lib)

if (APPLE)
  target_compile_definitions(lua_lib PUBLIC LUA_USE_MACOSX)
endif (APPLE)

if (UNIX)
  target_compile_definitions(lua_lib PUBLIC LUA_USE_LINUX LUA_USE_READLINE)
  
  include(FindPkgConfig)

  pkg_search_module(readline IMPORTED_TARGET readline libedit)

  if (NOT readline_FOUND) # if we can't find the pc, spoof it
    message(STATUS "searching for readline in system directories")
    find_library(readline_LIB readline REQUIRED)
    add_library(readline INTERFACE)
    target_link_libraries(readline INTERFACE ${readline_LIB})
    add_library(PkgConfig::readline ALIAS readline)
  endif (NOT readline_FOUND)

  target_link_libraries(lua_lib PkgConfig::readline)
endif (UNIX)
