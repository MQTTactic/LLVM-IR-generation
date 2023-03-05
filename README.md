# LLVM-IR-generation


## Frontends for LLVM
> Some projects are still in the development stage and may not fully support language features.

|  Language          |                Tool(s)                                              |
| ---------- | ------------------------------------------------------------ |
| python     | - [Numba](https://numba.readthedocs.io/en/stable/reference/envvars.html#envvar-NUMBA_DUMP_LLVM)<br />- [numba/llvmlite: A lightweight LLVM python binding for writing JIT compilers (github.com)](https://github.com/numba/llvmlite) |
| java       | - [polyglot-compiler/JLang: JLang: Ahead-of-time compilation of Java programs to LLVM](https://github.com/polyglot-compiler/JLang)<br />- [superblaubeere27/masxinlingvonta: Compiles Java ByteCode to LLVM IR (for obfuscation purposes)](https://github.com/superblaubeere27/masxinlingvonta)<br />- https://vmkit.llvm.org/<br />- [mirkosertic/Bytecoder: Framework to interpret and transpile JVM bytecode to JavaScript, OpenCL or WebAssembly.](https://github.com/mirkosertic/Bytecoder)<br />- [Kotlin/Native libraries \| Kotlin (kotlinlang.org)](https://kotlinlang.org/docs/native-libraries.html) |
| javascript | - [SirJosh3917/jssat: Compile JS into LLVM IR - JavaScript Static Analysis Tool](https://github.com/SirJosh3917/jssat)<br />- [wizardpisces/js-ziju: Compile javascript to LLVM IR, x86 assembly and self interpreting](https://github.com/wizardpisces/js-ziju) |
| erlang     | - [lumen/lumen: An alternative BEAM implementation, designed for WebAssembly](https://github.com/lumen/lumen) |
| golang     | - [gollvm - Git at Google](https://go.googlesource.com/gollvm/) |
| rust       | - [rust-osdev/cargo-xbuild: Automatically cross-compiles the sysroot crates core, compiler_builtins, and alloc.](https://github.com/rust-osdev/cargo-xbuild) |


------------------------

## Instruction

> The installation of tools and LLVM envrioments will not be repeated here.

##### C/C++

```bash
# install wllvm
$ pip3 install wllvm

# generate LLVM IR for a single file
$ clang  -emit-llvm -c -g -O0 -Xclang -disable-O0-optnone  -fno-discard-value-names example.c -o example.bc

# generate llvm ir for a project
# 1. Change the compiler to `clang`/`clang++` and change the compiler flags. You also can add compiler flags to CMakeLists.txt.
$ export CFLAGS="-g -O0 -Xclang -disable-O0-optnone  -fno-discard-value-names"
# 2. Compile and link multiple files
$ export LLVM_COMPILER=clang CC=wllvm make
# 3. Extract LLVM bitcode
$ extract-bc {bin/xx}

```


##### Golang
> https://go.googlesource.com/gollvm/
```bash
# Record the instructions compiled by Go compiler
$ go build -gccgoflags "-static-libgo -S" -work -x 1> transcript.txt 2>&1

# Extract gollvm instructions, remember to delete the cache before compiling
$ egrep '(WORK=|llvm-goc -c|cd)' transcript.txt > compile.sh

# clean the cache
$ rm -rf ~/.cache/go-build/*
$ go clean --modcache

# modify the instructions in compile.sh to generate LLVM IR:
/usr/local/bin/llvm-goc -c -O2 -g -m64 -fdebug-prefix-map=$WORK=/tmp/go-build -gno-record-gcc-switches -fgo-relative-import-path=_/build-debug/bin "-S -emit-llvm" -o test.ll -I $WORK/b001/_importcfgroot_ -static-libgo ./mypackage2.go $WORK/b001/_gomod_.go
```

And here is a simple script for the last step.
```python
import re

origin = open("compile.sh")
x = origin.read().split("\n")

result = []
count = 0
for i in x:
     m = re.findall("-fgo-pkgpath=(.*) -o", i)
     print(m)
     if(m == [] or m == None):
          result.append(i)
     elif("VolantMQ" in m[0]):
          n = re.sub("-o \$WORK/.*/_go_.o",f"-S -emit-llvm -fno-inline -o /tmp/{count}.ll",i)
          n = re.sub("-O2","-O1", n)
          n = re.sub("\./","`pwd`/",n)
          result.append(n)
          count += 1

output = open("result.sh",'w')
for i in result:
     output.write(i+'\n')
output.close()
origin.close()
```


##### Rust

```bash
$ cargo rustc -- --emit=llvm-ir -g -C no-prepopulate-passes -C link-dead-code=yes -C default-linker-libraries=no
```


##### Kotlin
> [Get started with Kotlin/Native using the command-line compiler | Kotlin (kotlinlang.org)](https://kotlinlang.org/docs/native-command-line-compiler.html#write-hello-kotlin-native-program)
> [Release Kotlin 1.7.10 · JetBrains/kotlin (github.com)](https://github.com/JetBrains/kotlin/releases/tag/v1.7.10)

```bash
$ kotlinc-native hello.kt -o hello -p bitcode
```




