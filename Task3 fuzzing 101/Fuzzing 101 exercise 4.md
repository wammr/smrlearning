# Fuzzing 101 exercise 4

### 下载并构建目标

首先获取模糊目标，为要模糊的项目创建一个新目录：

```
cd $HOME
mkdir fuzzing_tiff && cd fuzzing_tiff/
```

下载并解压缩 libtiff 4.0.4：

```
wget https://download.osgeo.org/libtiff/tiff-4.0.4.tar.gz
tar -xzvf tiff-4.0.4.tar.gz
```

现在构建和安装 libtiff：

```
cd tiff-4.0.4/
./configure --prefix="$HOME/fuzzing_tiff/install/" --disable-shared
make
make install
```

测试tiffinfo是否正常工作，输入：

```
$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w $HOME/fuzzing_tiff/tiff-4.0.4/test/images/palette-1c-1b.tiff
```

发现tiffinfo可以正常工作：

![image-20230716100643388](https://github.com/wammr/smrlearning/blob/master/picture/image-20230716100643388.png)

### 代码覆盖率

代码覆盖率是一个软件指标，显示每行代码被触发的次数。

首先，安装 lcov使用以下命令来完成：

```
sudo apt install lcov
```

安装lcov的时候出现问题：“/var/lib/dpkg/lock-frontend error”

解决方法：sudo rm /var/lib/dpkg/lock-frontend     &    sudo rm /var/lib/dpkg/lock

参考链接：[(15 封私信) 如何解决 Could not get lock /var/lib/dpkg/lock 问题？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/590708259)



现在需要用编译器和链接器重建 libTIFF：`--coverage`

```
rm -r $HOME/fuzzing_tiff/install
cd $HOME/fuzzing_tiff/tiff-4.0.4/
make clean
  
CFLAGS="--coverage" LDFLAGS="--coverage" ./configure --prefix="$HOME/fuzzing_tiff/install/" --disable-shared
make
make install
```



然后，通过输入以下内容来收集代码覆盖率数据：

```
cd $HOME/fuzzing_tiff/tiff-4.0.4/
lcov --zerocounters --directory ./
lcov --capture --initial --directory ./ --output-file app.info
$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w $HOME/fuzzing_tiff/tiff-4.0.4/test/images/palette-1c-1b.tiff
lcov --no-checksum --directory ./ --capture --output-file app2.info
```



对于每个命令的解释：

- `lcov --zerocounters --directory ./`：重置以前的计数器
- `lcov --capture --initial --directory ./ --output-file app.info`：返回“基线”覆盖率数据文件，其中包含每个检测行的零覆盖率
- `$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w $HOME/fuzzing_tiff/tiff-4.0.4/test/images/palette-1c-1b.tiff`：运行要分析的应用程序，可以使用不同的输入多次运行它
- `lcov --no-checksum --directory ./ --capture --output-file app2.info`：将当前覆盖状态保存到 app2.info 文件中

最后，生成 HTML 输出：

```
genhtml --highlight --legend -output-directory ./html-coverage/ ./app2.info
```



在该文件夹中创建代码覆盖率报告，打开文件`./html-coverage/index.html`  ，看到如下所示的内容：

![image-20230716105059464](https://github.com/wammr/smrlearning/blob/master/picture/image-20230716105059464.png)

### 模糊测试

现在将在启用 ASAN 的情况下编译 libtiff。

首先清理所有以前编译的对象文件和可执行文件：

```
rm -r $HOME/fuzzing_tiff/install
cd $HOME/fuzzing_tiff/tiff-4.0.4/
make clean
```

调用make之前设置AFL_USE_ASAN=1：

```
export LLVM_CONFIG="llvm-config-11"
CC=afl-clang-lto ./configure --prefix="$HOME/fuzzing_tiff/install/" --disable-shared
AFL_USE_ASAN=1 make -j4
AFL_USE_ASAN=1 make install
```

现在，使用以下命令运行模糊程序：

```
afl-fuzz -m none -i $HOME/fuzzing_tiff/tiff-4.0.4/test/images/ -o $HOME/fuzzing_tiff/out/ -s 123 -- $HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w @@
```

看到结果，这次时间非常迅速：

![image-20230717084245382](https://github.com/wammr/smrlearning/blob/master/picture/image-20230717084245382.png)

### ASan跟踪

输入命令：

```
$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w '/home/fuzz/fuzzing_tiff/out/default/crashes/id:000000,sig:06,src:000000,time:1287,execs:1016,op:havoc,rep:1'
```

![image-20230717084601733](https://github.com/wammr/smrlearning/blob/master/picture/image-20230717084601733.png)

### 代码覆盖率测量

跟上面的代码覆盖率过程类似，只不过使用crashes里的文件：

重建 libTIFF：`--coverage`

```
rm -r $HOME/fuzzing_tiff/install
cd $HOME/fuzzing_tiff/tiff-4.0.4/
make clean
  
CFLAGS="--coverage" LDFLAGS="--coverage" ./configure --prefix="$HOME/fuzzing_tiff/install/" --disable-shared
make
make install
```



然后，我们可以通过键入以下内容来收集代码覆盖率数据：

```
cd $HOME/fuzzing_tiff/tiff-4.0.4/
lcov --zerocounters --directory ./
lcov --capture --initial --directory ./ --output-file app.info
$HOME/fuzzing_tiff/install/bin/tiffinfo -D -j -c -r -s -w $HOME/fuzzing_tiff/out/default/crashes/id:000000,sig:06,src:000000,time:1287,execs:1016,op:havoc,rep:1
lcov --no-checksum --directory ./ --capture --output-file app2.info
```

结果如下：

![image-20230717091512980](https://github.com/wammr/smrlearning/blob/master/picture/image-20230717091512980.png)

本次实验引入了一个新的概念代码覆盖率，通过使用代码覆盖率，可以了解模糊器到达了代码的哪些部分，并可视化模糊处理过程。完成本练习后，对如何使用LCOV测量代码覆盖率有了一定的了解，在今后的学习过程中也可以通过使用该工具来可视化模糊处理。