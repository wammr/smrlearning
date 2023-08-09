# Fuzzing 101 exercise 9

### VS 2019

本机上已经安装

### **DynamoRIO 8.0.0**

解压zip内容

![image-20230809113836552](https://github.com/wammr/smrlearning/blob/master/picture//image-20230809113836552.png)

### 下载并构建WinAFL

从官方库下载WinAFL之后，打开**“VS2019的开发人员命令提示符”**并将工作目录更改为WinAFL目录，然后输入以下命令：

```
mkdir build32
cd build32
cmake -G"Visual Studio 16 2019" -A Win32 .. -DDynamoRIO_DIR=E:\fuzz_win_tool\DynamoRIO-Windows-8.0.0-1\cmake
cmake --build . --config Release
```

![image-20230809101154077](https://github.com/wammr/smrlearning/blob/master/picture/image-20230809101154077.png)

![image-20230809101210780](https://github.com/wammr/smrlearning/blob/master/picture/image-20230809101210780.png)

### 下载7zip、准备种子语料库

按步骤来即可

![image-20230809101726052](https://github.com/wammr/smrlearning/blob/master/picture/image-20230809101726052.png)

### 模糊测试

WinAFL 命令行与 AFL++ 略有不同。新选项的简要说明：

- *-coverage_module*：用于记录覆盖范围的模块。支持多个模块标志
- *-target_module*：包含要模糊测试的目标函数的模块
- *-target_offset*：从模块开始模糊的方法的偏移量

检查目标是否在 DynamoRIO 下正常运行：

在目录`E:\fuzz_win_tool\winafl-master\build32\bin\Release`中键入以下命令：

`E:\fuzz_win_tool\DynamoRIO-Windows-8.0.0-1\bin32\drrun.exe -c winafl.dll -debug -target_module 7z.exe -target_offset 0x02F3B3 -fuzz_iterations 10 -nargs 2 -- "E:\fuzz_win_tool\7-Zip\7z.exe" l E:\fuzz_win_tool\test.img`

![image-20230809110348456](https://github.com/wammr/smrlearning/blob/master/picture/image-20230809110348456.png)

最后，使用以下命令运行模糊器：

在目录`E:\fuzz_win_tool\winafl-master\build32\bin\Release`中输入：

`afl-fuzz.exe -i E:\fuzz_win_tool\afl_in -o E:\fuzz_win_tool\afl_out -t 2000 -D E:\fuzz_win_tool\DynamoRIO-Windows-8.0.0-1\bin32 -- -coverage_module 7z.exe -coverage_module 7z.dll -target_module 7z.exe -target_offset 0x02F3B3 -nargs 2 -- "E:\fuzz_win_tool\7-Zip\7z.exe" e -y @@`

看到WinAFL正在运行：

![image-20230809105806960](https://github.com/wammr/smrlearning/blob/master/picture/image-20230809105806960.png)