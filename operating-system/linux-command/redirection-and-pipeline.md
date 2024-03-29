# 管道和重定向

## :pencil2: 管道

在Linux中有很多标准的命令例如`find、sort、du`等，可以满足我们完成文档管理、系统管理等诸多需求，但是大多时候一些复杂的需求都需要多个命令搭配起来使用，对于Linux来说一个命令对应于一个进程，因此多个命令协同工作，就涉及到多个进程的通信，Linux提供一种管道的方式来完成进程间通信。

&#x20;管道在Linux中对应管道符号 `|`。

```bash
CommandA | CommandB

CommandA | xargs CommandB
```

CommandA执行的输出作为CommandB的输入，例如：

```bash
ls -l | xargs head -10   # 列出前十个文件
```

## :pencil2: 重定向

输出重定向符号： `>`(覆盖)， `>>` (追加)。重定向到文件时，如果文件不存在，这两个命令都会首先创建这个文件。

系统中默认的文件描述符号：

* 0 标准输入
* 1 标准输出
* 2 标准错误输出

例如：

```bash
ls -l 1 > file 2 > /dev/null # 将输出重定向到file，且将错误输出重定向到/dev/null中，1可以省略

command 1> a.txt 2>&1        # 将错误输出和正确输出保存到同一个文件

command 1>> a.txt 2>&1       # 将错误的和正确的信息重定向追加到同一个文件，注意不是2>>&1
```

这里`/dev/null`只有个特殊的文件，重定向这里的输出都会被其丢弃，因此有时候为了抑制错误输出，则只需要将其重定向到黑洞`/dev/null`即可。

输出重定向符号：`<`和`<<`

```bash
CommandA < file   # 输入重定向到file

CommandA << tag   # 将开始标记 tag 和结束标记 tag 之间的内容作为输入。
```
