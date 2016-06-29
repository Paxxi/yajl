# Copyright (c) 2007-2014, Lloyd Hilaiel <me@lloyd.io>
#
# Permission to use, copy, modify, and/or distribute this software for any
# purpose with or without fee is hereby granted, provided that the above
# copyright notice and this permission notice appear in all copies.
#
# THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES
# WITH REGARD TO THIS SOFTWARE INCLUDING ALL IMPLIED WARRANTIES OF
# MERCHANTABILITY AND FITNESS. IN NO EVENT SHALL THE AUTHOR BE LIABLE FOR
# ANY SPECIAL, DIRECT, INDIRECT, OR CONSEQUENTIAL DAMAGES OR ANY DAMAGES
# WHATSOEVER RESULTING FROM LOSS OF USE, DATA OR PROFITS, WHETHER IN AN
# ACTION OF CONTRACT, NEGLIGENCE OR OTHER TORTIOUS ACTION, ARISING OUT OF
# OR IN CONNECTION WITH THE USE OR PERFORMANCE OF THIS SOFTWARE.

SET (SRCS yajl.c yajl_lex.c yajl_parser.c yajl_buf.c
          yajl_encode.c yajl_gen.c yajl_alloc.c
          yajl_tree.c yajl_version.c
)
SET (HDRS yajl_parser.h yajl_lex.h yajl_buf.h yajl_encode.h yajl_alloc.h)
SET (PUB_HDRS yajl/yajl_parse.h yajl/yajl_gen.h yajl/yajl_common.h yajl/yajl_tree.h)

# useful when fixing lexer bugs.
#ADD_DEFINITIONS(-DYAJL_LEXER_DEBUG)

# Ensure defined when building YAJL (as opposed to using it from
# another project).  Used to ensure correct function export when
# building win32 DLL.
ADD_DEFINITIONS(-DYAJL_BUILD)

ADD_LIBRARY(yajl_s STATIC ${SRCS})

ADD_LIBRARY(yajl SHARED ${SRCS}) 
target_include_directories(yajl PUBLIC
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  PRIVATE .)
target_include_directories(yajl_s PUBLIC
  $<BUILD_INTERFACE:${CMAKE_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>
  PRIVATE .)
#### setup shared library version number
SET_TARGET_PROPERTIES(yajl PROPERTIES
                      DEFINE_SYMBOL YAJL_SHARED
                      SOVERSION ${YAJL_MAJOR}
                      VERSION ${YAJL_MAJOR}.${YAJL_MINOR}.${YAJL_MICRO})

#### ensure a .dylib has correct absolute installation paths upon installation
IF(APPLE)
  MESSAGE("INSTALL_NAME_DIR: ${CMAKE_INSTALL_PREFIX}/lib")
  SET_TARGET_PROPERTIES(yajl PROPERTIES
                        INSTALL_NAME_DIR "${CMAKE_INSTALL_PREFIX}/lib")
ENDIF(APPLE)

#### build up an sdk as a post build step


# generate build-time source
CONFIGURE_FILE(${CMAKE_SOURCE_DIR}/include/yajl/yajl_version.h.cmake yajl_version.h)
CONFIGURE_FILE(yajl.pc.cmake yajl.pc)

# at build time you may specify the cmake variable LIB_SUFFIX to handle
# 64-bit systems which use 'lib64'
INSTALL(TARGETS yajl yajl_s
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib)
INSTALL(DIRECTORY ${CMAKE_SOURCE_DIR}/include/ DESTINATION include)
INSTALL(FILES ${CMAKE_CURRENT_BINARY_DIR}/yajl_version.h DESTINATION include/yajl)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/yajl.pc DESTINATION share/yajl.pc)
