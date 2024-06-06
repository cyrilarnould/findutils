# MinGW port

The `mingw` branches of this repository are an attempt at porting `findutils` to Microsoft Windows. Most of the patches are based on Eli Zaretskii's [ezwinports](https://sourceforge.net/projects/ezwinports/files/) implementation. It is intended to be built using the [MSYS2](https://www.msys2.org/) toolchain.

## Testsuite

Since the port does not support all of the `findutils` features perfectly and the testsuite is not set up with Windows in mind, some of the `make check` tests will fail. Pretty much all of the tests will fail because of `CRLF` vs `LF` if not using the corresponding `mingw-w64-diffutils` package. A lot of `gnulib-tests` will fail as well, those have not been addressed.

## find Tests

The following `find` tests fail because of `ln` issues (https://github.com/msys2/MSYS2-packages/issues/3077):

```
find/testsuite/find.gnu/follow-basic
find/testsuite/find.gnu/fprintf-samefile
find/testsuite/find.gnu/ilname
find/testsuite/find.gnu/lname
find/testsuite/find.gnu/posix-dflt
find/testsuite/find.gnu/posix-h
find/testsuite/find.gnu/posix-l
find/testsuite/find.gnu/printf-h
find/testsuite/find.gnu/printf-symlink
find/testsuite/find.gnu/samefile-link
find/testsuite/find.gnu/samefile-p-brokenlink
find/testsuite/find.gnu/samefile-symlink
find/testsuite/find.gnu/xtype
find/testsuite/find.posix/sv-bug-19605
find/testsuite/find.posix/sv-bug-19613
find/testsuite/find.posix/sv-bug-25359
```

-------------------------------------------------------------------------------

The following tests fail because chmod doesn't seem to work properly (https://github.com/msys2/MSYS2-packages/issues/2612):

```
find/testsuite/find.posix/perm-X
find/testsuite/find.posix/perm-vanilla
find/testsuite/find.gnu/printf
```

The following test _also_ fails if `MSYS2_ARG_CONV_EXCL=*` isn't set:

```
find/testsuite/find.gnu/perm-slash
```

-------------------------------------------------------------------------------

The following test will fail because `-type l is not supported because symbolic links are not supported on the platform find was compiled on.` The behaviour is the same as in ezwinports:

```
find/testsuite/find.gnu/printf-nonlocal-symlink
```

The following test will fail because `the -d option is deprecated; please use -depth instead, because the latter is a POSIX-compliant feature.` This behaviour is the same for MSYS2's `usr/bin/find` though:

```
find/testsuite/find.gnu/depth-d
```

The following test will fail because `'-name' matches against basenames only, but the given pattern contains a directory separator ('/'), thus the expression will evaluate to false all the time.  Did you mean '-wholename'?` This behaviour is the same for `usr/bin/find` though. It also fails if `MSYS2_ARG_CONV_EXCL=`* isn't set:

```
find/testsuite/find.gnu/name-slash
```

-------------------------------------------------------------------------------

The following test will fail with `sh: No such file or directory` because it tries to execute a shell script which doesn't seem like a use case for the Windows variant:

```
find/testsuite/find.posix/exec-nogaps
```

-------------------------------------------------------------------------------

The following test fails if `MSYS2_ARG_CONV_EXCL=*` isn't set:

```
find/testsuite/find.gnu/execdir-root-only
```

The following test will fail if `export gl_cv_have_weak=no` hasn't been set prior to `./configure`:

```
find/testsuite/find.gnu/iregex1
```

## xargs Tests

The following tests will fail with the same behaviour as ezwinports. The problem is that spawnvp doesn't preserve whitespace if it isn't quoted. I guess this could be considered expected Windows behaviour?

```
xargs/testsuite/xargs.gnu/L2_2-0
xargs/testsuite/xargs.gnu/delim-o
xargs/testsuite/xargs.gnu/idef-0
xargs/testsuite/xargs.gnu/l1_2-0
xargs/testsuite/xargs.gnu/space-0
xargs/testsuite/xargs.gnu/space-t-0
xargs/testsuite/xargs.posix/IARG
xargs/testsuite/xargs.posix/quotes
xargs/testsuite/xargs.posix/rc-123
xargs/testsuite/xargs.sysv/idef
xargs/testsuite/xargs.sysv/iquotes
xargs/testsuite/xargs.sysv/sv-bug-18713
```

-------------------------------------------------------------------------------

The following test fails similarly to the ones above but the behaviour is different across the `mingw` branch, MSYS2's `/usr/bin/xargs` and ezwinports. Seems to be a similar whitespace issue as above though, it uses the weird `^L` to separate the text:

```
xargs/testsuite/xargs.posix/sv-bug-18714
xargs/testsuite/xargs.posix/sv-bug-18714b
```

-------------------------------------------------------------------------------

The following test fails because `true: Invalid argument`. Behaviour is different from both MSYS2's `/usr/bin/xargs` as well as ezwinports. The problem seems to be with the `ARG_MAX` limits which are inherently different in the `mingw` version:

```
xargs/testsuite/xargs.posix/arg_max_64bit_linux_bug
```

The following test fails because `unexpected failure, child process exited abnormally`, however both xargs.out and xargs.err are emtpy, same as when run with MSYS2'S `/usr/bin/xargs`. The problem seems to be with the `ARG_MAX` limits which are inherently different in the mingw version:

```
xargs/testsuite/xargs.posix/arg_max_32bit_linux_bug
```

The following test will fail because the outputs differ, but i can't reproduce this manually in the shell. The command to reproduce the bug relies on calling sh, which is not really a Windows use case I guess:

```
xargs/testsuite/xargs.posix/sv-bug-20273
```

-------------------------------------------------------------------------------

The following test will fail because `kill: usage: kill [-s sigspec | -n signum | -sigspec] pid | jobspec ... or kill -l [sigspec]`, which is the same behaviour as ezwinports.

```
xargs/testsuite/xargs.posix/rc-125
```

The following test will fail because it doesn't result in the following error: `sh: exited with status 255; aborting`, which is the same behaviour as ezwinports.

```
xargs/testsuite/xargs.posix/rc-124
```

The following test will fail because it doesn't result in the following error: `xargs: /: No such file or directory`, which is the same behaviour as ezwinports.

```
xargs/testsuite/xargs.posix/rc-126
```

The following test will fail because it doesn't result in the following error: `xargs: ./missing: No such file or directory`, which is the same behaviour as ezwinports.

```
xargs/testsuite/xargs.posix/rc-127
```

-------------------------------------------------------------------------------

The following test will fail because `warning: options --max-args and --replace/-I/-i are mutually exclusive, ignoring previous --max-args value`, which is the same behaviour as MSYS2's `/usr/bin/xargs`. It will also fail because `sleep: missing operand Try 'sleep --help' for more information.`, which is the same behaviour as ezwinports.

```
xargs/testsuite/xargs.gnu/P3-n1-IARG
```

-------------------------------------------------------------------------------

The following test will fail because `warning: options --max-lines and --max-args/-n are mutually exclusive, ignoring previous --max-lines value`. This behaviour is the same for MSYS2's `usr/bin/xargs` though:

```
xargs/testsuite/xargs.posix/L2-n2
xargs/testsuite/xargs.sysv/l1n4
```


## locate Tests

Pretty much all tests fail because they output `CRLF` instead of `LF`; and the tests use `cmp` rather than `diff` to verify. `cmp` performs a byte by byte comparison whereas the `mingw-w64-diffutils` `diff` command seems to ignore the difference in line endings.

-------------------------------------------------------------------------------

The following test is not only different in line endings but I cannot figure out what the problem is, the outputs look the same:

```
locate/testsuite/locate.gnu/littleendian
```

The following test fails because of missing UTF8 support. Same behaviour as ezwinports:

```
locate/testsuite/locate.gnu/sv-bug-14535
```
