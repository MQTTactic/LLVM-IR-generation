
#### 0x01 Dependency

```bash
$ git clone https://github.com/eclipse/mosquitto.git

# install llvm
$ sudo apt-get install clang-9 llvm-9 llvm-9-dev llvm-9-tools
$ sudo apt install build-essential cmake git

# install wllvm
$ pip3 install wllvm -i https://pypi.tuna.tsinghua.edu.cn/simple

# install dependencies for mosquitto
$ apt install libc-ares-dev libwebsockets-dev  libevent-pthreads-2.1-7  uthash-dev  xsltproc libcjson-dev cjson-dev 

```

#### Compile

```bash

# modify the compile flags in mosquitto-2.0.11/config.mk
BROKER_CFLAGS:=${CFLAGS} -stdlib=libc++ -static -g -O0 -Xclang -disable-O0-optnone  -fno-discard-value-names -disable-llvm-passes 


$ export  CFLAGS="-g -O0 -Xclang -disable-O0-optnone  -fno-discard-value-names"
$ export LLVM_COMPILER=clang CC=wllvm make
$ extract-bc mosquitto

# link mosquitto.bc and lib.a.bc
$ llvm-link ./mosquitto.bc ./libc.a.bc -o complete.bc
```