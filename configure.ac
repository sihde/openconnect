AC_INIT(openconnect, 4.00)
PKG_PROG_PKG_CONFIG
AC_LANG_C
AC_CANONICAL_HOST
AM_MAINTAINER_MODE([enable])
AM_INIT_AUTOMAKE([foreign])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES([yes])])

# Define htmldir and localedir for users of ancient autoconf building from git
AC_PREREQ([2.59c], [], [AC_SUBST([htmldir], [m4_ifset([AC_PACKAGE_TARNAME],
	  ['${datadir}/doc/${PACKAGE_TARNAME}'],
	  ['${datadir}/doc/${PACKAGE}'])
])])

AC_PREREQ([2.60], [], [AC_SUBST([localedir], ['$(datadir)/locale'])])

# Upstream's pkg.m4 (since 0.27) offers this now, but define our own
# compatible version in case the local version of pkgconfig isn't new enough.
# https://bugs.freedesktop.org/show_bug.cgi?id=48743
m4_ifdef([PKG_INSTALLDIR], [PKG_INSTALLDIR],
	  [AC_ARG_WITH([pkgconfigdir],
		       [AS_HELP_STRING([--with-pkgconfigdir],
		       [install directory for openconnect.pc pkg-config file])],
			[],[with_pkgconfigdir='$(libdir)/pkgconfig'])
	   AC_SUBST([pkgconfigdir], [${with_pkgconfigdir}])])

AC_ARG_WITH([vpnc-script],
	[AS_HELP_STRING([--with-vpnc-script],
	  [default location of vpnc-script helper])])

if test "$with_vpnc_script" = "yes" || test "$with_vpnc_script" = ""; then
   with_vpnc_script=/etc/vpnc/vpnc-script
   if ! test -x "$with_vpnc_script"; then
      AC_MSG_ERROR([${with_vpnc_script} does not seem to be executable.]
    [OpenConnect will not function correctly without a vpnc-script.]
    [See http://www.infradead.org/openconnect/vpnc-script.html for more details.]
    []
    [If you are building a distribution package, please ensure that your]
    [packaging is correct, and that a vpnc-script will be installed when the]
    [user installs your package. You should provide a --with-vpnc-script=]
    [argument to this configure script, giving the full path where the script]
    [will be installed.]
    []
    [The standard location is ${with_vpnc_script}. To bypass this error and]
    [build OpenConnect to use the script from this location, even though it's]
    [not present at the time you are building OpenConnect, pass the argument]
    ["--with-vpnc-script=${with_vpnc_script}"])
  fi
elif test "$with_vpnc_script" = "no"; then
   AC_ERROR([You cannot disable vpnc-script.]
   [OpenConnect will not function correctly without it.]
   [See http://www.infradead.org/openconnect/vpnc-script.html])
fi

AC_DEFINE_UNQUOTED(DEFAULT_VPNCSCRIPT, "${with_vpnc_script}")
AC_SUBST(DEFAULT_VPNCSCRIPT, "${with_vpnc_script}")

case $host_os in
 *linux* | *gnu*)
    AC_MSG_NOTICE([Applying feature macros for GNU build])
    AC_DEFINE(_POSIX_C_SOURCE, 200112L)
    # For strcasecmp() 
    AC_DEFINE(_BSD_SOURCE)
    # For asprintf()
    AC_DEFINE(_GNU_SOURCE)
    ;;
 *netbsd*)
    AC_MSG_NOTICE([Applying feature macros for NetBSD build])
    AC_DEFINE(_POSIX_C_SOURCE, 200112L)
    AC_DEFINE(_NETBSD_SOURCE)
    ;;
 *)
    # On FreeBSD the only way to get vsyslog() visible is to define
    #  *nothing*, which makes absolutely everything visible.
    # On Darwin enabling _POSIX_C_SOURCE breaks <sys/mount.h> because
    # u_long and other types don't get defined. OpenBSD is similar.
    ;;
esac

symver_time=
symver_getline=
symver_asprintf=

case $host_os in
 *solaris*|*sunos*)
    symver_time="openconnect__time;"
    ;;
esac

AC_CHECK_FUNC(fdevname_r, [AC_DEFINE(HAVE_FDEVNAME_R, 1)], [])
AC_CHECK_FUNC(getline, [AC_DEFINE(HAVE_GETLINE, 1)], [symver_getline="openconnect__getline;"])
AC_CHECK_FUNC(strcasestr, [AC_DEFINE(HAVE_STRCASESTR, 1)], [])
AC_CHECK_FUNC(asprintf, [AC_DEFINE(HAVE_ASPRINTF, 1)], [symver_asprintf="openconnect__asprintf;"])
if test -n "$symver_asprintf"; then
  AC_MSG_CHECKING([for va_copy])
  AC_LINK_IFELSE([AC_LANG_PROGRAM([
	#include <stdarg.h>
	va_list a;],[
	va_list b;
	va_copy(b,a);
	va_end(b);])],
	[AC_DEFINE(HAVE_VA_COPY, 1)
	AC_MSG_RESULT(va_copy)],
	[AC_LINK_IFELSE([AC_LANG_PROGRAM([
		#include <stdarg.h>
		va_list a;],[
		va_list b;
		__va_copy(b,a);
		va_end(b);])],
		[AC_DEFINE(HAVE___VA_COPY, 1)
		AC_MSG_RESULT(__va_copy)],
		[AC_MSG_RESULT(no)
		AC_MSG_ERROR([Your system lacks asprintf() and va_copy()])])
	])
fi
AC_SUBST(SYMVER_TIME, $symver_time)
AC_SUBST(SYMVER_GETLINE, $symver_getline)
AC_SUBST(SYMVER_ASPRINTF, $symver_asprintf)

AS_COMPILER_FLAGS(CFLAGS,
        "-Wall
         -Wextra
         -Wno-missing-field-initializers
         -Wno-sign-compare
         -Wno-unused-parameter
         -Werror=pointer-to-int-cast
         -Wdeclaration-after-statement
         -Werror-implicit-function-declaration
         -Wformat-nonliteral
         -Wformat-security
         -Winit-self
         -Wmissing-declarations
         -Wmissing-include-dirs
         -Wnested-externs
         -Wpointer-arith
         -Wwrite-strings")

AC_ENABLE_SHARED
AC_DISABLE_STATIC

AC_ARG_ENABLE([nls],
	[  --disable-nls           do not use Native Language Support],
	[USE_NLS=$enableval], [USE_NLS=yes])
LIBINTL=
if test "$USE_NLS" = "yes"; then
   AC_PATH_PROG(MSGFMT, msgfmt)
   if test "$MSGFMT" = ""; then
      AC_ERROR([msgfmt could not be found. Try configuring with --disable-nls])
   fi
fi
LIBINTL=
if test "$USE_NLS" = "yes"; then
   AC_MSG_CHECKING([for functional NLS support])
   AC_LINK_IFELSE([AC_LANG_PROGRAM([
    #include <locale.h>
    #include <libintl.h>],[
    setlocale(LC_ALL, "");
    bindtextdomain("openconnect", "/tmp");
    (void)dgettext("openconnect", "foo");])],
    [AC_MSG_RESULT(yes)],
    [oldLIBS="$LIBS"
     LIBS="$LIBS -lintl"
     AC_LINK_IFELSE([AC_LANG_PROGRAM([
      #include <locale.h>
      #include <libintl.h>],[
      setlocale(LC_ALL, "");
      bindtextdomain("openconnect", "/tmp");
      (void)dgettext("openconnect", "foo");])],
      [AC_MSG_RESULT(yes (with -lintl))]
       LIBINTL="-lintl",
      [AC_MSG_RESULT(no)
       USE_NLS=no])
     LIBS="$oldLIBS"])
fi

if test "$USE_NLS" = "yes"; then
   AC_SUBST(LIBINTL)
   AC_DEFINE(ENABLE_NLS, 1)
fi
AM_CONDITIONAL(USE_NLS, [test "$USE_NLS" = "yes"])

# We will use GnuTLS if it's requested, and if GnuTLS doesn't have DTLS
# support then we'll *also* use OpenSSL for that, but it appears *only*
# only in the openconnect executable and not the library (hence shouldn't
# be a problem for GPL'd programs using libopenconnect).
#
# If built with --with-gnutls --without-openssl then we'll even eschew
# OpenSSL for DTLS support and will build without any DTLS support at all
# if GnuTLS cannot manage.
#
# The default (for now) is to use OpenSSL for everything.

AC_ARG_WITH([gnutls],
	AS_HELP_STRING([--with-gnutls], 
		       [Use GnuTLS instead of OpenSSL (EXPERIMENTAL)]))
AC_ARG_WITH([openssl],
            AS_HELP_STRING([--with-openssl],
                           [Location of OpenSSL build dir]))
ssl_library=

if test "$with_gnutls" = "yes"; then
    PKG_CHECK_MODULES(GNUTLS, gnutls)
    if ! $PKG_CONFIG --atleast-version=2.12.16 gnutls; then
       AC_MSG_ERROR([Your GnuTLS is too old. At least v2.12.16 is required])
    fi
    oldlibs="$LIBS"
    LIBS="$LIBS $GNUTLS_LIBS"
    AC_CHECK_FUNC(gnutls_certificate_set_x509_system_trust,
		 [AC_DEFINE(HAVE_GNUTLS_CERTIFICATE_SET_X509_SYSTEM_TRUST, 1)], [])
    AC_CHECK_FUNC(gnutls_pkcs12_simple_parse,
		 [AC_DEFINE(HAVE_GNUTLS_PKCS12_SIMPLE_PARSE, 1)], [])
    AC_CHECK_FUNC(gnutls_certificate_set_key,
		 [AC_DEFINE(HAVE_GNUTLS_CERTIFICATE_SET_KEY, 1)], [])
    if test "$with_openssl" = "" || test "$with_openssl" = "no"; then
	AC_CHECK_FUNC(gnutls_session_set_premaster,
		 [have_gnutls_dtls=yes], [have_gnutls_dtls=no])
    else
	have_gnutls_dtls=no
    fi
    if test "$have_gnutls_dtls" = "yes"; then
	if test "$with_openssl" = "" || test "$with_openssl" = "no"; then
	    # They either said no OpenSSL or didn't specify, and GnuTLS can
	    # do DTLS, so just use GnuTLS.
            AC_DEFINE(HAVE_GNUTLS_SESSION_SET_PREMASTER, 1)    
	    ssl_library=gnutls
	    with_openssl=no
	else
	    # They specifically asked for OpenSSL, so use it for DTLS even
	    # though GnuTLS could manage.
	    ssl_library=both
	fi
    else
	if test "$with_openssl" = "no"; then
	    # GnuTLS doesn't have DTLS, but they don't want OpenSSL. So build
	    # without DTLS support at all.
	    ssl_library=gnutls
	else
	    # GnuTLS doesn't have DTLS so use OpenSSL for it, but GnuTLS for
	    # the TCP connection (and thus in the library).
	    ssl_library=both
	fi
    fi
    AC_CHECK_FUNC(gnutls_pkcs11_add_provider,
		 [PKG_CHECK_MODULES(P11KIT, p11-kit-1, [AC_DEFINE(HAVE_P11KIT)
    			      		  AC_SUBST(P11KIT_PC, p11-kit-1)], [:])], [])
    LIBS="$oldlibs -ltspi"
    AC_MSG_CHECKING([for tss library])
    AC_LINK_IFELSE([AC_LANG_PROGRAM([
		   #include <trousers/tss.h>
		   #include <trousers/trousers.h>],[
		   int err = Tspi_Context_Create((void *)0);
		   Trspi_Error_String(err);])],
		  [AC_MSG_RESULT(yes)
		   AC_SUBST([TSS_LIBS], [-ltspi])
		   AC_SUBST([TSS_CFLAGS], [])
		   AC_DEFINE(HAVE_TROUSERS, 1)],
		  [AC_MSG_RESULT(no)])
    LIBS="$oldlibs"
elif test "$with_gnutls" != "" && test "$with_gnutls" != "no"; then
    AC_MSG_ERROR([Values other than 'yes' or 'no' for --with-gnutls are not supported])
fi
if test "$with_openssl" = "yes" || test "$with_openssl" = "" || test "$ssl_library" = "both"; then
    PKG_CHECK_MODULES(OPENSSL, openssl, [],
	[oldLIBS="$LIBS"
	 LIBS="$LIBS -lssl -lcrypto"
	 AC_MSG_CHECKING([for OpenSSL without pkg-config])
	 AC_LINK_IFELSE([AC_LANG_PROGRAM([
			        #include <openssl/ssl.h>
				#include <openssl/err.h>],[
				SSL_library_init();
				ERR_clear_error();
				SSL_load_error_strings();
				OpenSSL_add_all_algorithms();])],
			[AC_MSG_RESULT(yes)
			 AC_SUBST([OPENSSL_LIBS], ["-lssl -lcrypto"])
			 AC_SUBST([OPENSSL_CFLAGS], [])],
			[AC_MSG_RESULT(no)
			 if test "$ssl_library" = "both"; then
			     ssl_library="gnutls";
			 else
			     AC_ERROR([Could not build against OpenSSL]);
			 fi])
	 LIBS="$oldLIBS"])
    if test "$ssl_library" != "both" && test "$ssl_library" != "gnutls"; then
        ssl_library=openssl
    fi
elif test "$with_openssl" != "no" ; then
    OPENSSL_CFLAGS="-I${with_openssl}/include"
    OPENSSL_LIBS="${with_openssl}/libssl.a ${with_openssl}/libcrypto.a -ldl -lz"
    AC_SUBST(OPENSSL_CFLAGS)
    AC_SUBST(OPENSSL_LIBS)
    enable_static=yes
    enable_shared=no
    AC_DEFINE(DTLS_OPENSSL, 1)
    if test "$ssl_library" != "both"; then
        ssl_library=openssl
    fi
fi

case "$ssl_library" in
    gnutls)
	AC_DEFINE(OPENCONNECT_GNUTLS, 1)
	AC_DEFINE(DTLS_GNUTLS, 1)
	AC_SUBST(SSL_LIBRARY, [gnutls])
	AC_SUBST(SSL_LIBS, ['$(GNUTLS_LIBS)'])
	AC_SUBST(SSL_CFLAGS, ['$(GNUTLS_CFLAGS)'])
	;;
    openssl)
	AC_DEFINE(OPENCONNECT_OPENSSL, 1)
	AC_DEFINE(DTLS_OPENSSL, 1)
	AC_SUBST(SSL_LIBRARY, [openssl])
	AC_SUBST(SSL_LIBS, ['$(OPENSSL_LIBS)'])
	AC_SUBST(SSL_CFLAGS, ['$(OPENSSL_CFLAGS)'])
	AC_SUBST(SYMVER_PRINT_ERR, ["openconnect_print_err_cb;"])
	;;
    both)
	# GnuTLS for TCP, OpenSSL for DTLS
	AC_DEFINE(OPENCONNECT_GNUTLS, 1)
	AC_DEFINE(DTLS_OPENSSL, 1)
	AC_SUBST(SSL_LIBRARY, [gnutls])
	AC_SUBST(SSL_LIBS, ['$(GNUTLS_LIBS)'])
	AC_SUBST(SSL_CFLAGS, ['$(GNUTLS_CFLAGS)'])
	AC_SUBST(DTLS_SSL_LIBS, ['$(OPENSSL_LIBS)'])
	AC_SUBST(DTLS_SSL_CFLAGS, ['$(OPENSSL_CFLAGS)'])
	AC_SUBST(SYMVER_PRINT_ERR, ["openconnect_print_err_cb;"])
	;;
    *)
	AC_MSG_ERROR([Neither OpenSSL nor GnuTLS selected for SSL.])
	;;
esac
AM_CONDITIONAL(OPENCONNECT_GNUTLS,  [ test "$ssl_library" != "openssl" ])
AM_CONDITIONAL(OPENCONNECT_OPENSSL, [ test "$ssl_library" = "openssl" ])

# Needs to happen after we default to static/shared libraries based on OpenSSL
AC_PROG_LIBTOOL

# Ick. This seems like it's likely to be very fragile, but I can't see a better
# way. I shall console myself with the observation that the failure mode isn't
# particularly horrible — you just don't get symbol versioning if it fails.
symvers=no
if test "$enable_shared" = "yes" ; then
   AC_MSG_CHECKING([if library symbol versioning is available]);
   echo 'FOO { global: foo; local: *; };' > conftest.map
   echo 'int foo = 0;' > conftest.$ac_ext
   if AC_TRY_EVAL(ac_compile); then
      soname=conftest
      libobjs=conftest.$ac_objext
      if AC_TRY_EVAL(archive_cmds ${wl}--version-script ${wl}conftest.map); then
         AC_SUBST(VERSION_SCRIPT_ARG, [--version-script])
         symvers="yes (with --version-script)"
      elif AC_TRY_EVAL(archive_cmds ${wl}-M ${wl}conftest.map); then
         AC_SUBST(VERSION_SCRIPT_ARG, [-M])
         symvers="yes (with -M)"
      fi
   fi
   AC_MSG_RESULT(${symvers})
fi
AM_CONDITIONAL(HAVE_SYMBOL_VERSIONING, [test "${symvers}" != "no"])

PKG_CHECK_MODULES(LIBXML2, libxml-2.0)

PKG_CHECK_MODULES(ZLIB, zlib, [AC_SUBST(ZLIB_PC, [zlib])],
		  [oldLIBS="$LIBS"
		  LIBS="$LIBS -lz" 
		  AC_MSG_CHECKING([for zlib without pkg-config])
		  AC_LINK_IFELSE([AC_LANG_PROGRAM([
		   #include <zlib.h>],[
		   z_stream zs;
		   deflateInit2(&zs, Z_DEFAULT_COMPRESSION, Z_DEFLATED,
		   		-12, 9, Z_DEFAULT_STRATEGY);])],
		  [AC_MSG_RESULT(yes)
		   AC_SUBST([ZLIB_LIBS], [-lz])
		   AC_SUBST([ZLIB_CFLAGS], [])],
  		  [AC_MSG_RESULT(no)
		   AC_ERROR([Could not build against zlib])])
		  LIBS="$oldLIBS"])

PKG_CHECK_MODULES(LIBPROXY, libproxy-1.0,
		[AC_SUBST(LIBPROXY_PC, libproxy-1.0)
		 AC_DEFINE([LIBPROXY_HDR], ["proxy.h"])
		 libproxy_pkg=yes],
		 libproxy_pkg=no)
dnl Libproxy *can* exist without a .pc file, and its header may be called
dnl libproxy.h in that case.
if (test "$libproxy_pkg" = "no"); then
   AC_MSG_CHECKING([for libproxy])
   oldLIBS="$LIBS"
   LIBS="$LIBS -lproxy"
   AC_LINK_IFELSE([AC_LANG_PROGRAM([#include <libproxy.h>],
			   [(void)px_proxy_factory_new();])],
	  [AC_MSG_RESULT(yes (with libproxy.h))
	   AC_DEFINE([LIBPROXY_HDR], ["libproxy.h"])
	   AC_SUBST([LIBPROXY_LIBS], [-lproxy])],
	  [AC_LINK_IFELSE([AC_LANG_PROGRAM([#include <proxy.h>],
				  [(void)px_proxy_factory_new();])],
		  [AC_MSG_RESULT(yes (with proxy.h))
		   AC_DEFINE([LIBPROXY_HDR], ["proxy.h"])
		   AC_SUBST([LIBPROXY_LIBS], [-lproxy])],
		   [AC_MSG_RESULT(no)])])
   LIBS="$oldLIBS"
fi

AC_CHECK_HEADER([if_tun.h],
    [AC_DEFINE([IF_TUN_HDR], ["if_tun.h"])],
    [AC_CHECK_HEADER([linux/if_tun.h],
        [AC_DEFINE([IF_TUN_HDR], ["linux/if_tun.h"])],
        [AC_CHECK_HEADER([net/if_tun.h],
            [AC_DEFINE([IF_TUN_HDR], ["net/if_tun.h"])],
            [AC_CHECK_HEADER([net/tun/if_tun.h],
                [AC_DEFINE([IF_TUN_HDR], ["net/tun/if_tun.h"])])])])])

if test "$ssl_library" = "openssl" || test "$ssl_library" = "both"; then
    oldLIBS="$LIBS"
    LIBS="$LIBS $OPENSSL_LIBS"

    if test "$ssl_library" = "openssl"; then
	AC_MSG_CHECKING([for ENGINE_by_id() in OpenSSL])
	AC_LINK_IFELSE([AC_LANG_PROGRAM([#include <openssl/engine.h>],
				        [ENGINE_by_id("foo");])],
		       [AC_MSG_RESULT(yes)
			AC_DEFINE(HAVE_ENGINE, [1], [OpenSSL has ENGINE support])],
		       [AC_MSG_RESULT(no)
			AC_MSG_NOTICE([Building without OpenSSL TPM ENGINE support])])
    fi

    AC_MSG_CHECKING([for dtls1_stop_timer() in OpenSSL])
    AC_LINK_IFELSE([AC_LANG_PROGRAM([#include <openssl/ssl.h>
				     #include <stdlib.h>
				     extern void dtls1_stop_timer(SSL *);],
				    [dtls1_stop_timer(NULL);])],
		   [AC_MSG_RESULT(yes)
		    AC_DEFINE(HAVE_DTLS1_STOP_TIMER, [1], [OpenSSL has dtls1_stop_timer() function])],
		   [AC_MSG_RESULT(no)])
    LIBS="$oldLIBS"
fi

AC_PATH_PROG(PYTHON, [python], [], $PATH:/bin:/usr/bin)
if (test -n "${ac_cv_path_PYTHON}"); then
   AC_SUBST(PYTHON, ${ac_cv_path_PYTHON})
else
   AC_MSG_NOTICE([Python not found; not building HTML pages])
fi
AM_CONDITIONAL(BUILD_WWW, [test -n "${ac_cv_path_PYTHON}"])

AC_SUBST([CONFIG_STATUS_DEPENDENCIES], ['$(top_srcdir)/po/LINGUAS $(top_srcdir)/openconnect.h ${top_srcdir}/libopenconnect.map.in'])
RAWLINGUAS=`sed -e "/^#/d" -e "s/#.*//" "${srcdir}/po/LINGUAS"`
# Remove newlines
LINGUAS=`echo $RAWLINGUAS`
AC_SUBST(LINGUAS)

APIMAJOR="`sed -n 's/^#define OPENCONNECT_API_VERSION_MAJOR \(.*\)/\1/p' ${srcdir}/openconnect.h`"
APIMINOR="`sed -n 's/^#define OPENCONNECT_API_VERSION_MINOR \(.*\)/\1/p' ${srcdir}/openconnect.h`"
AC_SUBST(APIMAJOR)
AC_SUBST(APIMINOR)

# We want version.c to depend on the files that would affect the
# output of version.sh. But we cannot assume that they'll exist,
# and we cannot use $(wildcard) in a non-GNU makefile. So we just
# depend on the files which happen to exist at configure time.
GITVERSIONDEPS=
for a in ${srcdir}/.git/index ${srcdir}/.git/packed-refs \
         ${srcdir}/.git/refs/tags ${srcdir}/.git/HEAD; do
    if test -r $a ; then
       GITVERSIONDEPS="$GITVERSIONDEPS $a"
    fi
done
AC_SUBST(GITVERSIONDEPS)

AC_OUTPUT(Makefile openconnect.pc po/Makefile www/Makefile libopenconnect.map \
	  openconnect.8 www/styles/Makefile www/inc/Makefile www/images/Makefile)
