I'll note down the issues I met during building and running SPEC2017 in RISC-V Ubuntu20.04.4 image with qemu-system-riscv64 virtual machine on this page.

# Useful sites


# SPEC2017 system requirement

https://www.spec.org/cpu2017/Docs/system-requirements.html#disk

Disk: 10 GB minimum but 250 GB preferred
Memory: 16 GB


# ISSUE

## During buildtool

### configure: error: no acceptable C compiler found in $PATH

```
    configure: error: in `/home/ubuntu/riscv_spec2017/tools/src/make-4.2.1':
    configure: error: no acceptable C compiler found in $PATH
    See `config.log' for more details
    + testordie error configuring make
    + test 1 -ne 0
    + echo !!! error configuring make
    !!! error configuring make
    + [ -z  ]
    + kill -TERM 2160
    + exit 1
    !!!!! buildtools killed
```

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

config.guess: http://git.savannah.gnu.org/gitweb/p=config.git;a=blob_plain;f=config.guess;hb=HEAD

config.sub: http://git.savannah.gnu.org/gitweb/?p=config.git;a=blob_plain;f=config.sub;hb=HEAD

