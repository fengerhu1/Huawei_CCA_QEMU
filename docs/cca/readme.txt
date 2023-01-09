ARM CCA Emulation
=================

This documentation describes how to build and run ARM CCA test cases in QEMU
emulation environment.

Compile
-------

export CROSS_COMPILE=aarch64-none-elf-
export PATH=/path-to-gcc-arm-10.3-2021.07-x86_64-aarch64-none-elf/bin:/path-to-cmake-3.25.1/bin:$PATH

mkdir qemu/build
cd qemu/build
../configure --target-list=aarch64-softmmu --disable-docs --enable-debug
make

cmake -DRMM_CONFIG=qemu_defcfg -DRMM_PLATFORM=qemu -S tf-rmm -B tf-rmm/build -DCMAKE_BUILD_TYPE=Debug -DLOG_LEVEL=50
cmake --build tf-rmm/build

make -C tf-a-tests PLAT=qemu DEBUG=1 USE_NVM=0 SHELL_COLOR=1 TESTS=realm-payload tftf
make -C trusted-firmware-a ARCH=aarch64 PLAT=qemu ARM_DISABLE_TRUSTED_WDOG=1 ENABLE_RME=1 DEBUG=1 BL33=../tf-a-tests/build/qemu/debug/tftf.bin all fip

rm -f flash.bin
dd if=trusted-firmware-a/build/qemu/debug/bl1.bin of=flash.bin bs=4096 conv=notrunc
dd if=trusted-firmware-a/build/qemu/debug/fip.bin of=flash.bin seek=64 bs=4096 conv=notrunc


Run Test Cases
--------------

./qemu/build/qemu-system-aarch64 -nographic -serial telnet::54341,server -smp sockets=2,cores=4 -machine virt,secure=on,virtualization=on,gic-version=3 -m 3072 -cpu max -d unimp,guest_errors,rme -D qemu.log -bios flash.bin
