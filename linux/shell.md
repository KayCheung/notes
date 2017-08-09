# shell 变量 $$、$!、$?、$#、$@、$0、$1、$2 的含义解释:
  1. $$ : shell 本身的PID(Process ID)
  2. $! : shell 最后运行的后台Process的ID
  3. $? : 最后运行的命令的结束代码（返回值）
  4. $# : 添加到Shell的参数个数
  5. $@ : 所有参数列表。循环输出参数列表。
  6. $- : 使用Set命令设定的Flag一览
  7. $* : 所有参数列表。如"$*"用 [] 括起来的情况、以"$1 $2 … $n"的形式输出所有参数，否则与 $@ 一致。
  8. $0 : Shell本身的文件名 
  9. $0-$n :  添加到Shell的各参数值。$1是第1参数、$2是第2参数…。
  
# shell 表达式
	-eq 等于
	-ne 不等于
	-gt 大于
	-ge 大于等于
	-lt 小于
	-le 小于等于
	< 	小于(需要双括号),如:(("$a" < "$b"))
		在ASCII字母顺序下.如:
			if [[ "$a" < "$b" ]]
			if [ "$a" \< "$b" ]
			注意:在[]结构中"<"需要被转义.
	
	> 	大于(需要双括号),如:(("$a" > "$b"))
		在ASCII字母顺序下.如:
			if [[ "$a" > "$b" ]]
			if [ "$a" \> "$b" ]
			注意:在[]结构中">"需要被转义. 具体参考Example 26-11来查看这个操作符应用的例子.
	<=	小于等于(需要双括号),如:(("$a" <= "$b"))
	>= 	大于等于(需要双括号),如:(("$a" >= "$b"))
	= 	等于,如:if [ "$a" = "$b" ]
	== 	等于,如:if [ "$a" == "$b" ],与=等价
       注意:==的功能在[[]]和[]中的行为是不同的,如下:
       1 [[ $a == z* ]]    # 如果$a以"z"开头(模式匹配)那么将为true
       2 [[ $a == "z*" ]] # 如果$a等于z*(字符匹配),那么结果为true
       3 [ $a == z* ]      # File globbing 和word splitting将会发生
       4 [ "$a" == "z*" ] # 如果$a等于z*(字符匹配),那么结果为true
       一点解释,关于File globbing是一种关于文件的速记法,比如"*.c"就是,再如~也是. 但是file globbing并不是严格的正则表达式,虽然绝大多数情况下结构比较像.
	!=  不等于,如:if [ "$a" != "$b" ] 这个操作符将在[[]]结构中使用模式匹配.
	=~  判断子字符串包含关系
	
	   
	   
  
# shell 的输入输出符号
  1. 0 表示标准输入 
  2. 1 表示标准输出 
  3. 2 表示标准错误输出
  4. 输入输出用符号 < 和 > 来表示  

  默认为标准输出重定向，与 1> 相同。
  2>&1 意思是把 标准错误输出 重定向到 标准输出. 
  &>file 意思是把 标准输出 和 标准错误输出 都重定向到文件file中
  >&与&>效果相同
  n>&m 表示使文件描述符n成为输出文件描述符m的副本。这样做的好处是，有的时候你查找文件的时候很容易产生无用的信息,
  如:2> /dev/null的作用就是不显示标准错误输出；另外当你运行某些命令的时候,出错信息也许很重要,便于你检查是哪出了毛病,如:2>&1 
  
# shell脚本中echo显示内容带颜色显示, 需要使用参数-e  
  格式如下：
	echo -e "\033[字背景颜色;文字颜色m{要输出的内容}\033[0m"
	echo -e "\033[41;36m something here \033[0m"
	
	其中41的位置代表底色， 36的位置是代表字的颜色
	
  注： 
　　1. 字背景颜色和文字颜色之间是英文的";"
　　2. 文字颜色后面有个m 
　　3. 字符串前后可以没有空格，如果有的话，输出也是同样有空格 

    字颜色：30—–37, 字背景颜色范围：40—–47
# 控制选项说明
  \033[0m 关闭所有属性 
　\033[1m 设置高亮度 
　\033[4m 下划线 
　\033[5m 闪烁 
　\033[7m 反显 
　\033[8m 消隐 
　\033[30m — \33[37m 设置前景色 
　\033[40m — \33[47m 设置背景色 
　\033[nA 光标上移n行 
　\033[nB 光标下移n行 
　\033[nC 光标右移n行 
　\033[nD 光标左移n行 
　\033[y;xH设置光标位置 
　\033[2J 清屏 
　\033[K 清除从光标到行尾的内容 
　\033[s 保存光标位置 
　\033[u 恢复光标位置 
　\033[?25l 隐藏光标 
　\033[?25h 显示光标

  背景闪烁: echo -e "\033[5m\033[41;36m something here \033[0m"

# shell if中 -a到-z 表达式
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
	[ FILE1 -nt FILE2 ] 如果 FILE1 has been changed more recently than FILE2, or 如果 FILE1 exists and FILE2 does not则为真。 比文件1比文件2是否要新 :  filename1 -nt filename2 如果 filename1比 filename2新，则为真。
	[ FILE1 -ot FILE2 ] 如果 FILE1 比 FILE2 要老, 或者 FILE2 存在且 FILE1 不存在则为真。 比较文件1比文件2是否要旧 :  filename1 -ot filename2 如果 filename1比 filename2旧，则为真。
	[ FILE1 -ef FILE2 ] 如果 FILE1 和 FILE2 指向相同的设备和节点号则为真。
	[ -o OPTIONNAME ] 如果 shell选项 “OPTIONNAME” 开启则为真。
	[ -z STRING ] “STRING” 的长度为零则为真。 字符串为"null".就是长度为0
	[ -n STRING ] or [ STRING ] “STRING” 的长度为非零 non-zero则为真。 字符串不为"null"
		注意: 使用-n在[]结构中测试必须要用""把变量引起来.使用一个未被""的字符串来使用! -z或者就是未用""引用的字符串本身,放到[]结构中。虽然一般情况下可以工作,但这是不安全的.习惯于使用""来测试字符串是一种好习惯.
	[ STRING1 == STRING2 ] 如果2个字符串相同。 “=” may be used instead of “==” for strict POSIX compliance则为真。
	[ STRING1 != STRING2 ] 如果字符串不相等则为真。
	[ STRING1 < STRING2 ] 如果 “STRING1” sorts before “STRING2” lexicographically in the current locale则为真。
	[ STRING1 > STRING2 ] 如果 “STRING1” sorts after “STRING2” lexicographically in the current locale则为真。

	

	


	
# shell中的进度条打印
<pre>
#!/bin/bash
b=''
i=0
while [ $i -le  100 ]
do
	printf "progress:[%-50s]%d%%\r" $b $i
	sleep 0.1
	i=`expr 2 + $i`        
	b=#$b
done
echo


#!/bin/bash
i=0
while [ $i -lt 20 ]
do
	for j in '-' '\' '|' '/'
	do
		printf "intel testing : %s\r" $j
		sleep 0.1
		((i++))
	done
done
</pre>



