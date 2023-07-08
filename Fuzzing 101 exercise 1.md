# Fuzzing 101 exercise 1

### AFL原理

实验开始前先记录AFL的原理，如下：

AFL（American Fuzzy Lop）是一种基于模糊测试（fuzz testing）的软件测试工具，用于发现软件中的漏洞和错误。它的原理可以概括为以下几个步骤：

1. 选择输入样本：AFL使用一个或多个输入样本（通常是测试用例）作为起点。这些样本可以是符合预期输入的有效数据，也可以是随机生成的数据。
2. 插桩目标程序：AFL通过插桩（instrumentation）修改目标程序的二进制代码，以便能够监控程序执行时的分支情况。插桩的目的是为了在运行过程中收集关于程序行为的信息。
3. 生成测试输入：AFL使用一种称为“字典”的技术，基于输入样本和观察到的程序分支情况生成新的测试输入。字典是一个表示输入空间的数据结构，通过变异和组合现有的输入生成新的输入，以尽可能覆盖不同的程序路径。
4. 执行目标程序：AFL将生成的测试输入提供给目标程序，并监控程序执行时的分支情况。每次执行时，AFL会记录经过的分支路径，以此来衡量测试输入的覆盖率。
5. 调整测试输入：根据目标程序执行时的分支情况，AFL会调整测试输入，并在下一次执行中使用更有可能导致新的分支的输入。这样，AFL不断迭代地生成和调整输入，以尽量发现新的程序路径和漏洞。
6. 持续监控：AFL通过持续监控目标程序的运行状态，自动进行测试输入的生成和调整，直到达到预设的停止条件（例如，时间限制或覆盖率不再增加）。

通过这种方式，AFL能够在大量输入中探索不同的程序路径，找到导致程序崩溃、内存错误、逻辑错误等漏洞的测试输入。它利用模糊测试的思想，尽可能地覆盖程序的不同执行路径，提高软件的安全性和稳定性。

### 下载Xpdf并测试

首先创建一个新目录fuzzing_xpdf

![image-20230708100403778](E:\typora_pictures\image-20230708100403778.png)



安装一些必备工具（make gcc等），我这里build-essential已经是最新版本了

![image-20230708100533610](E:\typora_pictures\image-20230708100533610.png)



下载Xpdf 3.02并解压缩

![image-20230708100945820](E:\typora_pictures\image-20230708100945820.png)



构建xpdf

```
cd xpdf-3.02
sudo apt update && sudo apt install -y build-essential gcc
./configure --prefix="$HOME/fuzzing_xpdf/install/"
make
make install
```

![image-20230708102205715](E:\typora_pictures\image-20230708102205715.png)



测试一下xpdf的功能，首先先下载一些用到的PDF示例，将PDF示例放在fuzzing_xpdf中的pdf_examples中：

![image-20230708102634245](E:\typora_pictures\image-20230708102634245.png)



测试一下pdfinfo：

```
$HOME/fuzzing_xpdf/install/bin/pdfinfo -box -meta $HOME/fuzzing_xpdf/pdf_examples/helloworld.pdf
```

![image-20230708102822790](E:\typora_pictures\image-20230708102822790.png)



### 安装并使用AFL++

由于教程中给的两种安装方法目前都没法用，所以我就选择直接sudo apt install AFL++了。输入afl-fuzz可以看到安装成功

![image-20230708104232006](E:\typora_pictures\image-20230708104232006.png)



安装好AFL之后，开始正式使用。首先，将清理所有以前编译好的对象文件和可执行文件：

```
rm -r $HOME/fuzzing_xpdf/install  #刚才在这安装的xpdf，都删了
cd $HOME/fuzzing_xpdf/xpdf-3.02/  
make clean #刚才在这构建的xpdf，这条命令是清理构建过程中生成的临时文件和目标文件，以便准备重新构建项目时保持干净的状态
```



接下来使用afl-clang-fast编译器来构建xpdf，不过需要注意的是，和教程不同，要把CC和CXX改成我自己的路径，如果不知道在哪可以用which搜索一下

![image-20230708105610478](E:\typora_pictures\image-20230708105610478.png)

./configure的设置和一开始构建xpdf的时候是一样的（相当于让afl参与重新构建的过程）

之后就是make 和make install



开始运行模糊程序：

使用的命令如下

```
afl-fuzz -i $HOME/fuzzing_xpdf/pdf_examples/ -o $HOME/fuzzing_xpdf/out/ -s 123 -- $HOME/fuzzing_xpdf/install/bin/pdftotext @@ $HOME/fuzzing_xpdf/output
```

每个选项的简要说明：

- *-i* 表示我们必须放置输入案例的目录（又名文件示例）
- *-o* 表示 AFL++ 将存储突变文件的目录
- *-s* 表示要使用的静态随机种子
- *@@*是占位符目标的命令行，AFL 将用每个输入文件名替换该命令行

模糊器将为每个不同的输入文件运行命令：

```
$HOME/fuzzing_xpdf/install/bin/pdftotext <input-file-name> $HOME/fuzzing_xpdf/output
```

几分钟之后可以看到出现崩溃：

![image-20230708110315562](E:\typora_pictures\image-20230708110315562.png)



接下来在目录中找到与崩溃相对应的文件：

![image-20230708110831236](E:/typora_pictures/image-20230708110831236.png)

将这个文件作为输入传递给pdftotext，来重现crash

```
#来到fuzzing_xpdf目录下
install/bin/pdftotext out/default/crashes/id:000000,sig:11,src:000877,time:97675,execs:77278,op:havoc,rep:8 output
```

![image-20230708111733760](E:/typora_pictures/image-20230708111733760.png)

可以看到分段错误导致程序崩溃



接下来使用gdb找出程序在此输入时崩溃的原因

首先使用调试信息重建xpdf以获得符号堆栈跟踪

```
rm -r $HOME/fuzzing_xpdf/install
cd $HOME/fuzzing_xpdf/xpdf-3.02/
make clean
CFLAGS="-g -O0" CXXFLAGS="-g -O0" ./configure --prefix="$HOME/fuzzing_xpdf/install/"
make
make install
```

过程跟前面构建的过程相似就不截图了



开始运行GDB

![image-20230708145042608](E:/typora_pictures/image-20230708145042608.png)



键入run，看到下面输出：

![image-20230708145124210](E:/typora_pictures/image-20230708145124210.png)



键入bt获取回溯

![image-20230708145543517](E:/typora_pictures/image-20230708145543517.png)

可以看到许多“Parser::getObj”方法的调用，查看CVE-2019-13288可以知道：在 Xpdf 4.01.01 中，Parser.cc 中的 Parser：：getObj（） 函数可能会通过构建的文件导致无限递归。远程攻击者可利用此漏洞进行 DoS 攻击。



查看xpdf4.02的源码可以看到限制了递归的最大次数从而修改了这个漏洞

![image-20230708155318056](E:/typora_pictures/image-20230708155318056.png)



### 心得体会

通过本我理解了AFL的原理、以及怎么使用AFL找到崩溃。本实验由于一开始做的时候没做好记录于是又做了第二遍...麻烦了点但是印象深刻一些。虽然预计时间2h但实际做的时间要稍微长一些，接下来还是不断提高自己的熟练程度、学习能力来达到基本要求。