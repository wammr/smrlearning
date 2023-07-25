# Fuzzing 101 exercise 6

### 下载并构建目标

首先获取模糊目标，为要模糊的项目创建一个新目录：

```
cd $HOME
mkdir Fuzzing_gimp && cd Fuzzing_gimp
```

现在，安装所需的依赖项：

```
sudo apt-get install build-essential libatk1.0-dev libfontconfig1-dev libcairo2-dev libgudev-1.0-0 libdbus-1-dev libdbus-glib-1-dev libexif-dev libxfixes-dev libgtk2.0-dev python2.7-dev libpango1.0-dev libglib2.0-dev zlib1g-dev intltool libbabl-dev
```

下载并构建 GEGL 0.2（通用图形库）：

```
wget https://download.gimp.org/pub/gegl/0.2/gegl-0.2.0.tar.bz2
tar xvf gegl-0.2.0.tar.bz2 && cd gegl-0.2.0
```

对源代码进行两个小的更改：

```
sed -i 's/CODEC_CAP_TRUNCATED/AV_CODEC_CAP_TRUNCATED/g' ./operations/external/ff-load.c
sed -i 's/CODEC_FLAG_TRUNCATED/AV_CODEC_FLAG_TRUNCATED/g' ./operations/external/ff-load.c
```

构建并安装 Gegl-0.2：

```
./configure --enable-debug --disable-glibtest  --without-vala --without-cairo --without-pango --without-pangocairo --without-gdk-pixbuf --without-lensfun --without-libjpeg --without-libpng --without-librsvg --without-openexr --without-sdl --without-libopenraw --without-jasper --without-graphviz --without-lua --without-libavformat --without-libv4l --without-libspiro --without-exiv2 --without-umfpack
make -j$(nproc)
sudo make install
```

下载并解压缩 GIMP 2.8.16：

```
cd ..
wget https://mirror.klaus-uwe.me/gimp/pub/gimp/v2.8/gimp-2.8.16.tar.bz2
tar xvf gimp-2.8.16.tar.bz2 && cd gimp-2.8.16/
```

使用 **afl-clang-lto** 作为编译器构建 GIMP 的时间：

```
CC=afl-clang-lto CXX=afl-clang-lto++ PKG_CONFIG_PATH=$PKG_CONFIG_PATH:$HOME/Fuzzing_gimp/gegl-0.2.0/ CFLAGS="-fsanitize=address" CXXFLAGS="-fsanitize=address" LDFLAGS="-fsanitize=address" ./configure --disable-gtktest --disable-glibtest --disable-alsatest --disable-nls --without-libtiff --without-libjpeg --without-bzip2 --without-gs --without-libpng --without-libmng --without-libexif --without-aa --without-libxpm --without-webkit --without-librsvg --without-print --without-poppler --without-cairo-pdf --without-gvfs --without-libcurl --without-wmf --without-libjasper --without-alsa --without-gudev --disable-python --enable-gimp-console --without-mac-twain --without-script-fu --without-gudev --without-dbus --disable-mp --without-linux-input --without-xvfb-run --with-gif-compression=none --without-xmc --with-shm=none --enable-debug  --prefix="$HOME/Fuzzing_gimp/gimp-2.8.16/install"
make -j$(nproc)
make install
```



### 持续模式

教程给出了两种方法，这里采用第二种：在函数中插入AFL_LOOP宏

![image-20230720102031144](https://github.com/wammr/smrlearning/blob/master/picture/image-20230720102031144.png)

### 种子语料库创建

将 [SampleInput.xcf](https://github.com/antonio-morales/Fuzzing101/blob/main/Exercise 6/SampleInput.xcf) 文件复制到 AFL 输入文件夹

### 模糊测试时间

由于该漏洞会影响 GIMP 核心，因此可以通过删除不需要的插件来节省一些启动时间：

```
rm ./install/lib/gimp/2.0/plug-ins/*
```

使用以下命令运行模糊程序：

```
ASAN_OPTIONS=detect_leaks=0,abort_on_error=1,symbolize=0 afl-fuzz -i './afl_in' -o './afl_out' -D -t 100 -- ./install/bin/gimp-console-2.8 --verbose -d -f @@
```

目前出现以下错误还没解决...

![image-20230725153700422](https://github.com/wammr/smrlearning/blob/master/picture/image-20230725153700422.png)