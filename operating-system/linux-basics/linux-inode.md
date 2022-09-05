# 理解inode

**inode**是一个重要概念，是理解Unix/Linux文件系统和硬盘储存的基础。

## :pencil2: 1、`inode`是什么

理解`inode`，要从文件储存说起，文件储存在硬盘上，硬盘的最小存储单位叫做"扇区"（Sector）。每个扇区储存512字节（相当于0.5KB）。

操作系统读取硬盘的时候，不会一个扇区一个扇区地读取，这样效率太低，而是一次性地连续读取多个扇区，即一次次那个读取一个“块”（block）。这种由多个扇区组成的块，是文件存取的最小单位。“块”的大小，最常见的是4KB，即连续八个sector组成一个`block`。

文件数据都存储在“块”中，很显然，我们必须找到一个地方存储文件的元信息，比如文件的创建者、文件的创建日期、文件的大小等。这种存储文件元信息的区域叫做`inode`，中文译名为“索引节点”。每一个文件都有对应的`inode`，里面包含了与该文件有关的一些信息。

`inode`包含文件的元信息，具体来说有字节数、属主`UserID`、属组`GroupID`、读写执行权限、时间戳等。 而文件名存放在目录当中，但`Linux`系统内部不使用文件名，而是使用`inode`号码识别文件。对于系统来说文件名只是`inode`号码便于识别的别称。可以用`stat`命令查看某个文件的`inode`信息：

![](../../.gitbook/assets/2020-07-06\_164519.png)

三个主要的时间属性：

`ctime`：`change time`是最后一次改变文件或目录（属性）的时间，例如执行`chmod`，`chown`等命令。\
`atime`：`access time`是最后一次访问文件或目录的时间。\
`mtime`：`modify time`是最后一次修改文件或目录（内容）的时间。

## :pencil2: 2、inode大小

`inode`也会消耗硬盘空间，所以硬盘格式化的时候，操作系统自动将硬盘分成两个区域。一个是数据区，存放文件数据；另一个是`inode`区（inode table），存放`inode`所包含的信息。

每个`inode`节点的大小，一般是128字节或256字节。`inode`节点的总数，在格式化时就给定，**一般是每1KB或每2KB就设置一个`inode`**。假定在一块1GB的硬盘中，每个`inode`节点的大小为128字节，每1KB就设置一个`inode`，那么**`inode table`**的大小就会达到128MB，占整块硬盘的12.8%。

查看每个硬盘分区的`inode`总数和已经使用的数量，可以使用**`df -i`**命令。

查看每个`inode`节点的大小，可以用如下命令：

```bash
$ dumpe2fs --help
dumpe2fs 1.44.1 (24-Mar-2018)
dumpe2fs: invalid option -- '-'
Usage: dumpe2fs [-bfghixV] [-o superblock=<num>] [-o blocksize=<num>] device

$ sudo dumpe2fs -h /dev/hda | grep "Inode size"
```

![](../../.gitbook/assets/640.webp)

由于每个文件都必须有一个`inode`，因此有可能发生`inode`已经用光，但是硬盘还未存满的情况。这时，就无法在硬盘上创建新文件。

### :paintbrush: 2.1、特有现象

由于`inode`号码与文件名分离，导致一些`Unix/Linux`系统具备以下几种特有的现象。

1.文件名包含特殊字符，可能无法正常删除。这时直接删除`inode`，能够起到删除文件的作用；

```bash
find ./* -inum 节点号 -delete
```

2.移动文件或重命名文件，只是改变文件名，不影响`inode`号码；\
3.打开一个文件以后，系统就以`inode`号码来识别这个文件，不再考虑文件名。

这种情况使得软件更新变得简单，可以在不关闭软件的情况下进行更新，不需要重启。因为系统通过`inode`号码，识别运行中的文件，不通过文件名。更新的时候，新版文件以同样的文件名，生成一个新的`inode`，不会影响到运行中的文件。等到下一次运行这个软件的时候，文件名就自动指向新版文件，旧版文件的`inode`则被回收。

### :paintbrush: 2.2、inode耗尽故障

由于硬盘分区的`inode`总数在格式化后就已经固定，而每个文件必须有一个`inode`，因此就有可能发生`inode`节点用光，但硬盘空间还剩不少，却无法创建新文件。同时这也是一种攻击的方式，所以一些公用的文件系统就要做磁盘限额，以防止影响到系统的正常运行。

至于修复，很简单，只要找出哪些大量占用`i节点`的文件删除就可以了。

```bash
# 1.先准备一个比较小的硬盘分区/dev/sdb1，并格式化挂载，这里挂载到了/data目录下。
[root@localhost ~]# df -hT /data/
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sdb1      xfs    29M  1.8M   27M   6% /data
# 2.先测试可以正常创建文件。
[root@localhost ~]# touch /data/test{1..5}.txt
[root@localhost ~]# ls /data/
test1.txt  test2.txt  test3.txt  test4.txt  test5.txt
# 3.查看i节点的使用情况。
[root@localhost ~]# df -i /data/
Filesystem     Inodes IUsed IFree IUse% Mounted on
/dev/sdb1       16384     8 16376    1% /data
# 4.编写一个测试程序，创建大量空文件，用于耗尽此分区中的i节点数。
[root@localhost ~]# vim killinode.sh
#!/bin/bash
i=1
while [ $i -le 16376 ]
do
touch /data/file$i
let i++
done
# 5.运行测试程序，结束后查看i节点占用情况，磁盘分区空间使用情况。
[root@localhost ~]# sh killinode.sh
[root@localhost ~]# df -i /data/
Filesystem     Inodes IUsed IFree IUse% Mounted on
/dev/sdb1       16384 16384     0  100% /data
[root@localhost ~]# df -hT /data/
Filesystem     Type  Size  Used Avail Use% Mounted on
/dev/sdb1      xfs    29M   11M   19M  36% /data
# 6.虽然还有很多剩余空间，但是i节点耗尽了，也无法创建创建新文件，这就是i节点耗尽故障。
[root@localhost ~]# touch /data/newfile.txt
touch: cannot touch ‘/data/newfile.txt’: No space left on device
```

## :pencil2: 3、inode号码

每个`inode`都有一个号码，操作系统用`inode`号码来识别不同的文件。这里值得重复一遍，Unix/Linux系统内部不使用文件名，而使用inode号码来识别文件。对于系统来说，文件名只是`inode`号码便于识别的别称或者绰号。

表面上，用户通过文件名打开文件，实际上，系统内部将这个过程分为三步：

1.系统找到这个文件名对应的`inode`号码；\
2.通过`inode`号码，获取`inode`信息；\
3.根据`inode`信息，找到文件数据所在的`block`，并读出数据。

其实系统还要根据`inode`信息，看用户是否具有访问的权限，有就指向对应的数据`block`，没有就返回权限拒绝。

使用`ls -i [fielname]`命令，可以看到文件名对应的inode号码。

## :pencil2: 4、目录文件

Unix/Linux系统中，**目录（directory）也是一种文件**。打开目录，实际上就是打开目录文件。

目录文件的结构非常简单，就是一系列目录项（dirent）的列表。每个目录项，组成由两部分：所包含文件的文件名，以及该文件名对应的inode号码。`ls -i [directory]`命令列出整个目录文件，即文件名和`inode`号码。

通过**`ls`**命令配合**`wc`**命令，可以查看某个文件夹下的文件数量。例如：

```bash
ls -lt /tmp | wc -l
```

## :pencil2: 5、软链接和硬链接

### :pen\_fountain: 5.1、硬链接

一般情况下，文件名和`inode`号码是"一一对应"关系，每个inode号码对应一个文件名。但是，Unix/Linux系统允许多个文件名指向同一个inode号码。

这意味着，可以用不同的文件名访问同样的内容；对文件内容进行修改，会影响到所有文件名；但是，删除一个文件名，不影响另一个文件名的访问。这种情况就被称为"硬链接"（hard link）。

`ln`命令可以创建硬链接：`ln 源文件 目标文件`

![](../../.gitbook/assets/640.png)

运行上面这条命令以后，源文件与目标文件的`inode`号码相同。`inode`信息中有一项叫做"链接数"，记录指向该`inode`的文件名总数，这时就会增加1。反过来，删除一个文件名，就会使得inode节点中的"链接数"减1。当这个值减到0，表明没有文件名指向这个`inode`，系统就会回收这个`inode`号码，以及其所对应`block`区域。

这里顺便说一下目录文件的"链接数"。创建目录时，**默认会生成两个目录项："."和".."，**前者的`inode`号码就是当前目录的`inode`号码，等同于当前目录的"硬链接"；后者的`inode`号码就是当前目录的父目录的inode号码，等同于父目录的"硬链接"。所以，**任何一个目录的"硬链接"总数，总是等于2加上它的子目录总数（含隐藏目录）**。

**需要注意的是不能对目录做硬链接**。通过`mkdir`命令创建一个新目录，其硬链接数应该有`2`个，因为常见的目录本身为`1`个硬链接，而目录下面的隐藏目录`.（点号）`是该目录的又一个硬链接，也算是`1`个连接数。

### :pen\_fountain: 5.2、软链接

除了硬链接以外，还有一种特殊情况：文件A和文件B的inode号码虽然不一样，但是文件A的内容是文件B的路径。读取文件A时，系统会自动将访问者导向文件B。因此，无论打开哪一个文件，最终读取的都是文件B。这时，文件A就称为文件B的"软链接"（soft link）或者"符号链接（symbolic link）。

这意味着，文件A依赖于文件B而存在，如果删除了文件B，打开文件A就会报错：`"`**`No such file or directory`**`"`。这是软链接与硬链接最大的不同：文件A指向文件B的文件名，而不是文件B的`inode`号码，文件B的`inode`"链接数"不会因此发生变化。

`ln -s`命令可以创建软链接：`ln -s 源文文件或目录 目标文件或目录`

![](<../../.gitbook/assets/640 (1).png>)

