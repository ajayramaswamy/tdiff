#  tdiff - tree diffs
#  configure.ac
#  Copyright (C) 1999, 2006, 2008, 2014, 2019 Philippe Troin <phil+github-commits@fifi.org>
#
#  This program is free software; you can redistribute it and/or modify
#  it under the terms of the GNU General Public License as published by
#  the Free Software Foundation; either version 3 of the License, or
#  (at your option) any later version.
#
#  This program is distributed in the hope that it will be useful,
#  but WITHOUT ANY WARRANTY; without even the implied warranty of
#  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#  GNU General Public License for more details.
#
#  You should have received a copy of the GNU General Public License
#  along with this program.  If not, see <http://www.gnu.org/licenses/>.

AC_INIT([tdiff],[0.8.5])
AC_CONFIG_AUX_DIR([config.aux])
AC_CANONICAL_TARGET()
FI_PROJECT()

# Get started
AC_CONFIG_SRCDIR([tdiff.c])
AC_LANG([C])

### 1) Checking for system
AC_USE_SYSTEM_EXTENSIONS
AC_SYS_LARGEFILE

### 2) Checking for programs
AC_PROG_CC
FI_COMPILER_WARNINGS
AC_PROG_CPP
AM_PROG_AR
AC_PROG_RANLIB
AC_PROG_EGREP
AC_PROG_INSTALL
AC_CHECK_PROG([FAKEROOT],[fakeroot],[fakeroot])
FI_FORMAT_MAN([tdiff.1], [html pdf])

FI_GIT_REVISION

FI_CHECK_SHELL_FUNCTIONS_DIR([bash],[$datarootdir/bash-completion/completions])
FI_CHECK_SHELL_FUNCTIONS_DIR([zsh],[$datarootdir/zsh/site-functions])

### 3) Checking for libraries

### 4) Checking for header files

AC_HEADER_TIOCGWINSZ

# Getdents
AH_TEMPLATE(HAVE_SYS_DIRENT_H, [Define if your system has <sys/dirent.h>])
AH_TEMPLATE(STRUCT_DIRENT, [Define to the name of the struct used by getdents()])
AH_TEMPLATE(HAVE_GETDENTS_SYSCALL_H, [Define to 1 if you have SYS_getdents in <syscall.h>])
AH_TEMPLATE(HAVE_GETDENTS64_SYSCALL_H, [Define to 1 if you have SYS_getdents64 in <syscall.h>])

getdents="not-found"

case "$target_os" in
  linux*)
    FI_CHECK_EGREP([getdents syscall],
		   [fi_found_sys_getdents],
		   [#include <syscall.h>
		    #ifdef SYS_getdents
		    fi_found_sys_getdents
		    #endif
		   ],
		   [],
		   [getdents=syscall
		    AC_DEFINE(HAVE_GETDENTS_SYSCALL_H,1)
		   ])
    FI_CHECK_EGREP([getdents64 syscall],
		   [fi_found_sys_getdents64],
		   [#include <syscall.h>
		    #ifdef SYS_getdents64
		    fi_found_sys_getdents64
		    #endif
		   ],
		   [],
		   [getdents=syscall64
		    AC_DEFINE(HAVE_GETDENTS64_SYSCALL_H,1)
		   ])
    ;;
esac

FI_CHECK_EGREP([getdents],
	       [getdents],
	       [#include <sys/dirent.h>],
	       [in libc],
	       [getdents="sys_dirent"
		AC_DEFINE(HAVE_SYS_DIRENT_H, 1)
	       ])

case "$getdents" in
    sys_dirent)
	getdents_decl="#include <sys/dirent.h>"
	AC_DEFINE(STRUCT_DIRENT, dirent)
	;;
    syscall)
	getdents_decl="#include \"linux_getdents.c\""
	AC_DEFINE(STRUCT_DIRENT, linux_dent)
	;;
    syscall64)
	getdents_decl="#include \"linux_getdents64.c\""
	AC_DEFINE(STRUCT_DIRENT, linux_dent64)
	;;
esac

AC_CACHE_CHECK([for random in stdlib.h], fi_cv_have_random, [
  AC_EGREP_CPP(random, [#include <stdlib.h>],
	       fi_cv_have_random=yes,
	       fi_cv_have_random=no)])
if test "x$fi_cv_have_random" = xyes
then
  AC_DEFINE(HAVE_RANDOM,1,
	    [Define HAVE_RANDOM to 1 if random() is declared in <stdlib.h>])
fi

# Major / minor
AH_TEMPLATE(HAVE_SYS_MKDEV_H, [Define to 1 if your system has major and minor definitions in mkdev.h])
AH_TEMPLATE(HAVE_SYS_SYSMACROS_H, [Define to 1 if your system has major and minor definitions in sys/sysmacros.h])
AH_TEMPLATE(HAVE_MAJOR_MINOR_FUNCTIONS, [Define to 1 if your system has major and minor functions])

major_keep_trying=yes
AC_CACHE_CHECK([for major and minor in mkdev.h], fi_cv_majmin_mkdev, [
  AC_EGREP_CPP(major,
	       [#include <sys/mkdev.h>],
	       fi_cv_majmin_mkdev=yes,
	       fi_cv_majmin_mkdev=no)])
if test "x$fi_cv_majmin_mkdev" = xyes
then
  AC_DEFINE(HAVE_SYS_MKDEV_H,1)
  unset major_keep_trying
fi

if test "$major_keep_trying"
then
  AC_CACHE_CHECK([for major and minor in sysmacros.h], fi_cv_majmin_sysmacros, [
    AC_EGREP_CPP(oh_yes,
		 [#include <sys/sysmacros.h>
		  #if defined(minor) && defined(major)
		  oh_yes
		  #endif
		 ],
		 fi_cv_majmin_sysmacros=yes,
		 fi_cv_majmin_sysmacros=no)])
  if test "x$fi_cv_majmin_sysmacros" = xyes
  then
    AC_DEFINE(HAVE_SYS_SYSMACROS_H, 1)
    unset major_keep_trying
  fi
fi

if test "$major_keep_trying"
then
  AC_CACHE_CHECK([for major() and minor() functions], [fi_cv_majmin_functions],
  [AC_COMPILE_IFELSE([AC_LANG_SOURCE([
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int main()
{
  return major(0)+minor(0);
}])],
		     [fi_cv_majmin_functions=yes],
		     [fi_cv_majmin_functions=no])])
  if test "x$fi_cv_majmin_functions" = xyes
  then
    AC_DEFINE(HAVE_MAJOR_MINOR_FUNCTIONS, 1)
    unset major_keep_trying
  fi
fi

if test "$major_keep_trying"
then
    AC_MSG_ERROR([Cannot find a definition for major() and minor()])
fi

# Struct winsize
AC_CACHE_CHECK([for struct winsize in sys/tty.h], [fi_cv_winsize_sys_tty_h],
	       [AC_COMPILE_IFELSE([AC_LANG_SOURCE([
#include <sys/tty.h>

int main()
{
  struct winsize ws;
  return 0;
}])],
		   [fi_cv_winsize_sys_tty_h=yes],
		   [fi_cv_winsize_sys_tty_h=no])])
if test "x$fi_cv_winsize_sys_tty_h" = xyes
then
  AC_DEFINE([HAVE_SYS_TTY_H], [1],
	    [Define to 1 if struct winsize is in sys/tty.h.])
fi

### 5) Checking for typedefs: NONE

### 6) Checking for macros

FI_CHECK_DEFINES([O_NOATIME O_DIRECTORY], [#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>])

FI_CHECK_DEFINES([S_IFLNK S_IFIFO S_IFSOCK S_IFDOOR S_IFWHT \
		  UF_NODUMP UF_IMMUTABLE UF_APPEND UF_OPAQUE UF_NOUNLINK \
		  UF_COMPRESSED UF_TRACKED UF_SYSTEM UF_SPARSE UF_OFFLINE \
		  UF_REPARSE UF_ARCHIVE UF_READONLY UF_HIDDEN \
		  SF_ARCHIVED SF_IMMUTABLE SF_APPEND SF_NOUNLINK SF_SNAPSHOT],
		 [#include <sys/stat.h>],
		 [in <sys/stat.h>])

### 6) Checking for structures:
FI_CHECK_IDENTIFIERS([st_atimespec st_flags],
		     [#include <sys/stat.h>],
		     [in <sys/stat.h>])

AC_CACHE_CHECK([for nanosecond resolution in stat], [fi_cv_have_st_timensec],
[AC_COMPILE_IFELSE([AC_LANG_SOURCE([
#include <sys/types.h>
#include <sys/stat.h>
#include <unistd.h>

int main()
{
  struct stat sbuf;
  return sbuf.st_atim.tv_nsec;
}])],
		   [fi_cv_have_st_timensec=yes],
		   [fi_cv_have_st_timensec=no])])
if test "x$fi_cv_have_st_timensec" = xyes
then
  AC_DEFINE([HAVE_ST_TIMENSEC], [1],
	    [Define to 1 if your system provides nanosecond timestamps (st_atimensec, st_ctimensec, st_mtimensec)])
fi

FI_CHECK_IDENTIFIER([lgetxattr], [#include <sys/xattr.h>], [in <sys/xattr.h>])

### 7) Checking for compiler characteristics
AC_C_INLINE
AC_CHECK_SIZEOF(short)
AC_CHECK_SIZEOF(int)
AC_CHECK_SIZEOF(long)
AC_CHECK_SIZEOF(long long)
AC_CHECK_SIZEOF(void *)
AC_CHECK_SIZEOF(ino_t)
AC_CHECK_SIZEOF(dev_t)

### 8) Checking for library functions
FI_LINK_IFELSE([for getopt_long()], [HAVE_GETOPT_LONG],
	       [AC_LANG_PROGRAM([[#include <unistd.h>
				  #include <getopt.h>
				 ]],
				[[char *a[1];
				  getopt_long(0, a, "foo", NULL, NULL);
				 ]])],
	       [Define to 1 if your system has the GNU getopt_long().])

FI_LINK_IFELSE([for fdopendir()], [HAVE_FDOPENDIR],
	       [AC_LANG_PROGRAM([[#include <sys/types.h>
				  #include <dirent.h>
				 ]],
				[[fdopendir(0);
				 ]])],
	       [Define to 1 if your system has fdopendir().])

FI_LINK_IFELSE([for mallinfo2()], [HAVE_MALLINFO2],
	       [AC_LANG_PROGRAM([[#include <malloc.h>]],
				[[mallinfo2();]])],
	       [Define to 1 if your system has mallinfo2().])

if test "x$getdents" != "xnot-found"
then
  AC_CACHE_CHECK([for working getdents],
		 [fi_cv_working_getdents],
		 [fi_saved_CPPFLAGS="$CPPFLAGS"
		 CPPFLAGS="$CPPFLAGS -I$srcdir"
		 AC_RUN_IFELSE([AC_LANG_SOURCE([[#include <sys/types.h>
						  #include <fcntl.h>
						  $getdents_decl
						  int main() {
						    char buf[10000];
						    int fd;
						    int nread;
						    fd = open(".", O_RDONLY);
						    nread = getdents(fd, buf, 10000);
						    return nread<=0;
						  }
						 ]])
				],
				[fi_cv_working_getdents=yes],
				[fi_cv_working_getdents=no],
				[fi_cv_working_getdents=cross])
		 CPPFLAGS="$fi_saved_CPPFLAGS"])

  if test "x$fi_cv_working_getdents" = xcross
  then
    AC_MSG_WARN([assuming that getdents() works on cross-compiling target])
    fi_cv_working_getdents=yes
  fi
  if test "x$fi_cv_working_getdents" = xyes
  then
    AC_DEFINE(HAVE_GETDENTS,1,[Define to 1 if your system had a working getdents().])
  fi
fi

FI_LINK_IFELSE([for lstat()], [HAVE_LSTAT],
	       [AC_LANG_PROGRAM([[#include <sys/stat.h>
				  #include <unistd.h>
				 ]],
				[[struct stat buf;
				  lstat("/", &buf);]
				])],
	       [Define to 1 if your system has lstat().])

AC_SEARCH_LIBS([acl_get_file],[acl],[
  AC_DEFINE([HAVE_ACL],1,[Define if your system has ACLs])
  have_acls=yes
],[have_acls=no])

FI_USE_XNET([no])
FI_CHECK_LIB_FOR_SOCKETS

### 9) Check for "system services"

### END: Define automake conditionals
AM_CONDITIONAL([HAVE_S_IFSOCK], [test x$fi_cv_S_IFSOCK = xyes])
AC_SUBST([HAVE_S_IFSOCK],[$fi_cv_S_IFSOCK])

AM_CONDITIONAL([HAVE_S_IFDOOR], [test x$fi_cv_S_IFDOOR = xyes])
AC_SUBST([HAVE_S_IFDOOR],[$fi_cv_S_IFDOOR])

# Define further flags for the Makefile, for testing
AC_SUBST([TARGET_OS],[$target_os])
AC_SUBST([HAVE_ST_FLAGS],[$fi_cv_st_flags])
AC_SUBST([HAVE_GETOPT_LONG],[$fi_cv_HAVE_GETOPT_LONG])
AC_SUBST([HAVE_XATTRS],[$fi_cv_getxattr])
AC_SUBST([HAVE_ACLS],[$have_acls])

if test "x$fi_cv_have_st_timensec" = xyes -o "x$fi_cv_st_atimespec" = xyes
then
  have_nanosecond_time_resolution=yes
else
  have_nanosecond_time_resolution=no
fi
AC_SUBST([HAVE_NANOSECOND_TIME_RESOLUTION],[$have_nanosecond_time_resolution])

### END: output

AC_CONFIG_HEADER(config.h)
AC_CONFIG_FILES([Makefile])
AC_OUTPUT

# Local variables:
# mode: sh
# end:
