I'll note down the issues I met during building and running SPEC2017 in RISC-V Ubuntu20.04.4 image with qemu-system-riscv64 virtual machine on this page.

# Useful sites

- https://github.com/donggyukim/Speckle-2017
- https://zhuanlan.zhihu.com/p/425497845
- https://gitee.com/lvxiaoqian/memo/blob/master/%E5%9C%A8unmatched%20Ubuntu21.04%E4%B8%8A%E8%B7%91cpu2017.md

# SPEC2017 system requirement

https://www.spec.org/cpu2017/Docs/system-requirements.html#disk

Disk: 10 GB minimum but recommend 250 GB

Memory: 16 GB

# Offical RISC-V config

It can be obtained here: https://www.spec.org/cpu2017/Docs/Example-gcc-linux-riscv.cfg

# Random Tips

Make sure to use all threads in the system to build the tools and the benchmarks, i.e. setting up "MAKEFLAGS" and "build_ncpus". The amount of time it took me to build the tools and benchmarks for SPEC2017 was significantly more than for SPEC2006. 

In 2022, SPEC releases version 1.1.9 for SPEC2017 which includes a tool for RISC-V 64-bit Linux systems ([link](https://www.spec.org/cpu2017/Docs/changes.html#v119)). Since I can not run `runcpu` command before installing SPEC2017, I ran the `runcpu --update` command on my x86-64 system and obtained the tool `linux-riscv64`. However, when I used it with the `install.sh` on my RISC-V disk, it couldn't detect the tool. If one can manage to use this tool to install SPEC2017, I think it might be better to do so than going though the time to build the tools. 

If the GCC version on the RISC-V machine is lower than 10, then the line `%define GCCge10 ` on the RISC-V config script must be removed.

## update in 2023

I was able to update SPEC2017 in x86 host and use the RISCV tool package to install and build SPEC2017 in RISCV enviornment.
It's much more convinence than going through the tool building process.

# ISSUE

## During buildtool

note: the tools' source files might be under `install_achieve/`.

### Replace outdated config files

Replace the `config.sub` and `config.guess` files in 
```
./specinvoke/
./specsum/build-aux/
./tar-1.28/build-aux/
./make-4.2.1/config/
./rxp-1.5.0/
./expat-2.1.0/conftools/
./xz-5.2.2/build-aux/
```

The files are from https://ftp.gnu.org/

config.guess: http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.guess;hb=HEAD

config.sub: http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD

(solution from [冬天已往](https://zhuanlan.zhihu.com/p/425497845))


### /usr/bin/ld: glob.o:glob.c:(.text+0x12a4): more undefined references to `__alloca' follow

```
/usr/bin/ld: glob.o: in function `.L0 ':
glob.c:(.text+0x4ac): undefined reference to `__alloca'
/usr/bin/ld: glob.o: in function `.L45':
glob.c:(.text+0x662): undefined reference to `__alloca'
/usr/bin/ld: glob.o: in function `.L50':
glob.c:(.text+0x6ca): undefined reference to `__alloca'
/usr/bin/ld: glob.o: in function `.L52':
glob.c:(.text+0x75c): undefined reference to `__alloca'
/usr/bin/ld: glob.o: in function `__glob_pattern_p':
glob.c:(.text+0x1202): undefined reference to `__alloca'
/usr/bin/ld: glob.o:glob.c:(.text+0x12a4): more undefined references to `__alloca' follow
collect2: error: ld returned 1 exit status
+ testordie error building make with build.sh
+ test 1 -ne 0
+ echo !!! error building make with build.sh
!!! error building make with build.sh
+ [ -z  ]
+ kill -TERM 11554
+ exit 1
!!!!! buildtools killed
```

In `[tools/src/make-4.2.1/glob/glob.c]`, <br>
then find ```#if !defined __alloca && !defined GNU_LIBRARY``` <br>
change it to ```#if !defined __alloca && defined GNU_LIBRARY```

(solution from [RyoTTa](https://ryotta-205.tistory.com/48))

### Failed to buld miniperl

```
./miniperl -w -Ilib -Idist/Exporter/lib -MExporter -e '<?>' || sh -c 'echo >&2 Failed to build miniperl.  Please run make minitest; exit 1'
Attempt to free unreferenced scalar: SV 0x557bafe5f570.
Segmentation fault (core dumped)
Failed to build miniperl. Please run make minitest
make: *** [makefile:384: lib/buildcustomize.pl] Error 1
+ testordie error building Perl
+ test 2 -ne 0
+ echo !!! error building Perl
!!! error building Perl
+ [ -z  ]
+ kill -TERM 2711317
+ exit 1
!!!!! buildtools killed
```

This might caused by a bug in build perl with gcc10.
In `[tools/src/perl-5.24.0/Configure]` and `[tools/src/perl-5.24.0/cflags.SH]`, find all `case "$gccversion" in` and change the `1*` to `1.*` if it is finding `1*`.

(solution from [冬天已往](https://zhuanlan.zhihu.com/p/425497845))


### Can't locate test_driver.pl in @INC 

```
cd tests && perl ./run_make_tests.pl -srcdir /home/ubuntu/riscv_spec2017/tools/src/make-4.2.1 -make ../make 
Can't locate test_driver.pl in @INC (@INC contains: /etc/perl /usr/local/lib/riscv64-linux-gnu/perl/5.30.0 /usr/local/share/perl/5.30.0 /usr/lib/riscv64-linux-gnu/perl5/5.30 /usr/share/perl5 /usr/lib/riscv64-linux-gnu/perl/5.30 /usr/share/perl/5.30 /usr/local/lib/site_perl /usr/lib/riscv64-linux-gnu/perl-base) at ./run_make_tests.pl line 61.
make[2]: *** [Makefile:1282: check-regression] Error 2
make[2]: Leaving directory '/home/ubuntu/riscv_spec2017/tools/src/make-4.2.1'
make[1]: *** [Makefile:1089: check-am] Error 2
make[1]: Leaving directory '/home/ubuntu/riscv_spec2017/tools/src/make-4.2.1'
make: *** [Makefile:798: check-recursive] Error 1
+ testordie error testing make
+ test 2 -ne 0
+ echo !!! error testing make
!!! error testing make
+ [ -z  ]
+ kill -TERM 16033
+ exit 1
!!!!! buildtools killed
```

In `[tools/src/make-4.2.1/tests/run_make_tests.pl]`, </b>
find `require "test_driver.pl"` and add 
```
use FindBin;
use lib "$FindBin::Bin";
```
above it. 

(solution from https://lists.gnu.org/archive/html/bug-make/2017-04/msg00015.html)


### tools/src/make-4.2.1/tests/scripts/functions/wildcard test failed

~~I edited the wildcard script to make it `return -1` before running any tests in the test suit.~~

This was caused by changing <br>
```#if _GNU_GLOB_INTERFACE_VERSION == GLOB_INTERFACE_VERSION``` <br>
to ```#if _GNU_GLOB_INTERFACE_VERSION >=GLOB_INTERFACE_VERSION```<br>
in `[tools/src/make-4.2.1/glob/glob.c]`.
The two failed tests in this test suits indicate the program is having SEG FAULT 11.



### SPECSUM test error

```
make[4]: Leaving directory '/home/ubuntu/riscv_spec2017/tools/src/specsum/tests'
/home/ubuntu/riscv_spec2017/tools/output/bin/make  check-TESTS
make[4]: Entering directory '/home/ubuntu/riscv_spec2017/tools/src/specsum/tests'
PASS: test-alloca-opt
PASS: test-binary-io.sh
PASS: test-c-ctype
PASS: test-c-strcase.sh
PASS: test-close
PASS: test-md5
PASS: test-sha1
PASS: test-ctype
PASS: test-dup2
PASS: test-environ
PASS: test-errno
PASS: test-fadvise
PASS: test-fcntl-h
PASS: test-fdopen
PASS: test-fgetc
PASS: test-fpending.sh
PASS: test-fputc
PASS: test-fread
PASS: test-freopen-safer
PASS: test-freopen
PASS: test-fstat
PASS: test-fwrite
PASS: test-getcwd-lgpl
PASS: test-getdelim
PASS: test-getdtablesize
PASS: test-getline
PASS: test-getopt
PASS: test-gettimeofday
PASS: test-ignore-value
PASS: test-intprops
PASS: test-inttypes
PASS: test-langinfo
PASS: test-locale
PASS: test-localename
Starting test_lock .../bin/bash: line 5: 50334 Aborted                 (core dumped) EXEEXT='' srcdir='.' LOCALE_FR='none' LOCALE_TR_UTF8='none' LOCALE_FR='none' LOCALE_FR_UTF8='none' LOCALE_JA='none' LOCALE_ZH_CN='none' MAKE='/home/ubuntu/riscv_spec2017/tools/output/bin/make' ${dir}$tst
FAIL: test-lock
PASS: test-lstat
PASS: test-malloca
PASS: test-memchr
PASS: test-open
PASS: test-pathmax
PASS: test-quotearg-simple
PASS: test-setenv
Skipping test: no locale for testing is installed
SKIP: test-setlocale1.sh
PASS: test-setlocale2.sh
PASS: md5sum-bsd.sh
PASS: md5sum-parallel.sh
PASS: test-specmd5sum.sh
PASS: test-specsha256sum.sh
PASS: test-specsha512sum.sh
PASS: test-stat
PASS: test-stdalign
PASS: test-stdbool
PASS: test-stddef
PASS: test-stdint
PASS: test-stdio
PASS: test-stdlib
PASS: test-strerror
PASS: test-string
PASS: test-strtoull
PASS: test-symlink
PASS: test-sys_stat
PASS: test-sys_time
PASS: test-sys_types
PASS: test-init.sh
PASS: test-thread_self
glthread_create failed
FAIL: test-thread_create
PASS: test-time
PASS: test-u64
PASS: test-unistd
PASS: test-unsetenv
PASS: test-verify
PASS: test-verify.sh
PASS: test-version-etc.sh
PASS: test-wchar
PASS: test-wctype-h
PASS: test-xalloc-die.sh
=================================
2 of 75 tests failed
(1 test was not run)
Please report to support@spec.org
=================================
make[4]: *** [Makefile:2333: check-TESTS] Error 1
make[4]: Leaving directory '/home/ubuntu/riscv_spec2017/tools/src/specsum/tests'
make[3]: *** [Makefile:2484: check-am] Error 2
make[3]: Leaving directory '/home/ubuntu/riscv_spec2017/tools/src/specsum/tests'
make[2]: *** [Makefile:2204: check-recursive] Error 1
make[2]: Leaving directory '/home/ubuntu/riscv_spec2017/tools/src/specsum/tests'
make[1]: *** [Makefile:2486: check] Error 2
make[1]: Leaving directory '/home/ubuntu/riscv_spec2017/tools/src/specsum/tests'
make: *** [Makefile:1016: check-recursive] Error 1
+ testordie error testing specsum
+ test 2 -ne 0
+ echo !!! error testing specsum
!!! error testing specsum
+ [ -z  ]
+ kill -TERM 709
+ exit 1
!!!!! buildtools killed
```

In `[tools/src/specsum/configure]`, </b>
find `USE_POSIX_THREADS` and change it from </b>
`#define USE_POSIX_THREADS 1` to </b>
`#define USE_POSIX_THREADS 0`.

(solution from [吕晓倩](https://gitee.com/lvxiaoqian/memo/blob/master/%E5%9C%A8unmatched%20Ubuntu21.04%E4%B8%8A%E8%B7%91cpu2017.md))

### Perl test fail

```
cpan/Tie-RefHash/t/refhash .................................... ok
cpan/Tie-RefHash/t/storable ................................... ok
cpan/Tie-RefHash/t/threaded ................................... skipped
cpan/Time-Local/t/Local ....................................... #   Failed test 'timelocal year for 1970 1 2 0 0 0'
#   at t/Local.t line 104.
#          got: '170'
#     expected: '70'
#   Failed test 'timegm year for 1970 1 2 0 0 0'
#   at t/Local.t line 120.
#          got: '170'
#     expected: '70'
# Looks like you failed 2 tests of 187.
FAILED at test 6
cpan/Time-Piece/t/01base ...................................... ok
cpan/Time-Piece/t/02core_dst .................................. ok
cpan/Time-Piece/t/02core ...................................... ok
## 
    here is a long log of test results
##
lib/vmsish .................................................... ok
lib/warnings .................................................. ok
Failed 1 test out of 2234, 99.96% okay.
        ../cpan/Time-Local/t/Local.t
### Since not all tests were successful, you may want to run some of
### them individually and examine any diagnostic messages they produce.
### See the INSTALL document's section on "make test".
### You have a good chance to get more information by running
###   ./perl harness
### in the 't' directory since most (>=80%) of the tests succeeded.
### You may have to set your dynamic library search path,
### LD_LIBRARY_PATH, to point to the build directory:
###   setenv LD_LIBRARY_PATH `pwd`:$LD_LIBRARY_PATH; cd t; ./perl harness
###   LD_LIBRARY_PATH=`pwd`:$LD_LIBRARY_PATH; export LD_LIBRARY_PATH; cd t; ./perl harness
###   export LD_LIBRARY_PATH=`pwd`:$LD_LIBRARY_PATH; cd t; ./perl harness
### for csh-style shells, like tcsh; or for traditional/modern
### Bourne-style shells, like bash, ksh, and zsh, respectively.
Elapsed: 3880 sec
u=63.97  s=33.65  cu=3037.97  cs=586.69  scripts=2234  tests=849751
make: *** [makefile:812: test] Error 1
+ [ 2 -ne 0 ]
+ set +x


Hey!  Some of the Perl tests failed!
If you think this is okay, enter y now:
```

After pressing y, there are more error messages:

```
================================================================
=== Building TimeDate-2.30
================================================================
+ /home/ubuntu/riscv_spec2017/tools/output/bin/perl Makefile.PL -n
Checking if your kit is complete...
Looks good
Generating a Unix-style Makefile
Writing Makefile for Date::Parse
Writing MYMETA.yml and MYMETA.json
+ testordie error making Makefile for TimeDate-2.30
+ test 0 -ne 0
+ MAKEFLAGS= /home/ubuntu/riscv_spec2017/tools/output/bin/make install
cp lib/Date/Language/Gedeo.pm blib/lib/Date/Language/Gedeo.pm
cp lib/Date/Language/Russian_koi8r.pm blib/lib/Date/Language/Russian_koi8r.pm
cp lib/Date/Language/Russian_cp1251.pm blib/lib/Date/Language/Russian_cp1251.pm
cp lib/Date/Language/Swedish.pm blib/lib/Date/Language/Swedish.pm
cp lib/Date/Language/French.pm blib/lib/Date/Language/French.pm
cp lib/Time/Zone.pm blib/lib/Time/Zone.pm
cp lib/Date/Parse.pm blib/lib/Date/Parse.pm
cp lib/Date/Language/Bulgarian.pm blib/lib/Date/Language/Bulgarian.pm
cp lib/Date/Language/TigrinyaEthiopian.pm blib/lib/Date/Language/TigrinyaEthiopian.pm
cp lib/Date/Language/Amharic.pm blib/lib/Date/Language/Amharic.pm
cp lib/Date/Language/Dutch.pm blib/lib/Date/Language/Dutch.pm
cp lib/Date/Language/Romanian.pm blib/lib/Date/Language/Romanian.pm
cp lib/Date/Language/TigrinyaEritrean.pm blib/lib/Date/Language/TigrinyaEritrean.pm
cp lib/Date/Language/Chinese.pm blib/lib/Date/Language/Chinese.pm
cp lib/Date/Language/English.pm blib/lib/Date/Language/English.pm
cp lib/Date/Language/Greek.pm blib/lib/Date/Language/Greek.pm
cp lib/Date/Language/Italian.pm blib/lib/Date/Language/Italian.pm
cp lib/Date/Language/Brazilian.pm blib/lib/Date/Language/Brazilian.pm
cp lib/Date/Language/Tigrinya.pm blib/lib/Date/Language/Tigrinya.pm
cp lib/Date/Language/Afar.pm blib/lib/Date/Language/Afar.pm
cp lib/Date/Language.pm blib/lib/Date/Language.pm
cp lib/Date/Language/Danish.pm blib/lib/Date/Language/Danish.pm
cp lib/Date/Language/Hungarian.pm blib/lib/Date/Language/Hungarian.pm
cp lib/Date/Language/Sidama.pm blib/lib/Date/Language/Sidama.pm
cp lib/Date/Language/Spanish.pm blib/lib/Date/Language/Spanish.pm
cp lib/Date/Language/Oromo.pm blib/lib/Date/Language/Oromo.pm
cp lib/Date/Language/Somali.pm blib/lib/Date/Language/Somali.pm
cp lib/Date/Language/Turkish.pm blib/lib/Date/Language/Turkish.pm
cp lib/Date/Language/Russian.pm blib/lib/Date/Language/Russian.pm
cp lib/Date/Language/Czech.pm blib/lib/Date/Language/Czech.pm
cp lib/Date/Language/Austrian.pm blib/lib/Date/Language/Austrian.pm
cp lib/Date/Format.pm blib/lib/Date/Format.pm
cp lib/Date/Language/Icelandic.pm blib/lib/Date/Language/Icelandic.pm
cp lib/Date/Language/Norwegian.pm blib/lib/Date/Language/Norwegian.pm
cp lib/Date/Language/Finnish.pm blib/lib/Date/Language/Finnish.pm
cp lib/Date/Language/Chinese_GB.pm blib/lib/Date/Language/Chinese_GB.pm
cp lib/Date/Language/German.pm blib/lib/Date/Language/German.pm
Manifying 6 pod documents
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Format.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Parse.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Dutch.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Oromo.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Russian.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Amharic.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Brazilian.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Spanish.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Gedeo.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Sidama.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Somali.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Danish.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Greek.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Russian_koi8r.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Norwegian.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Turkish.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Chinese.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/French.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/TigrinyaEthiopian.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Russian_cp1251.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Hungarian.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Czech.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Finnish.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Chinese_GB.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Italian.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Swedish.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/TigrinyaEritrean.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Austrian.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Tigrinya.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Afar.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Bulgarian.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Icelandic.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/English.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/German.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Date/Language/Romanian.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/site_perl/5.24.0/Time/Zone.pm
Installing /home/ubuntu/riscv_spec2017/tools/output/man/man3/Date::Parse.3
Installing /home/ubuntu/riscv_spec2017/tools/output/man/man3/Date::Language::Hungarian.3
Installing /home/ubuntu/riscv_spec2017/tools/output/man/man3/Time::Zone.3
Installing /home/ubuntu/riscv_spec2017/tools/output/man/man3/Date::Format.3
Installing /home/ubuntu/riscv_spec2017/tools/output/man/man3/Date::Language.3
Installing /home/ubuntu/riscv_spec2017/tools/output/man/man3/Date::Language::Bulgarian.3
Appending installation info to /home/ubuntu/riscv_spec2017/tools/output/lib/perl5/5.24.0/riscv64-linux/perllocal.pod
+ testordie error building/installing TimeDate-2.30
+ test 0 -ne 0
+ [ -f TimeDate-2.30/spec_do_no_tests ]
+ /home/ubuntu/riscv_spec2017/tools/output/bin/make test
PERL_DL_NONLAZY=1 "/home/ubuntu/riscv_spec2017/tools/output/bin/perl" "-MExtUtils::Command::MM" "-MTest::Harness" "-e" "undef *Test::Harness::Switches; test_harness(0, 'blib/lib', 'blib/arch')" t/*.t
t/cpanrt.t ... ok   
t/date.t ..... Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
t/date.t ..... 1/148 Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
Redundant argument in printf at t/date.t line 180.
t/date.t ..... ok       
t/format.t ... ok       
t/getdate.t .. Failed 146/146 subtests 
t/lang.t ..... ok   

Test Summary Report
-------------------
t/getdate.t (Wstat: 0 Tests: 0 Failed: 0)
  Parse errors: Bad plan.  You planned 146 tests but ran 0.
Files=5, Tests=362,  3 wallclock secs ( 0.73 usr  0.18 sys +  2.31 cusr  0.63 csys =  3.85 CPU)
Result: FAIL
Failed 1/5 test programs. 0/362 subtests failed.
make: *** [Makefile:965: test_dynamic] Error 255
+ testordie error running TimeDate-2.30 test suite
+ test 2 -ne 0
+ echo !!! error running TimeDate-2.30 test suite
!!! error running TimeDate-2.30 test suite
+ [ -z  ]
+ kill -TERM 58483
+ exit 1
!!!!! buildtools killed
```

This is a bug. The error will happen when the system date is after 2020.

To fix this bug, in `[TimeDate-2.30/t/getdate.t]`, </b>
find `my $offset = Time::Local::timegm(0,0,0,1,0,70);` and change it to </b>
`my $offset = Time::Local::timegm(0,0,0,1,0,1970);`.

(solution from [吕晓倩](https://gitee.com/lvxiaoqian/memo/blob/master/%E5%9C%A8unmatched%20Ubuntu21.04%E4%B8%8A%E8%B7%91cpu2017.md))
