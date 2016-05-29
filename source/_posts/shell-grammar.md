layout: post
title: Shell流程控制语句学习
tags: Shell
date: 2016-05-29 09:44:21
---

本博文主要讲述Shell \*sh 的流程控制语句,主要有分支语句和循环语句，分支语句包括if、case。循环语句有for、while、until。
<!--more-->
# 分支语句 #

## if 语句 ##

语法：
```shell
if cond1
  then
  cmd 1
  ...
  cmd n
elif cond2
  then
  cmd 1
  ...
  cmd n
else
  cmd 1
  ...
  cmd n
fi
```
tips:
- else 之后没有 then 命令
- 最后一定要用 fi 结束

这些是语法上的强制要求，违反就会运行报错

## case语句 ##

语法：
```shell
case $value in
regex1)
  cmd 1
  ...
  cmd n
  ;;
... #多个case
esac
```

tip:
- 最后一个case可加可不加;;,其他case最后必须有;;命令
- 最后必须要有 esac 结束命令
- 匹配条件可以有多个，用 | 并列

匹配规则

- * 任意多个字符
- ? 单个字符
- [] 在[]里面的任意单个字符

Demo
```shell
case $1 in
1|2|3)
  echo "input: " $1
  ;;
4?)
  echo "input: " 40~49
  ;;
[a-z]|[A-Z]j)
  echo "input: letter"
  ;;
*)
  echo "any"
esac
```
---

# 循环语句 #

## for语句 ##

语法：
```shell
for i in $(seq start end)
do
  cmd 1
  ...
  cmd n
done
```
notes:

- 循环变量和起始和结束变量都没有带 $ 符号

Demo
```shell
for i in $(seq 0 10)
do
  echo hello world!
done
```

## while语句 ##
```shell
while condition
do
  cmd1
  ...
  cmd2
done
```
Demo
```shell
read i
declare -i target=10
while ((i<target))
  do
    echo $i
    let i++
  done
```

## until语句 ##

语法：
```shell
until condition
do
  cmd 1
  ...
  cmd n
done
```
tips:
- 条件为假的时候执行循环

Demo
```shell
read i
declare -i target=10
until ((i > target))
do
  echo $1
  ((i++))
done
```
---

- 和c语言比较,关键字条件之间必须有空格
