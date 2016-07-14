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

CMAKE_MINIMUM_REQUIRED(VERSION 2.6)

PROJECT(YetAnotherJSONParser C)

option(BUILD_TOOLS "Enables building of utilities" OFF)
option(BUILD_EXAMPLES "Enabled building of examples" OFF)

SET (YAJL_MAJOR 2)
SET (YAJL_MINOR 1)
SET (YAJL_MICRO 1)

SET (YAJL_DIST_NAME "yajl-${YAJL_MAJOR}.${YAJL_MINOR}.${YAJL_MICRO}")

if(MSVC)
    set(CMAKE_DEBUG_POSTFIX "d")
endif()
IF (WIN32)
ELSE (WIN32)
  SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall")
  IF(CMAKE_COMPILER_IS_GNUCC)
    INCLUDE(CheckCCompilerFlag)
    CHECK_C_COMPILER_FLAG(-fvisibility=hidden HAVE_GCC_VISIBILITY)
    IF(HAVE_GCC_VISIBILITY)
      SET(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fvisibility=hidden")
    ENDIF(HAVE_GCC_VISIBILITY)
  ENDIF(CMAKE_COMPILER_IS_GNUCC)
  SET(CMAKE_C_FLAGS
      "${CMAKE_C_FLAGS} -std=c99 -pedantic -Wpointer-arith -Wno-format-y2k -Wstrict-prototypes -Wmissing-declarations -Wnested-externs -Wextra	-Wundef -Wwrite-strings -Wold-style-definition -Wredundant-decls -Wno-unused-parameter -Wno-sign-compare -Wmissing-prototypes")

  SET(CMAKE_C_FLAGS_DEBUG "-DDEBUG -g")
  SET(CMAKE_C_FLAGS_RELEASE "-DNDEBUG -O2 -Wuninitialized")
ENDIF (WIN32)

SET (SRCS
    src/yajl.c
    src/yajl_lex.c
    src/yajl_parser.c
    src/yajl_buf.c
    src/yajl_encode.c
    src/yajl_gen.c
    src/yajl_alloc.c
    src/yajl_tree.c
    src/yajl_version.c
)

# useful when fixing lexer bugs.
#ADD_DEFINITIONS(-DYAJL_LEXER_DEBUG)

ADD_LIBRARY(yajl ${SRCS}) 
target_include_directories(yajl PRIVATE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include;${CMAKE_CURRENT_BINARY_DIR}/include>
  INTERFACE
  $<INSTALL_INTERFACE:include/yajl>
  )
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

if(MSVC)
    target_compile_definitions(yajl PRIVATE WIN32 _CRT_SECURE_NO_DEPRECATE _CRT_NONSTDC_NO_DEPRECATE YAJL_BUILD)
endif()

# generate build-time source
CONFIGURE_FILE(${CMAKE_CURRENT_SOURCE_DIR}/include/yajl/yajl_version.h.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/include/yajl/yajl_version.h)
CONFIGURE_FILE(src/yajl.pc.cmake yajl.pc)

if(BUILD_TOOLS)
    add_executable(json_reformat reformatter/json_reformat.c)
    target_link_libraries(json_reformat yajl)

    add_executable(json_verify verify/json_verify.c)
    target_link_libraries(json_verify yajl)
endif()

if(BUILD_EXAMPLES)
    add_executable(parse_config example/parse_config.c)
    target_link_libraries(parse_config yajl)
endif()

enable_testing()
add_executable(yajl_test test/parsing/yajl_test.c)
target_link_libraries(yajl_test yajl)
add_test(yajl_test yajl_test
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/test/parsing)

# ADD_SUBDIRECTORY(test)
# ADD_SUBDIRECTORY(perf)

INCLUDE(YAJLDoc.cmake)

# a test target
# ADD_CUSTOM_TARGET(test
#                   ./run_tests.sh ${CMAKE_CURRENT_BINARY_DIR}/test/parsing/yajl_test
#                   WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/test/parsing)

ADD_CUSTOM_TARGET(test-api ${CMAKE_CURRENT_SOURCE_DIR}/test/api/run_tests.sh
                  WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/test/api)

INSTALL(TARGETS yajl EXPORT yajl
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib
  ARCHIVE DESTINATION lib
)
install(EXPORT yajl
  FILE
    yajl.cmake
  NAMESPACE
    yajl::
  DESTINATION
    lib/cmake/yajl
)
install(
  FILES
    cmake/yajl-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/yajl-config-version.cmake
  DESTINATION
    lib/cmake/yajl
)

export(TARGET yajl
    FILE yajl.cmake
    NAMESPACE yajl::
)

install(DIRECTORY
    include/
    DESTINATION include
 )
install(FILES 
    ${CMAKE_CURRENT_BINARY_DIR}/include/yajl/yajl_version.h
    DESTINATION include/yajl)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/yajl.pc DESTINATION share)


if(BUILD_TOOLS)
    install(TARGETS json_reformat json_verify
        RUNTIME DESTINATION bin)
endif()
if(BUILD_EXAMPLES)
    install(TARGETS parse_config
        RUNTIME DESTINATION bin)
endif()