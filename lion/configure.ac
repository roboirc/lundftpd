AC_INIT([liblion],[2.6], [bug-report@lundman.net])
AC_CONFIG_AUX_DIR([build-aux])
AM_INIT_AUTOMAKE([foreign])
# The big search for OpenSSL
# copied from openssh's configure.ac
tryssldir=""
AC_ARG_WITH(ssl-dir,
        [  --with-ssl-dir=PATH     Specify path to OpenSSL installation ],
        [
                if test "x$withval" != "xno" ; then
                        tryssldir=$withval
                fi
        ]
)
AC_PROG_CC
AC_PROG_RANLIB

AC_TYPE_INT64_T
AC_TYPE_UINT64_T
AC_TYPE_SIGNAL
AC_C_INLINE

AC_SYS_LARGEFILE

AC_HEADER_TIME
AC_CHECK_HEADERS([stdarg.h varargs.h time.h sys/time.h sys/types.h])

AC_CHECK_FUNC(snprintf,AC_DEFINE_UNQUOTED(HAVE_SNPRINTF,1,[Presence of snprintf found]),[
 AC_MSG_WARN(Adding internal snprintf function)])

if test "x$ac_cv_prog_gcc" = xyes ; then
   CFLAGS="$CFLAGS -Wall -g"
fi

dnl ------------------------------------------------------
dnl Where do you live, openssl?  And how do we call you?
AC_ARG_WITH([openssl],
            [AS_HELP_STRING([--without-openssl],[Disable openssl support])],
            [],
            [with_openssl=yes])

dnl If SSL was NOT disable, then go look for it.
if test x"$with_openssl" != x"no" ; then

AC_CACHE_CHECK([for OpenSSL directory], ac_cv_openssl_dir, [
  saved_LIBS="$LIBS"
  saved_LDFLAGS="$LDFLAGS"
  saved_CPPFLAGS="$CPPFLAGS"
  ssl_found=no
  for ssldir in $tryssldir "" $prefix /usr/local/openssl /usr/lib/openssl /usr/local/ssl /usr/lib/ssl /usr/local /usr/athena /usr/pkg /opt /opt/openssl ; do
    LDFLAGS="$saved_LDFLAGS"
    LIBS="$saved_LIBS -lssl -lcrypto"

    # Skip the directory if it isn't there.
    if test ! -z "$ssldir" -a ! -d "$ssldir" ; then
       continue;
    fi
    if test ! -z "$ssldir" ; then
      if test -d "$ssldir/lib" ; then
        LDFLAGS="-L$ssldir/lib $LDFLAGS"
      else
        LDFLAGS="-L$ssldir $LDFLAGS"
      fi
      if test -d "$ssldir/include" ; then
        CPPFLAGS="-I$ssldir/include $CPPFLAGS"
      else
        CPPFLAGS="-I$ssldir $CPPFLAGS"
      fi
    fi
    # Can I link it?
    AC_TRY_LINK([#include <openssl/rand.h>],
       [ RAND_add((void*)0,0,0); ],
       [ openssl_linked=yes ], [ openssl_linked=no ])
    if test $openssl_linked = yes; then
       if test ! -z "$ssldir" ; then
         ac_cv_openssl_dir=$ssldir
       else
         ac_cv_openssl_dir="(system)"
       fi
       ssl_found=yes
       break
    fi
  done
  LIBS="$saved_LIBS"
  LDFLAGS="$saved_LDFLAGS"
  CPPFLAGS="$saved_CPPFLAGS"
  if test $ssl_found = no ; then
    AC_MSG_ERROR([Could not find a linkable OpenSSL. You can specify an explicit path using --with-ssl-dir])
  fi
])
AC_DEFINE([WITH_SSL], 1, [Define if we found OpenSSL - Legacy name])
LIBS="$LIBS -lssl -lcrypto"
if test "$ac_cv_openssl_dir" != "(system)"; then
  if test -d "$ac_cv_openssl_dir/lib" ; then
    LDFLAGS="-L$ac_cv_openssl_dir/lib $LDFLAGS"
    ssl_libdir="$ac_cv_openssl_dir/lib"
  else
    LDFLAGS="-L$ac_cv_openssl_dir $LDFLAGS"
    ssl_libdir="$ac_cv_openssl_dir"
  fi
  if test -d "$ac_cv_openssl_dir/include" ; then
    CPPFLAGS="-I$ac_cv_openssl_dir/include $CPPFLAGS"
  else
    CPPFLAGS="-I$ac_cv_openssl_dir $CPPFLAGS"
  fi
fi

if test -z $CROSS_COMPILE
then
AC_CACHE_CHECK([whether we need extra options to link OpenSSL],
               ac_cv_openssl_linker_option, [
    saved_LDFLAGS="$LDFLAGS"
    ssl_runs=no
    linked_with=nothing
    for ssl_extra in "" "-Wl,-R$ssl_libdir" "-R$ssl_libdir" ; do
      LDFLAGS="$ssl_extra $saved_LDFLAGS"
      AC_TRY_RUN([
#include <string.h>
#include <openssl/rand.h>
int main(void)
{
        char a[2048];
        memset(a, 0, sizeof(a));
        RAND_add(a, sizeof(a), sizeof(a));
        return(RAND_status() <= 0);
}
                        ],
                openssl_runs=yes, openssl_runs=no)
      if test $openssl_runs = yes ; then
        if test "$linked_with" = nothing; then
           linked_with="$ssl_extra"
        fi
        AC_TRY_RUN([
#include <openssl/opensslv.h>
#include <openssl/crypto.h>
int main(void) {
return (OPENSSL_VERSION_NUMBER == SSLeay()) == 0;
}],
        right_version=yes, right_version=no)
        if test "$right_version" = yes; then
          if test -z "$ssl_extra" ; then
            ac_cv_openssl_linker_option='(none)'
          else
            ac_cv_openssl_linker_option=$ssl_extra
          fi
          ssl_runs=yes
          break
        fi
     fi
    done
    if test $ssl_runs = no ; then
      if test "$linked_with" = 'nothing' ; then
        AC_MSG_ERROR([Found linkable OpenSSL in $ac_cv_openssl_dir, but it doesn't run, even with -R.  Maybe specify another using --with-ssl-dir?])
      else
        if test -z "$linked_with" ; then
            ac_cv_openssl_linker_option='(none)'
        else
            ac_cv_openssl_linker_option=$linked_with
        fi
        AC_MSG_WARN([I managed to make OpenSSL link and run, but I couldn't make it link against the same version I found header files for.])
      fi
    fi
    LDFLAGS="$saved_LDFLAGS"
])

if test "$ac_cv_openssl_linker_option" != '(none)' ; then
   LDFLAGS="$ac_cv_openssl_linker_option $LDFLAGS"
fi
fi

dnl NOSSL
fi


AM_CONFIG_HEADER([config.h])
AC_CONFIG_FILES([Makefile src/Makefile contrib/Makefile contrib/libdirlist/Makefile contrib/libdirlist/win32/Makefile python/Makefile win32/Makefile samples/Makefile samples/fork/Makefile samples/get-file/Makefile samples/getdirs/Makefile samples/relay/Makefile samples/send-file/Makefile samples/ssl-connect/Makefile samples/ssl-listen/Makefile samples/system-send/Makefile samples/system/Makefile samples/template/Makefile samples/timers/Makefile samples/udp-pong/Makefile])
AC_OUTPUT

