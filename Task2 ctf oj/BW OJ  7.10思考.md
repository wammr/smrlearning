# BW OJ  7.10思考

今日分数：148

## crypto

### What's RSA?

使用openssl 工具，命令为

```
openssl rsautl -decrypt -in flag.enc(加密文件) -inkey private.pem(私钥文件)
```



### RSA2

先按照教程装好rsactftool，教程参考：[(4条消息) kali Linux下RsaCtfTool的安装以及使用_一小小学徒的博客-CSDN博客](https://blog.csdn.net/m0_74039565/article/details/128633654?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-128633654-blog-109124650.235^v38^pc_relevant_sort&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~CTRLIST~Rate-1-128633654-blog-109124650.235^v38^pc_relevant_sort&utm_relevant_index=2)

命令为：

```
python RsaCtfTool.py --publickey 公钥文件 --uncipherfile 加密的文件
```



## Misc

### Invisible Flag

下载图片后发现少半截，用010 editor修改图片高度与宽度一致，可以得到flag



### LSB

下载图像后使用stegsolve打开，查看红色最低通道，存在二维码，扫码得到flag



### docx

下载file.docx文件，将文件改为zip格式,得到flag



### Birthday

下载flag.zip，发现压缩包加密，提示是一串代表生日的数字，且又是90后，设置密码格式为：19xxxxxx，起始密码为19000000，暴力破解得到生日，解压得到Flag



### LSB2

安装zsteg软件，运行zsteg -a secret.png 即可以得到flag



## Web

### Welcome to web

由exercise 0的提示，使用hackbar，在网址后面加?key=areyousure，可以得到flag