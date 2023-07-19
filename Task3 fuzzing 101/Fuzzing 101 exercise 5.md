# Fuzzing 101 exercise 5

### 下载并构建目标

获取模糊目标并为要模糊的项目创建一个新目录：

```
cd $HOME
mkdir Fuzzing_libxml2 && cd Fuzzing_libxml2
```

下载并解压缩 libxml2-2.9.4.tar.gz：

```
wget http://xmlsoft.org/download/libxml2-2.9.4.tar.gz
tar xvf libxml2-2.9.4.tar.gz && cd libxml2-2.9.4/
```

构建并安装 libxml2：

```
sudo apt-get install python-dev
CC=afl-clang-lto CXX=afl-clang-lto++ CFLAGS="-fsanitize=address" CXXFLAGS="-fsanitize=address" LDFLAGS="-fsanitize=address" ./configure --prefix="$HOME/Fuzzing_libxml2/libxml2-2.9.4/install" --disable-shared --without-debug --without-ftp --without-http --without-legacy --without-python LIBS='-ldl'
make -j$(nproc)
make install
```

测试一切是否正常工作：

```
./xmllint --memory ./test/wml.xml
```

输出为：

![image-20230717150204742](E:/typora_pictures/image-20230717150204742.png)



### 种子语料库创建

首先，需要获取一些 XML 示例。我们将使用此存储库中提供的 **SampleInput.xml**：

```
mkdir afl_in && cd afl_in
wget https://raw.githubusercontent.com/antonio-morales/Fuzzing101/main/Exercise%205/SampleInput.xml
cd ..
```



### 自定义词典

现在需要创建一个 XML 字典或使用 AFL++ 提供的 XML 字典：

```
mkdir dictionaries && cd dictionaries
wget https://raw.githubusercontent.com/AFLplusplus/AFLplusplus/stable/dictionaries/xml.dict
cd ..
```



### 模糊测试

使用 -**x** 标志设置字典路径，并使用 **-D **启用确定性突变（仅适用于主模糊器）：`--valid`

运行模糊器：

```
afl-fuzz -m none -i ./afl_in -o afl_out -s 123 -x ./dictionaries/xml.dict -D -M master -- ./xmllint --memory --noenc --nocdata --dtdattr --loaddtd --valid --xinclude @@
```

运行另一个从属实例：

```
afl-fuzz -m none -i ./afl_in -o afl_out -s 234 -S slave1 -- ./xmllint --memory --noenc --nocdata --dtdattr --loaddtd --valid --xinclude @@
```

非常非常非常长的时间以后，终于可以看到崩溃...：

![image-20230719161315906](E:/typora_pictures/image-20230719161315906.png)

![image-20230719161321754](E:/typora_pictures/image-20230719161321754.png)



### ASan

向程序提供崩溃文件，发现程序出现栈溢出错误，命令为:

```
./xmllint --memory --noenc --nocdata --dtdattr --loaddtd --valid --xinclude './afl_out/default/crashes/id:000000,sig:06,src:003963,time:12456489,op:havoc,rep:4'
```



本此实验的难点在于等待超长时间来运行出crash，教程参考时间为15h但实际上要等待更久，非常考验实验的耐心...

