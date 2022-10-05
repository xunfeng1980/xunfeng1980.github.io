---
layout: post
title:  "mac 环境构建 openjdk10"
date:   2017-10-11 23:44:18
categories: jdk
---


先把openjdk代码拉取下来，估计几个小时吧。或者下载，也很慢。

github:https://github.com/dmlloyd/openjdk

heiduyun:https://pan.baidu.com/s/1skVhHlN

如果没有安装xcode command line tools，还需要执行以下命令

```Shell
xcode-select --install
```

还需要安装freetype，执行

```shell
brew install freetype
```

最后确认一下，有没有安装jdk，为啥编译jdk还需要先安装jdk，可以看看编译原理的自举。

```shell
openjdk git:(jdk10/master) ✗ java -version
java version "9.0.1"
Java(TM) SE Runtime Environment (build 9.0.1+11)
Java HotSpot(TM) 64-Bit Server VM (build 9.0.1+11, mixed mode)
```



下面开始jdk编译配置:

```shell
bash configure --with-jvm-variants=server  --with-target-bits=64 --enable-debug   --disable-warnings-as-errors
```
需要其他配置可以查看，或者查看doc目录里的building.html或building.md

```shell 
bash configure -h
```
配置成功大抵是下面这样

```shell
Configuration summary:
* Debug level:    fastdebug
* HS debug level: fastdebug
* JDK variant:    normal
* JVM variants:   server
* OpenJDK target: OS: macosx, CPU architecture: x86, address length: 64
* Version string: 10-internal+0-adhoc.xxx.openjdk (10-internal)

Tools summary:
* Boot JDK:       java version "9.0.1" Java(TM) SE Runtime Environment (build 9.0.1+11) Java HotSpot(TM) 64-Bit Server VM (build 9.0.1+11, mixed mode)  (at /Library/Java/JavaVirtualMachines/jdk-9.0.1.jdk/Contents/Home)
* Toolchain:      clang (clang/LLVM)
* C Compiler:     Version 9.0.0 (at /usr/bin/clang)
* C++ Compiler:   Version 9.0.0 (at /usr/bin/clang++)
```

下面开始构建:

```shell 
make
```
```shell
Building target 'default (exploded-image)' in configuration 'macosx-x86_64-normal-server-fastdebug'
Compiling 8 files for BUILD_TOOLS_LANGTOOLS
Creating support/modules_libs/java.base/libjsig.dylib from 1 file(s)
Creating hotspot/variant-server/tools/adlc/adlc from 13 file(s)
Compiling 2 files for BUILD_JVMTI_TOOLS
Warning: No mercurial configuration present and no .src-rev
Parsing 1 properties into enum-like class for jdk.compiler
Compiling 13 properties into resource bundles for jdk.javadoc
Compiling 12 properties into resource bundles for jdk.jdeps
Compiling 20 properties into resource bundles for jdk.compiler
Compiling 7 properties into resource bundles for jdk.jshell
Compiling 117 files for BUILD_INTERIM_java.compiler
Compiling 397 files for BUILD_INTERIM_jdk.compiler
...
clang: warning: libstdc++ is deprecated; move to libc++ with a minimum deployment target of OS X 10.9 [-Wdeprecated]
clang: warning: libstdc++ is deprecated; move to libc++ with a minimum deployment target of OS X 10.9 [-Wdeprecated]
Compiling 4 files for BUILD_JIGSAW_TOOLS
Stopping sjavac server
Finished building target 'default (exploded-image)' in configuration 'macosx-x86_64-normal-server-fastdebug'
```

构建成功，在MacBook Pro 10.13上编译大概需要十几分钟左右



验证一下：

```shell
cd build/macosx-x86_64-normal-server-fastdebug/jdk/bin
bin git:(jdk10/master) ✗ ./java -version
openjdk version "10-internal"
OpenJDK Runtime Environment (fastdebug build 10-internal+0-adhoc.xxx.openjdk)
OpenJDK 64-Bit Server VM (fastdebug build 10-internal+0-adhoc.xxx.openjdk, mixed mode)
```



本来要构建jdk9，结果失败了，但是jdk10成功了，那就先这样了。
下次再来好好研究9的构建。





