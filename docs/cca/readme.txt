ARM CCA Emulation
=================

This documentation describes how to run ARM CCA test cases in QEMU emulation
environment.

Prepare the environment
-----------------------

Prepare the source code:

tf-a-tests and trusted-firmware-a need to be patched with our CCA
implementation.

git clone https://git.trustedfirmware.org/TF-A/tf-a-tests.git
cd tf-a-tests
git checkout 911fa6ba3
git apply path/to/tftf-911fa6ba3.patch

git clone https://git.trustedfirmware.org/TF-A/trusted-firmware-a.git
cd trusted-firmware-a
git checkout c3bdd3d3c
git apply path/to/tfa-c3bdd3d3c.patch

The current directory contains three subdirectories:
 * qemu (this repo)
 * tf-a-tests
 * trusted-firmware-a

Prepare the compiler:
export CROSS_COMPILE=aarch64-none-elf-
export PATH=/path/to/gcc-arm-10.3-2021.07-x86_64-aarch64-none-elf/bin:$PATH

Compile
-------

mkdir qemu/build
cd qemu/build
../configure --target-list=aarch64-softmmu --disable-docs --enable-debug
make
cd ../..

make -C tf-a-tests PLAT=qemu DEBUG=1 USE_NVM=0 SHELL_COLOR=1 TESTS=realm-payload tftfr
make -C trusted-firmware-a ARCH=aarch64 PLAT=qemu ARM_DISABLE_TRUSTED_WDOG=1 ENABLE_RME=1 DEBUG=1 BL33=../tf-a-tests/build/qemu/debug/tftfr.bin all fip

rm -f flash.bin
dd if=trusted-firmware-a/build/qemu/debug/bl1.bin of=flash.bin bs=4096 conv=notrunc
dd if=trusted-firmware-a/build/qemu/debug/fip.bin of=flash.bin seek=64 bs=4096 conv=notrunc

Run Test Cases
--------------

./qemu/build/qemu-system-aarch64 -nographic -serial telnet::54341,server -smp sockets=2,cores=4 -machine virt,secure=on,virtualization=on,gic-version=3 -m 2048 -cpu max -d unimp,guest_errors,rme -D qemu.log -bios flash.bin

In another console, run `telnet localhost 54341`. You should get the following output:
$ telnet localhost 54341
Trying ::1...
Connected to localhost.
Escape character is '^]'.
NOTICE:  Booting Trusted Firmware
NOTICE:  BL1: v2.6(debug):v2.6-1001-g8aaa6d629
...
******************************* Summary *******************************
> Test suite 'Realm payload tests'
                                                                Passed
=================================
Tests Skipped : 1
Tests Passed  : 17
Tests Failed  : 0
Tests Crashed : 0
Total tests   : 18
=================================
NOTICE:  Exiting tests.
