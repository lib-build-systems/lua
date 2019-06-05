# Copyright (C) 2007-2011 LuaDist.
# Created by Peter Drahoš, Peter Kapec
# Redistribution and use of this file is allowed according to the terms of the MIT license.
# For details see the COPYRIGHT file distributed with LuaDist.
# Please note that the package source code is licensed under its own license.
# Changed by Vinicius Jarina (viniciusjarina@gmail.com)
# Updated by Ad Steijves, 2019

project(lua C)
cmake_minimum_required(VERSION 2.8.7)

if (CMAKE_SIZEOF_VOID_P EQUAL 8)
	set (LIB_SUFFIX "64" CACHE STRING "Suffix of the directory name, e.g. 64 for lib64")
else (CMAKE_SIZEOF_VOID_P EQUAL 8)
	set (LIB_SUFFIX "" CACHE STRING "Suffix of the directory name, e.g. 64 for lib64")
endif (CMAKE_SIZEOF_VOID_P EQUAL 8)

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

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib${LIB_SUFFIX}" CACHE STRING "Library build output directory")
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib${LIB_SUFFIX}")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin${LIB_SUFFIX}")
set(EXECUTABLE_OUTPUT_PATH         "${CMAKE_BINARY_DIR}/bin${LIB_SUFFIX}")

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

if (LUA_WIN)

  # Use static linked runtime for DLL to avoid issues with people without the vcredist installed
  if (MSVC)
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set(CMAKE_CXX_FLAGS_MINSIZEREL "${CMAKE_CXX_FLAGS_DEBUG} /MT")
    set(CMAKE_C_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
    set(CMAKE_C_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set(CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set(CMAKE_C_FLAGS_MINSIZEREL "${CMAKE_CXX_FLAGS_DEBUG} /MT")
  endif()
  
  # Add extra rc files to the windows build
  if (MSVC OR MINGW)
    set (LUA_DLL_RC src/lua_dll.rc)
    set (LUA_RC src/lua.rc)
    set (LUAC_RC src/luac.rc)
  endif ()
endif ()


# Sources and headers
include_directories (include)
set (SRC_LIB 
	 src/lapi.c
	 src/lauxlib.c
	 src/lbaselib.c
	 src/lbitlib.c
	 src/lcode.c
	 src/lcorolib.c
	 src/lctype.c
	 src/ldblib.c
	 src/ldebug.c
	 src/ldo.c
	 src/ldump.c
	 src/lfunc.c
	 src/lgc.c
	 src/linit.c
	 src/liolib.c
	 src/llex.c
	 src/lmathlib.c
	 src/lmem.c
	 src/loadlib.c
	 src/lobject.c
	 src/lopcodes.c
	 src/loslib.c
	 src/lparser.c
	 src/lstate.c
	 src/lstring.c
	 src/lstrlib.c
	 src/ltable.c
	 src/ltablib.c
	 src/ltm.c
	 src/lundump.c
	 src/lutf8lib.c
	 src/lvm.c
	 src/lzio.c
   )
		
set (SRC_LUA  src/lua.c)
set (SRC_LUAC src/luac.c)

## BUILD
# Create dynamic library (dll, so)

add_library (liblua SHARED ${SRC_LIB} ${LUA_DLL_RC})

target_link_libraries (liblua ${LIBS})
set_target_properties (liblua PROPERTIES
                       OUTPUT_NAME lua${LIB_LUA_VER}
                       CLEAN_DIRECT_OUTPUT 1
                       ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX}
                       ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       LIBRARY_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX}
                       ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX}
                       ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       LIBRARY_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX})

if (LUA_WIN)
	set_target_properties (liblua PROPERTIES COMPILE_FLAGS -DLUA_BUILD_AS_DLL)
endif (LUA_WIN)


# Create static library, this is needed to compile luac in the 5.1.x, 5.2 Lua series

add_library (liblua_static STATIC ${SRC_LIB})
target_link_libraries (liblua_static ${LIBS})

set_target_properties (liblua_static PROPERTIES
                       OUTPUT_NAME lua${LIB_LUA_VER}_static
                       CLEAN_DIRECT_OUTPUT 1
                       ARCHIVE_OUTPUT_DIRECTORY_MINSIZEREL ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       LIBRARY_OUTPUT_DIRECTORY_MINSIZEREL ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       ARCHIVE_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       LIBRARY_OUTPUT_DIRECTORY_RELEASE ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       ARCHIVE_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       LIBRARY_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       ARCHIVE_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX}
                       LIBRARY_OUTPUT_DIRECTORY_DEBUG ${PROJECT_BINARY_DIR}/lib${LIB_SUFFIX})


add_executable (lua ${SRC_LUA} ${LUA_RC})
target_link_libraries (lua liblua ${LIBS})

set_target_properties (lua PROPERTIES
                       RUNTIME_OUTPUT_DIRECTORY_DEBUG          ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_RELEASE        ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL     ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX})


add_executable (luac ${SRC_LUAC} ${LUAC_RC})
target_link_libraries (luac liblua_static)
set_target_properties (luac PROPERTIES
                       RUNTIME_OUTPUT_DIRECTORY_DEBUG          ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_RELWITHDEBINFO ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_RELEASE        ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX}
                       RUNTIME_OUTPUT_DIRECTORY_MINSIZEREL     ${PROJECT_BINARY_DIR}/bin${LIB_SUFFIX})


# Installation of the Lua libraries
install(TARGETS liblua liblua_static lua luac
        EXPORT lua-targets
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
        
install(DIRECTORY ${PROJECT_SOURCE_DIR}/inc/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/lua
        FILES_MATCHING PATTERN "*.h"
        PATTERN ".git" EXCLUDE)

install(EXPORT lua-targets
        NAMESPACE lua::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/lua)


# Testing targets
enable_testing()

set (TEST_PARAM -e _U=true)

add_test (NAME test_all 
          WORKING_DIRECTORY ${PROJECT_SOURCE_DIR}/test 
		  COMMAND lua ${TEST_PARAM} ${PROJECT_SOURCE_DIR}/test/all.lua)

