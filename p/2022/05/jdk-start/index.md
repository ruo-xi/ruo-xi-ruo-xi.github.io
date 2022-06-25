# Jdk Start

# Jdk Start

## Environment

> 操作系统:  Linux yu 5.17.9-arch1-1 #1 SMP PREEMPT Wed, 18 May 2022 17:30:11 +0000 x86_64 GNU/Linux
>
> bootjdk:   openjdk version "17.0.3" 2022-04-19
>
> jdk源码:   jdk17u-jdk-17.0.3-ga
>
> 阅读工具:  vscode

## Prepration

### Source

* openjdk [https://hg.openjdk.java.net/](https://hg.openjdk.java.net/)
* [GitHub](https://github.com/openjdk?type=source)

### Compile

#### compile tools

* basic tools
* bear

  用于生成**compile_commands.json**文件

#### compile

```bash
# jdk 17   
# maybe with --disable-javac-server

# clang
bash configure --with-conf-name=clang --disable-warnings-as-errors --with-boot-jdk=/usr/lib/jvm/java-17-openjdk --with-toolchain-type=clang
bear -- make CONF_NAME=clang

# gcc
bash configure --with-conf-name=gcc --disable-warnings-as-errors --with-boot-jdk=/usr/lib/jvm/java-17-openjdk --with-toolchain-type=gcc
bear -- make CONF_NAME=gcc
```

test build java with `./build/clang/jdk/bin/java --version`  command

output:

```bash
openjdk version "17.0.3-internal" 2022-04-19
OpenJDK Runtime Environment (build 17.0.3-internal+0-adhoc.yu.jdk17u-jdk-17.0.3-ga)
OpenJDK 64-Bit Server VM (build 17.0.3-internal+0-adhoc.yu.jdk17u-jdk-17.0.3-ga, mixed mode)
```

### Tools

* vscode

  * Plugin
    * clangd
    * Java Extension Pack
* jps

  * -l 查看java线程
* jhsdb

  * sudo jhsdb hsdb 打开ui界面

  需要安装[Courier Prime](https://quoteunquoteapps.com/courierprime/)  字体

  ```
  yay -S ttf-courier-prime
  ```

## Dir Structure

* cpu
* os
* os_cpu
* share
  * adlc      (Architecture Description Language Compiler)      平台描述文件
  * aot        汇编器接口
  * c1           client编译器("又称c1")
  * ci            动态编译器的公共服务/从动态编译器到VM的接口
  * classfile  类文件的处理(类加载和系统符号表等等)
  * code        动态生成的代码的管理
  * compiler   从VM调用动态编译器的接口
  * gc            GC的实现
  * include       比较重要的头文件,就三个jvm.h/jmm.h/cds.h
  * interpreter   解释器
  * jfr          //todo
  * jvmci     JVM Compiler Interface
  * libadt
  * logging
  * memory   内存管理
  * metaprogramming
  * oops,   HotSpot VM的对象系统的实现
  * opto  C2编译器(又称optp)
  * precompiled 预编译
  * prims    HotSpot VM对外接口,包括部分native和JVMTI的实现
  * runtime   运行时支持库
  * service    JMX
  * utilities  工具类

## Tips

### how to see JIT-compile code

1. add '-XX:+UnlockDiagnosticVMOptions -XX:+PrintAssembly' flags
2. need disassembler.so library.

```bash
  wegt https://codeload.github.com/liuzhengyang/hsdis/zip/refs/heads/master
  unzip hsdis-master.zip
  tar -zxvf binutils-2.26.tar.gz
  cd hsdis-master
  make BINUTILS=binutils-2.26 ARCH=amd64
  sudo cp build/linux-amd64/hsdis-amd64.so /usr/lib/jvm/java-11-openjdk/lib/
```

## Reference

* [Building the JDK](https://openjdk.java.net/groups/build/doc/building.html)
* [How to see JIT-compiled code in JVM?](https://stackoverflow.com/questions/1503479/how-to-see-jit-compiled-code-in-jvm)
* 'man java' command

