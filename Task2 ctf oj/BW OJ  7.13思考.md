# BW OJ 7.13思考

## Web

### Calculator

题目要求在1.5秒之内就计算中算式中的结果并提交

查看网页源代码发现算式可以在源代码中找到，那么接下来可以用Python把算式先爬下来（用正则表达式的方式），计算好之后再GET上去，这个过程中要记得加上cookie相关信息

解题脚本如下：

```python
import re
import requests

# 定义目标网页的URL
url = "http://vps1.blue-whale.me:23331/calculator/"

# 发送HTTP GET请求获取网页内容
response = requests.get(url)
html_content = response.text

# 使用正则表达式提取算式
pattern = r'<span id="exp">(.*?) = </span>'
equation = re.search(pattern, html_content).group(1)

# 输出算式
print("Equation:", equation)

# 计算算式结果
result = eval(equation)

print(result)


cookie = (requests.utils.dict_from_cookiejar(response.cookies))['PHPSESSID']
headers = {
     'Cookie': 'PHPSESSID='+ cookie
          }

url = ("http://vps1.blue-whale.me:23331/calculator/?answer="+str(result))
r = requests.get(url,headers = headers)
print(r.text)

```



### Rapid Typing

题目要求在2.5秒内将很长的验证码输入

查看网页源代码发现，图片使用svg+xml，也就是说其实验证码的字符就在网页上显示了已经。不过首先发现验证码用base64加密了，那就先解密之后再获取。接下来通过x坐标的顺序将验证码输入再传给原网页就好了~

```python
import requests
import base64
import re

url = 'http://vps1.blue-whale.me:23331/captcha/'
sess = requests.Session()
urlcontent = sess.get(url).content.decode('utf-8')


picture = re.findall(r'<img src="data:image/svg\+xml;base64,(.*?)" />',urlcontent)
#print(picture) 

svg = base64.b64decode(picture[0]) #picture是列表形式，只有一个元素，所以要加[0]
#print(svg)

#print(re.findall(
        r'<text x="(\d+)" y="\d+" style="fill: rgb\(\d+, \d+, \d+\);">(\w)</text>', svg.decode('utf-8')
    ))
captchas = {
    int(item[0]): item[1] for item in re.findall(
        r'<text x="(\d+)" y="\d+" style="fill: rgb\(\d+, \d+, \d+\);">(\w)</text>', svg.decode('utf-8')
    )
} #使用字典推导式，将x坐标的值强制类型转换为数字，并将其作为字典中的键值

#print(sorted(captchas.keys()))
captcha = ''.join([captchas[x] for x in sorted(captchas.keys())])
#print(captcha)
print(sess.get(url + '?code=' + captcha).content)
```



### Basic PHP

题目源码如下：

```PHP
require_once('flag.php');

if (isset($_GET['name']) and isset($_GET['password']) && isset($_GET['test'])){
    // ========== Stage 1 ========== 
    $test=$_GET['test']; 
    $test=md5($test); 

    if($test=='0') { 
        print 'You passed stage 1.<br />';
    }
    else{
        print "Game over at stage 1."; 
        exit();
    }
    
    // ========== Stage 2 ========== 
    if ($_GET['name'] == $_GET['password']){
        print 'Your password can not be your name.';
        exit();
    }
    else if (sha1($_GET['name']) === sha1($_GET['password'])){
        print 'You passed stage 2.<br />';
        print 'Flag: '.$flag;
    }
    else{
        print 'Invalid password';
        exit();
    }

}
echo '<hr />';
show_source(__FILE__);
?>
</body>
</html>
```



一共有两关，**第一关**要求test**经过md5后的值为0**，md5($test)=='0'，上网查了一下MD5加密后=='0'的总结，如下：

> ```
> 240610708，aabg7XSs，aabC9RqS
> s878926199a
> 0e545993274517709034328855841020 //0e开头的会被识别成科学计数法，为0×10^x，结果均为0
> s155964671a
> 0e342768416822451524974117254469
> s214587387a
> 0e848240448830537924465865611904
> s214587387a
> 0e848240448830537924465865611904
> s878926199a
> 0e545993274517709034328855841020
> s1091221200a
> 0e940624217856561557816327384675
> s1885207154a
> 0e509367213418206700842008763514
> s1502113478a
> 0e861580163291561247404381396064
> s1885207154a
> 0e509367213418206700842008763514
> s1836677006a
> 0e481036490867661113260034900752
> s155964671a
> 0e342768416822451524974117254469
> s1184209335a
> 0e072485820392773389523109082030
> s1665632922a
> 0e731198061491163073197128363787
> s1502113478a
> 0e861580163291561247404381396064
> s1836677006a
> 0e481036490867661113260034900752
> s1091221200a
> 0e940624217856561557816327384675
> s155964671a
> 0e342768416822451524974117254469
> s1502113478a
> 0e861580163291561247404381396064
> s155964671a
> 0e342768416822451524974117254469
> s1665632922a
> 0e731198061491163073197128363787
> s155964671a
> 0e342768416822451524974117254469
> s1091221200a
> 0e940624217856561557816327384675
> s1836677006a
> 0e481036490867661113260034900752
> s1885207154a
> 0e509367213418206700842008763514
> s532378020a
> 0e220463095855511507588041205815
> s878926199a
> 0e545993274517709034328855841020
> s1091221200a
> 0e940624217856561557816327384675
> s214587387a
> 0e848240448830537924465865611904
> s1502113478a
> 0e861580163291561247404381396064
> s1091221200a
> 0e940624217856561557816327384675
> s1665632922a
> 0e731198061491163073197128363787
> s1885207154a
> 0e509367213418206700842008763514
> s1836677006a
> 0e481036490867661113260034900752
> s1665632922a
> 0e731198061491163073197128363787
> s878926199a
> 0e545993274517709034328855841020
> ```

链接：[(6条消息) MD5加密后==’0’ 总结_加密后为==_江judge的博客-CSDN博客](https://blog.csdn.net/qq_41281571/article/details/81292786)



选择test为240610708，先测试一下    <u>/?test=240610708&name=0&password=0</u>看第一阶段是否成功，发现提示"You passed stage 1" ，说明通过第一阶段

**第二关**挑战要求name和password不能相等，且name和password的**SHA1结果计算后强等**，这里选择用**数组绕过强比较**。

传入的不是字符串而是数组，不但sha1()函数不会报错，结果还会返回为NULL，在强比较里NULL=NULL为true即可以绕过。

测试一下  <u>/?test=240610708&name[]=1&password[]=2</u>

提示成功通过了两关并拿到了flag~~



![image-20230713092404468](E:\typora_pictures\image-20230713092404468.png)



关于SHA-1与MD5碰撞绕过可能情况，见以下链接总结：

[SHA-1与MD5绕过 - king_kb - 博客园 (cnblogs.com)](https://www.cnblogs.com/king-kb/p/15531476.html)



## Misc

### NTFS

下载工具ntfsstreamseditor，然后用工具扫描flag文件夹，导出后，得到flag。

本题主要是对ntfs数据流的理解和工具的使用。

工具下载链接：[ntfsstreamseditor.7z_免费高速下载|百度网盘-分享无限制 (baidu.com)](https://pan.baidu.com/s/1O504jNFr48bUFdAFM2wnhg)



## Reserve

### reserve sign in

将rev导入ida，查看main函数：

```C
int64 __fastcall main(int a1, char **a2, char **a3)
{
  __int64 result; // rax
  char s[40]; // [rsp+0h] [rbp-30h] BYREF
  unsigned __int64 v5; // [rsp+28h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  printf("Please input your flag:");
  __isoc99_scanf("%32s", s);
  if ( strlen(s) == 32 )
  {
    if ( (unsigned int)sub_400686(s) )
      puts("Right!");
    else
      puts("Wrong!");
    result = 0LL;
  }
  else
  {
    puts("Wrong!");
    result = 0LL;
  }
  return result;
}
```

s的长度要是32，同时要满足sub_400686的要求，查看这个函数：

```C
int64 __fastcall sub_400686(__int64 a1)
{
  int i; // [rsp+Ch] [rbp-Ch]

  for ( i = 0; i <= 31; ++i )
  {
    if ( (char)(*(_BYTE *)(i + a1) ^ byte_400818[i]) != i )
      return 0LL;
  }
  return 1LL;
}
```

说明如果对于字符串中的每个字符，将其与对应位置的`byte_400818`字节数组中的值进行异或操作后得到的结果等于该字符在字符串中的索引值，那么`sub_400686`函数会返回1，否则返回0

查看byte_400818数组的内容：

```
0x66, 0x6D, 0x63, 0x64, 0x7F, 0x3C, 0x36, 0x72, 0x57, 0x42, 0x64, 0x3B, 0x7B, 0x52, 0x7C, 0x3C, 0x66, 0x54, 0x60, 0x60, 0x27, 0x4A, 0x49, 0x7F, 0x71, 0x58, 0x52, 0x72, 0x7D, 0x75, 0x2A, 0x62
```

写一段python代码找到目标字符串：

```python
b = [0x66, 0x6D, 0x63, 0x64, 0x7F, 0x3C, 0x36, 0x72, 0x57, 0x42, 0x64,
     0x3B, 0x7B, 0x52, 0x7C, 0x3C, 0x66, 0x54, 0x60, 0x60, 0x27,
     0x4A, 0x49, 0x7F, 0x71, 0x58, 0x52, 0x72, 0x7D, 0x75, 0x2A, 0x62]
a = []

for i in range(32):
    a.append(chr(b[i] ^ i))

print(a)
result = ''.join(a)
print("Result:", result)
```

运行程序即可得到flag ~