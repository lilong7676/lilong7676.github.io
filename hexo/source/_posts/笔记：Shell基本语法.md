---
title: 笔记：Shell基本语法
date: 2019-06-11 19:35:01

tags:
  - shell
  
categories: 
  - shell分类
  
comments: false
---
```
// test.sh
#! 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种 Shell。
#!/bin/bash
echo "hello world"
```
<!--more -->
```    
运行Shell脚本的方法
1 作为可执行程序
chmod +x ./test.sh  #使脚本具有执行权限
./test.sh  #执行脚本
    
2 作为解释器参数
/bin/sh test.sh
```

# Shell变量

```
#定义变量
my_name="foo"

#使用变量 只需要在变量名前加上 $ 符号
echo $my_name
#加花括号视为了帮助解释器识别变量的边界
echo ${my_name}

for skill in Java Coffee; do
    echo "I am good at ${skill}Script"
done

#使用readonly将变量设为只读变量
my_name="foo"
readonly my_name #如果尝试修改变量值，会报错
my_name="bar" #my_name: readonly variable

#删除变量
#变量被删除后不能再次使用。unset 命令不能删除只读变量。
unset my_name
echo my_name #将没有任何输出

```

## shell变量类型 三种
1. 局部变量

局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量

2. 环境变量

所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。 

3. shell变量

 shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行
 
 ### shell字符串
 字符串可以用单引号，也可以用双引号，也可以不用引号。单双引号的区别跟PHP类似。
 ```
 your_name="foo"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1
# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3

输出
hello, foo ! hello, foo !
hello, foo ! hello, ${your_name} !

#获取字符串的长度
str="abcd"
echo ${#str} #输出 4

#提取子串 
#以下实例从字符串第 2 个字符开始截取 4 个字符
str="hello world"
echo ${str:1:4} $输出 ello

#查找子字符串的位置
#查找字符 l 或 o 的位置(哪个字母先出现就计算哪个)：
string="hello world"
echo `expr index "$string" lo`  # 输出 2
```

### shell 数组
```
#数组 数组名=(值1 值2 ... 值n)
arr=(
	v1
	v2
	v3
	)
echo "数组 ${arr[1]}"
#使用 @ 符号可以获取数组中的所有元素
echo “数组中的所有元素 ${arr[@]}”
#获取数组的长度
echo "数组的长度 ${#arr[@]}"
#或者
echo "数组的长度 ${#arr[*]}"
#获取数组单个元素的长度
echo "数组第二个元素的长度 ${#arr[1]}"

#还可以单独定义数组的各个分量：

array_name[0]=value0
array_name[1]=value1
array_name[n]=valuen
```

### shell传递参数
```
# $ ./test.sh 1 222 3 4

#!/bin/bash
echo "shell 传递参数实例"
echo "执行的文件名 $0"
echo "第一个参数 $1"
echo "第二个参数 $2"
echo "第三个参数 $3"
echo "参数个数 $#"
echo "所有参数 $*"
echo "所有参数 加引号 \"$@\""
echo "脚本当前进程ID $$"
echo "后台运行的最后一个进程的ID $!"
echo "shell使用的当前选项 $-"

echo "-- \$* 演示--"
for i in "$*"; do
	echo $i
done

echo "-- \$@ 演示--"
for i in "$@"; do
	echo $i
done

#输出
shell 传递参数实例
执行的文件名 ./test.sh
第一个参数 1
第二个参数 222
第三个参数 3
参数个数 4
所有参数 1 222 3 4
所有参数 加引号 "1 222 3 4"
脚本当前进程ID 20530
后台运行的最后一个进程的ID 
shell使用的当前选项 hB
-- $* 演示--
1 222 3 4
-- $@ 演示--
1
222
3
4

```

### shell算数运算
```
#!/bin/bash

a=10
b=20
echo "a $a, b $b"

#加法
val=`expr $a + $b`
echo "a + b: $val"

#乘法 需要转义
val=`expr $a \* $b`
echo "a * b: $val"

#除法
val=`expr $b / $a`
echo "b / a: $val"

#取余
val=`expr $a % $b`
echo "a % b: $val"

if [ $a == $b ]
then
	echo "a 等于 b"
fi

if [ $a != $b ]
then
	echo "a 不等于 b"
fi

#输出
a 10, b 20
a + b: 30
a * b: 200
b / a: 2
a % b: 10
a 不等于 b


#扩展 三种加法运算写法
a=10
b=20

c=`expr $a + $b`

echo "c: $c"

d=$[ `expr $a + $b` ]
echo "d: $d"

e=$[ 10 + 20 ]
echo "e: $e"
```

### shell 关系运算符
假设变量a=10 b=20

|运算符 |   说明    |   用法示例
| ------| ---------| ------|
| -eq   | 检测两个数是否相等 | [ $a -eq $b ] 返回 false
| -nq   | 检测两个数是否不相等| [ $a -nq $b]
| -gt   | >
| -lg   | < 
| -ge   | >=
| -le   | <=

### 布尔运算符

|运算符|说明|用法
|-----|-----|----
| !   | 非运算 | [ !false ] 返回true
| -o  | 或运算 | [ $a -lt 20 -o $b -gt 100]
| -a  | 与运算

### 逻辑运算符
|运算符|说明|用法
|------|----|----|
| &&    |逻辑AND| [[ $a -lt 100 && \$b -gt 100 ]]
| `||`  |逻辑OR | 

```
#!/bin/bash

a=10
b=20

if [[ $a -lt 100 && $b -gt 100 ]]
then
   echo "true"
else
   echo "false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
   echo "true"
else
   echo "false"
fi
```

### 字符串运算符
假设变量a="abc" b="efg"

|运算符|说明|举例|
|------|----|----|
|  =   | 检测两个字符串是否相等| [ $a =$b ]返回false
| !=   | | |
| -z   | 检测字符串长度是否为0，为0返回true| [ -z $a ] false
| -n   | 检测字符串长度是否不为0           | [ -n "$a" ] true
| str  | 检测字符串是否为空，不为空返回true| [ $a ] true

```
tips


使用 [[ ... ]] 条件判断结构，而不是 [ ... ]，能够防止脚本中的许多逻辑错误。
比如，&&、||、< 和 > 操作符能够正常存在于 [[ ]] 条件判断结构中，但是如果出现在 [ ] 结构中的话，会报错。
```


### printf输出
```
printf 命令模仿 C 程序库（library）里的 printf() 程序。

printf 由 POSIX 标准所定义，因此使用 printf 的脚本比使用 echo 移植性好。

printf 使用引用文本或空格分隔的参数，外面可以在 printf 中使用格式化字符串，还可以制定字符串的宽度、左右对齐方式等。默认 printf 不会像 echo 自动添加换行符，我们可以手动添加 \n。

printf 命令的语法：

printf  format-string  [arguments...]
参数说明：

format-string: 为格式控制字符串
arguments: 为参数列表。

#!/bin/bash

 
printf "%-10s %-8s %-4s\n" 姓名 性别 体重kg  
printf "%-10s %-8s %-4.2f\n" 郭靖 男 66.1234 
printf "%-10s %-8s %-4.2f\n" 杨过 男 48.6543 
printf "%-10s %-8s %-4.2f\n" 郭芙 女 47.9876 

# format-string为双引号
printf "%d %s\n" 1 "abc"

# 单引号与双引号效果一样 
printf '%d %s\n' 1 "abc" 

# 没有引号也可以输出
printf %s abcdef

# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def

printf "%s\n" abc def

printf "%s %s %s\n" a b c d e f g h i j

# 如果没有 arguments，那么 %s 用NULL代替，%d 用 0 代替
printf "%s and %d \n"

#输出
姓名     性别   体重kg
郭靖     男      66.12
杨过     男      48.65
郭芙     女      47.99
1 abc
1 abc
abcdefabcdefabc
def
a b c
d e f
g h i
j  
 and 0 
```


### shell流程控制
if 条件
```
#!/bin/bash

a=10
b=20

if test $a -lt $b; then
	printf '%s\n' "a < b"
elif [[ $a -gt $b ]]; then
	echo "a > b"
fi


```

for 循环
```
for a in 1 2 3 4 5; do
	echo $a  #不要忘了变量前加 $
done

#第二种写法,类似于C语言写法

#!/bin/bash
for((i=1;i<=5;i++));do
    echo "这是第 $i 次调用"
done;

```

while 语句
```
while condition
do
    command
done


#!/bin/bash

i=1
while(( $i<5 ))
do
	echo $i
	i=`expr $i + 1`
done

```

until 语句
```
#!/bin/bash

a=0

until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

case 语句
```
case的语法和C family语言差别很大
它需要一个esac（就是case反过来）作为结束标记
每个case分支用右圆括号，用两个分号表示break。


#!/bin/bash
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in               #变量后面必须跟上 in
    1)  echo '你选择了 1'   #每一个模式必须以右括号结束
    ;;                      #相当于break;
    2)  echo '你选择了 2'
    ;;
    3)  echo '你选择了 3'
    ;;
    4)  echo '你选择了 4'
    ;;
    # * 相当于 default
    *)  echo '你没有输入 1 到 4 之间的数字'
    ;;
esac    #结束case 
```

跳出循环
```
break
continue
```

### 函数

```
#!/bin/bash
#定义函数
demoFun(){
	echo "第一个shell函数"
}

#执行函数
demoFun


###带返回值的函数
funcWithReturn () {
	echo "输入a"
	read a
	echo "输入b"
	read b

	echo -e "a + b = \c"
	return $(($a + $b))
}
#执行函数
funcWithReturn
#通过 $?获取函数执行后的返回值
echo "$?"


###给函数传参
funWithParam () {
	echo $@
	echo "参数总数有 $# 个!"
    echo "作为一个字符串输出所有参数 $* !"
	i=1
	for arg in $@; do
		echo "第${i}个参数为 $arg"
		i=$(($i+1))
	done
}

funWithParam 1 2 3 4 5


```
