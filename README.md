Build the latest riscv toolchain with v and zfh extension on ubuntu 20.04.3

You need to build qemu from the git for running the binary

```shell
# install build requirements
sudo apt-get update
sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev device-tree-compiler

git clone https://github.com/riscv/riscv-gnu-toolchain.git
cd riscv-gnu-toolchain
git checkout 28271f03bb538d926ad2889dc8ad1b0cb1b3b45c

git submodule update --init --recursive --depth 1 riscv-binutils
git submodule update --init --recursive --depth 1 riscv-glibc
git submodule update --init --recursive --depth 1 riscv-dejagnu
git submodule update --init --recursive --depth 1 riscv-newlib
git submodule update --init --recursive --depth 1 riscv-gdb

# upgrade the riscv-gcc folder manually
rm -rf riscv-gcc
git clone -b riscv-gcc-10.1-rvv-dev --single-branch --depth=1 https://github.com/riscv-collab/riscv-gcc.git

# revert vfredsum -> vfredusum changes for compatibility atm
pushd riscv-gcc
wget https://github.com/riscv-collab/riscv-gcc/commit/227d14bbdbcef65cc3f8ea96e6e3fb905d32368f.patch
patch -p1 -R -i 227d14bbdbcef65cc3f8ea96e6e3fb905d32368f.patch
popd

# fix compile error
sed -i '/__OBSOLETE_MATH/d' riscv-newlib/newlib/libm/common/math_errf.c

# build toolchain, you need at least 20GB free memory for 4 parallel jobs
./configure --prefix=`pwd`/rv64gcv-install --with-arch=rv64gcv_zfh
make linux -j 4

# strip debug symbols
find rv64gcv-install -type f | xargs -i strip -g {} || true

# fix sysroot/lib/tls missing error
pushd rv64gcv-install/sysroot/lib
ln -s ./ tls
popd

zip -9 --symlinks -r rv64gcv-install.zip rv64gcv-install
```
