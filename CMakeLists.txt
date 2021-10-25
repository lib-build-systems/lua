# Copyright (C) 2007-2011 LuaDist.
# Created by Peter Drahoš, Peter Kapec
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.
# Changed by Vinicius Jarina (viniciusjarina@gmail.com)
# Updated by Ad Steijvers, 2019

project(lua C)
cmake_minimum_required(VERSION 2.8.7)

if (CMAKE_SIZEOF_VOID_P EQUAL 8)
	set (LIB_SUFFIX "64" CACHE STRING "Suffix of the directory name, e.g. 64 for lib64")
else ()
	set (LIB_SUFFIX "" CACHE STRING "Suffix of the directory name, e.g. 64 for lib64")
endif ()

set (CMAKE_BUILD_WITH_INSTALL_RPATH FALSE)

# $ORIGIN support in rpath.
if (UNIX AND NOT APPLE)
  set (USE_RPATH_ORIGIN TRUE)
  set (CMAKE_INSTALL_RPATH "\$ORIGIN/../lib${LIB_SUFFIX}")
endif ()

if (APPLE)
  set(SKIP_BUILD_RPATH, TRUE)
  set(CMAKE_MACOSX_RPATH 1)
  add_definitions (-DLUA_USE_MACOSX)
  list (APPEND LIBS readline)
endif ()

set (LIB_LUA_VER "53" CACHE STRING "Suffix of version (for library name) e.g lua53")

if (WIN32 AND NOT CYGWIN)
  # Windows systems
  option (LUA_WIN "Windows specific build." ON)
  option (LUA_BUILD_AS_DLL "Build  Lua library as Dll." ON)
  
else ()
  # Posix systems (incl. Cygwin)
  option (LUA_USE_POSIX "Use POSIX functionality." ON)
  option (LUA_USE_DLOPEN "Use dynamic linker to load modules." ON)
  
endif ()

if (LUA_USE_POSIX)
  # On POSIX Lua links to standard math library "m"
  list (APPEND LIBS m)
  add_definitions (-DLUA_USE_POSIX)
endif ()

if (LUA_USE_DLOPEN)
  # Link to dynamic linker library "dl"
  list (APPEND LIBS dl)
endif ()


# Sources and headers
set (LIB_SRCS
	 lapi.c
	 lauxlib.c
	 lbaselib.c
	 #lbitlib.c
	 lcode.c
	 lcorolib.c
	 lctype.c
	 ldblib.c
	 ldebug.c
	 ldo.c
	 ldump.c
	 lfunc.c
	 lgc.c
	 linit.c
	 liolib.c
	 llex.c
	 lmathlib.c
	 lmem.c
	 loadlib.c
	 lobject.c
	 lopcodes.c
	 loslib.c
	 lparser.c
	 lstate.c
	 lstring.c
	 lstrlib.c
	 ltable.c
	 ltablib.c
	 ltm.c
	 lundump.c
	 lutf8lib.c
	 lvm.c
	 lzio.c
   )
		
set (SRC_LUA  lua.c)
#set (SRC_LUAC luac.c)

## BUILD
# Create dynamic library (dll, so)

add_library (liblua SHARED ${LIB_SRCS})

target_link_libraries (liblua ${LIBS})
set_target_properties (liblua PROPERTIES
                       OUTPUT_NAME lua${LIB_LUA_VER}
                       CLEAN_DIRECT_OUTPUT 1)

if (LUA_WIN)
	set_target_properties (liblua PROPERTIES COMPILE_FLAGS -DLUA_BUILD_AS_DLL)
endif (LUA_WIN)


# Create static library, this is needed to compile luac in the 5.1.x, 5.2 Lua series

add_library (liblua_static STATIC ${LIB_SRCS})
target_link_libraries (liblua_static ${LIBS})
set_target_properties (liblua_static PROPERTIES
                       OUTPUT_NAME lua${LIB_LUA_VER}_static
                       CLEAN_DIRECT_OUTPUT 1
		       PUBLIC_HEADER "lua.h;lualib.h;lauxlib.h;luaconf.h;lua.hpp")

add_executable (lua ${SRC_LUA})
target_link_libraries (lua liblua ${LIBS})

#add_executable (luac ${SRC_LUAC})
#target_link_libraries (luac liblua_static)


# Installation of the Lua libraries
install(TARGETS liblua liblua_static lua #luac
        EXPORT lua-targets
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
	PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lua)
        
install(EXPORT lua-targets
        NAMESPACE lua::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lua)

# export(TARGETS lua-targets
#        NAMESPACE lua::)

