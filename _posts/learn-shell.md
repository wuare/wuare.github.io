---
title: Shell脚本简单知识
date: 2019-04-23 22:24:39
tags: Shell
---
Shell脚本语法学习笔记
<!-- more -->
## 变量
### 变量定义
variable=value
variable='value'
variable="value"
variable是变量名，value是赋给变量的值。=周围不能有空格。
变量的命名规范
* 变量名由数字、字母、下划线组成
* 必须以字母或者下划线开头
* 不能使用shell关键字

### 使用变量
使用一个定义过的变量，只要在变量名前面加美元符号$即可
```bash
name="名字"
echo $name
echo ${name}
```
### 单引号和双引号的区别
单引号里面的值是什么就输出什么，双引号里面的值会先解析里面的变量和命令
### 将命令的结果赋值给变量
常见的有以下两种方式
variable=`command`
variable=$(command)
例：
log=$(cat log.txt)
echo $log
## 变量作用域
局部变量：只能在函数内部使用。
全局变量：在当前Shell进程中使用。
环境变量：可以在子进程中使用。
shell函数中定义的变量默认是局部变量
```bash
# 定义函数
function func() {
	a = 1
}
# 调用函数
func

echo $a

#输出结果：1
```
在定义变量时加`local`命令，此时变量就成了局部变量。

全局变量的作用范围是当前的 Shell 进程。**打开一个 Shell 窗口就创建了一个 Shell 进程，打开多个 Shell 窗口就创建了多个 Shell 进程，每个 Shell 进程都是独立的，拥有不同的进程 ID**。
如果使用export命令将全局变量导出，那么它就在所有的子进程中也有效了，这称为“环境变量”。
## 位置参数
执行shell脚本可以传递参数，在脚本文件内使用`$n`的方式接收参数。函数内部使用同样的方式来接收。
## Shell特殊变量及含义
| 变量 | 含义 |
| ------ | ------- |
| $0 | 当前脚本的文件名 |
| $n | 传递给脚本或函数的参数，n为数字 |
| $# | 传递给脚本或函数的参数个数 |
| $* | 传递给脚本或函数的所有参数 |
| $@ | 传递给脚本或函数的所有参数，当被双引号`""`包含时，$@和$*不同 |
| $? | 上个命令的退出状态，或函数的返回值 |
| $$ | 当前Shell进程ID |

`$*`和`$@`不被双引号`" "`包围时，它们之间没有任何区别，都是将接收到的每个参数看做一份数据，彼此之间以空格来分隔。
当被双引号包含时，`$*`会将所有的参数从整体上看做一份数据，而不是把每个参数都看做一份数据。`$@`仍然将每个参数都看作一份数据，彼此之间是独立的。
例：
```bash
echo "print each param from \"\$*\""
for var in "$*"
do
    echo "$var"
done

echo "print each param from \"\$@\""
for var in "$@"
do
    echo "$var"
done
```
输出结果：
```
[wwxinjie@izhp3ik0nolcnxbu8oe8tiz learn-shell]$ ./test05.sh a b c d
print each param from "$*"
a b c d
print each param from "$@"
a
b
c
d

```
## 字符串
获取字符串的长度：
```bash
${#string_name}
```
字符串拼接：`$name$age`（中间不能有空格）或者`"$name $age"`（中间空格是新字符串的一部分）
截取字符串：`${string: start: length}`
## 数组
用括号表示数组，数组元素之间用空格分割：`arr=(e1 e2 e3)`
数组是弱类型的，不要求所有元素类型相同，例：
```bash
arr=(1 2 3 4 5)
oarr=(1 2 "aaa")
# 数组的长度不是固定的，可以增加元素
arr[5]=6
```
获取数组元素：`${array_name[index]}`
使用`*`或`@`可以获取所有元素：`${arr[*]}`或者`${arr[@]}`
获取数组长度：
```bash
${#array_name[*]}
${#array_name[@]}
```
删除数组元素：`unset array_name[index]`
## 语句
if else语句：
```bash
if conditon
then
	statement(s)
fi
# 或者
if condition; then
statement(s)
fi
```
case in语句：
```bash
case expression in
    pattern1)
        statement1
        ;;
    pattern2)
        statement2
        ;;
    pattern3)
        statement3
        ;;
    ……
    *)
        statementn
esac
```
case 会将 expression  的值与 pattern1、pattern2、pattern3 逐个进行匹配，如果 expression 和某个模式（比如 pattern2）匹配成功，就会执行这模式（比如 pattern2）后面对应的所有语句（该语句可以有一条，也可以有多条），直到遇见双分号;;才停止；然后整个 case 语句就执行完了，程序会跳出整个 case 语句，执行 esac 后面的其它语句。如果 expression 没有匹配到任何一个模式，那么就执行`*)`后面的语句`（*`表示其它所有值），直到遇见双分号;;或者esac才结束。`*)`相当于多个 if 分支语句中最后的 else 部分。
while语句：
```bash
while condition
do
	statements
done
```