# Fuzzing 101 exercise 2

### 下载并构建目标

本此测试的对象libexif库是一个用来读取数码相机照片中包含的exif信息的C语言库。首先下载libexif库并进行解压和安装：

```
//创建目录
cd $HOME
mkdir fuzzing_libexif && cd fuzzing_libexif/

//下载并解压缩libexif-0.6.14
wget https://github.com/libexif/libexif/archive/refs/tags/libexif-0_6_14-release.tar.gz
tar -xzvf libexif-0_6_14-release.tar.gz

//构建并安装libexif
cd libexif-libexif-0_6_14-release/
sudo apt-get install autopoint libtool gettext libpopt-dev
autoreconf -fvi
./configure --enable-shared=no --prefix="$HOME/fuzzing_libexif/install/"
make
make install
```



### 选择接口应用程序

libexif是一个库，接下来我们需要应用程序，它会利用这个库并将被模糊处理。

这个任务将使用exif，键入以下内容下载和解压缩exif0.6.15

```
cd $HOME/fuzzing_libexif
wget https://github.com/libexif/exif/archive/refs/tags/exif-0_6_15-release.tar.gz
tar -xzvf exif-0_6_15-release.tar.gz
```

接下来就是构建并安装exif命令行实用程序：

```
cd exif-exif-0_6_15-release/
autoreconf -fvi
./configure --enable-shared=no --prefix="$HOME/fuzzing_libexif/install/" PKG_CONFIG_PATH=$HOME/fuzzing_libexif/install/lib/pkgconfig
make
make install
```

不过需要注意，我的电脑没安装pkgconf，需要重新通过sudo apt install安装一下



测试exif是否能够正常工作：

![image-20230709093744846](E:/typora_pictures/image-20230709093744846.png)



### 种子语料库的创建

获得一些exif的样本：

```
cd $HOME/fuzzing_libexif
wget https://github.com/ianare/exif-samples/archive/refs/heads/master.zip
unzip master.zip
```

然后测试一下exif:

```
$HOME/fuzzing_libexif/install/bin/exif $HOME/fuzzing_libexif/exif-samples-master/jpg/Canon_40D_photoshop_import.jpg
```

![image-20230709095909286](E:/typora_pictures/image-20230709095909286.png)



### afl-clang-lto

接下来使用afl-clang-lto作为编译器来构建libexif，和上一个实验的步骤类似，把自己编译安装的内容清理掉，重新构建libexif

```
rm -r $HOME/fuzzing_libexif/install
cd $HOME/fuzzing_libexif/libexif-libexif-0_6_14-release/
make clean
export LLVM_CONFIG="llvm-config-11"
CC=afl-clang-lto ./configure --enable-shared=no --prefix="$HOME/fuzzing_libexif/install/"
make
make install
```

```
cd $HOME/fuzzing_libexif/exif-exif-0_6_15-release
make clean
export LLVM_CONFIG="llvm-config-11"
CC=afl-clang-lto 
./configure --enable-shared=no --prefix="$HOME/fuzzing_libexif/install/" PKG_CONFIG_PATH=$HOME/fuzzing_libexif/install/lib/pkgconfig
make
make install
```

不要忘了对libexif和exif分别操作一下（这里建议CC这句和后面./configure不要一起，我这边一一起就错误，分开两次执行就好了）



这里使用afl-clang-lto而不是afl-clang-fast，因为它是一种无碰撞的仪器而且比afl-clang-fast更快。



### 模糊测试

接下来就可以用命令进行模糊测试了：

```
afl-fuzz -i $HOME/fuzzing_libexif/exif-samples-master/jpg/ -o $HOME/fuzzing_libexif/out/ -s 123 -- $HOME/fuzzing_libexif/install/bin/exif @@
```



### 使用Eclipse-CDT 进行调试

首先先安装JAVA-SDK

```
sudo apt install default-jdk
```

然后用已下命令提取：

```
tar -xzvf eclipse-cpp-2021-03-R-linux-gtk-x86_64.tar.gz
```



...未完