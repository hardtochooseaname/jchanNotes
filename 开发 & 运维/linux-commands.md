# 基本命令 

### 用户切换：su
`su username`   
&emsp;&emsp;切换到对应用户  
`su`  
&emsp;&emsp;切换到root  
### 以root权限执行命令：sudo

<br/>

### 关机/重启/挂起：shutdown · reboot
`shutdown [options...] [TIME] [WALL...]`  

&emsp;&emsp;TIME参数设定的是关机等操作执行的时间，时间设定有三种：
1. `hh:mm`设置具体的时间  
2. `+m`设置延后m分钟后执行，默认是`+1`也就是1分钟后执行  
3. `now`表示立刻执行，是`+0`的别名  

&emsp;&emsp;WALL参数设定的是通过系统广播发送给所有登录用户终端上的通知信息，以告诉用户系统将进行什么操作   

关机：  
&emsp;&emsp;`shutdown -h now`  
重启：  
&emsp;&emsp;`shutdown -r now`  
&emsp;&emsp;`reboot`（reboot命令会立刻执行，默认没有延时，它也能用来关机和挂起）  
挂起：  
&emsp;&emsp;`shutdown -halt now`   

<br/>

### 切换目录：cd
`cd dir`  
&emsp;&emsp;切换到指定dir  
`cd ..  `  
&emsp;&emsp;切换到父目录  
`cd` or `cd ~`   
&emsp;&emsp;切换到当前用户的home目录，即"/home/username"  
`cd -`  
&emsp;&emsp;切换到上次的目录


<br/>

### 查看当前目录下的文件：ls
`ls`  
&emsp;&emsp;只列出可见文件  
`ls -a`  
&emsp;&emsp;列出所有包括隐藏的文件  
`ls -l`  
&emsp;&emsp;列出所有可见文件的详细信息  
`ls -al`  
&emsp;&emsp;列出所有包括隐藏文件的详细信息，等同于`ll`
`ls -t`  
&emsp;&emsp;根据修改时间排序，越近修改的在越前面  
`ls -R`  
&emsp;&emsp;递归当前目录，列出所有子目录和子文件  

<br/>

### 查看当前路径：pwd
如果当前目录是链接文件  
&emsp;&emsp;`pwd -p`：返回源dir的实际物理路径  
&emsp;&emsp;`pwd`：返回当前的实际路径

<br/>

### 清屏：clear

<br/>

### 查看历史命令：history
`history`  
&emsp;&emsp;查看之前执行过的命令的历史记录  
快捷键 ctrl+r  
&emsp;&emsp;按下ctrl+r后，输入字符就可以搜索与之匹配的历史命令，可以帮助快速找到使用过的复杂命令，再按ctrl+r可以继续搜索直到找到目标命令

<br/>

### 查看手册文档：man
`man [option] [section] page`  
man 命令通常会将手册页分为多个节（section），每个节包含了特定类型的信息。常见的手册页节包括：  
> Section 1: 用户命令，如常用的命令行工具  
> Section 2: 系统调用和内核函数  
> Section 3: 库函数和函数接口  
> Section 4: 特殊文件和设备  
> Section 5: 文件格式和配置文件  
> Section 6: 游戏和屏幕保护程序  
> Section 7: 杂项  

这也是man命令可以查询的范围，不局限于命令，还能查询系统调用、配置文件等

<br/>

### 查询命令用途：whatis
display one-line manual page descriptions

<br/>



# 文件操作
### 创建文件：touch
`touch [filename]`  
&emsp;&emsp;创建一个名为*filename*的文件  

<br/>

### 查看文件类型：file
`file [filename]`  
&emsp;&emsp;查看文件类型的详细信息  
> `jchan@jchan-G3:~/Downloads$ file google-chrome-stable_current_amd64.deb`  
> `jchan@jchan-G3:~/Downloads$ google-chrome-stable_current_amd64.deb: Debian binary package (format 2.0), with control.tar.xz, data compression xz`  

<br/>

### 拷贝：cp
`cp [source] [dest]`  
&emsp;&emsp;把源文件复制成目标文件  
`cp [source1] [source2] ... [dir]`  
&emsp;&emsp;把多个源文件复制到目标*目录*中  

cp命令的一些默认规则  
- 目的文件已存在的情况下，cp会直接覆盖该文件
- 新文件的所有者和组群会默认更新为执行cp命令的用户，若不想改变权限则需添加选项-p（保留不变：权限mod，拥有者ownership，时间戳timestamp） 
- 复制的文件默认是实际文件而不是链接本身，若想复制链接文件，则需添加-d选项（软&硬链接） 

常用选项：  
> -i：在覆写已存在的目的文件之前发出询问  
> -s：制作源文件的**符号链接**到目标文件  
> -l：制作源文件的**硬链接**到目标文件  
> -r：递归复制源目录中的所有文件，用于**目录**的复制  
> -u：对比源和目的文件，只有源文件有更新或者目的文件丢失才会进行复制，常用于**备份**  

<br/>

### 移动文件：mv
`mv file dir`  
&emsp;&emsp;移动文件到目标目录，可以移动多个file，files用逗号隔开  
`mv oldname newname`  
&emsp;&emsp;**重命名**文件or目录  

常用选项：  
> -i：类似cp，覆盖前询问  
> -b / --backup：用于**备份**，详情consult the man  

<br/>

### 创建目录：mkdir
`mkdir -m 766 dir`  
&emsp;&emsp;指定权限  
`mkdir -p dir/dir1/dir2`  
&emsp;&emsp;递归创建目录（when dir and dir1 don't exist）  

<br/>

### 删除空目录：rmdir
仅能删除**空**目录  
`rmdir -p dir/dir1/dir2`  
&emsp;&emsp;递归删除，从下往上直到非空目录为止  

<br/>

### 删除文件：rm
`rm -f`：忽略不存在的文件，删除之前不询问  
`rm -i`：删除前询问  
`rm -r`：递归删除目录及其中的内容，删除非空目录必须加-r  
`rm -d`：删除空目录  

<br/>

### 制作链接：ln
`ln [options] target link-name`  
&emsp;&emsp;制作*target*的硬链接*link-name*（默认硬链接）  
常用选项：  
> -s：制作符号链接  
> -f：如果由同名文件存在则覆盖  
> -i：如果有同名文件存在则询问  

<br/>

### 查看目录的树状结构：tree
`tree`：查看当前目录  
`tree /dir`：查看指定目录  

常用选项：  
> -l：显示文件的详细信息  
> -h：以可读的方式显示文件大小  
> -o file：把树状结构输出到指定文件而不是stdout  


# 文件权限
### 修改组群：chgrp
`chgrp [options] group file1...`  
### 修改拥有者：chown
`chown [options] owner[:group] file1...`  
两者常用选项：  
> -c：显示每一项修改的信息  
> -h：符号链接文件本身也要修改  
> -R：递归修改指定目录下的所有文件，但是不改变符号链接本身  
> -RH：遇到符号链接只改变其本身  
> -RL：遇到符号链接也要改变其所指向的文件  

<br/>

### 修改权限：chmod
方式一：通过数字修改  
`chmod 744 file`  
&emsp;&emsp;每个file的权限由三组x三个一共九个组成，每组的权限由一个数字表示，该数字由该组所拥有的权限所对应的数字累加所得  
> 读：r = 4  
> 写：w = 2  
> 执行：x = 1  

方式二：通过符号修改
`chmod u=rwx,go=r file`  
&emsp;&emsp;注意：中间那一项之间不能有空格！！
&emsp;&emsp;另外，也可以用+-来直接增加或移除权限  
> u：拥有者  
> g：所属组群  
> o：其他人  
> a：所有人，相当于ugo  

<br/>



# 文件内容的查看与打印
### 从第一行开始打印：cat
`cat [file1][file2]...`  
&emsp;&emsp;以此从每个文件的第一行开始打印  
`cat -n [file1][file2]...`  
&emsp;&emsp;文件每行标记序号,如果有多个文件的时候，这些文件的序号会接续递增，不会在后面的文件又从0开始  

### 从最后一行开始打印：tac
相当于cat倒序打印  

<br/>

### 打印开头数行：head
`head [-n num] [file]`  
&emsp;&emsp;默认打印开头10行，或者用选项n指定行数  

### 打印结尾数行：tail
用法同head

<br/>

### 统计行数
`nl [file]`  
&emsp;&emsp;每行开头标上行号并打印  
&emsp;&emsp;等同于`cat -n [file]`   

<br/>

## 长文件：more & less
### 高配版：less
*Less is more*  
以页面(like man page)的形式显示文件  
> 使用技巧：
> - h：Get help
> - q：quit
> - f：Forward one page
> - b / SPACE：Back one page
> - g：跳到文件开头
> - G：跳到文件结尾
> - 在文件中搜索：
>    - /pattern：从当前位置往下搜索
>    - ?pattern：从当前位置往上搜索
>    - &pattern：只显示匹配到的行
>    - n：跳到下一个匹配到pattern的位置
>    - N：跳到上一个匹配到pattern的位置

### 基础版：more
*More is less*  


<br/>



# 数据处理·文件处理
### 排序：sort
`sort [options] [file1][file2]...`  
&emsp;&emsp;把files合起来，按字典顺序，按行排序  
常用选项:  
> `-o [result]`：把排序的结果输出到*result*中  
> `-f`：忽略大小写差异  
> `-b`：忽略开头的空字符  
> `-r`：反向排序  
> `-n`：改为按照数值大小进行排序  
> `-k <n>`：指定字段号n，然后按照该字段排序  
> `-t <sep>`：指定分隔符sep，sep直接写，不用放引号里面  

<br/>

### 获取非重复行：uniq
`uniq [options] [input]`  
&emsp;&emsp;去掉*input*中的重复行  
&emsp;&emsp;*input*通常可以是文件，也常结合管道命令来处理其他命令的输出结果  

常用选项：  
> `-i`：忽略大小写的不同  
> `-c`：对每行重复次数计数  

<br/>

### 文件内容计数：wc
`wc [file]`  
&emsp;&emsp;统计*file*的行数、单词数、字节数  
> `-c`：字节数  
> `-m`：字符数  
> `-l`：行数  
> `-w`：单词数  

<br/>

### 查找匹配的行：grep -> riprep
`grep [options] pattern [files]`  
&emsp;&emsp;pattern：要查找的字符串或正则表达式  
&emsp;&emsp;files：可以同时查找多个文件  
&emsp;&emsp;注：grep是**以行为单位**进行筛选的，也就是说grep筛选出的是匹配到的字符串所在的行  
常用选项：  
> -i：忽略大小写进行匹配  
> -v：反向查找，只打印不匹配的行  
> -n：显示匹配行（在原文件中）的行号  
> -r：递归查找子目录中的文件   
> -l：只打印匹配的文件名  
> -c：只打印匹配的行数  
> -An：A后面添加的数字n，表示同时列出匹配行的后面n行  
> -Bn：B后面添加的数字n，表示同时列出匹配行的前面n行  

在目录中查找有两种方式：  
1. 只在指定目录中的非目录文件中查找  
`grep -d skip PATTERN dir/*`   
&emsp;&emsp;其中`-d skip`的意思是遇到目录文件就跳过，因为目录文件本身就是个inode，grep本身就不支持对其查找  
2. 在指定目录中递归查找  
`grep -r PATTERN [dir]`  

<br/>

### Stream Editor: sed
`sed [options] operate input`  
&emsp;&emsp;input通常是stdin或者文件  
&emsp;&emsp;对input的处理结果默认是输出到stdout，并不会改变原file  
常用操作（operate）：  
- 删除行：d  
> `'2,5d'`：删除2~5行  
> `'4d'`：删除第四行  
> `'/leave/d'`：删除存在“leave”的行，**两个斜杠之间可用正则表达式**  
- 添加行：a（在某行之后），i（在某行之前）  
> `'6a Welcome back'`： 在第6行之后新增一行"Welcome back"  
> `'6i Uzi'`：在第6行之前新增一行"Uzi"  
> `'6a Forever\[这里按Enter，在下一行自动出现的>之后输入下一行] > God'`：在第6行之后连续添加两行  
- 替换行：c  
> `'3,7c FMVP of S13'`：把第3~7行替换为一行"FMVP of S13"  
- 替换字符串：s  
> `'s/old-pattern/new-words/g'`：三个斜杠隔出四块，s开头g结尾，中间是旧字符串/新字符串  
> `'s/#.*//g'`：删除#开头的行
- 打印选择的行：p  
> `-n '10,20p'`：打印10~20行，必须加上-n选项，否则会输出多余的和重复的行    

常用选项：  
- -i：对文件的操作会直接修改文件本身，不再仅仅输出结果到stdout  
- -n：使用安静模式，只打印选中的行  
- -e：使用多点编辑，即一次性执行多个操作  
> `sed -e '/leave/d' -e '1i ADC: Uzi' EDG.file`  

<br/>

### Text processing language: awk
`awk 'condition1{operation1} condition2{operation2} ...' file`   
&emsp;&emsp;awk是一个复杂且功能强大的文本处理工具，它会把每行数据按分隔字符（默认是空格）分为多个字段，再按行处理行中字段。awk有许多内置变量，也可自己设置变量，awk对文本的处理经常用到这些变量。awk的处理语句括在单引号中，具体操作括在花括号中。  
&emsp;&emsp;awk本质上是一个处理文本文件的语言，可以用它写出包含复杂逻辑的文本处理语句。  

常用内置变量：
- \$0：代表整行数据  
- \$1, $2, ...：代表每行第一个字段、第二个字段……  
- NF：当前行拥有的字段数目  
- NR：当前处理的是第几行  
- FS：当前的分隔字符，默认是空格键  

awk命令的工作流程：  
1. 读入一行，用当前分隔字符分段，再将字段数据写入$0,$1等变量中  
2. 根据condition的判断结果，决定是否执行其后的operation  
3. 依次完成后续所有的condition判定与operation执行  
4. 如果还有下一行，对下一行重复步骤1~3  

使用实例：  
> 以冒号分割字段，取出每行的第一个和第四个字段，用空格分开：  
&emsp;`awk -F: '{print $1 " " $4}' file`  
&emsp;`awk 'BEGIN {FS=":"} {print $1 " " $4}' file` 
&emsp;关键词BEGIN作用是在读入第一行数据之前预先执行设置分隔符的操作，否则第一行的数据依然会以空格分隔  

> 存在字符串"root"的行中，如果第1个字段小于10，则取出第2个和第五个字段：  
&emsp;`grep 'root' file | awk '$1<10 {printf "%-10s %-5s",$2,$5}'`  
&emsp;可以使用printf进行格式化输出，格式串的写法类似于C语言的printf  

更多信息参考以下网址：  
&emsp;&emsp;菜鸟教程 https://www.runoob.com/linux/linux-comm-awk.html  

<br/>



# 查找
### 查找一切：find
`find [path] [expression]`  
&emsp;&emsp;path：要查找的目录，若省略则默认为当前目录  
&emsp;&emsp;expression：可选参数，用于指定查找条件  

> -name pattern：按文件名查找，支持使用通配符 * 和 ?。  
> -type type：按文件类型查找，可以是 f（普通文件）、d（目录）、l（符号链接）等  
> -size [+-]size[cwbkMG]：按文件大小查找，支持使用 + 或 - 表示大于或小于指定大小，单位可以是 c（字节）、w（字数）、b（块数）、k（KB）、M（MB）或 G（GB）  
> -mtime days：按修改时间查找，支持使用 + 或 - 表示在指定天数前或后   
> -user username：按文件所有者查找  
> -group groupname：按文件所属组查找   

&emsp;&emsp;find是一个非常强大且复杂的命令，可以实现很精准的查找，其他许多使用方法ask the man  

<br/>

### The Better One: fd-find/fd
&emsp;&emsp;fd相较于find具有更高效的性能和更快的搜索速度，语法也相对简化，且提供了ripgrep等工具的支持，可以更方便实现一些复杂查找  
&emsp;&emsp;fd默认的查找路径是当前目录以及其中的子目录（不包括符号链接指向的目录），返回的文件路径也是基于当前目录的相对路径    
&emsp;&emsp;fd的语法和find有所不同，能够实现find的大部分功能  

常见选项：  
> -s：case-sensitive，fd默认是case-insensitive  
> -a：返回绝对路径  
> -g：以glob模式，即类似bash通配符的语法来匹配文件，而不是使用正则表达式  
> -F：使用固定字符串精确匹配，而不是用正则表达式，即匹配文件名与字符串完全相同的文件  
> -l：同时列出找到的文件的详细信息（ls -l会列出的那些）  
> -L：把符号链接指向的目录也纳入搜索范围  
> -p：匹配完整路径，即pattern不仅会与文件名匹配，还会与文件的完整路径匹配    
> --max-results=n：查找到n个文件后停止搜索  
> -1：找到一个目标文件就结束搜索，相当于‘--max-results=1’的别名  
> --prune dir：搜索时跳过指定的目录，可以是绝对路径和相对路径，且能够指定多个目录    
> -t：指定搜索的文件类型（f-file, d-dir, l-symlink, x-exec, e-empty），可以同时指定多个类型  
&emsp;&emsp;`fdfind -td -te --max-results=20`：搜索空目录  
&emsp;&emsp;`fdfind -tf -tl`：搜素文件的符号链接  
> -e：搜索指定拓展名的文件  
&emsp;&emsp;`fdfind -e md`：搜索拓展名为md的文件   

<br/>

### 查找命令（可执行文件）：which
`which [command]`  
&emsp;&emsp;在环境变量\$PATH指定的路径内查找命令所在位置  
&emsp;&emsp;\$PATH指定的是可执行文件的搜索路径，例如在shell中执行ls命令，shell就是通过\$PATH找到ls命令的位置，而有些命令例如history是bash的内置命令，不是存放在系统的$PAHT中的，就不能用which查找到  

<br/>

### 查找命令有关的文件：whereis
`whereis [-bsm] [command]`  
> -b：二进制文件/可执行文件  
> -s：源文件  
> -m：命令的帮助文档  
> -l：查看whereis查找的目录范围  

<br/>

### 查找一般文件：locate
locate会为文件系统中的文件建立数据库，在查找文件时直接从数据库中查询，查找速度比find快很多  
但ubantu中没有安装locate，需要手动安装并初始化数据库  
locate的查找不需要记住完整文件名，它会查找所有与输入的pattern匹配的文件，使用方便  

<br/>



# 包的安装管理
### How does it work
&emsp;&emsp;程序的开发者称为upstream provider，他们是所有软件所有packages的提供者  
&emsp;&emsp;provider提供的包都会汇总到各种各样的package repo里，这些repos就相当于software source，是hub of packages  
&emsp;&emsp;各个发行版在安装时都会pre-install一些可靠的软件源（指向这些源/repo的link），以便使用命令行安装包时，系统知道从哪些地方获取我们需要的软件  
> ubantu中这些source存放在文件：`/etc/apt/sources.list`  

### 包安装：dpkg(debian)
`dpkg [-ir] some-debian-package.deb`  
&emsp;&emsp;dpkg的安装只会安装包本身，不包括包的依赖dependencies  
> -i：安装  
> -r：删除  
> -l：列出已安装的packages  

### 功能更强大的package manager：apt
`apt [option] command package-name`  
&emsp;&emsp;apt会连带包的依赖一起安装  
&emsp;&emsp;apt的执行需要root权限  
常用命令：
- 列出所有可更新的软件清单：`sudo apt update`
- 列出指定软件的更新信息：`sudo apt update package-name`      
- 列出可更新的软件包及版本信息：`apt list --upgradeable`  
- 安装指定的软件：`sudo apt upgrade package-name`  
- 更新指定的软件：`sudo apt update package-name`  
- 显示软件包具体信息,例如：版本号，安装大小，依赖关系等等：`sudo apt show package-name`  
- 删除软件包：`sudo apt remove package-name`  
- 移除软件包及配置文件: `sudo apt purge package-name`  
- 查找软件包： `sudo apt search <keyword>`  
- 列出所有已安装的包：`apt list --installed`  
- 列出所有已安装的包的版本信息：`apt list --all-versions`


## 包的压缩打包
### gzip：只压缩单个文件成.gz文件，不能压缩目录
`gzip -k file`  
&emsp;&emsp;压缩file同时保留原文件，否则gz文件会覆盖原文件  
`gzip -c apple > banana.gz`  
&emsp;&emsp;-c把输出流改为标准输出，同时保留原文件，这样可以在压缩的同时改名  
`gzip -d file.gz`  
&emsp;&emsp;解压缩gz文件（默认覆盖原文件）  

### tar：多个文件或目录打包成单一文件
`tar [options] target file1 file2 ...`  
tar打包的核心就是：把一堆文件或目录包装成一个单一的文件，让这一堆乱七八糟的东西以一个单一文件的方式存在
常用选项：  
> -c：压缩包   
> -x：解包，默认解压到当前工作目录  
> -t：列出文档中包含文件的文件名  

> -z：gzip压缩/解压缩  
> -j：bzip2压缩/解压缩  
> -J：xz压缩/解压缩  

> -f：选择目标文件（欲压缩为的 or 待解压的）  
> -v：显示指令执行过程的信息   
> -C：更改输出到指定目录  
> -p：保留文件的原本权限与属性（常用于备份重要的配置文件）  

常见用法：
- `tar -cf archive.tar file1 file2 ...`：打包为tar文件（即archive）  
- `tar -czf package.tar.gz file1 file2 ...`：打包后再gzip压缩为.tar.gz文件  
- `tar -xzvf package.tar.gz`：把.tar.gz文件解压缩并打印详细信息

上大当：  
&emsp;&emsp;用tar打包一个目录时，tar其实是把这个目录看做一个单一的文件，然后再把这一个文件打包成一个.tar文件。
&emsp;&emsp;又因为tar在打包一堆文件（其中包括普通文件和目录）时，不会破坏目录内部的结构，也不会更改任何文件或目录的名字，tar做的只是在这一堆文件外面套层皮，这层皮由用户命名为.tar文件。  
&emsp;&emsp;所以在用tar打包目录时，它做的也只是在目录外面套层皮，把这个目录包含在了.tar文件。于是在解包的时候，里面的目录会被还原出来，就成了一个a.tar文件解包成了一个b目录的故事。

<br/>



# 进程管理
### 查看进程状态：ps 
&emsp;&emsp;ps显示当前机器上运行进程的状态，是一个时刻系统进程的快照，它有非常多的参数，提供了灵活的输出格式和过滤选项，常用于脚本编写和批处理任务  
一些基本用法：  
> `ps au`：列出所有terminal中运行的进程(a)，以及它们的详细信息(u)  
> `ps aux`：列出所有进程的详细信息，不只是和terminal有关的  

<br/>

### “Linux任务管理器”：top
&emsp;&emsp;top动态显示系统状态和进程的运行情况，类似于Windows的任务管理器，可以按照cpu使用率、内存占用等指标来排序，并可以使用交互命令对进程进行操作（例如终止进程）  

进程信息各字段如下：  
> PID：进程的标识符  
> USER：运行进程的用户名  
> PR（优先级）：进程的优先级  
> NI（Nice值）：进程的优先级调整值  
> VIRT（虚拟内存）：进程使用的虚拟内存大小  
> RES（常驻内存）：进程实际使用的物理内存大小  
> SHR（共享内存）：进程共享的内存大小  
> %CPU：进程占用 CPU 的使用率  
> %MEM：进程占用内存的使用率  

top窗口中常见交互操作：  
- 滚动查看进程列表：  
&emsp;&emsp;默认情况下，top显示的是按 CPU 使用率排序的进程列表。你可以使用向上和向下箭头键滚动查看更多进程  
- 切换排序方式：  
&emsp;&emsp;按 M 键可以按内存使用率排序进程列表，按 P 键可以按 CPU 使用率排序进程列表，按 T 键可以按运行时间排序进程列表  
- 进程过滤：  
&emsp;&emsp;按 o 键可以打开进程过滤器，输入关键字可以根据进程名进行过滤。输入过滤条件后，只有符合条件的进程会显示在列表中  
- 终止进程：  
&emsp;&emsp;按 k 键可以终止选定的进程。输入要终止的进程的 PID，然后按回车键确认  
- 刷新频率：  
&emsp;&emsp;按 s 键可以更改刷新频率。输入新的刷新频率（以秒为单位），然后按回车键确认  
- 显示系统概况：  
&emsp;&emsp;按 1 键可以切换到系统概况页面，显示有关 CPU、内存、交换空间和进程总数等系统整体状态的信息  
- 显示帮助：  
&emsp;&emsp;按 h 键可以打开 top 的帮助页面，显示可用的交互命令列表和说明  
<br/>

### 终止进程：kill
`kill [-9] PID`  
&emsp;&emsp;杀死指定PID的进程，若有选项-9则指定一个kill的signal  

<br/>



# 网络信息
### 检测与目标主机的网络连接：ping
`ping <IP或域名>`   
&emsp;&emsp;ping发送 ICMP Echo 请求到目标主机，并接收并显示回应时间和可达性信息  
`ping -c <次数> <IP地址或域名>`  
&emsp;&emsp;指定发送ICMP请求的次数，不然shell会一直发送，直到用ctrl+c打断  
### 跟踪到目标主机的路径：traceroute
`traceroute <IP地址或域名>`：跟踪数据包从源到目标的路径。  
`traceroute -n <IP地址或域名>`：禁用地址解析，以 IP 地址形式显示跟踪结果。  
`traceroute -p <端口号> <IP地址或域名>`：指定目标端口进行跟踪。  
### 查看系统的网络接口信息：ifconfig
`ifconfig`：显示当前系统的网络接口信息。  
`ifconfig <网络接口名称> up`：启用指定的网络接口。  
`ifconfig <网络接口名称> down`：禁用指定的网络接口。  
`ifconfig <网络接口名称> <IP地址> netmask <子网掩码>`：设置网络接口的 IP 地址和子网掩码。  
### 查看网络连接信息：netstat
`netstat`：显示活动的网络连接、监听端口、路由表、网络接口统计信息等。  
`netstat -tuln`：显示所有活动的 TCP 和 UDP 监听端口。  
`netstat -r`：显示系统的路由表。  
`netstat -s`：显示网络统计信息，如接收和发送的数据包数量。  
### 查看和管理系统的路由表：route
`route`：显示当前路由表。  
`route add default gw <默认网关>`：添加默认网关。  
`route del default gw <默认网关>`：删除默认网关。  

<br/>


# 文件系统与磁盘
### 挂载文件系统：mount
`mount`: 显示当前已挂载的文件系统列表  
`mount /dev/sda1 /mnt`: 将设备 /dev/sda1 挂载到 /mnt 目录  
`mount -t ext4 /dev/sdb1 /data`: 指定文件系统类型为 ext4，将设备 /dev/sdb1 挂载到 /data 目录  

### 卸载文件系统：unmount
`umount /mnt`: 卸载挂载点 /mnt 下的文件系统  
`umount /dev/sda1`: 卸载设备 /dev/sda1 上挂载的文件系统  

### 计算目录或文件的磁盘使用情况：du
`du`: 显示当前目录下所有文件和子目录的磁盘使用情况  
`du -h`: 以人类可读的格式显示磁盘使用情况  
`du -sh /path/to/directory`: 显示指定目录的总磁盘使用情况  

### 查看文件系统的磁盘使用情况：df
`df`: 显示所有已挂载文件系统的磁盘空间使用情况   
`df -h`: 以人类可读的格式显示磁盘空间使用情况  
`df -hT`: 显示磁盘空间使用情况，并显示文件系统类型  

### 树状打印块设备上的文件系统信息：lsblk
&emsp;&emsp;lsblk是一个用于列出块设备信息的命令。它显示系统上的磁盘、分区和其他块设备的层次结构和相关信息。相比于df能更清晰地显示出磁盘的层次结构  

`lsblk`: 显示所有块设备的层次结构和详细信息。
`lsblk -a`: 显示所有块设备，包括未挂载的设备。
`lsblk -l`: 以列表的形式显示块设备信息，而不显示子设备（如分区）。
`lsblk -f`: 显示块设备及其文件系统的信息。
`lsblk -o NAME,SIZE,MOUNTPOINT`: 自定义输出列，只显示设备名称、大小和挂载点。

<br/>



# 一些bash使用技巧
### 常用快捷键
Tab：补全命令/文件名，连按两下查看可以补全成那些命令（可以用于查看比如以ls开头的所有命令）  
ctrl+c：终止当前程序  
ctrl+d：退出terminal，相当于exit  
ctrl+u：清除当前输入  
ctrl+a：光标移到行首  
ctrl+e：光标移到行尾    
shift+ctrl+[page up]/[page down]:往前\/后翻页  

<br/>  

### 通配符
*：表示任何字符或字符串   
&emsp;&emsp;`ll abc*`：列出当前目录下所有以“abc”开头的文件的详细信息  
？：只表示单个字符  
[]：表示括号里面有的某个字符  
&emsp;&emsp;`ls apple[89]*` ：会列出apple系列中的"apple8 8G","apple8 16G", "apple9 32G"

<br/>

### 重定向
`command > [file]`  
&emsp;&emsp;输出重定向，把命令的执行结果的输出流从标准输出stdout改为*file*  
`command < [file]`  
&emsp;&emsp;输入重定向，把命令的执行结果的输入流从标准输入stdin改为*file*  

重定向常与其他命令结合使用，以echo为例：
echo与输出重定向同时使用，常用来快速创建文件与追加内容  
echo与输入重定向同时使用，可用来复制文件  
`echo [text] > [file]`  
&emsp;&emsp;输出重定向到*file*，*file*不存在则创建，存在则覆写（text里面有特殊字符需要用引号括起来，有空格不用）  
`echo [text] >> [file]`  
&emsp;&emsp;同上，只不过*file*存在时则追加  
`echo < [file1] > [file2]`  
&emsp;&emsp;把echo的输入流改为*file1*，在把输出流定位到*file2*，从而实现把*file1*复制为*file2*  

<br/>

### 增加一个输出流到文件：tee
&emsp;&emsp;tee从stdin输入数据，然后输出到stdout和指定文件，相当于重定向的同时保留对stdout的输出流  
`ls | tee files-details.txt`

<br/>

### 管道
`command1 | command2`  
&emsp;&emsp;把*command1*的输出作为*command2*的输入,也可以利用多个管道组合多个命令，以完成较复杂的功能    
`ls -la /etc | less`  
&emsp;&emsp;就是把ls的输出结果输入给less，这样就可以很方便地用less来查看ls的输出结果  
