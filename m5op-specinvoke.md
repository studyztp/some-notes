# How to M5OP specinvoke

## 1. get gem5 m5op needed files

Follow the guidance on the [gem5 website](https://www.gem5.org/documentation/general_docs/m5ops/) to build m5 and libm5 libraries.

Copy the gem5/include, gem5/util/m5/src/m5_mmap.h, and gem5/util/m5/build/{TARGET_ISA}/out to the SPEC2006 tools directory.

For the following examples, I stored them under `{SPEC2006 directory}/tools/gem5lib`. Below is their layout:
```
gem5lib
    |___ include
    |       |___ gem5
    |              |___ asm
    |              |    |__generic
    |              |        |__m5ops.h
    |              |____ m5ops.h
    |                     
    |___ {TARGET_ISA}
    |       |___out
    |           |___libm5.a
    |           |___m5
    |
    |___m5_mmap.h
            
```

## 2. edit buildtools

In the `{SPEC2006 directory}/tools/buildtools` script, find the `specinvoke` section and add the compile flags for gem5 files and libraries. For example:

``` shell
# In the output directory, create directory for gem5 files and libraries
    mkdir $INSTALLDIR/gem5lib
    mkdir $INSTALLDIR/gem5lib/lib
    mkdir $INSTALLDIR/gem5lib/include
# Copy the gem5 files and libraries into output directory   
    cp $INSTALLDIR/../gem5lib/{TARGET_ISA}/out/* $INSTALLDIR/gem5lib/lib
    cp -r $INSTALLDIR/../gem5lib/include/* $INSTALLDIR/gem5lib/include
    cp $INSTALLDIR/../gem5lib/m5_mmap.h $INSTALLDIR/gem5lib/include
# Create flags for gem5 files and gem5 libraries
    GEM5LDFLAGS="-L$INSTALLDIR/gem5lib/lib"
    GEM5CFLAGS="-I$INSTALLDIR/gem5lib/include"
    GEM5EXTRALINK="-lm5"
# Pass the flags
    CFLAGS="$GEM5CFLAGS $ALLCFLAGS $SPECINVOKECFLAGS"; export CFLAGS
    LDFLAGS="$GEM5LDFLAGS $ALLLDFLAGS $SPECINVOKELDFLAGS"; export LDFLAGS
    LIBS="$ALLLIBS $SPECINVOKELIBS $GEM5EXTRALINK"; export LIBS
```
The `$INSTALLDIR` is the `{SPEC2006 directory}/tools/output`.

## 3. edit specinvoke Makefile.in

The `Makefile.in` can be found under the `{SPEC2006 directory}/tools/src/specinvoke/` directory. The `Makefile.in` configs the Makefile for `specinvoke`, so we need to make sure it links the gem5 libraries as they needed.
The `-lm5` has to be linked after the object files, so we can add the `$(LIBS)` after the object files. For example: 
``` shell
    specinvoke: specinvoke.o unix.o getopt.o
        $(CC) $(LDFLAGS) -o $@ specinvoke.o unix.o getopt.o $(LIBS)

    specinvoke_pm: specinvoke.pm.o unix.pm.o getopt.o pmfuncs.o
        $(CC) $(LDFLAGS) -o $@ specinvoke.pm.o unix.pm.o getopt.o pmfuncs.o $(LIBS)
```
