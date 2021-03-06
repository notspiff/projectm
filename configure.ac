AC_INIT([projectM], [3.1.0], [mischa@mvstg.biz], [projectM], [https://github.com/revmischa/projectm])
AM_INIT_AUTOMAKE([-Wall -Werror foreign subdir-objects tar-pax])

m4_ifdef([AM_PROG_AR], [AM_PROG_AR])
LT_INIT

# Check if we should disable rpath.
#
# For advanced users: In certain configurations, the rpath attributes
# added by libtool cause problems as rpath will be preferred over
# LD_LIBRARY_PATH. This does not seem to be a problem with
# clang. When using --disable-rpath you will likely need to set
# LD_LIBRARY_PATH if you are using libraries in non-system locations.
# YMMV.
#

DISABLE_RPATH

AC_PROG_CXX
AC_LANG(C++)

AC_CONFIG_MACRO_DIRS([m4 m4/autoconf-archive])

dnl emscripten
AC_ARG_ENABLE([emscripten],
AS_HELP_STRING([--enable-emscripten], [Build for web with emscripten]),
[], [enable_emscripten=no])
AS_IF([test "x$enable_emscripten" = "xyes" || test "x$EMSCRIPTEN" = "xyes"], [
  dnl Set up emscripten
  m4_include([m4/emscripten.m4])
  AC_DEFINE([EMSCRIPTEN], [1], [Define EMSCRIPTEN])
  enable_threading=no
  enable_gles=yes
  enable_sdl=yes
], [
  dnl Running in a normal OS (not emscripten)
  AX_CHECK_GL

  # check OS
  AC_CANONICAL_HOST
  AC_MSG_CHECKING(Freedom)
  case $host_os in
  darwin*)
    # OSX needs CoreFoundation
    AC_MSG_RESULT(Apple hoarderware detected)
    LIBS="$LIBS -framework CoreFoundation"
    ;;
  linux*)
    # limux needs dl
    AC_MSG_RESULT(GNU/LINUX detected)
    LIBS="$LIBS -ldl"
    ;;
  esac
])


AC_CHECK_LIB(c, dlopen, LIBDL="", AC_CHECK_LIB(dl, dlopen, LIBDL="-ldl"))

AC_CHECK_FUNCS_ONCE([aligned_alloc posix_memalign])

AC_CONFIG_HEADERS([config.h])
AC_CONFIG_FILES([
  Makefile
  src/Makefile
  src/libprojectM/Makefile
  src/libprojectM/Renderer/Makefile
  src/libprojectM/NativePresetFactory/Makefile
  src/libprojectM/MilkdropPresetFactory/Makefile
  src/libprojectM/libprojectM.pc
  src/NativePresets/Makefile
  src/projectM-sdl/Makefile
  src/projectM-emscripten/Makefile
  src/projectM-qt/Makefile
  src/projectM-pulseaudio/Makefile
])

dnl SDL
AC_ARG_ENABLE([sdl],
  AS_HELP_STRING([--enable-sdl], [Build SDL2 app]),
  [], [enable_sdl=no])
AS_IF([test "x$enable_sdl" = "xyes"], [
  m4_include([m4/sdl2.m4])
  SDL_VERSION=2.0.5
  AS_IF([test "$TRAVIS"], [SDL_VERSION=2.0.2])  # travis has old SDL, we don't care
  AS_IF([test "EMSCRIPTEN"], [SDL_VERSION=2.0.0])  # emscripten has old SDL, we don't care
  AM_PATH_SDL2($SDL_VERSION, :, AC_MSG_ERROR([*** SDL version $SDL_VERSION not found!]))
])

dnl glm
AS_IF([test "x$enable_emscripten" != "xyes"], [
  AC_CHECK_HEADER([glm/glm.hpp],, AC_MSG_ERROR(libglm is required.))
])

dnl Threading
AC_ARG_ENABLE([threading],
AS_HELP_STRING([--enable-threading], [multhreading]),
[], [enable_threading=yes])
AS_IF([test "x$enable_threading" = "xyes" && ! test "$EMSCRIPTEN"], [
  m4_include([m4/autoconf-archive/ax_pthread.m4])
  AX_PTHREAD([
    AC_DEFINE([USE_THREADS], [1], [Define USE_THREADS])
    LIBS="$LIBS $PTHREAD_LIBS $PTHREAD_CFLAGS"
    CFLAGS="$CFLAGS $PTHREAD_CFLAGS"
    CXXFLAGS="$CXXFLAGS $PTHREAD_CFLAGS"
    echo "LIBS=$LIBS"
  ], [
    AC_MSG_ERROR([pthreads not found])
  ])
])

AC_ARG_ENABLE([gles],
  AS_HELP_STRING([--enable-gles], [OpenGL ES support]),
  [], [enable_gles=no])
  AS_IF([test "x$enable_gles" = "xyes"], [
    AC_DEFINE([USE_GLES], [1], [Define USE_GLES])
])

dnl from https://stackoverflow.com/questions/30897170/ac-subst-does-not-expand-variable answer: https://stackoverflow.com/a/30960268
dnl ptomato https://stackoverflow.com/users/172999/ptomato
AC_SUBST([PACKAGE])
AC_PROG_SED
AC_CONFIG_FILES([src/libprojectM/config.inp.in])

AC_PREFIX_DEFAULT([/usr/local])

AC_PROG_MKDIR_P

AX_CHECK_COMPILE_FLAG([-stdlib=libc++], [
                        CXXFLAGS="$CXXFLAGS -stdlib=libc++"])

AX_CHECK_COMPILE_FLAG([-std=c++11], [
                        CXXFLAGS="$CXXFLAGS -std=c++11"])

dnl Qt
AC_ARG_ENABLE([qt],
  AS_HELP_STRING([--enable-qt], [Build Qt]),
  [], [enable_qt=no])
AS_IF([test "x$enable_qt" = "xyes"], [
  PKG_CHECK_MODULES(QT, [Qt5Core, Qt5Gui, Qt5Widgets Qt5OpenGL], [], [AC_MSG_ERROR([Qt libraries are required.])])
  qt_CPPFLAGS="`$PKG_CONFIG --cflags-only-I Qt5Core Qt5Gui Qt5Widgets Qt5OpenGL` $CPPFLAGS"
  qt_LDFLAGS="`$PKG_CONFIG --libs-only-L Qt5Core Qt5Gui Qt5Widgets Qt5OpenGL` $LDFLAGS"
  qt_LIBS="`$PKG_CONFIG --libs-only-l Qt5Core Qt5Gui Qt5Widgets Qt5OpenGL` $LIBS"

  if ! `$PKG_CONFIG --atleast-version=5.0.0 Qt5Core`; then
    AC_MSG_ERROR([Qt >= 5.0.0 is required. Try installing qtdeclarative5-dev])
  fi

  AC_CHECK_PROGS(MOC, [moc-qt5 moc])
  AC_CHECK_PROGS(UIC, [uic-qt5 uic])
  AC_CHECK_PROGS(RCC, [rcc-qt5 rcc])
  if test -z "$MOC" || test -z "$UIC" || test -z "$RCC"; then
    AC_MSG_ERROR([Qt utility programs moc, uic, and rcc are required.])
  fi

  PKG_CHECK_MODULES(LIBPULSE, [libpulse], [], [AC_MSG_ERROR([Pulseaudio library libpulse is required.])])
])

AM_CONDITIONAL([ENABLE_SDL], [test "$enable_sdl" = yes])
AM_CONDITIONAL([ENABLE_QT], [test "$enable_qt" = yes])
AM_CONDITIONAL([ENABLE_EMSCRIPTEN], [test "$enable_emscripten" = yes])


my_CFLAGS="-Wall -Wchar-subscripts -Wformat-security -Wmissing-declarations -Wpointer-arith -Wshadow -Wsign-compare -Wtype-limits "
my_CFLAGS+='-DDATADIR_PATH=\""$(pkgdatadir)"\" '
my_CFLAGS+='-I$(top_srcdir)/vendor '
AC_SUBST([my_CFLAGS])



AC_OUTPUT

AC_MSG_RESULT([
projectM v$VERSION
=====

prefix:                 ${prefix}
sysconfdir:             ${sysconfdir}
libdir:                 ${libdir}
includedir:             ${includedir}

compiler:               ${CC}
cflags:                 ${CFLAGS} ${my_CFLAGS}
ldflags:                ${LDFLAGS}

- - -

Applications:
=====

libprojectM:            yes
Threading:              ${enable_threading}
SDL:                    ${enable_sdl}
Qt & Pulseaudio:        ${enable_qt}
OpenGLES:               ${enable_gles}
Emscripten:             ${enable_emscripten}
])
