# Fuzzing 101 exercise 3

### 下载并构建目标

首先获取模糊目标，为要模糊的项目创建一个新目录：

```
cd $HOME
mkdir fuzzing_tcpdump && cd fuzzing_tcpdump/
```



下载并解压缩 tcpdump-4.9.2.tar.gz

```
wget https://github.com/the-tcpdump-group/tcpdump/archive/refs/tags/tcpdump-4.9.2.tar.gz
tar -xzvf tcpdump-4.9.2.tar.gz
```



下载libpcap，这是TCPdump需要的跨平台库，下载并解压缩 libpcap-1.8.0.tar.gz：

```
wget https://github.com/the-tcpdump-group/libpcap/archive/refs/tags/libpcap-1.8.0.tar.gz
tar -xzvf libpcap-1.8.0.tar.gz
```



重命名为 libpcap-1.8.0,否则，tcpdump 找不到本地路径

```
mv libpcap-libpcap-1.8.0/ libpcap-1.8.0
```



构建并安装 libpcap：

```
cd $HOME/fuzzing_tcpdump/libpcap-1.8.0/
./configure --enable-shared=no
make
```



现在，我们可以构建和安装 tcpdump：

```
cd $HOME/fuzzing_tcpdump/tcpdump-tcpdump-4.9.2/
./configure --prefix="$HOME/fuzzing_tcpdump/install/"
make
make install
```



要测试一切是否正常工作，只需键入：

```
$HOME/fuzzing_tcpdump/install/sbin/tcpdump -h
```

能够看到以下输出：

![image-20230715150153509](E:/typora_pictures/image-20230715150153509.png)



### 种子语料库创建

可以在“./tests”文件夹中找到很多.pcacp示例。可以使用以下命令行运行这些 .pcap 文件：

```
$HOME/fuzzing_tcpdump/install/sbin/tcpdump -vvvvXX -ee -nn -r [.pcap file]
```

例如：

```
$HOME/fuzzing_tcpdump/install/sbin/tcpdump -vvvvXX -ee -nn -r ./tests/geneve.pcap
```

输出应如下所示：

![image-20230715150403380](E:/typora_pictures/image-20230715150403380.png)





## ASan

**ASan**是C和C++的快速内存错误检测器，它由编译器检测模块和运行时库组成，该工具能够查找对堆、堆栈和全局对象的越界访问，以及释放后使用、双重释放和内存泄漏错误。

接下来要在启用ASan的情况下构建tcpdump（和libpcap）。

首先，我们将清理所有以前编译的对象文件和可执行文件：

```
rm -r $HOME/fuzzing_tcpdump/install
cd $HOME/fuzzing_tcpdump/libpcap-1.8.0/
make clean

cd $HOME/fuzzing_tcpdump/tcpdump-tcpdump-4.9.2/
make clean
```



现在，我们在调用之前设置和：`AFL_USE_ASAN=1  configure  make`

```
cd $HOME/fuzzing_tcpdump/libpcap-1.8.0/
export LLVM_CONFIG="llvm-config-11"
CC=afl-clang-lto ./configure --enable-shared=no --prefix="$HOME/fuzzing_tcpdump/install/"
AFL_USE_ASAN=1 make

cd $HOME/fuzzing_tcpdump/tcpdump-tcpdump-4.9.2/
AFL_USE_ASAN=1 CC=afl-clang-lto ./configure --prefix="$HOME/fuzzing_tcpdump/install/"
AFL_USE_ASAN=1 make
AFL_USE_ASAN=1 make install
```



### 模糊测试

使用以下命令运行模糊程序：

```
afl-fuzz -m none -i $HOME/fuzzing_tcpdump/tcpdump-tcpdump-4.9.2/tests/ -o $HOME/fuzzing_tcpdump/out/ -s 123 -- $HOME/fuzzing_tcpdump/install/sbin/tcpdump -vvvvXX -ee -nn -r @@
```

**注意：64 位系统上的 ASAN 请求大量虚拟内存。这就是设置了标志“-m none”来禁用 AFL 中的内存限制的原因**

一段时间后，出现多次崩溃：

![image-20230715211944641](E:/typora_pictures/image-20230715211944641.png)



### Triage

调试使用 ASan 构建的程序比前面的练习要容易，向程序提供崩溃文件：

```
$HOME/fuzzing_tcpdump/install/sbin/tcpdump -vvvvXX -ee -nn -r '/home/fuzz/fuzzing_tcpdump/out/default/crashes/id:000000,sig:06,src:007367,time:10720920,execs:4393777,op:havoc,rep:2'
```



获得崩溃的良好摘要，包括执行跟踪：

![image-20230715215336460](E:/typora_pictures/image-20230715215336460.png)

发现有堆溢出heap-buffer-overflow

官方给出的修复方案见：[Use nd_ types, add EXTRACT_, fix a bounds check. · the-tcpdump-group/tcpdump@85078ee (github.com)](https://github.com/the-tcpdump-group/tcpdump/commit/85078eeaf4bf8fcdc14a4e79b516f92b6ab520fc#diff-05f854a9033643de07f0d0059bc5b98f3b314eeb1e2499ea1057e925e6501ae8L381)



本此实验加深了对AFL的理解，熟悉了tcpdump工具，以及AScan的使用过程。另外，换成官方建议给出的Ubuntu20版本能感到实验确实顺利了很多，看来环境的建议还是要听哇T_T