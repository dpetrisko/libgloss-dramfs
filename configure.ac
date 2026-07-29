AC_INIT([libgloss-dramfs], [0.1])

AC_PROG_CC
AC_CHECK_TOOL([AR], [ar])
AC_CHECK_TOOL([LD], [ld])
AC_CHECK_TOOL([SIZE], [size])

AC_CANONICAL_HOST
AC_SUBST([host])
AC_SUBST([build])
AC_SUBST([target])

AC_SUBST([AR])
AC_SUBST([LD])
AC_SUBST([SIZE])

AC_ARG_VAR([CC_FOR_BUILD], [Build system C compiler])
AC_CHECK_PROGS([CC_FOR_BUILD], [gcc cc])
AC_SUBST([CC_FOR_BUILD])

AC_COMPILE_IFELSE([AC_LANG_PROGRAM([], [
#ifndef __riscv
#error "A RISC-V compiler is required"
#endif
])], [],
[AC_MSG_FAILURE([The target architecture of the C compiler does not appear to be RISC-V.
If cross-compiling, specify a target triplet such as '--host=riscv64-unknown-elf'.])])

AC_ARG_WITH([multilib-generator],
    [AS_HELP_STRING([--with-multilib-generator=STRING],
        [build libraries using riscv-gnu-toolchain generator string format (implies --enable-multilib)])],
    [],
    [with_multilib_generator=no])

AC_MSG_CHECKING([whether multilib is enabled])
AC_ARG_ENABLE([multilib],
    [AS_HELP_STRING([--enable-multilib],
        [build libraries for multiple march/mabi combinations @<:@--disable-multilib@:>@])],
    [],
    [enable_multilib=default])

AS_IF([test "x$with_multilib_generator" != xno], [
    # Process --with-multilib-generator using newlib's awk parsing snippet.
    # Adjusted for M4/shell syntax and libgloss's expected arch/abi format.
    multilib=`echo "$with_multilib_generator" | tr ';' '\n' | sed '/^$/d' | awk '{split([$]0,a,"-"); printf "%s/%s", (NR==1?a[[1]]:" "a[[1]]),a[[2]]}'`
    
    # Validate the resulting multilib string
    for mlib in $multilib ; do
        AS_CASE(["$mlib"], [rv32?*/?*|rv64?*/?*], [], [
            AC_MSG_RESULT([unknown])
            AC_MSG_ERROR([illegal march-mabi value: ${mlib} derived from ${with_multilib_generator}])])
    done
], [
    # Fall back to standard --enable-multilib logic
    AS_CASE(["$enable_multilib"],
        [yes], [multilib="rv32i/ilp32 rv32iac/ilp32 rv32im/ilp32 rv32imac/ilp32 rv32imafc/ilp32f rv64imac/lp64 rv64imafdc/lp64d"],
        [no], [multilib=""],
        [default], [multilib="`$CC -print-multi-lib | sed '/.*;@march=\(.*\)@mabi=\(.*\).*/{s//\1\/\2/;H;};$!d;x;s/\n/ /g'`"],
        [
            for mlib in $enable_multilib ; do
                AS_CASE(["$mlib"], [rv32?*/?*|rv64?*/?*], [], [
                    AC_MSG_RESULT([unknown])
                    AC_MSG_ERROR([illegal march-mabi value: ${mlib}])])
            done
            multilib="$enable_multilib"
        ])
])

AS_IF([test "x$multilib" != x],
    [AC_MSG_RESULT([yes ($multilib)])],
    [AC_MSG_RESULT([no])])

AC_SUBST([MULTILIB], ["$multilib"])

AC_CONFIG_FILES([Makefile])
AC_OUTPUT
