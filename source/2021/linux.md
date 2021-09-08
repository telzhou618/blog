# Linux 常用命令

## less

从文件开头查看少量内容命令，命令格式：less [参数] 文件 。

**实例及说明**

- 查看日志, -N 显示行号

```shell
less -N /var/a.log
```

- /字符串：向下搜索
- ?字符串：向上搜索

- n：重复前一个搜索
- N：反向重复前一个搜索
- 翻一页
  - f：前进一页
  - b：回退一页
- 翻半页：
  - d：前进半页
  - u：后退半页
- 翻一行：
  - e：前进一行
  - y：后退一行

## chown

设置文件所有者对文件的权限。

**实例**

- 给admin用户授于/var/log.a.log文件权限

```sh
chown admin /var/log.a.log
```

- 递归给某个用户授于整个目录权限

```sh
chown -R admin /var/log/
```



## chmod

设置用户对文件的读、写、执行权限。

![img](https://raw.githubusercontent.com/telzhou618/images/main/img/rwx-standard-unix-permission-bits.png)

命令格式：chomd  [ugo+rwx]  file

- u - 文件拥有者、g - 拥有者同组的其他人、o - 其他用户、a - 所有用户。

- r - 读、w - 写、x - 执行
- -R 递归收取目录和子目录下所有文件权限

**实例**

- 将文件 file1.txt 设为所有人皆可读取

```shell
chmod ugo+r file1.txt
# 等价
chmod a+r file1.txt
```

- 所有文件所有人可读。

```shell
chmod -R a+r *
```

- 所有人读写执行

```shell
chmod 777 file
# 等价
chmod a=rwx file
```

