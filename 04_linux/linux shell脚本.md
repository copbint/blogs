[toc]



# shell 内置功能example

## for循环

* 方式1

```bash
sum=0
for((i=0;i<=10;i++))
do
  sum=$((sum + i))
done

echo $sum
```

**这个语法sh不支持，仅bash支持。**



* 方式2

```bash
sum=0
for i in $(seq 1 10)
do
  sum=$((sum + i))
done

echo $sum
```

sh和bash都支持





## 获取命令输出的每一行，并逐行处理

* 方式1：管道符方式

```bash
echo '1 a
2 b
3 c
4
5
6
7
8
9' >a.log
sum=0
cat a.log | while read line
do
  number=$(echo $line | awk '{print $1}')
  echo $number
	sum=$((sum+number))
done
echo sum: $sum

```

这种方式，**在while 循环中对变量赋值，在循环外无效**。所以最后输出的sum=0

可以参考：

[Shell： Shell中while循环的陷阱, 变量实效, 无法赋值变量](https://justcode.ikeepstudying.com/2018/02/shell：-shell中while循环的陷阱-变量实效-无法赋值变量/)

* 方式2：重定向

```bash
sum=0
while read line
do
  number=$(echo $line | awk '{print $1}')
  echo $number
	sum=$((sum+number))
done <<<'1 a
2 b
3 c
4
5
6
7
8
9'
echo sum: $sum
```

这种方式bash支持，sh不支持。

* 方式3： 重定向文件

```bash
sum=0
while read line
do
  number=$(echo $line | awk '{print $1}')
  echo $number
	sum=$((sum+number))
done <a.log

echo sum: $sum
```

* 方式4： 重定向命令输出

```bash
sum=0
while read line
do
  number=$(echo $line | awk '{print $1}')
  echo $number
	sum=$((sum+number))
done < <(cat a.log)

echo sum: $sum
```

这种方式bash支持，sh不支持。

所以，也可利用这种方式从变量中读取

```bash
nums='1 a
2 b
3 c
4
5
6
7
8
9'
sum=0
while read line
do
  number=$(echo $line | awk '{print $1}')
  echo $number
	sum=$((sum+number))
done < <(echo "$nums") #变量周围的双引号不能少
echo sum: $sum

```



读取文件每一行并忽略空行

```bash
grep -v "^[ ]*$" routes | while read line
do
    echo "line: $line"
done
```



## 获取命令输出的内容，按空格分隔部分，分别处理

```bash
for word in `sysctl -a | grep "\\.forwarding"`
do 
	echo "$word" 
done

```

上面这种写法，如果输出中有*号，即使加上双引号也没用，还是会被转义

只想到一个规避的办法，将*号去掉：

```bash
for word in `sysctl -a | grep "\\.forwarding" | sed "s/*//g" `
do 
	echo "$word" 
done

```



可以修改IFS，达到按行分割的效果：

```bash
IFS=$'\n' #设置换行符为分隔符。默认空格也会被当做分隔符，无法按行处理

```



## while死循环

```bash
sum=0
while true
do
  sum=$((sum + 1))
  echo $sum
  sleep 0.5
done

```



## if  语句

```bash
a=$1
if [ "$a" -gt 100 ];then
  echo  "too big!"
elif [ "$a" -lt 10 ]; then
  echo "too small!"
else
  echo "this ok: $a"
fi

```

注意点：

* 注意”[“的后面，"]"的前面需要空格
* 变量要用双引号引用起来，应对变量为空的情况

### if的与/或 表达

```bash
if [ "$useProxy" = yes ] || [ "$useProxy" = y ];then

fi

if [ "$useProxy" != yes ] && [ "$useProxy" != y ];then

fi

```

### if的简洁写法

```bash
[ "$useProxy" != yes ] && [ "$useProxy" != y ] && echo "do not use proxy!"

```



## 算术运算

* expr:

```bash
val=$(expr 2 + 2)

```

只支持整数运算

* shell自带功能

```bash
a=2
b=3
echo $((a*b))

```

只支持整数运算

* bc

```bash
a=2
b=3
echo $(echo "scale=2; $a/$b" | bc)

```



## shell变量设置默认值

```bash
#!/bin/bash

b="default"

# 当变量a为null时
echo ${a-$b}
# default

#当变量a为null或为空字符串时
a=""
echo ${a:-$b}  
# default


```



## shell正则

示例：

```bash
#!/bin/sh
regex=$2
if [[ "$1" =~ $regex ]]; then
 echo matched!
else
 echo No!
fi

```

先记几个重要的坑：

1. 网上有文章提到只有bash才支持正则，但是不知道为啥我写的#!/bin/sh也正常支持

2. 在[[  ]]内部的$regex周围一定不能加引号。如下两种写法都是不对的：

if [[ "$1" =~ "$regex" ]];

if [[ "$1" =~ "[0-9]{3}" ]];

参考： http://blog.zengrong.net/post/1563.html

下面写几个用法示例，方便以后参考：

将上面那段脚本保存为test.sh，然后开始实验：

普通正则用法

SIG-OM-Global-Base01:~/tempPeng # ./test.sh aaa [a-z]{3}

matched!

是一种比较松散的正则检验，以下也是可以通过的：

SIG-OM-Global-Base01:~/tempPeng # ./test.sh aabbcc bb

matched!

但是反过来就不：

SIG-OM-Global-Base01:~/tempPeng # ./test.sh bb aabbcc

No!

## 白名单检验参数合法性

```bash
echo "$*" | grep -q -E "^[ 0-9a-zA-Z./:_@,-]*$"  # 如果有中划线，需要放在末尾
[ $? -ne 0 ] && echo "illega para" && exit 1

```

（但是这种校验方式没办法校验住换行符）

参考：

https://stackoverflow.com/questions/4542732/how-do-i-negate-a-test-with-regular-expressions-in-a-bash-script

https://stackoverflow.com/questions/28256178/how-can-i-match-spaces-with-a-regexp-in-bash

校验换行符

```bash
[ "$(echo "$*" | wc -l)" -ne 1 ]  && echo "illega para" && exit 1

```



## shell脚本写UT

```bash
#!/bin/bash
#script name:paraCheck.sh
para=$*
echo "$para" | grep -q -E '^[ 0-9a-zA-Z./:_-]*$'
result=$?
if [ "$result" -ne 0 ]
then
        echo "parameter error!"
        exit 1
fi

exit 0

```

对应的UT脚本：

```bash
#!/bin/bash
#script name:paraCheckTest.sh
checkTrue()
{
    result=$1
    lineNumber=$2
    lineNumber=$((lineNumber - 1))
    if [ "$result" -ne 0 ]
    then
        echo "error. Test failed. expect 0 but return $result."
        echo "test case:"
        awk 'NR=="'$lineNumber'"' $0
        exit 1
    fi
}

checkFalse()
{
    result=$1
    lineNumber=$2
    lineNumber=$((lineNumber - 1))
    if [ "$result" -eq 0 ]
    then
        echo "error. Test failed. expect 1 but return 0."
        echo "test case:"
        awk 'NR=="'$lineNumber'"' $0
        exit 1
    fi
}

sh paraCheck.sh hahah kdsjflkd
checkTrue $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 >/dev/null
checkTrue $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1  >/dev/null
checkTrue $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 fe80::a00:27ff:fe99:7888 >/dev/null
checkTrue $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 fe80::a00:27ff:fe99:7888 "/var/log/" >/dev/null
checkTrue $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 fe80::a00:27ff:fe99:7888 "/var/log/" "_-" >/dev/null
checkTrue $? $LINENO 

 

sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 fe80::a00:27ff:fe99:7888 ";" >/dev/null
checkFalse $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103  ";"  ::1 fe80::a00:27ff:fe99:7888>/dev/null
checkFalse $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 fe80::a00:27ff:fe99:7888 "#" >/dev/null
checkFalse $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 fe80::a00:27ff:fe99:7888 "&&" >/dev/null
checkFalse $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 fe80::a00:27ff:fe99:7888 "||" >/dev/null
checkFalse $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 fe80::a00:27ff:fe99:7888 "|" >/dev/null
checkFalse $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 fe80::a00:27ff:fe99:7888 "<>" >/dev/null
checkFalse $? $LINENO
sh paraCheck.sh hahah kdsjflkd 192.168.0.103 ::1 "*" fe80::a00:27ff:fe99:7888 >/dev/null
checkFalse $? $LINENO
sh paraCheck.sh hahah 'fdsf\' >/dev/null
checkFalse $? $LINENO

echo "all test case succeed!"

```

## 基于shell脚本实现UDP端口探测

方案请参考：

https://blog.csdn.net/qq_31567335/article/details/86568254

为了实现简单，采用了连续两次发送报文，并对返回结果进行判断的方式：

```bash
	exec 8<>/dev/udp/$rsIp/$rsPort
	echo "" >&8 #第一次发送报文
	echo "" >&8 #再次发送报文：如果上一次发送报文时，收到了ICMP 端口不可达报文，此会返错误
	if [ $? -ne 0 ];then
		lvs_sh_log "$LINENO send udp packet return error. exit with 1. rsIp:$rsIp, rsPort:$rsPort"
		exit 1
	fi
	exit 0

```

奇怪的是，脚本却不能正常工作，探测**未开启**的端口，绝大部分情况下，都依然返回成功，只有极少数情况能够返回失败。

并且通过抓包的情况来看，每个报文都是正常返回了ICMP udp port xxx unreachable报文的。

而在命令行中执行的现象都是固定的，第一次发送返回成功，第二发送返回失败。

由于不太理解shell执行脚本的原理，担心这种实现方式可能有问题。

搜了很久，没找到结果。

最后终于发现......

妈的，都是由于计算机实在是太快了：连续发送两个报文，发送第二个报文的时候，第一个报文的ICMP udp port xxx unreachable报文还未回来呢....

在命令行手动执行的时候，人这个慢吞吞的速度.....自然是没有问题的。



改进写法：

```bash
	##建立连接
	exec 8<>/dev/udp/$rsIp/$rsPort
	
	echo "" >&8 #发送报文
	
	read -t 3 <&8 2>&1 | grep -q "onnection refused"
	#如果返回了Connection refused说明端口未开启，返回失败。
	if [ $? -eq 0 ];then
		lvs_sh_log "$LINENO send udp packet return error. exit with 1. rsIp:$rsIp, rsPort:$rsPort"
		exit 1
	fi

	exit 0

```

最简单的改进写法，当然是在两个发包之间加一个sleep。但是直接sleep 3秒，则不论什么情况都要sleep 3秒，太慢了。



## 解析json

```bash
# 函数约束：
# 1. 仅支持匹配最简单的单层json，比如：{"a":"100","b":"22"}
# 2. value中不能有'"}{'，会被直接替换掉
function get_json_value()
{
  local json=$1
  local key=$2

  echo "${json}" | tr -d '"}{'  |  awk -v key="${key}" -F"[,:]" '{
      if( NF%2 != 0){ #元素个数不为偶数个
        exit
      }
      for(i=1;i<NF;i=i+2){ # 每次+2，只匹配key
        if($i==key){
          print $(i+1)
          exit
        }
      }
    }'
}

```



## shell脚本上传下载文件到ftp服务器

* ftp自动登陆上传文件

```bash
####本地的/home/databackup to ftp服务器上的/home/data####
\#!/bin/bash
ftp -n<<!
open 192.168.1.171
user guest 123456
binary
hash
cd /home/data
lcd /home/databackup
prompt
mput *
close
bye
!

```



注：

1. -n 表示不读取.netrc文件中的配置。（ftp默认为读取.netrc文件中的设定）
2. << 是使用即时文件重定向输入。
3. !是标志，它必须成对出现，以标识即时文件的开始和结尾。

* ftp自动登录下载文件

```bash
#####从ftp服务器上的/home/data 到 本地的/home/databackup####
#!/bin/bash
ftp -n<<!

open 192.168.1.171

user guest 123456

binary

cd /home/data

lcd /home/databackup

prompt

mget *(也可以指定单个文件)

close

bye

!

```

 

## lftp

网上搜了一下，lftp是ftp的加强版。功能更为强大。

* 自动登陆上传文件

```bash
lftp <<!

open 10.180.167.14 

user ftpuser ftpuser 

cd /V300R006/CloudSOPAV1R6C10/CloudSOP_3rdTool 

mput POPT-5.4.13-binary-$timestamp.zip

close 

bye 

!

```



* 自动登陆下载文件

将mput换成mget命令即可。

附：

shell脚本中的EOF以及文件重定向

https://blog.csdn.net/huangjin0507/article/details/50352684



## 批量git clone代码仓

```bash
repos='
http://code-cbu.huawei.com/CB-OBS/OBS_AIOps/obs-monitor.git	master
http://code-cbu.huawei.com/CB-OBS/OBS_DataFuse/streaming.git	master
http://code-cbu.huawei.com/CB-OBS/OBS_DataHub/obs-service.git	master
http://code-cbu.huawei.com/CB-OBS/OBS_DataPlus/obs_bigdataonobs.git	master
http://code-cbu.huawei.com/obs-aiops/obs-secas-tomcat.git	master
http://codehub-g.huawei.com/cis/obsd/obs/obs-ux-console/obs-console.git	master
http://codehub-g.huawei.com/cis/obsd/obs/obs-ux-sftp/obs-ux-sftp.git	master
'
IFS=$'\n' #设置换行符为分隔符。默认空格也会被当做分隔符，无法按行处理
ROOT_DIR=$(PWD)

failed_repos=""

for line in $repos
do
    repo=$(echo $line | awk '{print $1}')
    branch=$(echo $line | awk '{print $2}')

    dir=$(basename $repo | sed 's/.git//')
    [[ -e $dir ]] && continue
    
    echo -e "\n\n"
    git clone $repo
    if [ $? != 0 ]
    then
        echo "clone failed!!!!!!!!!!!!!!!!"
        failed_repos="${failed_repos}$repo\n" 
        continue
    fi
    
    cd $ROOT_DIR/$dir
    git checkout -B $branch origin/$branch
    cd $ROOT_DIR
done

echo -e "\n\nfailed repos:\n$failed_repos"


```

## 获取当前脚本路径
```bash
scriptDir=$(cd `dirname $0`; pwd)
######切换到脚本所在目录#################
cd $scriptDir
```
在当前目录下，不论调用什么目录下的脚本，脚本中的pwd输出的目录都依然是当前目录。

所以说，在脚本中，如果要切换到脚本所在目录，那么就只需要获取脚本的执行名字中的路径部分，然后cd到对应目录即可。
然后用pwd的方式，就可以得到脚本所在绝对路径。


如果某个脚本是以source的方式被调用，那么情况又有些不同。
由于source的执行方式并没有新开一个shell去执行脚本，$0等变量都依然与调用此脚本的shell相同，所以，无法在脚本中通过$(cd `dirname $0`; pwd)来获取脚本所在的绝对路径

# 知识点

## sh和bash的区别

<https://stackoverflow.com/questions/5725296/difference-between-sh-and-bash>

简言之，sh是一个标准。

bash功能更丰富。

### sh不支持[[]]

这个语法是进行条件判断的

[[]]相比于[]的好处时，允许参数为空：

```bash
srcPort=12345
protocol=udp
oldSrcPort=""
oldProtocol=""
[[ $oldSrcPort != "$srcPort" ]] || [[ $oldProtocol != "$protocol" ]] && echo "not equal!"

```

[]不允许参数为空，必须使用双引号引起来

```
srcPort=12345
protocol=udp
oldSrcPort=""
oldProtocol=""
[ "$oldSrcPort" != "$srcPort" ] || [ "$oldProtocol" != "$protocol" ] && echo "not equal!"

```

否则脚本执行时会报错：

```bash
test.sh: line 5: [: !=: unary operator expected

```



## shell默认提供的参数

```
$0 这个程式的执行名字
$n 这个程式的第n个参数值，n=1..9
$* 这个程式的所有参数,此选项参数可超过9个。
$# 这个程式的参数个数
$$ 这个程式的PID(脚本运行的当前进程ID号)
$! 执行上一个背景指令的PID(后台运行的最后一个进程的进程ID号)
$? 执行上一个指令的返回值(显示最后命令的退出状态。0表示没有错误，其他任何值表明有错误)
$- 显示shell使用的当前选项，与set命令功能相同
$@ 跟$*类似，但是可以当作数组用
$PPID 父进程ID
$UID 执行脚本的用户ID

```

## Shell中判断语句if中-a至-z的意思

```
[ -a FILE ] 如果 FILE 存在则为真。
[ -b FILE ] 如果 FILE 存在且是一个块特殊文件则为真。
[ -c FILE ] 如果 FILE 存在且是一个字特殊文件则为真。
[ -d FILE ] 如果 FILE 存在且是一个目录则为真。
[ -e FILE ] 如果 FILE 存在则为真。
[ -f FILE ] 如果 FILE 存在且是一个普通文件则为真。
[ -g FILE ] 如果 FILE 存在且已经设置了SGID则为真。
[ -h FILE ] 如果 FILE 存在且是一个符号连接则为真。
[ -k FILE ] 如果 FILE 存在且已经设置了粘制位则为真。
[ -p FILE ] 如果 FILE 存在且是一个名字管道(F如果O)则为真。
[ -r FILE ] 如果 FILE 存在且是可读的则为真。
[ -s FILE ] 如果 FILE 存在且大小不为0则为真。
[ -t FD ] 如果文件描述符 FD 打开且指向一个终端则为真。
[ -u FILE ] 如果 FILE 存在且设置了SUID (set user ID)则为真。
[ -w FILE ] 如果 FILE 如果 FILE 存在且是可写的则为真。
[ -x FILE ] 如果 FILE 存在且是可执行的则为真。
[ -O FILE ] 如果 FILE 存在且属有效用户ID则为真。
[ -G FILE ] 如果 FILE 存在且属有效用户组则为真。
[ -L FILE ] 如果 FILE 存在且是一个符号连接则为真。
[ -N FILE ] 如果 FILE 存在 and has been mod如果ied since it was last read则为真。
[ -S FILE ] 如果 FILE 存在且是一个套接字则为真。
[ FILE1 -nt FILE2 ] 如果 FILE1 has been changed more recently than FILE2, or 如果 FILE1 exists and FILE2 does not则为真。
[ FILE1 -ot FILE2 ] 如果 FILE1 比 FILE2 要老, 或者 FILE2 存在且 FILE1 不存在则为真。
[ FILE1 -ef FILE2 ] 如果 FILE1 和 FILE2 指向相同的设备和节点号则为真。
[ -o OPTIONNAME ] 如果 shell选项 “OPTIONNAME” 开启则为真。
[ -z STRING ] “STRING” 的长度为零则为真。
[ -n STRING ] or [ STRING ] “STRING” 的长度为非零 non-zero则为真。 如果string是用变量表示，一定要用双引号括起来，否则空字符串会判断错误为真
[ STRING1 == STRING2 ] 如果2个字符串相同。 “=” may be used instead of “==” for strict POSIX compliance则为真。
[ STRING1 != STRING2 ] 如果字符串不相等则为真。
[ STRING1 < STRING2 ] 如果 “STRING1” sorts before “STRING2” lexicographically in the current locale则为真。
[ STRING1 > STRING2 ] 如果 “STRING1” sorts after “STRING2” lexicographically in the current locale则为真。
[ ARG1 OP ARG2 ] “OP” is one of -eq, -ne, -lt, -le, -gt or -ge. These arithmetic binary operators return true if “ARG1” is equal to, not equal to, less than, less than or equal to, greater than, or greater than or equal to “ARG2”, respectively. “ARG1” and “ARG2” are integers.

```



## shell脚本中单引号双引号与参数传递

为了研究一个shell脚本参数的注入问题，学习了一些这方面的知识。

**单引号与双引号的区别**

**单引号**

由单引号括起来的**任意字符**都被当做普通字符：

![img](http://image.huawei.com/tiny-lts/v1/images/8073725f2e1ef280e85c_482x148.png@900-0-90-f.png)



那么可能会有个疑问，如果单引号里面我还想要单引号出现，我应该怎么写呢？

第一反应肯定是使用\来进行转义。但是刚刚才说了， **任意字符**都被当做普通字符：

![img](http://image.huawei.com/tiny-lts/v1/images/4847625f2e01feac9a07_350x49.png@900-0-90-f.png)

解决方法也很简单。只需要把转义单引号的操作放到单引号外面就行了。

![img](http://image.huawei.com/tiny-lts/v1/images/d966025f2e0688099901_398x94.png@900-0-90-f.png)





**双引号**

双引号与单引号的区别就是，被双引号括起来的内容，$(美元符），\（反斜杆），`（反引号）这3个符号依然保留特殊功能。

![img](http://image.huawei.com/tiny-lts/v1/images/0317125f2e1c4f9992a4_649x305.png@900-0-90-f.png)



双引号中出现的单引号，会被当做普通字符处理：

![img](http://image.huawei.com/tiny-lts/v1/images/da77c25f2e15da615bfd_648x83.png@900-0-90-f.png)





**shell参数传递与注入问题**

当向脚本中传入&&, ||, ;等特殊字符时，是否能进行命令注入呢？

有两个用来测试的脚本文件：

![img](http://image.huawei.com/tiny-lts/v1/images/75c0825f2e2a3809f5e4_452x199.png@900-0-90-f.png)

当向脚本中传入特殊字符时：

![img](http://image.huawei.com/tiny-lts/v1/images/cd86225f2e2b55decd52_440x81.png@900-0-90-f.png)

赋值操作与传递参数的操作，都不会引发命令的执行。

所以说，即使是特殊字符，在当做shell参数传递时，也是被当做普通字符处理。无法越权执行命令。



所以说，依据个人的理解，得出的结论：

**任何可执行文件本身不是拼接的，就不存在越权风险。**

比如说，这样：

command=$1

sh $command

或者：

fileName=$1

sh /path1/path2/$fileName

这种一定会存在越权风险的。

但是如果传递进来的参数，只作为命令执行的参数，则不存在越权的问题。 













##  bash-shell脚本调试技术

测试脚本test.sh

```bash
a=1
if [ "$a" -eq 1 ]
then
    b=2
else
    b=1
fi
c=3
echo "end"

```

调试方式

```bash
sh -x test.sh #默认会直接输出到屏幕
sh -x test.sh &> log.txt  # 重定向到文件

```

输出：

```bash
copbint@debian2:~$ sh -x test.sh
+ a=1
+ [ 1 -eq 1 ]
+ b=2
+ c=3
+ echo end
end

```

在上面的结果中，前面有“+”号的行是shell脚本实际执行的命令，前面有“++”号的行是执行trap机制中指定的命令），其它不带+的行则是输出信息。

* -x 增强

在命令行先执行：

```bash
export PS4='+{$LINENO:${FUNCNAME[0]}} '

```

再运行脚本，输出：

```bash
copbint@debian2:~$ bash -x test.sh
+{2:main}  a=1
+{3:main}  '[' 1 -eq 1 ']'
+{5:main}  b=2
+{9:main}  c=3
+{10:main}  echo end
end

```

原理解释请参考： [Shell脚本调试技术](https://www.ibm.com/developerworks/cn/linux/l-cn-shell-debug/)

* 这种方式仅bash支持，shell不支持
* 在ubuntu上测试，发现bash -x test.sh运行脚本时，无法获取到外面设置的PS4变量，所以没办法只能修改test.sh，将修改PS4的命令放到脚本最前面。



