AC_INIT([onig], [5.9.6])

AC_CONFIG_MACRO_DIRS([m4])

AM_INIT_AUTOMAKE([foreign subdir-objects])

AC_CONFIG_HEADERS([config.h])

dnl ----------------------------------------------------------------------
dnl Options
dnl ----------------------------------------------------------------------

RUBYDIR=".."

AC_ARG_WITH(
  [rubydir],
  [AS_HELP_STRING([--with-rubydir=RUBYDIR],
                  [specify Ruby source directory @<:@default ..@:>@])],
  [RUBYDIR=$withval]
)

AC_SUBST([RUBYDIR])

STATISTICS=""

AC_ARG_WITH(
  [statistics],
  [AS_HELP_STRING([--with-statistics],
                  [enable matching time statistics])],
  [STATISTICS=-DONIG_DEBUG_STATISTICS]
)

AC_SUBST([STATISTICS])

AC_ARG_ENABLE(
  [combination-explosion-check],
  [AS_HELP_STRING([--enable-combination-explosion-check],
                  [enable combination explosion check])],
  [],
  [enable_combination_explosion_check=no]
)

AS_IF(
  [test "x$enable_combination_explosion_check" = xyes],
  [AC_DEFINE(
     [USE_COMBINATION_EXPLOSION_CHECK],
     [1],
     [Define to enable combination explosion checking]
   )]
)

AC_ARG_ENABLE(
  [crnl-as-line-terminator],
  [AS_HELP_STRING([--enable-crnl-as-line-terminator],
                  [enable CR+NL as line terminator])],
  [],
  [enable_crnl_as_line_terminator=no]
)

AS_IF(
  [test "x$enable_crnl_as_line_terminator" = xyes],
  [AC_DEFINE(
     [USE_CRNL_AS_LINE_TERMINATOR],
     [1],
     [Define to enable CR+NL as line terminator]
   )]
)

dnl ----------------------------------------------------------------------
dnl Toolchain
dnl ----------------------------------------------------------------------

AC_PROG_CC
AC_PROG_INSTALL
AM_PROG_AR
LT_INIT

dnl ----------------------------------------------------------------------
dnl Headers
dnl ----------------------------------------------------------------------

AC_CHECK_HEADERS([
  stdlib.h
  string.h
  strings.h
  sys/time.h
  unistd.h
  sys/times.h
])

dnl ----------------------------------------------------------------------
dnl Types / compiler characteristics
dnl ----------------------------------------------------------------------

AC_CHECK_SIZEOF([int], [4])
AC_CHECK_SIZEOF([short], [2])
AC_CHECK_SIZEOF([long], [4])

dnl ----------------------------------------------------------------------
dnl Functions
dnl ----------------------------------------------------------------------

AC_FUNC_ALLOCA
AC_FUNC_MEMCMP

dnl ----------------------------------------------------------------------
dnl Language features
dnl ----------------------------------------------------------------------

AC_CACHE_CHECK(
  [for C prototypes],
  [onig_cv_have_prototypes],
  [
    AC_COMPILE_IFELSE(
      [AC_LANG_PROGRAM(
        [[int foo(int x) { return x; }]],
        [[return foo(10);]]
      )],
      [onig_cv_have_prototypes=yes],
      [onig_cv_have_prototypes=no]
    )
  ]
)

AS_IF(
  [test "x$onig_cv_have_prototypes" = xyes],
  [AC_DEFINE(
     [HAVE_PROTOTYPES],
     [1],
     [Define if the compiler supports prototypes]
   )]
)

AC_CACHE_CHECK(
  [for variable length prototypes and stdarg.h],
  [onig_cv_stdarg],
  [
    AC_COMPILE_IFELSE(
      [AC_LANG_PROGRAM(
        [[
#include <stdarg.h>
int foo(int x, ...)
{
  va_list va;
  va_start(va, x);
  va_arg(va, int);
  va_arg(va, char *);
  va_arg(va, double);
  va_end(va);
  return 0;
}
        ]],
        [[return foo(10, "", 3.14);]]
      )],
      [onig_cv_stdarg=yes],
      [onig_cv_stdarg=no]
    )
  ]
)

AS_IF(
  [test "x$onig_cv_stdarg" = xyes],
  [AC_DEFINE(
     [HAVE_STDARG_PROTOTYPES],
     [1],
     [Define if the compiler supports stdarg prototypes]
   )]
)

AC_CONFIG_FILES([
  Makefile
  sample/Makefile
])

AC_OUTPUT
