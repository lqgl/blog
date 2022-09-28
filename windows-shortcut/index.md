# Windows 与 Linux 基础使用


## Windows 常用快捷键

1. Ctrl + C : 复制
2. Ctrl + V : 粘贴
3. Ctrl +  X : 剪切
4. Ctrl + Z : 撤销
5. Alt + F4 : 关闭窗口
6. Ctrl + Shift + Esc : 任务管理器
7. Win + R : 运行
8. Win + E  : 资源管理器

---

## 常用 Dos 命令

```bash
#正斜杠与反斜杠  / 和 \  前正后反
#切换盘符
dir #查看当前目录下的所有文件 
cd #切换目录 cd change directory
cd .. #返回上一级目录
cd /d E:\a #/d参数用于跨盘符切换
cls   #清理屏幕(clear screen)
exit  #关闭终端
ipconfig #查看ip信息  查看ip的详细信息 ipconfig \all
#打开应用
mspaint #打开画图工具
calc #打开计算器
notepad #打开记事本
#ping 命令
ping www.baidu.com

#文件操作
md 目录名 #创建文件夹
cd> 文件名 #创建文件
del 文件名 #删除文件
rd 目录名 #移除文件夹
```

---

## 常用 Linux 命令
```bash
##### 显示和修改文本文件 #####
$ echo "Hello World!" > hello-world.txt
$ cat hello-world.txt
Hello World! 
$ echo "Go is the best!" >> hello-world.txt
$ cat hello-world.txt
Hello World! 
Go is the best!

##### 搜索文件和在文件内搜索 #####
$ find /etc -name hosts
/etc/hosts
/etc/avahi/hosts
$ find /etc -name "hosts*"
/etc/hosts
/etc/hosts.allow
/etc/hosts.deny
/etc/avahi/hosts

#####  管理进程 #####
$ ps aux  # ps aux命令列出当前正在运行的进程，并显示他们的PID。
# 比如查询 docker
$ ps aux | grep docker
root  115   0.0  0.0  4997956  168   ??  Ss   13Sep22  0:00.57 /Library/PrivilegedHelperTools/com.docker.vmnetd
# 向进程发送信号 kill -signal pid
# 通过kill -l可以看到所有的信号变量
$ kill -l
# 常用的几个信号, TERM(请求优雅的终止)、KILL(强制杀死)。
$ kill -TERM pid

# 命令后台运行 命令后面跟 &
$ ping www.google.com & 

# 显示在后台运行的进程列表
$ jobs

# 将恢复一个工作到前台
$ fg %job-number # job-number(对于前台)

# 将一个正在前台执行的命令放到后台,暂停流程并恢复对命令行的控制
Control+Z组合键

# 重新启动该流程在后台
$ bg %job-number # (用于后台)。

##### 管理权限  ######
# 三类 users: u:user g:group o:other 三种权限：r:read w:write x:execute 八进制表示:421
# changes the owner of the file
$ chown user file 
# alters the owner group
$ chgrp group file 
# changes the permissions for the file
$ chmod rights file 
# example
$ chmod 755 hello-world.txt

##### 获取系统信息和日志 #####
# 显示内存信息
$ free
# 报告可用的文件系统中挂载的每个磁盘上的磁盘空间
$ df -h # df means disk free -h human readable
# 显示运行会话的用户的身份，以及他们所属的组的列表
# 由于对某些文件或设备的访问可能只限于小组成员，检查可用的小组成员资格可能很有用
$ id
# 返回一行记录内核名称(Linux)、主机名、内核版本、内核版本、机器类型(一个体系结构字符串，如x86_64)、以及操作系统的名称(GNU/Linux)
# 此命令的输出通常应该包含在错误报告中，因为它清楚地定义了使用的内核和硬件平台上运行。
$ uname -a 
# 检索内核日志
# 通常你需要查阅日志来了解你的计算机上发生了什么。比如一个新的USB设备被插入。一个失败的硬盘操作，或者启动时的初始硬件检测
$ dmesg

##### 发现硬件 #####
# lists PCI devices
$ lspci
# lists USB devices
$ lsusb
# lists PCMCIA cards.
$ lspcmcia 
```
