I'll note down the issues I met during building and running SPEC2017 in RISC-V Ubuntu20.04.4 image with qemu-system-riscv64 virtual machine on this page.

# Useful sites

- https://github.com/donggyukim/Speckle-2017
- https://zhuanlan.zhihu.com/p/425497845


# SPEC2017 system requirement

https://www.spec.org/cpu2017/Docs/system-requirements.html#disk

Disk: 10 GB minimum but only run fully with 250 GB

Memory: 16 GB


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
find ```#if _GNU_GLOB_INTERFACE_VERSION == GLOB_INTERFACE_VERSION``` <br>
change it to ```#if _GNU_GLOB_INTERFACE_VERSION >=GLOB_INTERFACE_VERSION```<br>
then find ```#if !defined __alloca && !defined GNU_LIBRARY``` <br>
change it to ```#if !defined __alloca && defined GNU_LIBRARY```

(solution from [RyoTTa](https://ryotta-205.tistory.com/48))