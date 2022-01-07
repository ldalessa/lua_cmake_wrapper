cmake_minimum_required(VERSION 3.14)

project(lua_cmake_wrapper VERSION 5.4.3 LANGUAGES C)

# grab the lua project, version 5.4.3
include (FetchContent)
FetchContent_Declare(lua
  GIT_REPOSITORY     https://github.com/lua/lua.git
  GIT_TAG            v5.4.3)
FetchContent_MakeAvailable(lua)
FetchContent_GetProperties(lua SOURCE_DIR lua_SRCDIR)

# create a cmake target library for the fetched project
add_library(lua STATIC
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lapi.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lcode.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lctype.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/ldebug.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/ldo.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/ldump.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lfunc.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lgc.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/llex.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lmem.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lobject.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lopcodes.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lparser.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lstate.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lstring.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/ltable.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/ltm.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lundump.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lvm.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lzio.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/ltests.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lauxlib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lbaselib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/ldblib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/liolib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lmathlib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/loslib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/ltablib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lstrlib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lutf8lib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/loadlib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/lcorolib.c
  $<BUILD_INTERFACE:${lua_SRCDIR}>/linit.c)

target_include_directories(lua PUBLIC $<BUILD_INTERFACE:${lua_SRCDIR}>)
target_compile_features(lua PUBLIC c_std_99)
target_link_libraries(lua m dl)

if (APPLE)
  target_compile_definitions(lua PUBLIC LUA_USE_MACOSX)
endif (APPLE)

if (UNIX)
  target_compile_definitions(lua PUBLIC LUA_USE_LINUX LUA_USE_READLINE)
  
  include(FindPkgConfig)

  pkg_search_module(readline IMPORTED_TARGET readline libedit)

  if (NOT readline_FOUND) # if we can't find the pc, spoof it
    message(STATUS "searching for readline in system directories")
    find_library(readline_LIB readline REQUIRED)
    add_library(readline INTERFACE)
    target_link_libraries(readline INTERFACE ${readline_LIB})
    add_library(PkgConfig::readline ALIAS readline)
  endif (NOT readline_FOUND)

  target_link_libraries(lua PkgConfig::readline)
endif (UNIX)
