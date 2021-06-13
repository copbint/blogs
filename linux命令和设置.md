[toc]



# awk

## 指定分隔符

 awk -F '/bin/java' '{print $1}'`

## 找出文件的最长行和最短行

```bash
awk '(NR==1||length(min)>length()){min=$0}END{print min}'  data.txt #最短行
awk '{if (length(max)<length()) max=$0}END{print max}'  data.txt # 最长行
awk '{if (length(max)<length()) max=$0}END{print length(max)}'  data.txt  # 长度
```



# sed



# grep

grep linuxgrep使用的是正则表达式 如grep ".*aaa"表示以a结束的字符

grep同时过滤多个关键字 egrep "a|b" grep -E "a|b"



# 网络相关的命令

* 查看端口占用情况

```bash
netstat -npl | grep 9000
```

windows上查看：

```bash
netstat -nao | findstr 808
```

* 发送UDP包

```bash
echo "hello" > /dev/udp/192.168.1.81/5060
```

* 远程检查某台主机的某个tcp端口是否打开

```bash
telnet ip port
wget ip:port
```

* 重启网络服务

```bash
/etc/init.d/network restart
```

* 查看节点加入的多播组

```bash
netstat -gn
```

* 清空arp缓存

arp -d

```bash
arp -n | awk '/^[1-9]/{print "arp -d " $1}' | bash -x
```

* 局域网内ip查主机

```bash
nbtstat -A 10.40.42.148
ping -a 10.40.42.148

```



# ssh相关命令和设置

* 为机器添加ssh连接用的公钥

1.修改/etc/ssh/sshd_config（权限和公钥路径）
2.在用户主目录下添加.ssh文件夹，权限600
3.在.ssh文件夹下添加authorized_keys，文件权限600

* 设置连接超时时间

vim /etc/profile
设置TMOUT=0
sed-i '/TMOUT=/aTMOUT=0' /etc/profile

* 消除history中连续的重复命令

```bash
export HISTCONTROL=ignoredups 

```

* 重启sshd服务

```bash
/etc/init.d/ssh restart
/etc/init.d/sshd restart

```



# 时间相关命令

* 设置linux机器硬件时间

```bash
hwclock --show #显示时间
hwclock --systohc #将系统时间同步到硬件

```

* 同步时间

```bash
ntpdate ip #客户端
ntp #server端

```



# 文件相关操作

* 解压

```bash
xz -d linux-3.1-rc4.tar.xz #解压.xz文件
tar -xf linux-3.1-rc4.tar # 解压.tar文件
tar -zxf app.tar.gz 
tar -zxf jre-8u144-linux-x64.tar.gz -C /usr/lib/java #tar解压到指定文件夹
tar -cf hahah.tar haha.txt #打包成tar

```

* tar压缩文件

tar -zcf hahah.tar.gz haha.txt

tar 参数解释
-c: 建立压缩档案
-x：解压
-t：查看内容
-r：向压缩归档文件末尾追加文件
-u：更新原压缩包中的文件
这五个是独立的命令，压缩解压都要用到其中一个，可以和别的命令连用但只能用其中一个。下面的参数是根据需要在压缩或解压档案时可选的。
-z：有gzip属性的
-j：有bz2属性的
-Z：有compress属性的
-v：显示所有过程
-O：将文件解开到标准输出

下面的参数-f是必须的
-f: 使用档案名字，切记，这个参数是最后一个参数，后面只能接档案名。

* 递归删除

```bash
find ./ -name "*.o"  | xargs rm -r

```

* 递归修改文件用户和用户组

```bash
chown -R ossuser:ossgroup .

```

* 查找文件

```bash
find / -name cacerts 2>/dev/null #如果搜索某个目录是软件链接时，一定要在目录后面加

```

* 按时间查找

find ./ -mtime -2 | grep "\.c$" 最近两天内修改过的.c文件

find ./ -mtime +2 | grep "\.c$" 两天前修改过的.c文件

find ./ -mmin -60 | grep "\.c$" 60分钟内修改过的.c文件

* 按大小查找

find ./ -size +100M 大于100M的文件

find ./ -size +1G 大于1G的文件

更多用法：

http://blog.51cto.com/zhaizhenxing/134885

* 查找文件内容

find ./ | xargs grep "xxxx" 

* cp 时保留文件属性

```bash
cp -p from to

```

* 修改文件时间属性

```bash
touch haha.txt -t 201811112222  #将文件的修改时间设置为2018/11/11 22:22  
touch haha.txt -r hehe.txt  #将haha.txt的时间属性设置为和hehe.txt一样

```

* 生成一个指定大小的文件

```bash
dd if=/dev/zero of=lvs_sh_journal_root.jou.test bs=1024 count=10239

```

bs ：block size

count: block count

# 查看操作系统信息

查看内核版本（同时可以查看编译内核使用的gcc版本）
cat /proc/version

查看操作系统版本

cat /etc/os-release



# 内核相关

查看某个模块是否加载

```bash
lsmod |grep $modName

```

反汇编某个目录下所有ko文件

```bash
objdump -D xx.ko > xx.dis

```

# 系统命令

查看系统启动时间：

who -b

查看系统运行时间：

uptime

重启机器

reboot

# 其他

* bash命令行提示符的格式

ubuntu

PS1='\u@\h:\w \A \$ '
SUSE
PS1="[\e[32;1m\u@\h \w \A]\\$ "

* linux系统自动补齐忽略大小写

```bash
echo "set completion-ignore-case on">>~/.inputrc

```

当前用户重新登录后生效。
(没有这个文件也没有关系，创建就可以)

强制生效版本：

```bash
bind 'set completion-ignore-case on'

```

* You have new mail in /var/mail/root

```bash
echo "unset MAILCHECK" >> /etc/profile;source /etc/profile

```

* top命令按内存和cpu排序

P，按CPU大小排序

M ，按内存大小排序



* rpm

rpm -ivh your-package.rpm

查询已经安装的软件

rpm -q -a | grep jpr

删除已经安装的软件（软件名是上一步查询的结果）

rpm -e jprofiler-9.0.2-1.i386



* 进程相关

查看某进程的所有环境变量

cat /proc/108505/environ |tr '\0' '\n'

查看进程启动时间

ps -p 130272 -o lstart

* echo不换行

```bash
echo -e "haha\c"

```

