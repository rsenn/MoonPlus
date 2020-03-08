project ( moonp CXX )
cmake_minimum_required ( VERSION 3.1 )

MESSAGE(STATUS "Lua: using information from luarocks")

#[[
MESSAGE(STATUS "LUA_LIBDIR: " ${LUA_INCDIR}/../../lib)
MESSAGE(STATUS "LUA_INCDIR: " ${LUA_INCDIR})
MESSAGE(STATUS "LUA: " ${LUA})
]]

#include(CheckCCompilerFlag)
include(CheckCXXCompilerFlag)
include(CheckLibraryExists)

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
#[[  check_c_compiler_flag("-O0" CFLAGS_O_OPT_NONE)
  if(CFLAGS_O_OPT_NONE)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O0")
  endif(CFLAGS_O_OPT_NONE)]]
  check_cxx_compiler_flag("-O0" CXXFLAGS_O_OPT_NONE)
  if(CXXFLAGS_O_OPT_NONE)
    string(REGEX REPLACE "-O[0-9s]" "" CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS}")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O0")
  endif(CXXFLAGS_O_OPT_NONE)
endif()

check_cxx_compiler_flag("-std=c++17" CXXFLAGS_STD_CXX17)

if(CXXFLAGS_STD_CXX17)
      set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++17")
endif()

check_library_exists(pthread pthread_create  /usr HAVE_LIBPTHREAD)

if(HAVE_LIBPTHREAD)
  set(PTHREAD_LIBRARY pthread)
endif()


find_program(RESTY resty)
find_program(LUAJIT luajit)

if(NOT LUA)
find_program(LUA lua)
endif()



if(EXISTS "${RESTY}")
  set(LUA "${RESTY}")
  string(REGEX REPLACE "/bin.*" "/lib" LUA_LIBDIR "${LUA}")
  string(REGEX REPLACE "/lib.*" "/include" LUA_INCDIR "${LUA_LIBDIR}")
  set(LUA_LIBRARIES luajit-5.1)
endif()

SET(LUA_EXECUTABLE "${LUA}")
SET(LUA_INCLUDE_DIR "${LUA_INCDIR}")

foreach(VAR RESTY LUAJIT LUA LUA_LIBDIR LUA_INCDIR LUA_LIBRARIES)

MESSAGE(STATUS "${VAR}: " ${${VAR}})

endforeach()



GET_FILENAME_COMPONENT(LUA_EXEC_NAME "${LUA}" NAME_WE)
IF(LUA_EXEC_NAME STREQUAL "luajit")
    FIND_LIBRARY(LUA_LIBRARIES
        NAMES luajit libluajit
        PATHS ${LUA_INCDIR}/../../lib
        NO_DEFAULT_PATH)
ELSEIF(LUA_EXEC_NAME MATCHES "lua.*")
    FIND_LIBRARY(LUA_LIBRARIES
        NAMES lua lua53 lua52 lua51 liblua liblua53 liblua52 liblua51
        PATHS ${LUA_INCDIR}/../../lib
        NO_DEFAULT_PATH)
ENDIF()
MESSAGE(STATUS "Lua library: ${LUA_LIBRARIES}")

enable_language( CXX )

find_package( Lua REQUIRED )


include_directories( src ${LUA_INCLUDE_DIR} )

set(DO_MODULE OFF)


if(DO_MODULE)
add_definitions( -DLIBMOONP )

  add_library( moonp  MODULE
    src/MoonP/ast.cpp
    src/MoonP/ast.hpp
    src/MoonP/moon_ast.h
    src/MoonP/moon_compiler.cpp
    src/MoonP/moon_compiler.h
    src/MoonP/moon_parser.cpp
    src/MoonP/moon_parser.h
    src/MoonP/parser.cpp
    src/MoonP/parser.hpp
    src/Moonscript.h
    src/StackTracePlus.h

    src/moonp.cpp)

  set_target_properties( moonp PROPERTIES PREFIX "")
  target_link_libraries( moonp ${LUA_LIBRARIES} )
  set(TARGETS moonp )

else()

  add_library( libmoonp
    src/MoonP/ast.cpp
    src/MoonP/ast.hpp
    src/MoonP/moon_ast.h
    src/MoonP/moon_compiler.cpp
    src/MoonP/moon_compiler.h
    src/MoonP/moon_parser.cpp
    src/MoonP/moon_parser.h
    src/MoonP/parser.cpp
    src/MoonP/parser.hpp
    src/Moonscript.h
    src/StackTracePlus.h
    )

  set_target_properties( libmoonp PROPERTIES PREFIX "" SONAME moonp)
  target_link_libraries( libmoonp ${LUA_LIBRARIES} ${PTHREAD_LIBRARY})

  add_executable( moonp  src/moonp.cpp )
  target_link_libraries( moonp  libmoonp)

  set(TARGETS moonp libmoonp)

endif()


INSTALL(TARGETS ${TARGETS}
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
)

install(CODE "")
