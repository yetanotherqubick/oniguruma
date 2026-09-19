cmake_minimum_required(VERSION 3.16)

project(oniguruma VERSION 5.9.6 LANGUAGES C)

include(CheckIncludeFile)
include(CheckSymbolExists)
include(CheckTypeSize)
include(CMakePackageConfigHelpers)
include(GNUInstallDirs)
include(CTest)

option(BUILD_SHARED_LIBS "Build shared libraries" ON)
option(ONIG_BUILD_SAMPLES "Build sample programs" ON)
option(ONIG_ENABLE_STATISTICS "Enable matching statistics" OFF)
option(ONIG_ENABLE_COMBINATION_EXPLOSION_CHECK
  "Enable combination explosion checking" OFF)
option(ONIG_ENABLE_CRNL_AS_LINE_TERMINATOR
  "Treat CR+NL as a line terminator" OFF)

# 5.9.6 predates the C23 callback fixes. Keep the default build compatible
# with the source as it exists on this branch; this can be overridden by
# setting CMAKE_C_STANDARD explicitly.
set(CMAKE_C_STANDARD 17 CACHE STRING "C standard")
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS ON)

check_include_file(alloca.h     HAVE_ALLOCA_H)
check_include_file(inttypes.h   HAVE_INTTYPES_H)
check_include_file(stdarg.h     HAVE_STDARG_H)
check_include_file(stdint.h     HAVE_STDINT_H)
check_include_file(stdlib.h     HAVE_STDLIB_H)
check_include_file(string.h     HAVE_STRING_H)
check_include_file(strings.h    HAVE_STRINGS_H)
check_include_file(sys/time.h   HAVE_SYS_TIME_H)
check_include_file(sys/times.h  HAVE_SYS_TIMES_H)
check_include_file(sys/types.h  HAVE_SYS_TYPES_H)
check_include_file(time.h       HAVE_TIME_H)
check_include_file(unistd.h     HAVE_UNISTD_H)

if(HAVE_ALLOCA_H)
  check_symbol_exists(alloca "alloca.h" HAVE_ALLOCA)
else()
  check_symbol_exists(alloca "stdlib.h;malloc.h" HAVE_ALLOCA)
endif()

check_type_size(int SIZEOF_INT)
check_type_size(short SIZEOF_SHORT)
check_type_size(long SIZEOF_LONG)

set(HAVE_PROTOTYPES 1)
set(HAVE_STDARG_PROTOTYPES 1)

set(PACKAGE "onig")
set(PACKAGE_NAME "Oniguruma")
set(PACKAGE_TARNAME "onig")
set(PACKAGE_VERSION "${PROJECT_VERSION}")
set(VERSION "${PROJECT_VERSION}")

configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/config.h.in"
  "${CMAKE_CURRENT_BINARY_DIR}/config.h"
  @ONLY
)

set(ONIG_SOURCES
  regerror.c
  regparse.c
  regext.c
  regcomp.c
  regexec.c
  reggnu.c
  regenc.c
  regsyntax.c
  regtrav.c
  regversion.c
  st.c
  regposix.c
  regposerr.c
  enc/unicode.c
  enc/ascii.c
  enc/utf8.c
  enc/utf16_be.c
  enc/utf16_le.c
  enc/utf32_be.c
  enc/utf32_le.c
  enc/euc_jp.c
  enc/sjis.c
  enc/iso8859_1.c
  enc/iso8859_2.c
  enc/iso8859_3.c
  enc/iso8859_4.c
  enc/iso8859_5.c
  enc/iso8859_6.c
  enc/iso8859_7.c
  enc/iso8859_8.c
  enc/iso8859_9.c
  enc/iso8859_10.c
  enc/iso8859_11.c
  enc/iso8859_13.c
  enc/iso8859_14.c
  enc/iso8859_15.c
  enc/iso8859_16.c
  enc/euc_tw.c
  enc/euc_kr.c
  enc/big5.c
  enc/gb18030.c
  enc/koi8_r.c
  enc/cp1251.c
)

set(ONIG_PUBLIC_HEADERS
  oniguruma.h
  oniggnu.h
  onigposix.h
)

add_library(onig ${ONIG_SOURCES})
add_library(oniguruma::onig ALIAS onig)

target_include_directories(onig
  PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

if(ONIG_ENABLE_STATISTICS)
  target_compile_definitions(onig PRIVATE ONIG_DEBUG_STATISTICS)
endif()

if(ONIG_ENABLE_COMBINATION_EXPLOSION_CHECK)
  target_compile_definitions(onig PRIVATE USE_COMBINATION_EXPLOSION_CHECK)
endif()

if(ONIG_ENABLE_CRNL_AS_LINE_TERMINATOR)
  target_compile_definitions(onig PRIVATE USE_CRNL_AS_LINE_TERMINATOR)
endif()

if(MSVC AND BUILD_SHARED_LIBS)
  target_compile_definitions(onig PRIVATE EXPORT)
endif()

# Original Autotools: LTVERSION=2:0:0.
set_target_properties(onig PROPERTIES
  VERSION 2.0.0
  SOVERSION 2
)

if(BUILD_TESTING)
  add_executable(testc testc.c)
  target_link_libraries(testc PRIVATE onig)

  add_executable(testp testc.c)
  target_compile_definitions(testp PRIVATE POSIX_TEST)
  target_link_libraries(testp PRIVATE onig)

  add_executable(testcu testu.c)
  target_link_libraries(testcu PRIVATE onig)

  add_test(NAME oniguruma-testc COMMAND testc)
  add_test(NAME oniguruma-testp COMMAND testp)
  add_test(NAME oniguruma-testcu COMMAND testcu)
endif()

if(ONIG_BUILD_SAMPLES)
  add_subdirectory(sample)
endif()

install(
  TARGETS onig
  EXPORT onigurumaTargets
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

install(
  FILES ${ONIG_PUBLIC_HEADERS}
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

set(ONIG_CMAKE_INSTALL_DIR
  "${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME}"
)
set(ONIG_GENERATED_DIR
  "${CMAKE_CURRENT_BINARY_DIR}/generated"
)
set(ONIG_VERSION_CONFIG
  "${ONIG_GENERATED_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
)
set(ONIG_PROJECT_CONFIG
  "${ONIG_GENERATED_DIR}/${PROJECT_NAME}Config.cmake"
)

write_basic_package_version_file(
  "${ONIG_VERSION_CONFIG}"
  COMPATIBILITY SameMajorVersion
)

configure_package_config_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/cmake/Config.cmake.in"
  "${ONIG_PROJECT_CONFIG}"
  INSTALL_DESTINATION "${ONIG_CMAKE_INSTALL_DIR}"
)

install(
  FILES "${ONIG_PROJECT_CONFIG}" "${ONIG_VERSION_CONFIG}"
  DESTINATION "${ONIG_CMAKE_INSTALL_DIR}"
)

install(
  EXPORT onigurumaTargets
  FILE onigurumaTargets.cmake
  NAMESPACE oniguruma::
  DESTINATION "${ONIG_CMAKE_INSTALL_DIR}"
)

configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/oniguruma.pc.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/oniguruma.pc"
  @ONLY
)

install(
  FILES "${CMAKE_CURRENT_BINARY_DIR}/oniguruma.pc"
  DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig"
)

configure_file(
  "${CMAKE_CURRENT_SOURCE_DIR}/onig-config.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/onig-config"
  @ONLY
)

install(
  PROGRAMS "${CMAKE_CURRENT_BINARY_DIR}/onig-config"
  DESTINATION "${CMAKE_INSTALL_BINDIR}"
)
