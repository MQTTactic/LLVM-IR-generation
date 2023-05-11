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

* Test Enviroment
> $ cat /etc/os-release
```
NAME="Ubuntu"
VERSION="20.04.3 LTS (Focal Fossa)"
ID=ubuntu
ID_LIKE=debian
PRETTY_NAME="Ubuntu 20.04.3 LTS"
VERSION_ID="20.04"
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
VERSION_CODENAME=focal
UBUNTU_CODENAME=focal
```

> $ uname -a
```
5.15.0-69-generic #76~20.04.1-Ubuntu SMP Mon Mar 20 15:54:19 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

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


## Open-source MQTT brokers

| Broker Name | Repository |
| --- | --- |
| ejabberd | processone/ejabberd |
| Emitter | emitter-io/emitter |
| EMQ X | emqx/emqx |
| FlashMQ | halfgaar/FlashMQ |
| HBMQTT | beerfactory/hbmqtt |
| HiveMQ | hivemq/hivemq-community-edition |
| Jmqtt | Cicizz/jmqtt |
| Moquette | moquette-io/moquette |
| Mosca | moscajs/mosca |
| Mosquitto | eclipse/mosquitto |
| MQTTnet | dotnet/MQTTnet |
| MqttWk | Wizzercn/MqttWk |
| NanoMQ | emqx/nanomq |
| RabbitMQ | rabbitmq/rabbitmq-server |
| Cassandana | mtsoleimani/cassandana |
| Apache ActiveMQ | apache/activemq |
| Apache ActiveMQ Artemis | apache/activemq-artemis |
| Solace | Solace |
| SwiftMQ | iitsoftware/swiftmq-ce |
| VerneMQ | vernemq/vernemq |
| mainflux | mainflux/mainflux |
| hmq | fhmq/hmq |
| mqtt | xamarin/MQTT |
| uMQTTBroker | martin-ger/uMQTTBroker |
| volantmq | VolantMQ/volantmq |
| mqtt | luelan/mqtt |
| crossbar | crossbario/crossbar |
| Erl.mqtt.server | alekras/erl.mqtt.server |
| gmqtt | Drmagic/gmqtt |
| smqtt | quickmsg/smqtt |
| enmasse | EnMasseProject/enmasse |
| mqtt | mochi-co/mqtt |
| gnatmq | gnatmq/gnatmq |
| gossipd | luanjunyi/gossipd |
| mica-mqtt | lets-mica/mica-mqtt |
| rumqtt | bytebeamio/rumqtt |
| jo-mqtt | joey-happy/jo-mqtt |
| gmqtt | hb-chen/gmqtt |
| sol | codepr/sol |
| mqtt-broker | bschwind/mqtt-broker |
| mqtt | atilaneves/mqtt |
| iot-mqtt | ShiCloud/iot-mqtt |
| hermes | c16a/hermes |
| esp-idf-mqtt-broker | nopnop2002/esp-idf-mqtt-broker |
| mmqtt | MrHKing/mmqtt |
| JetMQ | butaji/JetMQ |
| mqttools | eerimoq/mqttools |
| mqtt-gateway | FarmBot-Labs/mqtt-gateway |
| TinyMqtt | hsaturn/TinyMqtt |
| whsnbg | homewsn/whsnbg |
| mithqtt | longkerdandy/mithqtt |
| skyline | hansonkd/skyline |
| Aedes | moscajs/aedes |
| hrotti | alsm/hrotti |
| KMQTT | davidepianca98/KMQTT |
| Mystique | TheThingsIndustries/mystique |
| SurgeMQ | zentures/surgemq |
| creep | ConnorRigby/creep |
| amlen | eclipse/amlen |
| rmqtt | rmqtt/rmqtt |
| PronghornGateway | oci-pronghorn/PronghornGateway |
| DovakinMQ | Dovakin-IO/DovakinMQ |
| ioBroker.mqtt | ioBroker/ioBroker.mqtt |
| node-red-contrib-aedes | martin-doyle/node-red-contrib-aedes |
| mqttcpp | atilaneves/mqttcpp |
| LV-MQTT-Broker | LabVIEW-Open-Source/LV-MQTT-Broker |
| haskell-hummingbird | lpeterse/haskell-hummingbird |
| mhub | funkygao/mhub |
| clima-link | PatrickKalkman/clima-link |
| GS.GRID | GRIDSystemSAS/GS.GRID |
| pyrinas-server-rs | pyrinas-iot/pyrinas-server-rs |
| wave | gbour/wave |
| lannister | anyflow/lannister |
| JoramMQ | JoramMQ |

