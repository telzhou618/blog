# Linux & mac 常用命令

## less

从文件开头查看少量内容命令，命令格式：less [参数] 文件 。

参数：

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

实例：

- 查看日志, -N 显示行号

```shell
less -N /var/a.log
```

## chown

设置文件所有者对文件的权限。

实例：

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

命令格式：

chomd  [ugo+rwx]  file

参数：

- u - 文件拥有者、g - 拥有者同组的其他人、o - 其他用户、a - 所有用户。
- r - 读、w - 写、x - 执行
- -R 递归收取目录和子目录下所有文件权限

实例：

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

## lsof

查看端口占用情况。

实例：

- 查看 占用9876 端口的进程。

```shell
lsof -i tcp:9876

# 执行结果
COMMAND   PID       USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
java    71184 zhougaojun   69u  IPv6 0xd4d49114bb1da0f5      0t0  TCP *:sd (LISTEN)
```

- 另外可以用java自带的jps命令查看java应用进程

```shell
jps

#执行结果
21873 Main
71184 NamesrvStartup
```

## rz

上传本地文件到远程linux服务器。

- centos 输入rz命令唤起文件选择弹框。

```shell
rz # 回车
```

## sz

下载服务文件到本地。

- 下载文件a.txt到本地机器。

```sh
sz a.txt
```

## scp

远程和本地之间传输文件, 加-r 拷贝整个文件夹。

实例：

- 考配置远程服务器tomct.log文件到本地指定位置

```shell
scp root@192.168.1.100:/var/log/tomcat/tomcat.log  /home/myfile/ 

scp -r root@192.168.1.100:/data/ /home/myfile/ # 拷贝文件夹到本地服务
```

- 拷贝本地文件到远程服务器

```shell
scp /home/myfile/test.txt root@192.168.1.100:/data/ # 单文件拷贝

scp /home/myfile/test1.txt test2.cpp test3.bin test.* root@192.168.1.100:/data/ # 多文件拷贝

scp -r /home/myfile/ root@192.168.1.100:/data/ # 拷贝文件夹
```

## find

查找文件。

实例：

- 在home目­录下查­找名为­test的文件。

```shell
find /home -name test
```

- 在home目录下查找以test开头的文件。

```shell
find /home -name "test*"
```

-  在home目­录下查­找大小­大于1­00k的文件

```shell
find /home -size +100k
```

## netstat

显示各种网络信息，如查看TCP连接信息。

参数：

- a (all)显示所有选项，默认不显示LISTEN相关
- t (tcp)仅显示tcp相关选项
- u (udp)仅显示udp相关选项
- n 拒绝显示别名，能显示数字的全部转化成数字。
- l 仅列出有在 Listen (监听) 的服務状态
- p 显示建立相关链接的程序名
- r 显示路由信息，路由表
- e 显示扩展信息，例如uid等
- s 按各个协议进行统计
- c 每隔一个固定时间，执行该netstat命令。

实例：

-  查看所有TCP连接的端口和进程信息。

```shell
netstat -antp
```

## ps

显示进程的状态。

参数：

- a：显示一个终端的所有进程，除会话引线外；
- u：显示进程的归属用户及内存的使用情况；
- x：显示没有控制终端的进程；
- l：长格式显示更加详细的信息；
- e：显示所有进程；

实例：

- 显示进程及进程启动详细情况。

```shell
ps -ef
```

- 线程带CPU占用、内存信息等详细情况。

```shell
ps -aux
```

## ln 

生成链接文件。

实例：

- 生成软连接，也称为符号链接，相当于快捷方式。

```shell
ln -s  source_file target_file
```

- 生成硬链接，也称为实体链接，相当于拷贝。

```shell
ln source_file target_file
```

## brew

mac 软件包管理工具。

- brew uninstall  [包名] 安装包。
- brew install [包名] 卸载包。
- brew search [包名] 查询包。
- brew list   列出已安装的软件。
- brew update 更新brew。 -
- brew outdated 查看可更新的包。 
- brew upgrade(更新所有) 更新所有包。
- brew upgrade [包名] 更新置顶包。
- brew cleanup 清理所有包的旧版本。
- brew cleanup [包名] 清理指定包的旧版本。
- brew cleanup -n 查看可清理的旧版本包，不执行实际操作。
- brew home 用浏览器打开brew的官方网站。
- brew info [包名] 显示软件信息。
- brew deps [包名] 显示包依赖。

