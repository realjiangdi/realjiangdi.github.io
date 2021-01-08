---
layout: post
title: Shell 笔记
tags: Linux
---
# 基础 Basics
变量赋值时 需要注意等号两端无需空格。
> Value assignment is done using the "=" sign. Note that no space permitted on either side of = sign when initializing variables.


变量也可以使用某个命令的输出
```shell
FILELIST=`ls`
FileWithTimeStamp=/tmp/my-dir/file_$(/bin/date +%Y-%m-%d).txt
```


变量的运算需要再如下表达式`$((expression))`中实现
```
A=3
B=$((100 * $A + 5))
```

查看字符串的长度
```
#!/bin/bash
STRING="This is a string"
echo ${#STRING}         #16
```

字符串子串提取
```
STRING="this is a string"
POS=1
LEN=3
echo ${STRING:$POS:$LEN} # his
```

数组
```

# basic construct
# array=(value1 value2 ... valueN)
array=(23 45 34 1 2 3)
#To refer to a particular value (e.g. : to refer 3rd value)
echo ${array[2]}

#To refer to all the array values
echo ${array[@]}

#To evaluate the number of elements in an array
echo ${#array[@]

```

# 流程 与 循环
## if else
```
if [ expression ]; then

else

fi

"$a" = "$b"     $a is the same as $b
#note1: whitespace around = is required
#note2: use "" around string variables to avoid shell expansion of special characters as *
```

## case
```
case "$variable" in
    "$condition1")

    ;;

    "$conditon2")

    ;;
esac
```
## for
```
for arg in [list]
do
    commands
done

# loop on array member
NAMES=(Joe Jenny Sara Tony)
for N in ${NAMES[@]} ; do
  echo "My name is $N"
done


# loop on command output results
for f in $( ls prog.sh /etc/localtime ) ; do
  echo "File is: $f"
done
```

## while
```
while [condition]
do
    commands
done

COUNT=4

while [ $COUNT -gt 0 ]; do
  echo "Value of count is: $COUNT"
  COUNT=$(($COUNT - 1))
done
```

# 参考资料
- [Learn Shell - Free Interactive Shell Tutorial](https://www.learnshell.org/)
- [Shell Scripting Tutorial](https://www.shellscript.sh/)
