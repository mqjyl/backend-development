# 内存

## :pencil2: 1、`brk & sbrk`

**不属于Linux系统调用，而是C语言提供的。**

**函数原型：**

```c
#include<unistd.h>
int brk(void* addr); 
void* sbrk(intptr_t increment);
```

这两个函数都用来改变 “program break” (程序间断点)的位置，改变数据段长度（Change data segment size），即 **扩展heap的上界`brk`，**实现虚拟内存到物理内存的映射。 `brk()`函数直接修改有效访问范围的末尾地址实现分配与回收。当`addr`参数合理、系统有足够的内存并且不超过最大值时`brk()`函数将数据段结尾设置为`addr`，即间断点设置为`addr`。`sbrk()`函数中：当increment为正值时，间断点位置向后移动increment字节。同时返回移动之前的位置，相当于分配内存。当increment为负值时，位置向前移动increment字节，相当于释放内存，其返回值没有实际意义。当increment为0时，不移动位置只返回当前位置。参数increment的符号决定了是分配还是回收内存。

![](<../../.gitbook/assets/image (1).png>)

**返回值：**

`brk()`成功返回0，失败返回-1并且设置`errno`值为`ENOMEM`。 `sbrk()`成功返回之前的程序间断点地址。如果间断点值增加，那么这个指针（指的是返回之前的间断点地址）是指向分配的新的内存的首地址。如果出错失败，就返回一个指针并设置`errno`全局变量的值为`ENOMEM`。

### 测试

`sbrk()`与`brk()`是C标准函数的底层实现，其机制较为复杂（测试中，死循环是为了查看maps文件，不至于进程消亡文件随之消失）。

![](<../../.gitbook/assets/image (2).png>)

虽然，`sbrk()`与`brk()`均可分配回收兼职，但是我们一般用`sbrk()`分配内存，而用`brk()`回收内存，上例中回收内存可以这样写：

```c
int err = brk(old);
/**或者brk(p);效果与sbrk(-MAX*MAX);是一样的，但brk()更方便与清晰明了。**/
if(-1 == err){
    perror("brk");
    exit(EXIT_FAILURE);
}
```

## :pencil2: 2、`mmap & munmap`

&#x20;`mmap`函数第一种用法是映射磁盘文件到内存中，对该内存区域的存取即是直接对该文件内容的读写；而**`malloc`使用的`mmap`函数的第二种用法，即匿名映射，匿名映射不映射磁盘文件，而是向映射区申请一块内存**。

**函数原型：**

```c
#incldue<sys/mman.h>
void * mmap(void * addr, size_t length,int prot,int flags,int fd,off_t offset);

int munmap(void * addr, size_t length);
// addr为mmap函数返回接收的地址，length为请求分配的长度。
```

| 参数     | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| ------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| start  | 指向欲对应的内存起始地址，通常设为NULL，代表让系统自动选定地址，对应成功后该地址会返回。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| length | 代表将文件中多大的部分对应到内存。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `prot` | <p> 代表映射区域的保护方式，有下列组合：</p><ul><li><code>PROT_EXEC</code>  映射区域可被执行；</li><li><code>PROT_READ</code>  映射区域可被读取；</li><li><code>PROT_WRITE</code>  映射区域可被写入；</li><li><code>PROT_NONE</code>  映射区域不能存取。</li></ul><p>注意<code>PROT_XXXXX</code>与文件本身的权限不冲突，如果在程序中不设定任何权限，即使本身存在读写权限，该进程也不能对其操作</p>                                                                                                                                                                                                                                                                                                                            |
| flags  | <p>会影响映射区域的各种特性：</p><ul><li><code>MAP_FIXED</code>  如果参数 start 所指的地址无法成功建立映射时，则放弃映射，不对地址做修正。通常不鼓励用此旗标。</li><li><code>MAP_SHARED</code>  对应射区域的写入数据会复制回文件内，而且允许其他映射该文件的进程共享。</li><li><code>MAP_PRIVATE</code>  对应射区域的写入操作会产生一个映射文件的复制，即私人的"写入时复制" (copy on write)对此区域作的任何修改都不会写回原来的文件内容。</li><li><code>MAP_ANONYMOUS</code>  建立匿名映射，此时会忽略参数<code>fd</code>，不涉及文件，而且映射区域无法和其他进程共享。</li><li><code>MAP_DENYWRITE</code>  只允许对应射区域的写入操作，其他对文件直接写入的操作将会被拒绝。</li><li><code>MAP_LOCKED</code>  将映射区域锁定住，这表示该区域不会被置换(swap)。</li></ul><p><br>在调用<code>mmap()</code>时必须要指定<code>MAP_SHARED</code> 或 <code>MAP_PRIVATE</code>。</p> |
| `fd`   | open()返回的文件描述词，代表欲映射到内存的文件。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| offset | 文件映射的偏移量，通常设置为0，代表从文件最前方开始对应，offset必须是分页大小的整数倍。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

> 返回值：失败返回`MAP_FAILED`，即`(void * (-1))`并设置`errno`全局变量。成功返回指向`mmap area`的指针`pointer`。
>
> 常见`errno`错误：

> **①`ENOMEM`**：内存不足；\
> **②`EAGAIN`**：文件被锁住或有太多内存被锁住；\
> **③`EBADF`**：参数`fd`不是有效的文件描述符；\
> **④`EACCES`**：存在权限错误，。如果是MAP\_PRIVATE情况下文件必须可读；使用MAP\_SHARED则文件必须能写入，且设置`prot`权限必须为`PROT_WRITE`。\
> **⑤`EINVAL`**：参数`addr`、length或者offset中有不合法参数存在。

范例：利用`mmap()`来读取`/etc/passwd` 文件内容：

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
main(){
    int fd;
    void *start;
    struct stat sb;
    fd = open("/etc/passwd", O_RDONLY); /*打开/etc/passwd */
    fstat(fd, &sb); /* 取得文件大小 */
    start = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    if(start == MAP_FAILED) /* 判断是否映射成功 */
        return;
    printf("%s", start); 
    munmap(start, sb.st_size); /* 解除映射 */
    closed(fd);
}
```

在`mmap`和`munmap`执行过程的任何时刻，被映射文件的`st_atime`可能被更新。若`st_atime`字段在上述情况下没有得到更新，首次对映射区的第一个页索引时会更新该字段的值。

使用`PROT_WRITE` 和 `MAP_SHARED`标志建立起来的文件映射，其`st_ctime(c: change)` 和 `st_mtime(m: modification)` 在对映射区写入之后与`msync()[memory synchronize]`通过`MS_SYNC` 和 `MS_ASYNC[memory asynchronize]`两个标志调用之前会被更新。

### 测试

测试`framebuffer`的小程序：显示红色方块。（对`Framebuffer`进行读写可以直接操作显存，该程序需要在Linux的命令行模式下运行。）

```c
#include <unistd.h>  
#include <stdio.h>  
#include <stdlib.h>  
#include <fcntl.h>  
#include <linux/fb.h>  
#include <sys/mman.h>  

int main()
{
    int fp = 0;
    struct fb_var_screeninfo  vinfo;
    struct fb_fix_screeninfo  finfo;
    long screensize = 0;
    char* fbp = 0;
    int x = 0, y = 0;
    long location = 0;
    fp = open("/dev/fb0", O_RDWR);
    sleep(10);
    if (fp < 0)
    {
        printf("Error : Can not open framebuffer device\n");
        exit(1);
    }
    if (ioctl(fp, FBIOGET_FSCREENINFO, &finfo))
    {
        printf("Error reading fixed information\n");
        exit(2);
    }
    if (ioctl(fp, FBIOGET_VSCREENINFO, &vinfo))
    {
        printf("Error reading variable information\n");
        exit(3);
    }

    screensize = vinfo.xres * vinfo.yres * vinfo.bits_per_pixel / 8;
    /*这就是把fp所指的文件中从开始到screensize大小的内容给映射出来，得到一个指向这块空间的指针*/
    fbp = (char*)mmap(0, screensize, PROT_READ | PROT_WRITE, MAP_SHARED, fp, 0);
    if ((int)fbp == -1)
    {
        printf("Error: failed to map framebuffer device to memory./n");
        exit(4);
    }
    /*这是你想画的点的位置坐标,(0，0)点在屏幕左上角*/
    x = 100;
    y = 100;

    for (x = 100; x < 200; x++)
    {
        for (y = 100; y < 200; y++)
        {
            location = x * (vinfo.bits_per_pixel / 8) + y * finfo.line_length;
            *(fbp + location) = 0;  /* 蓝色的色深 */  /*直接赋值来改变屏幕上某点的颜色*/
            *(fbp + location + 1) = 0; /* 绿色的色深*/
            *(fbp + location + 2) = 200; /* 红色的色深*/
            *(fbp + location + 3) = 0;  /* 是否透明*/
        }
    }

    munmap(fbp, screensize); /*解除映射*/
    close(fp);    /*关闭文件*/
    return 0;
}

```
