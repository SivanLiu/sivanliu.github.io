---
layout: linux
title: Linux 常用命令
date: 2018-01-28 12:11:47
tags:
---

平时经常用到 linux shell 命令，今天就来总结下使用频率较高的命令，便于以后查询。

### 一、ls 命令
ls 命令就是list的缩写，用来打印出当前目录的清单，如果 ls 指定其他目录那么就会显示指定目录里的文件及文件夹清单。 通过ls 命令不仅可以查看linux文件夹包含的文件，而且可以查看文件权限(包括目录、文件夹、文件权限)，查看目录信息等等。

#### 1. 命令格式：

```shell
ls [选项] [目录名]
```
#### 2. 常用参数：
| 选项参数 | 说明 |
| --- | --- |
| -a | all 列出目录下的所有文件，包括以 . 开头的隐含文件|
| -A | 同 -a，但不列出"."(表示当前目录)和“..”(表示当前目录的父目录) |
| -c | 根据 ctime 排序及显示 ctime (文件状态最后更改的时间)配合 -l：显示 ctime 但根据名称排序否则：根据 ctime 排序  |
| -d | –directory 将目录象文件一样显示，而不是显示其下的文件。 |
| -D | –dired产生适合 Emacs 的 dired 模式使用的结果 |
| -f | 对输出的文件不进行排序，-aU 选项生效，-lst 选项失效 |
| -g | 类似 -l,但不列出所有者 |
| -G | –no-group 不列出任何有关组的信息 |
| -h | –human-readable 以容易理解的格式列出文件大小 (例如 1K 234M 2G) |
| -H | –dereference-command-line 使用命令列中的符号链接指示的真正目的地 |
| -i | –inode 印出每个文件的 inode 号 |
| -k |  –block-size=1K,以 k 字节的形式表示文件的大小 |
| -l | 除了文件名之外，还将文件的权限、所有者、文件大小等信息详细列出来 |
| -m | 所有项目以逗号分隔，并填满整行行宽 |
| -r  | –reverse 依相反次序排列 |
| -s | –size 以块大小为单位列出所有文件的大小 |

#### 3.示例：
* 列出当前文件下的所有文件和目录的详细资料：

```shell
ls -l -R .
```
* 列出当前目录中所有以“t”开头的目录的详细内容：

```shell
ls -l t**
```

* 只列出文件下的子目录：

```shell
ls -F . |grep /$  
```

* 列出目前工作目录下所有名称是 s 开头的文件：

```shell
ls -ltr s*
```

* 列出当前工作目录下所有档案及目录，目录于名称后加"/", 可执行档于名称后加"*" ：

```shell
ls -AF
```

* 计算当前目录下的文件数和目录数：

```shell
ls -l * |grep "^-"|wc -l ---文件个数  

ls -l * |grep "^d"|wc -l    ---目录个数
```

* 列出文件的绝对路径：

```shell
ls | sed "s:^:`pwd`/:"
```

* 列出当前目录下的所有文件（包括隐藏文件）的绝对路径，对目录不做递归：

```shell
find $PWD -maxdepth 1 | xargs ls -ld
```

* 递归列出当前目录下的所有文件（包括隐藏文件）的绝对路径：

```shell
find $PWD | xargs ls -ld 
```

### 二、cd 命令：Linux cd 命令可以说是 Linux 中最基本的命令语句，其他的命令语句要进行操作，都是建立在使用 cd 命令上的

#### 1.命令格式：

```shell
cd [目录名]
```

#### 2.示例：
* 进入系统根目录：

```shell
cd /
```

* 使用 cd 命令进入当前用户主目录：

```shell
cd
```

* 跳转到指定目录：

```shell
cd ~/Downloads
```

* 返回进入此目录之前所在的目录：

```shell
cd -
```

* 把上个命令的参数作为 cd 参数使用：

```shell
cd !$
```

### 三、pwd 命令来查看当前工作目录的完整路径

#### 1. 命令格式：

```shell
pwd [选项]
```
#### 2. 常用参数：一般情况下不带任何参数，如果目录是链接时：

```shell
pwd -P  显示出实际路径，而非使用连接（link）路径
```

#### 3. 示例：
* 查看默认工作目录的完整路径：

```shell
pwd
```

* 查看指定文件夹：

```shell
pwd ~/Downloads
```

* 目录连接链接时，pwd -P 显示出实际路径，而非使用连接（link）路径；pwd显示的是连接路径：

```shell
pwd -p
```

* 查看默认工作目录的完整路径：

```shell
/bin/pwd [选项]
选项：
-L 目录连接链接时，输出连接路径
-P 输出物理路径

SivanLiu:Downloads sivanliu$ /bin/pwd -L
/Users/sivanliu/Downloads

SivanLiu:Downloads sivanliu$ /bin/pwd -P
/Users/sivanliu/Downloads
```

### 四、mkdir 命令，用来创建指定的名称的目录，要求创建目录的用户在当前目录中具有写权限，并且指定的目录名不能是当前目录中已有的目录。

#### 1. 命令格式：

```shell
mkdir [选项] 目录...
```

#### 2. 命令参数：

| 参数选项 | 说明 |
| :-: | :-: |
| -m | --mode=模式，设定权限<模式> (类似 chmod)|
| -p  | --parents 可以是一个路径名称。此时若路径中的某些目录尚不存在，加上此选项后,系统将自动建立好那些尚不存在的目录，即一次可以建立多个目录|
| -v | --verbose  每次创建新目录都显示信息 |

#### 3. 示例：

* 创建一个空目录：

```shell
mkdir test
```

* 递归创建多个目录：

```shell
mkdir -p parent/son
```

* 创建权限为 777 的目录 ：

```shell
mkdir -m 777 test 
```

#### 五、rm 命令，为删除一个目录中的一个或多个文件或目录，它也可以将某个目录及其下的所有文件及子目录均删除。对于链接文件，只是删除了链接，原有文件均保持不变。

#### 1. 命令格式：

```shell
rm [选项] 文件… 
```
#### 2. 命令参数：

| 参数选项 | 说明 |
| --- | --- |
| -f | --force 忽略不存在的文件，从不给出提示 |
| -i | --interactive 进行交互式删除 |
| -r | --recursive 指示 rm 将参数中列出的全部目录和子目录均递归地删除。 |
| -v | --verbose 详细显示进行的步骤 |

#### 3. 示例：
* 删除文件file，系统会先询问是否删除：

```shell
rm 文件名
``` 

* 强行删除file，系统不再提示：

```shell
rm -f 文件名
```

* 删除 test 文件，删除前逐一询问确认 ：

```shell
rm -i test
```

* 将 test 子目录及子目录中所有档案删除：

```shell
rm -r test
```

* rm -rf test 命令会将 test 子目录及子目录中所有档案删除，并且不用一一确认：

```shell
rm -rf test
```

* 删除以 -f 开头的文件：

```shell
rm -- -f
```

### 六、rmdir 命令，删除空目录，一个目录被删除之前必须是空的。

#### 1. 命令格式：

```shell
mdir [选项]... 目录...
```

#### 2. 命令参数：

| 参数选项 | 说明 |
| --- | --- |
| -p | 递归删除目录 dirname，当子目录删除后其父目录为空时，也一同被删除 |
| -v | --verbose 显示指令执行过程  |

#### 3. 示例：

* rmdir 不能删除非空目录：

```shell
rmdir test(test 为空)
```

* rmdir -p 当子目录被删除后使它也成为空目录的话，则顺便一并删除 ：

```shell
rmdir -p test
```

### 七、mv 命令是 move 的缩写，用来移动文件或者将文件改名（move (rename) files），是Linux系统下常用的命令，经常用来备份文件或者目录

#### 1. 命令格式：

```shell
mv [选项] 源文件或目录 目标文件或目录
```
#### 2. 命令参数：

| 参数选项 | 说明 |
| --- | --- |
| -b | 若需覆盖文件，则覆盖前先行备份 |
| -f | force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖 |
| -i | 若目标文件 (destination) 已经存在时，就会询问是否覆盖 |
| -u | 若目标文件已经存在，且 source 比较新，才会更新(update)|
| -t | 指定 mv 目标目录，该选项适用于移动多个源文件到一个目录的情况 |

#### 3. 示例：

* 文件改名：

```shell
mv file1.txt file2.txt
```

* 移动文件：

```shell
mv file1.txt file
```

* 将文件 file1.txt，file2.txt，file3.txt 移动到目录 test 中：

```shell
mv file1.txt file2.txt file3.txt test
```

* 将文件 file1 改名为 file2，如果 file2 已经存在，则询问是否覆盖：

```shell
mv -i file1 file2
```

* 将文件 file1 改名为 file2，即使 file2 存在，也是直接覆盖掉。：

```shell
mv -f file1 file2
```

* 目录的移动：

```shell
mv test1 test2
```

* 移动当前文件夹下的所有文件到上一级目录：

```shell
mv * ../
```

* 把当前目录的一个子目录里的文件移动到另一个子目录里：

```shell
mv test/* test2
```

* 文件被覆盖前做简单备份，前面加参数-b：

```shell
mv file1.txt -b file2.txt
```

### 八、cp 命令用来复制文件或者目录，在命令行下复制文件时，如果目标文件已经存在，就会询问是否覆盖，不管你是否使用 -i 参数。但是如果是在 shell 脚本中执行 cp 时，没有 -i 参数时不会询问是否覆盖。

#### 1. 命令格式：

```shell
cp [选项]... [-T] 源 目的
```

#### 2. 命令参数：

| 参数选项 | 说明 |
| --- | --- |
| -a | --archive 等于-dR --preserve=all --backup[=CONTROL    为每个已存在的目标文件创建备份 |
| -b | 类似--backup 但不接受参数 |
| -d | 等于--no-dereference --preserve=links |
| -f | 如果目标文件无法打开则将其移除并重试(当 -n 选项存在时则不需再选此项) |
| -i | 覆盖前询问(使前面的 -n 选项失效) |
| -l | 链接文件而不复制 |
| -L | 总是跟随符号链接 |
| -n | 不要覆盖已存在的文件(使前面的 -i 选项失效) |
| -P | 不跟随源文件中的符号链接 |
| -p | 等于--preserve=模式,所有权,时间戳 |
| -R | --recursive 复制目录及目录内的所有项目 |

#### 3. 示例：

* 复制单个文件到目标目录，文件在目标文件中不存在：

```shell
cp file1.txt test
```

* 将文件 file 复制到目录 ~/Downloads/test下，并改名为 file1：

```shell
cp file ~/Downloads/test/file1
```

* 将目录 files 下的所有文件及其子目录复制到目录 ~/Downloads/test 中：

```shell
cp -r files ~/Downloads/test
```

* 交互式地将目录files 中的以 m 打头的所有 .txt 文件复制到目录 ~/Downloads/test 中：

```shell
cp -i files m*.txt ~/Downloads/test
```

### 九、touch 在新疆文件的时候可能会用到，用来修改文件时间戳，或者新建一个不存在的文件

#### 1. 命令格式：

```shell
touch [选项]... 文件...
```

#### 2. 命令参数：

| 参数选项 | 说明 |
| --- | --- |
| -a | --time = atime 或 --time = access 或 --time = use 只更改存取时间 |
| -c | --no-create 不建立任何文档  |
| -d | 使用指定的日期时间，而非现在的时间 |
| -f | 此参数将忽略不予处理，仅负责解决 BSD 版本 touch 指令的兼容性问题 |
| -m | -time = mtime 或 --time=modify 只更改变动时间。 |
| -r | 把指定文档或目录的日期时间，统统设成和参考文档或目录的日期时间相同 |
| -t | 使用指定的日期时间，而非现在的时间 |

#### 3. 示例：

* 创建不存在的文件：

```shell
touch file.txt
```

* 更新 file1.txt 的时间和 file2.txt 时间戳相同：

```shell
touch -r file1.txt file2.txt
```

* 设定文件的时间戳：

```shell
touch -t 201801142234.54 file.txt
```

### 十、cat 是连接文件或标准输入并打印，常用来显示文件内容，或者将几个文件连接起来显示，或者从标准输入读取内容并显示，它常与重定向符号配合使用

#### 1. 命令格式：

```shell
cat [选项] [文件]...
```

#### 2. 命令参数：

| 参数选项 | 说明 |
| --- | --- |
| -A | --show-all 等价于|
| -b | --number-nonblank 对非空输出行编号|
| -E | 在每行结束处显示 $ |
| -n | 对输出的所有行编号，由 1 开始对所有输出的行数编号|
| -s | 有连续两行以上的空白行，就代换为一行的空白行  |
| -T | 将跳格字符显示为 ^I |

#### 3. 示例：

* 把 file1.txt 的文件内容加上行号后输入 file2.txt 这个文件里：

```shell
cat -n file1.txt file2.txt
```

* 把 file1.txt 和 file2.txt 的文件内容加上行号（空白行不加）之后将内容附加到 file3.txt 里。：

```shell
cat -b file1.txt file2.txt file3.txt
```

* 使用 here doc 生成文件：

```shell
cat >file1.txt<<EOF
```

先写到这儿了，后续再补充！


