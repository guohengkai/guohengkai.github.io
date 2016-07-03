title: Linux里有用的shell命令总结
date: 2016-01-22 00:13:05
tags:
- Linux
- Shell
categories:
- Engineering
---
最近简单地翻了一本书叫[《Linux Shell Scripting Cookbook》](http://book.douban.com/subject/5974281/)，有很多有用的东西，在这里简单记录备忘一下，其中略过一些比较熟或者很少用的命令。
<!--more-->
## 1. 变量与环境变量
#### 查看进程的环境变量
``` bash
$ cat /proc/$PID/environ | tr '\0' '\n'
```
其中PID设置为相应进程的ID，tr命令将\0替换成\n。

#### 获取字符串长度
``` bash
$ length=${#var}
```


#### 检查是否为超级用户
``` bash
if [ $UID -ne 0 ]; then
  echo Non root user. Please run as root.
else
  echo Root user
fi
```


#### 查看和修改Bash提示字符串
``` bash
$ cat ~/.bashrc | grep PS1  # 查看
$ PS1="PROMPT>"  # 修改
```


#### 使用函数添加环境变量
``` bash
prepend() { [ -d "$2" ] && eval $1=\"$2\$\{$1:+':'\$$1\}\" && export $1; }
```
该函数首先会检查第二个参数的目录是否存在，若存在就将其插入到第一个参数指定的变量，若变量为空则不插入冒号。它利用了shell参数扩展：
``` bash
${parameter:+expression}
```
若parameter不为空则使用experssion的值。


## 2. 数学运算
#### 整数运算
``` bash
$ let result=no1+no2
$ result=$[ no1 + no2 ]
$ result=$(( no1 + no2))
$ result=$(expr $no1 + $no2)
```


#### bc命令
``` bash
$ result=`echo "$no * 1.5" | bc`  # 支持浮点运算
$ result=`echo "scale=2;3/8" | bc`  # 设置小数精度为2位
$ result=`echo "obase=2;ibase=10;$no" | bc`  # 十进制转二进制
$ result=`echo "sqrt(100)+10^2" | bc`  # 平方根与平方
```


## 3. 文件描述符及重定向
#### tee命令
``` bash
$ ls * &> | tee -a log.txt
```
将stdout和stderr既输出到log.txt，又作为stdout。-a选项用于追加内容。

#### 使用stdin作为命令参数
``` bash
$ cmd1 | cmd2 -
```

#### 自定义文件描述符
``` bash
$ exec 3>>input.txt
$ echo hehe >&5
```
其中>>操作符用于追加模式的文件写入。

## 4. 数组和关联数组
#### 数组
``` bash
$ array_var={1 2 3 4 5 6}  # 列表定义
$ array_var[0]=0; array_var[1]=1;  # 索引定义
$ echo ${array_var[0]}  # 特定索引的元素
$ echo ${array_var[*]}  # 数组所有值
$ echo ${array_var[@]}  # 同上
$ echo ${#array_var[*]}  # 数组长度
```

#### 关联数组
``` bash
$ declare -A ass_array  # 声明
$ ass_array=([index1]=val1 [index2]=val2)  # 列表定义
$ ass_array[index1]=val1  # 索引定义
$ echo ${!ass_array[*]}  # 数组索引列表
$ echo ${!ass_array[@]}  # 同上
```

## 5. 别名
#### 设置rm保留副本
``` bash
$ alias rm='cp $@ ~/backup && rm $@'
```

#### 删除或忽略别名
``` bash
$ unalias cmd  # 删除别名
$ alias cmd=  # 另一种写法
$ \cmd  # 忽略别名
```

## 6. 获取终端信息
#### tput工具
``` bash
$ tput cols  # 获取列数
$ tput lines  # 获取行数
$ tput cup 100 100  # 移动光标
$ tputed  # 删除从当前光标位置到行尾的所有内容
```

#### 输入密码脚本
``` bash
echo -e "Enter password: "
stty -echo  # 禁止将输出发送到终端
read password
stty echo
echo
echo Password read.
```

## 7. 日期与时间
#### 读取与设置日期
``` bash
$ date  # 打印类似于：Sun Jan 24 11:44:00 IST 2016
$ date +%s # 打印纪元时
$ date --date "input_date" +%s  # 日期格式转换
$ date "+%d %B %Y"  # 打印年月日
$ date -s "input_date"  # 设置日期
```

#### 脚本计时
``` bash
$ time cmd
```

#### 延时命令
``` bash
$ sleep no_of_seconds
```

## 8. 调试脚本
#### 脚本调试开关
``` bash
$ set -x  # 在执行时显示参数和命令
$ set +x  # 禁止调试
$ set -v  # 在命令进行读取时显示输入
$ set +v  # 禁止打印输入
```
也可以在shebang上加入：
``` bash
#!/bin/bash -xv
```

#### 自定义调试
``` bash
function DEBUG()
{
  [ "$_DEBUG" == "on" ] && $@ || :
}
```
其中“:”表示不执行任何操作。使用时只需要在每一个在调试时想执行的语句前加DEBUG，并在执行脚本前定义_DEBUG变量为on即可。

## 9. 函数与参数
#### 函数参数
``` bash
fname()
{
  echo $1, $2;  # 参数1和参数2
  echo "$@"  # 以列表方式打印所有参数
  echo "$*"  # 参数被作为一个字符串
  return 0;
}
```

#### Fork炸弹
``` bash
:() { :|:& };:
```
该递归函数能够调用本身不断生成新的进程。

#### 检测命令是否成功结束
``` bash
$CMD
if [ $? -eq 0 ];
then
  echo "$CMD executed successfully"
else
  echo "$CMD terminated unsuccessfully"
fi
```

## 10. 子shell
#### 将命令输出读入变量
``` bash
$ cmd_output = "$(CMDS)"  # 加引号可以保留空格和换行符
$ cmd_output = `CMDS`
```

#### 新建进程执行命令
``` bash
$ pwd;
$ (cd /bin; ls);
$ pwd;
```

## 11. 获取输入
#### read命令
``` bash
$ read -n 2 var  # 从输入读取2个字符存入变量var
$ read -s var  # 无回显读取
$ read -p "Enter input:" var  # 显示提示信息
$ read -t 2 var  # 2秒内读取输入
$ read -d ":" var  # 以冒号作为输入结束
```

## 12. 字符分隔符和迭代器
#### 读取csv数据
``` bash
oldIFS=$IFS
IFS=,
for item in $data;
do
  echo Item: $item
done
IFS=$oldIFS
```

#### for循环
``` bash
for i in {a..z}; do cmds; done;
```
也可以采用C语言的格式：
``` bash
for((i=0;i<10;i++))
{
  cmds;
}
```

#### 其他循环
``` bash
while condition  # 直到条件不成立时退出
do
  cmds;
done
until condition;  # 直到条件成立时退出
do
  cmds;
done
```

## 13. 比较与测试
#### 算术比较
``` bash
$ [ $var -gt 0 -a $var -le 2 ]  # 大于0且小于等于2
$ [ $var -lt 0 -o $var -ge 2 ]  # 小于0或大于等于2
```

#### 文件系统测试
``` bash
$ [ -d $var ]  # 是否目录
$ [ -e $var ]  # 文件是否存在
```

#### 字符串测试
``` bash
$ [[ $str1 = $str2 ]]  # 是否相同，等号前后如果不加空格则为赋值
$ [[ $str1 != $str2 ]]  # 是否不同
$ [[ -z $str ]]  # 是否为空
$ [[ -n $str ]]  # 是否不为空
```

#### test命令
``` bash
$ test $var -eq 0  # 等价于 [ $var -eq 0 ]
```

## 14. cat命令
#### 拼接标准输入
``` bash
$ echo 'Text from stdin' | cat - file.txt
```

#### 其他选项
``` bash
$ cat -s file  # 压缩相邻空白行
$ cat -T file  # 将制表符显示为^|
$ cat -n file  # 显示行号
```

## 15. 录制并回放终端会话
#### 录制会话
``` bash
$ script -t 2> timing.log -a output.session
cmds;
...
exit
```
其中timing.log存储时序信息，output.session存储命令输出信息。

#### 回放会话
``` bash
$ scriptreplay timing.log output.session
```

## 16. 文件查找
#### 基础find命令
``` bash
$ find . -print0  # 在当前目录向下遍历查找文件，使用'\0'作为定界符
$ find . \( -name "*.txt" -o -name "*.pdf" \) -print  # 匹配对应文件名
$ find . -path "*/ghk/*" -print  # 匹配路径
$ find . ! -name "*.txt" -print  # 否定匹配
$ find . -maxdepth 1 -print  # 指定最大深度
$ find . -type d -print  # 只列出目录（文件为f，符号链接为l）
$ find . \( -name ".git" -prune \) -o \( -type f -print \)  # 排除目录
```

#### 结合动作的find命令
``` bash
$ find . -type f -name "*.swp" -delete  # 删除特定文件
$ find . -type f -user root -exec chown ghk {} \;  # 改变文件所有权
```
其中{}为与-exec选项搭配的特殊字符串，它将依次替换为每个匹配的文件名。（当需要对找到的文件名列表作为参数时用+代替;）

## 17. xargs命令
#### 基础xargs用法
``` bash
$ cat args.txt | xargs  # 把多行输入转换成单行输出（即将\n替换为空格）
$ cat args.txt | xargs -n 3  # 按空格和换行符拆成每行3个参数
$ echo "splitXsplitXsplit" | xargs -d X  # 定义定界符为X
$ cat args.txt | xargs -I {} cmd.sh -p {} -l  # 依次将每个参数传进cmd.sh执行（-I指定参数替换字符串）
```

#### xargs应用
``` bash
$ find . -type f -name "*.txt" -print0 | xargs -0 rm -f  # 删除所有txt文件（指定\0为输入定界符）
$ find . -type f -name "*.c" -print0 | xargs -0 wc -l  # 统计C的代码行数
```

## 18. 用tr进行转换
#### 替换字符
``` bash
$ tr $str 'A-Z' 'a-z'  # 大写转换为小写
```
当第二个字符集不够长时，会不断复制最后一个字符直到长度与第一个集合相等。

#### 删除字符
``` bash
$ tr -d '0-9'  # 删除数字
$ tr -d -c '0-9 \n'  # 删除除了数字、空格和换行符的字符
$ tr -s '\n'  # 删除多余的换行符
```

## 19. 排序与去重
#### sort命令
``` bash
$ sort file1 file2
$ sort -n file  # 按数字顺序排序
$ sort -r file  # 逆序
$ sort -m sorted1 sorted2  # 合并有序文件
$ sort -bk 2 file  # 按第二列排序（-b忽略前导空白行）
```

#### uniq命令
``` bash
$ sort unsorted.txt | uniq
$ uniq -u sorted  # 只显示唯一的行
$ uniq -c sorted  # 统计次数
$ uniq -d sorted  # 找出重复的行
$ uniq -s 2 -w 2  # 忽略前2个字符（-s 2），最多比较2个字符（-w 2）
```

## 20. 临时文件命名
#### mktemp命令
``` bash
$ mktemp  # 生成临时文件并返回文件名
$ mktemp -d  # 临时目录
$ mktemp -u  # 仅返回名字，不进行创建
$ mktemp test.XXX  # 根据模板生成文件（至少3个X）

## 21. 文件名切分
#### %操作符和#操作符
``` bash
$ ${VAR%.*}  # 从右到左非贪婪匹配通配符.*并删除匹配结果（贪婪为%%）
$ ${VAR##*.}  # 从左到右贪婪匹配通配符*.并删除匹配结果（非贪婪为#）
```

## 22. 批量重命名和移动
#### 重命名当前目录下的图像
``` bash
count=1;
for img in `find . -iname '*png' -o -iname '*.jpg' -type f -maxdepth 1`
do
  new=img-$count.${img##*.}
  echo "Renaming $img to $new"
  mv "$img" "$new"
  let count++
done
```

#### rename命令
``` bash
$ rename 's/ /_/g' *  # 将文件名的空格替换为_
$ rename 'y/a-z/A-Z' *  # 转换文件名大小写
```

## 23. 交互输入自动化
#### 自动ssh的脚本
``` bash
#!/usr/bin/expect -f
spawn ssh $USERNAME@$IP
expect ":"
send "$PASSWORD\n"
interact
```

#### 自动实现ssh无密钥登录
``` bash
#!/usr/bin/expect
set username [lindex $argv 0]
set password [lindex $argv 1]
set host [lindex $argv 2]
spawn scp ~/.ssh/id_dsa.pub $username@$host:~/.ssh/pub_key
expect "*password*"
send "$password\n"
spawn ssh $username@$host "cat ~/.ssh/pub_key >> ~/.ssh/authorized_keys"
expect "*password*"
send "$password\n"
expect eof
```

## 24. 查找文件差异及修补
#### 生成差异文件
``` bash
$ diff -u version1.txt version2.txt > version.patch
```

#### 修补文件
``` bash
$ patch -p1 version1.txt < version.patch
```

## 25. 分割文件和数据
#### split命令
``` bash
$ split -b 10k data.file -d -a 4 split_file  # 将文件分割为10KB的小文件，前缀名为split_file，后缀为4位数字
$ split -l 10 data.file  # 分割成多个文件，每个10行
```

## 26. 切分文件名
#### 提取扩展名和域名
``` bash
$ # 假设URL="www.google.com"
$ ${URL%.*}  # 非贪婪匹配最右并删除：www.google
$ ${URL%%.*}  # 贪婪匹配最右并删除：www
$ ${URL#*.}  # 非贪婪匹配最左并删除：google.com
$ ${URL##*.}  # 贪婪匹配最左并删除：com
```

## 27. 文件权限和所有权
#### chmod命令
``` bash
$ chmod u=rwx g=rw o=r filename  # 指定用户权限为读、写和执行，用户组权限为读和写，其他实体权限为读
$ chmod o+x filename  # 添加可执行权限
$ chmod a-x filename  # 给所有用户删除可执行权限
$ chmod 764 filename  # rwx rw- rwx
$ chmod a+t directory_name  # 设置粘滞位，使得只有目录所有者才能删除目录中的文件
$ chmod 777 . -R  # 递归修改所有文件的权限
$ chmod +s executable_file  # 给二进制文件添加setuid权限，使文件在执行阶段具有文件所有者的权限
```

#### chown命令
``` bash
$ chown user.group filename  # 更改文件所有权
$ chown user.group . -R  # 递归设置所有权
```

## 28. 创建不可修改的文件
#### chattr命令
``` bash
$ chattr +i file  # 任何用户不能删除该文件
$ chattr -i file  # 恢复可写状态
```

## 29. 文本处理
#### grep命令搜索文本
``` bash
$ grep "pattern" file1 file2 --color=auto  # 在多个文件中查找pattern，并着重标记匹配的单词
$ grep -E "[a-z]+" filename  # 使用正则表达式
$ grep -v pattern file  # 打印除包含pattern以外的行
$ grep -c pattern file  # 统计包含匹配字符串的行数
$ cat file | grep pattern -n  # 打印行号
$ grep -o -b pattern file  # -o只输出匹配的部分，-b打印字符或字节偏移
$ grep -l pattern file1 file2  # 搜索多个文件并找出匹配文本位于哪个文件（-L则是不匹配的列表）
$ grep pattern . -R -n  # 多级目录对文本进行递归搜索
$ grep -i pattern file  # 忽略大小写
$ grep -e pattern1 -e pattern2 file  # 匹配多个样式
$ grep -f pattern_file file  # 用文件指定pattern
$ grep "main()" . -r --include *.{c,cpp} --exclude "README"  # 指定和排除某些文件
$ grep pattern . -r --exclude-dir path --exclude-from file_list  # 排除目录和文件列表
$ grep "test" file* -lZ | xargs -0 rm  # 删除带test文本的文件
$ grep -q pattern file  # 静默输出
$ grep pattern file -A line_num  # 打印匹配结果之后的line_num行（-B是之前，-C是前后）
```

#### cut命令按列切分文本
``` bash
$ cut -f 2,3 -s filename  # 打印第2、3列，对于不含定界符'\t'的不打印该行'
$ cut -f 3 --complement filename  # 打印除了第3列的所有列
$ cut -f 2 -d ";" filename  # 以分号为定界符
$ cut -c 1-5 filename  # 打印前5个字符
$ cut -c 1-3,6-9 --output-delimiter ","  # 以逗号分隔打印多个字段
```

#### sed命令进行文本替换
``` bash
$ sed 's/pattern/replace_string/' file  # 替换给定文本中的字符串并打印，每行只替换第一处（如果需要保存，使用-i）
$ sed 's/pattern/replace_string/g' file  # 替换每行所有匹配字符串
$ sed 's/pattern/replace_string/3g' file  # 从第3处开始替换
$ sed '/^$/d' file  # 删除空白行
$ sed 's/\w\+/[&]/g' file  # 将每个单词加上中括号
$ sed 's/\([a-z]\+\) \([A-Z]\+\)/\2 \1' file  # 将第一个单词和第二个进行互换（\1表示第一个匹配子串）
$ sed -e 'expression' -e 'expression' file  # 组合多个表达式（也可以用管道或者单引号内的分号来替代）
$ sed "s/$text/HELLO" file  # 引用变量text，使用双引号
```

#### awk命令
``` bash
$ awk 'BEGIN { statements } { statements } END { statements }' file  # 首先执行BEGIN，然后对每一行执行中间的命令，最后执行END
$ awk 'BEGIN { print "start" } { print } END { print "end" }' file  # 打印每一行
$ awk '{ print "line no:"NR",field no:"NF, "$0="$0 }' file  # 打印行号、每行字段数和第一个字段
$ awk 'END { print NR }' file  # 统计行数
$ awk '{ print v1,v2 }' v1=$var1 v2=$var2 file  # 传递外部变量
$ awk 'BEGIN { getline; print "first row: ", $0 } { print $0 }'  # 在BEGIN读取一行
$ awk 'NR==1,NR==4' file  # 读取行号为1到5的行
$ awk '/linux/'  # 包含样式linux的行
$ awk -F : '{ print NF }' /etc/passwd  # 将定界符设为冒号
$ echo | awk '{ "command" | getline cmdout ; print cmdout }'  # 将外部shell命令读入变量cmdout
$ awk '/start_pattern/, /end_pattern/' filename  # 打印位于start_pattern与end_pattern之间的文本
```

#### 统计词频
``` bash
egrep -o "\b[[:alpha]]+\b" $filename | \
awk '{ count[$0]++ } END { printf("%-14s%s\n","Word","Count");
for(ind in count)
{
  printf("%-14s%d\n",ind,count[ind]);
}}'
```

#### 按列合并文件
``` bash
$ paste file1 file2 file3 -d ","  # 指定逗号为定界符
```

#### 文本切片
``` bash
$ ${var/old_str/new_str}  # 将变量中的old_str替换为new_str
$ ${var:start_position:length}  # 得到子串（-1表示最后一位，必须用括号括起来 ）
```

## 30. Web页面下载
#### wget命令
``` bash
$ wget URL -O save_name  # 从URL下载文件，输出文件名为save_name
$ wget URL -o log -t 5  # 将日志输出到log，重试次数最多为5（若为0则是不断重试）
$ wget --limit-rate 20k -Q 100m URL  # 限速20k，最大下载配额100m
$ wget -c URL  # 断点续传
$ wget --mirror --convert-links URL  # 递归下载整个网站
$ wget -r -N -l -k DEPTH URL  # 递归下载整个网站，最大深度DEPTH
$ wget --user username --password pass URL  # 提供认证信息（--ask-password则手动输入密码）
```

## 31. cURL命令
#### curl命令
``` bash
$ curl URL --silent  # 从URL下载文件输出到终端，避免显示进度信息
$ curl URL -o file_name --progress  # 从URL下载到文件，用井号输出进度
$ curl -C - URL  # 断点续传
$ curl -u user:pass URL  # 用户认证
$ curl -I URL  # 打印HTTP头部信息
$ curl URL -d "postvar=postdata&postvar2=postdata2"  # 发送POST请求
```

## 32. 归档与压缩
#### tar命令
``` bash
$ tar -cvf output.tar file1 file2 --totals # 归档文件并打印总归档字节数
$ tar -xvf archive.tar -C path  # 提取文件到指定文件夹
$ tar -Af file1.tar file2.tar  # 合并归档文件
$ tar -f archive.tar --delete file1 file2  # 从归档文件中删除文件
$ tar acvf archive.tar.gz file1 file2  # 归档并压缩文件
$ tar -cf archive.tar * --exclude "*.txt"  # 归档时排除文件
$ tar -cf archive.tar * -X list  # 基于列表排除文件
```

#### zip命令
``` bash
$ zip -r archive.zip folder1 folder2  # 递归压缩文件夹
$ zip -e archive.zip file  # 添加压缩密码
$ unzip file.zip  # 解压缩
$ zip -d archive.zip file  # 删除压缩包中文件
$ unzip -l archive.zip  # 列出压缩包中的文件
```

## 33. 备份与同步
#### rsync命令
``` bash
$ rsync -av source_path destination_path  # 复制文件，使用归档并输出进度（源路径如果末尾带/则会把目录中的内容复制，否则把目录本身复制）
$ rsync -avz source_path destination_path --delete  # 使用压缩模式进行同步，删除不存在的文件
$ rsync -avc source_path destination_path --exclude "pattern" --exclude-from list  # 传输时使用校验，且排除一些文件
$ mkdir /tmp/test && rsync --delete-before -aHv --progress --stats /tmp/test path  # 快速删除有海量文件的文件夹path
```

## 34. 远程访问和传输
#### ssh命令
``` bash
$ ssh user@host 'COMMANDS'  # 在远程执行命令
$ ssh -X user@host  # 开启图形化输出并显示在本地
$ ssh-copy-id user@host  # 自动将私钥加入远程服务器的authorized_keys文件中
$ ssh -fL 8000:www.kernel.org:80 user@localhost -N  # 将本地端口8000上的流量转发到www.kernel.org的端口80，并转入后台执行
$ ssh -R 8000:localhost:80 user@host  # 将远程主机端口8000上的流量转发到本地主机的端口80上
```

#### scp命令
``` bash
$ scp filename user@host:path -o port # 将文件传输到远程主机，使用port端口
$ scp -r src_path dest_path  # 递归复制
```

## 35. 监视磁盘使用情况
#### du命令
``` bash
$ du -h file1 file2  # 查看文件占用的磁盘空间
$ du -a directory_name  # 查看目录下每个文件占用的空间
$ du -c file1 file2  # 查看总共占用的空间
```

#### df命令
``` bash
$ df -h  # 查看磁盘可用空间信息
```

## 36. 计算命令执行时间
#### time命令
``` bash
$ time COMMAND -o output.txt # 统计运行时间并写入output.txt文件
$ time -f "Time: %U" COMMAND  # 格式化输出（real: %e为命令从开始到结束的时间，user: %U为实际执行时间，sys: %S为花费在内核的时间）
```

## 37. 系统相关登录信息
#### 常用命令
``` bash
$ who  # 当前登录用户信息
$ w  # 更为详细的信息
$ users  # 当前登录主机的用户列表
$ uptime  # 系统已经运行时间
$ last  # 上一次启动及登录的信息
```

## 38. 监视命令输出
#### watch命令
``` bash
$ watch COMMAND  # 以2秒间隔更新输出（-n设置秒数）
$ watch -d COMMAND  # 将差异之处用颜色突显出来
```

## 39. 收集进程信息
#### ps命令
``` bash
$ ps -fe  # 显示详细的所有进程信息
$ ps -eo comm,pcpu | head  # 显示命令和CPU占用率
$ ps -C COMMAND_NAME  # 显示特定命令的进程信息
$ ps -eo cmd e  # 输出环境变量
```

#### 其他常用命令
``` bash
$ top  # 输出占用CPU最多的进程列表
$ pgrep COMMAND  # 列出特定命令的进程ID列表
$ pgrep -u root COMMAND  # 指定用户
$ pgrep -c COMMAND  # 返回匹配的进程数量
$ which COMMAND  # 获取命令位置
$ whereis COMMAND  # 获取命令位置和源码路径
$ file FILENAME  # 获取文件信息
$ whatis COMMAND  # 获取命令信息
```

## 40. 发送信号
#### kill命令
``` bash
$ kill -l  # 打印信号编号和名称
$ kill PID  # 发出TERM信号给进程
$ kill -s SIGNAL PID  # 强行杀死进程
```

#### killall命令
``` bash
$ killall -s SIGNAL process_name  # 通过名称发送信号
$ killall -u USERNAME process_name -i  # 通过用户名和进程名指定进程，并需要进行确认
```

## 41. 采集系统信息
#### 常用命令
``` bash
$ hostname  # 打印主机名（或者uname -n）
$ uname -a  # 打印Linux内核信息
$ uname -r  # 打印内核发行版本
$ uanme -m  # 打印主机类型
$ cat /proc/cpuinfo  # 打印CPU相关信息
$ cat /proc/meminfo  # 打印内存信息
$ cat /proc/patitions  # 打印系统分区信息（也可以用fdisk -l）
$ lshw  # 获取系统详细信息
$ lspci  # 查看驱动
$ cat /etc/issue  # 查看系统版本
$ nvidia-smi  # 查看显卡状态
```

## 42. 用户信息管理
#### 常用命令
``` bash
$ useradd USER -p PASSWORD  # 创建新用户
$ deluser USER --remove-all-files  # 删除用户
$ chsh USER -s SHELL  # 修改用户默认shell
$ usermod -L USER  # 锁定用户账户（-U用于解锁）
$ passwd USER  # 修改密码
$ addgroup GROUP  # 新建用户组
$ addgroup USER GROUP  # 将已有的用户添加到一个组
$ delgroup GROUP  # 删除用户组
$ figer USER  # 显示用户信息
```

## 43. 媒体处理
#### convert命令
``` bash
$ convert INPUT OUTPUT  # 转换图像格式
$ convert image.png -resize 1024x768 image.png  # 缩放图像（也可按照百分比：50%）
```

#### import命令
``` bash
$ import -window root screenshot.png  # 取整个屏幕
$ import screenshot.png  # 手动选取区域
```

#### ffmpeg命令
``` bash
$ ffmpeg -i input_file output_file  # 格式转换
$ ffmpeg -i input_file -vcodec copy -an output_file_video  # 视频流分离
$ ffmpeg -i input_file -acodec copy -vn output_file_audio  # 音频流分离
$ ffmpeg –i test.avi –r 1 –f image2 image-%3d.jpeg  # 图片提取
$ ffmpeg -i movie.mp4 2>&1 | grep Duration  # 获取视频时长
$ ffmpeg -ss START -t LENGTH -i INPUTFILE -vcodec copy -acodec copy OUTFILE  # 视频切片
$ ffmpeg -i in.mkv -codec copy -map 0 -f segment -segment_time 10 out%03d.mkv  # 按10秒切片
```

## 44. 调试相关
#### 常用命令
``` bash
$ ulimit –c unlimited  # 使出现segmentation fault时保存core文件，之后可用gdb查看
$ objdump -T file  # 查看文件的动态符号
$ ldd file  # 查看应用程序对库的依赖
```
