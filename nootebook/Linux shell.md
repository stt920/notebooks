# Linux shell

## vim命令

**移动光标的方法**

|     vim命令     | 功能                                    |
| :-------------: | --------------------------------------- |
|   [Ctrl]+[f]    | 屏幕向下移动一页，相当于[Page Down]按键 |
|   [Ctrl]+[b]    | 屏幕向上移动一页，相当于[Page Up]按键   |
| 0或功能键[Home] | 移动到这一行最前面字符处                |
| $或功能键[End]  | 移动到这一行最后面字符处                |
|        G        | 移动到这个文件最后一行                  |
|       nG        | 移动到这个文件的第n行                   |
|       gg        | 移动到这个文件的第一行，相当于1G        |
|    N[enter]     | n为数字，光标向下移动n行                |
|        u        | 复原前一个操作（撤销）                  |

**查找与替换**

|        vim命令        | 功能                                                         |
| :-------------------: | ------------------------------------------------------------ |
|         /word         | 向下寻找一个名称为word的字符串                               |
|        ？word         | 向上寻找一个名称为word的字符串                               |
|           n           | 重复前一个查找操作                                           |
|           N           | 反向进行前一个查找操作                                       |
| :n1,n2s/word1/word2/g | n1,n2为数字。在第n1和n2行之间寻找word1这个字符串，并将字符串替换为word2 |
|  :1,$s/word1/word2/g  | 从第一行到最后一行查找word1字符串，并将该字符串替换为word2   |

**删除、复制与粘贴**

| vim命令 | 功能                                                         |
| :-----: | :----------------------------------------------------------- |
|   x,X   | 在一行中，x为向后删除一个字符（相当于[Del]键），X为向前删除一个字符（相当于[Backspace]键） |
|   nx    | 连续向后删除n个字符                                          |
|   dd    | 删除光标所在的一整行                                         |
|   ndd   | 删除光标所在向下的n行                                        |
|   yy    | 复制光标所在的一行                                           |
|   nyy   | 复制光标所在的向下n行                                        |
|  p，P   | p为将已复制的数据在光标下一行粘贴，P则为粘贴在光标上一行     |

**进入插入或替换的编辑模式**

| vim命令 | 功能                                                         |
| :-----: | ------------------------------------------------------------ |
|   i,I   | 进入插入模式：i为从目前光标所在处插入，I为从目前光标所在行的第一个非空格字符处开始插入 |
|   a,A   | 进入插入模式：a为从目前光标所在的下一个字符处开始插入，A 为从光标所在行的最后一个字符处开始插入 |
|   o,O   | 进入插入模式：o为在目前光标所在的下一行处插入新一行，O为在目前光标所在处的上一行插入新的一行 |
|   r,R   | 进入替换模式：r只会替换光标所在的那一个字符一次，R会一直替换光标所在的字符，直到按下[ESC]为止 |
|  [ESC]  | 退出编辑模式，回到一般模式中                                 |

**命令行的保存离开等**

| vim命令 | 功能                             |
| ------- | -------------------------------- |
| :w      | 将编辑内容写入硬盘文件中         |
| :q      | 离开vim                          |
| :q!     | 若修改文件，不想保存，强制离开   |
| :wq     | 保存后离开，“:wq!”强制保存后离开 |
| :set nu | 显示行号；":set nonu"取消行号    |

## 网络相关

##文本操作

### head、tail

### grep

grep命令常见用法

在文件中搜索一个单词，命令会返回一个包含**“match_pattern”**的文本行：

```shell
grep match_pattern file_name
grep "match_pattern" file_name
```

在多个文件中查找：

```shell
grep "match_pattern" file_1 file_2 file_3 ...
```

输出除之外的所有行 **-v **选项：

```shell
grep -v "match_pattern" file_name
```

标记匹配颜色 **--color=auto** 选项：

```shell
grep "match_pattern" file_name --color=auto
```

使用正则表达式 **-E** 选项：

```shell
grep -E "[1-9]+"
或
egrep "[1-9]+"
```

只输出文件中匹配到的部分 **-o **选项：

```shell
echo this is a test line. | grep -o -E "[a-z]+\."
line.

echo this is a test line. | egrep -o "[a-z]+\."
line.
```

统计文件或者文本中包含匹配字符串的行数 **-c** 选项：

```shell
grep -c "text" file_name
```

输出包含匹配字符串的行数 **-n **选项：

```shell
grep "text" -n file_name
或
cat file_name | grep "text" -n

#多个文件
grep "text" -n file_1 file_2
```

打印样式匹配所位于的字符或字节偏移：

```shell
echo gun is not unix | grep -b -o "not"
7:not

#一行中字符串的字符便宜是从该行的第一个字符开始计算，起始值为0。选项 -b -o 一般总是配合使用。
```

搜索多个文件并查找匹配文本在哪些文件中：

```shell
grep -l "text" file1 file2 file3...
```

grep递归搜索文件

在多级目录中对文本进行递归搜索：

```shell
grep "text" . -r -n
# .表示当前目录。
```

忽略匹配样式中的字符大小写：

```shell
echo "hello world" | grep -i "HELLO"
hello
```

选项** -e** 制动多个匹配样式：

```shell
echo this is a text line | grep -e "is" -e "line" -o
is
line

#也可以使用-f选项来匹配多个样式，在样式文件中逐行写出需要匹配的字符。
cat patfile
aaa
bbb

echo aaa bbb ccc ddd eee | grep -f patfile -o
```

在grep搜索结果中包括或者排除指定文件：

```shell
#只在目录中所有的.php和.html文件中递归搜索字符"main()"
grep "main()" . -r --include *.{php,html}

#在搜索结果中排除所有README文件
grep "main()" . -r --exclude "README"

#在搜索结果中排除filelist文件列表里的文件
grep "main()" . -r --exclude-from filelist
```

使用0值字节后缀的grep与[xargs](http://man.linuxde.net/xargs)：

```shell
#测试文件：
echo "aaa" > file1
echo "bbb" > file2
echo "aaa" > file3

grep "aaa" file* -lZ | xargs -0 rm

#执行后会删除file1和file3，grep输出用-Z选项来指定以0值字节作为终结符文件名（\0），xargs -0 读取输入并用0值字节终结符分隔文件名，然后删除匹配文件，-Z通常和-l结合使用。
```

grep静默输出：

```shell
grep -q "test" filename

#不会输出任何信息，如果命令运行成功返回0，失败则返回非0值。一般用于条件测试。
```

打印出匹配文本之前或者之后的行：

```shell
#显示匹配某个结果之后的3行，使用 -A 选项：
seq 10 | grep "5" -A 3
5
6
7
8
#显示匹配某个结果之前的3行，使用 -B 选项：
seq 10 | grep "5" -B 3
2
3
4
5
#显示匹配某个结果的前三行和后三行，使用 -C 选项：
seq 10 | grep "5" -C 3
2
3
4
5
6
7
8
#如果匹配结果有多个，会用“--”作为各匹配结果之间的分隔符：
echo -e "a\nb\nc\na\nb\nc" | grep a -A 1
a
b
--
a
b
```

###sed

对行做处理

```
[root@localhost ruby] # sed '1,2d' ab           #删除第一行到第二行

[root@localhost ruby] # sed -n '1,2p' ab        #显示第一行到第二行

[root@localhost ruby] # sed -n '/ruby/p' ab    #查询包括关键字ruby所在所有行
```

### awk

将一行分为多个字段处理

