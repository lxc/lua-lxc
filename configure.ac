#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_INIT([lua-lxc], [lxc_version])

# We need pkg-config
PKG_PROG_PKG_CONFIG

AC_CONFIG_SRCDIR([configure.ac])
AC_CONFIG_AUX_DIR([config])
AC_CONFIG_HEADERS([src/config.h])
AM_INIT_AUTOMAKE([-Wall -Werror -Wno-portability subdir-objects])
AC_CANONICAL_HOST
AM_PROG_CC_C_O
AC_GNU_SOURCE

# libtool
LT_INIT
AC_SUBST([LIBTOOL_DEPS])

AC_ARG_ENABLE([werror],
	[AC_HELP_STRING([--disable-werror],
	[do not treat warnings as errors])],
	[], [enable_werror=yes])

# Configuration examples
AC_ARG_ENABLE([examples],
	[AC_HELP_STRING([--enable-examples], [install examples [default=yes]])],
	[], [enable_examples=yes])
AM_CONDITIONAL([ENABLE_EXAMPLES], [test "x$enable_examples" = "xyes"])

# Lua module and scripts
AC_ARG_WITH([lua-pc],
	[AS_HELP_STRING(
		[--with-lua-pc=PKG],
		[Specify pkg-config package name for lua]
	)], [], [with_lua_pc=no])

if test "x$with_lua_pc" != "xno"; then
	# exit with error if not found
	PKG_CHECK_MODULES([LUA], [$with_lua_pc], [LUAPKGCONFIG=$with_lua_pc])
fi

if "x$with_lua_pc" != "xno"; then
	PKG_CHECK_MODULES([LUA], [$with_lua_pc],
		[LUAPKGCONFIG=$with_lua_pc
		enable_lua=yes],
		[enable_lua=no])
fi

PKG_CHECK_MODULES([LUA], [lua], [LUAPKGCONFIG=lua],
	[PKG_CHECK_MODULES([LUA], [lua5.2], [LUAPKGCONFIG=lua5.2],
		[PKG_CHECK_MODULES([LUA], [lua5.1], [LUAPKGCONFIG=lua5.1],
			[AS_IF([test "x$enable_lua" = "xyes"],
				[AC_MSG_ERROR([Lua not found. Please use --with-lua-pc=PKG])],
				[enable_lua=no])]
		)]
	)])

AC_MSG_CHECKING([Lua version])
PKG_CHECK_VAR([LUA_VERSION], [$LUAPKGCONFIG], [V],,
	[PKG_CHECK_VAR([LUA_VERSION], [$LUAPKGCONFIG], [major_version])])
AC_MSG_RESULT([$LUA_VERSION])
AC_SUBST([LUA_LIBDIR], [$libdir/lua/$LUA_VERSION])
AC_SUBST([LUA_SHAREDIR], [$datadir/lua/$LUA_VERSION])

# Not in older autoconf versions
# AS_VAR_COPY(DEST, SOURCE)
# -------------------------
# Set the polymorphic shell variable DEST to the contents of the polymorphic
# shell variable SOURCE.
m4_ifdef([AS_VAR_COPY], [],
[AC_DEFUN([AS_VAR_COPY],
    [AS_LITERAL_IF([$1[]$2], [$1=$$2], [eval $1=\$$2])])
])

dnl PKG_CHECK_VAR was introduced with pkg-config 0.28
m4_ifdef([PKG_CHECK_VAR], [],
[AC_DEFUN([PKG_CHECK_VAR],
    [AC_REQUIRE([PKG_PROG_PKG_CONFIG])dnl
     AC_ARG_VAR([$1], [value of $3 for $2, overriding pkg-config])dnl
     _PKG_CONFIG([$1], [variable="][$3]["], [$2])
     AS_VAR_COPY([$1], [pkg_cv_][$1])
     AS_VAR_IF([$1], [""], [$5], [$4])dnl
    ])# PKG_CHECK_VAR
])

# Expand some useful variables
AS_AC_EXPAND(PREFIX, "$prefix")
AS_AC_EXPAND(LIBDIR, "$libdir")
AS_AC_EXPAND(BINDIR, "$bindir")
AS_AC_EXPAND(SBINDIR, "$sbindir")
AS_AC_EXPAND(LIBEXECDIR, "$libexecdir")
AS_AC_EXPAND(INCLUDEDIR, "$includedir")
AS_AC_EXPAND(LXCPATH, "$with_config_path")

# We need the install path so criu knows where to reference the hook scripts.
AC_DEFINE_UNQUOTED([DATADIR], "$DATADIR", ["Prefix for shared files."])

# Check for some standard binaries
AC_PROG_GCC_TRADITIONAL
AC_PROG_SED

if test "x$GCC" = "xyes"; then
	CFLAGS="$CFLAGS -Wall"
	if test "x$enable_werror" = "xyes"; then
		CFLAGS="$CFLAGS -Werror"
	fi
fi

# Files requiring some variable expansion
AC_CONFIG_FILES([
	Makefile
	src/Makefile
])
AC_CONFIG_COMMANDS([default],[[]],[[]])
AC_OUTPUT

# Configuration overview
cat << EOF

----------------------------
Environment:
 - compiler: $CC
 - with_lua_pc: $with_lua_pc

EOF
