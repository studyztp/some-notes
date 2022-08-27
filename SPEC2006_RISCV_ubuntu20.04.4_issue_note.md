I'll note down the issues I met during building and running SPEC2006 in RISC-V Ubuntu20.04.4 image with qemu-system-riscv64 virtual machine on this page. 


# Useful sites

SUPER thankful to all the people who documented their issues and solutions during building and running SPEC2006.

- https://GQBBBB/GQBBBB.github.io/issues/10

- https://www.okqubit.net/runspec.html

- http://pfzuo.github.io/2016/06/12/Compile-and-debug-spec-cpu-2006-in-linux/

- https://ryotta-205.tistory.com/48

- https://sjp38.github.io/post/spec_cpu2006_install/

- https://github.com/ccelio/Speckle

- https://groups.google.com/a/groups.riscv.org/g/sw-dev/c/Pr60YZFQXzY



# Issues

## configure: error: cannot guess build type; you must specify one
```
+ testordie error configuring make
+ test 1 -ne 0
+ echo !!! error configuring make
!!! error configuring make
+ kill -TERM 17710
+ exit 1
!!!!! buildtools killed
```

The solution to this was downloading the latest config.sub and config.guess from https://ftp.gnu.org/
Below are the links to the config files:
- http://git.savannah.gnu.org/gitweb/p=config.git;a=blob_plain;f=config.guess;hb=HEAD
- http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD

(solution from [GQBBBB](https://github.com/GQBBBB/GQBBBB.github.io/issues/10))

## /usr/bin/ld: glob.o:glob.c:(.text+0x126e): more undefined references to `__alloca' follow
```
/usr/bin/ld: glob.o: in function `.L143':
glob.c:(.text+0x11cc): undefined reference to `__alloca'
/usr/bin/ld: glob.o:glob.c:(.text+0x126e): more undefined references to `__alloca' follow
collect2: error: ld returned 1 exit status
+ testordie error building make with build.sh
+ test 1 -ne 0
+ echo !!! error building make with build.sh
!!! error building make with build.sh
+ kill -TERM 71479
+ exit 1
!!!!! buildtools killed
```

The solution to this was in ```[tools/src/make-3.80/glob/glob.c]``` <br>
find ```#if _GNU_GLOB_INTERFACE_VERSION == GLOB_INTERFACE_VERSION``` <br>
change it to ```#if _GNU_GLOB_INTERFACE_VERSION >=GLOB_INTERFACE_VERSION```<br>
then find ```#if !defined __alloca && !defined GNU_LIBRARY``` <br>
change it to ```#if !defined __alloca && defined GNU_LIBRARY```

(solution from [RyoTTa](https://ryotta-205.tistory.com/48))


## make: *** [md5sum.o] Error 1

```
md5sum.c: In function 'main':
md5sum.c:682:15: warning: format '%X' expects argument of type 'unsigned int', but argument 2 has type 'size_t' {aka 'long unsigned int'} [-Wformat=]
  682 |   printf ("%08X", size);
      |            ~~~^   ~~~~
      |               |   |
      |               |   size_t {aka long unsigned int}
      |               unsigned int
      |            %08lX
make: *** [md5sum.o] Error 1
+ testordie error building specmd5sum
+ test 2 -ne 0
+ echo !!! error building specmd5sum
!!! error building specmd5sum
+ kill -TERM 75459
+ exit 1
!!!!! buildtools killed
```

The solution to this was commenting out ```#include "getline.h"``` in ```[tools/src/specmd5sum/md5sum.c]```

(solution from [Pengfei Zuo](http://pfzuo.github.io/2016/06/12/Compile-and-debug-spec-cpu-2006-in-linux/))


## collect2: error: ld returned 1 exit status
```
/usr/bin/ld: libperl.a(pp_pack.o): in function `.L759':
pp_pack.c:(.text+0x315e): undefined reference to `floor'
collect2: error: ld returned 1 exit status
make: *** [miniperl] Error 1
+ testordie error building perl
+ test 2 -ne 0
+ echo !!! error building perl
!!! error building perl
+ kill -TERM 91442
+ exit 1
!!!!! buildtools killed
```

The solution to this was linking the math library when compiling perl. I tried different ways. In my case, adding a flag before running the build or install script worked the best for me without causing furthur errors. <br>

This is the command I used:
```
PERLFLAGS="-A libs=-lm -A libs=-ldl" ./buildtools
```
(solution from [RyoTTa](https://ryotta-205.tistory.com/48))

Beside linking the math library, you might need to comment out <br>```#include <asm/page.h>``` <br> in ```[tools/src/perl-5.8.7/ext/IPC/SysV/SysV.x]```

(solution from [GQBBBB](https://github.com/GQBBBB/GQBBBB.github.io/issues/10))

## You haven't done a "make depend" yet!
```
make[1]: Entering directory `/home/ubuntu/spec2006/mnt/tools/src/perl-5.8.7/x2p'
You haven't done a "make depend" yet!
make[1]: *** [hash.o] Error 1
make[1]: Leaving directory `/home/ubuntu/spec2006/mnt/tools/src/perl-5.8.7/x2p'
make: *** [translators] Error 2
+ testordie error building perl
+ test 2 -ne 0
+ echo !!! error building perl
!!! error building perl
+ kill -TERM 166551
+ exit 1
!!!!! buildtools killed
```

The solution to this was running the following command:
```
sudo rm /bin/sh
sudo ln -s /bin/bash /bin/sh
```

(solution from [SeongJae Park](https://sjp38.github.io/post/spec_cpu2006_install/))

## make: *** No rule to make target 'command-line', needed by `miniperlmain.o'.  Stop.

```
+ testordie 'error configuring perl'
+ test 0 -ne 0
+ /home/ubuntu/spec2006/mnt/tools/output/bin/make
make: *** No rule to make target `<command-line>', needed by `miniperlmain.o'.  Stop.
+ testordie 'error building perl'
+ test 2 -ne 0
+ echo '!!! error building perl'
!!! error building perl
+ kill -TERM 217079
+ exit 1
!!!!! buildtools killed
```

The solution to this was finding ```-e '/^#.*<command line>/d'``` <br>
in ```[tools/src/perl-5.87/makedepend.SH]``` <br>
and adding ```-e '/^#.*<command-line>/d' \``` after it.

(solution from [RyoTTa](https://ryotta-205.tistory.com/48))


# Some thoughts

- Every run takes more than half hour with buildtools.sh. With install.sh, every run takes more than one hour.
- The part where making the perl is causing most of the error.
- The community and the documentations are being very helpful. 

# Some stupid mistakes

## Something related to RISC-V Ubuntu 20.04.4 preinstalled image
Make sure using the Hirsute's version of u-boot-qemu <http://old-releases.ubuntu.com/ubuntu/pool/main/u/u-boot/u-boot-qemu_2021.01+dfsg-3ubuntu9_all.deb>. Otherwise, this error will haunt you:
```
Filename 'boot.scr.uimg'.
Load address: 0x84000000
Loading: *
TFTP error: 'Access violation' (2)
```

## Make sure to use the right riscv tool when using [Speckle](https://github.com/ccelio/Speckle)
The riscv.config uses riscv64 newlib tools. Check ```/usr/bin/``` before trying ```./gen_binaries.sh```.
