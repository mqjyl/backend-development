# Shell编程

## :pencil2: 1、认识Shell

在计算机科学中，Shell俗称壳（用来区别于核），是指“提供使用者使用界面”的软件（命令解析器）。它类似于DOS下的command和后来的`cmd.exe`。它接收用户命令，然后调用相应的应用程序。同时它又是一种程序设计语言。作为命令语言，它交互式解释和执行用户输入的命令或者自动地解释和执行预先设定好的一连串的命令；作为程序设计语言，它定义了各种变量和参数，并提供了许多在高级语言中才具有的控制结构，包括循环和分支。

![](<../.gitbook/assets/image (22).png>)

### :pen\_fountain: 1.1、Shell分类 <a href="#11shell-fen-lei" id="11shell-fen-lei"></a>

基本上shell分两大类：

一：图形界面shell（Graphical User Interface shell，即 GUI shell）

> 例如：应用最为广泛的 Windows Explorer （微软的windows系列操作系统），还有也包括广为人知的 Linux shell，其中linux shell 包括 X window manager (`BlackBox`和`FluxBox`），以及功能更强大的CDE、GNOME、KDE、 XFCE。

二：命令行式shell（Command Line Interface shell，即CLI shell）

> 例如：
>
> * bash / sh / ksh / csh（Unix/linux 系统），Ken Thompson 的 sh 是第一种 Unix Shell
> * command（MS-DOS系统）
> * cmd.exe / 命令提示字符（Windows NT 系统）
> * Windows PowerShell（支援 .NET Framework 技术的 Windows NT 系统）

我们常说的shell是指命令行shell。

### :pen\_fountain: 1.2、常见的linux命令行Shell

**常见的 Shell 有 sh、bash、csh、tcsh、ash、zsh 等。**

1. sh ：sh 的全称是 Bourne shell，由 AT\&T 公司的 Steve Bourne 开发，为了纪念他，就用他的名字命名了。sh 是 UNIX 上的标准 shell，很多 UNIX 版本都配有 sh。sh 是第一个流行的 Shell。
2. csh ：sh 之后另一个广为流传的 shell 是由柏克莱大学的 Bill Joy 设计的，这个 shell 的语法有点类似C语言，所以才得名为 C shell ，简称为 csh。Bill Joy 是一个风云人物，他创立了 BSD 操作系统，开发了 vi 编辑器，还是 Sun 公司的创始人之一。BSD 是 UNIX 的一个重要分支，后人在此基础上发展出了很多现代的操作系统，最著名的有 FreeBSD、OpenBSD 和 NetBSD，就连 Mac OS X 在很大程度上也基于BSD。
3. tcsh ：tcsh 是 csh 的增强版，加入了命令补全功能，提供了更加强大的语法支持。
4. ash ：一个简单的轻量级的 Shell，占用资源少，适合运行于低内存环境，但是与下面讲到的 bash shell 完全兼容。
5. bash ：bash shell 是 Linux 的默认 shell，本教程也基于 bash 编写。bash 由 GNU 组织开发，保持了对 sh shell 的兼容性，是各种 Linux 发行版默认配置的 shell。

> bash 兼容 sh 意味着，针对 sh 编写的 Shell 代码可以不加修改地在 bash 中运行。

尽管如此，bash 和 sh 还是有一些不同之处：

1. 一方面，bash 扩展了一些命令和参数；
2. 另一方面，bash 并不完全和 sh 兼容，它们有些行为并不一致，但在大多数企业运维的情况下区别不大，特殊场景可以使用 bash 代替 sh。

## :pencil2: 2、终端（Terminal）

终端 (Terminal) = `TTYtty`（Teletypewriter， 电传打印机） = 文本输入/输出环境 作用是提供一个命令的输入输出环境，在 `linux` 下使用组合键 ctrl+alt+T 打开的就是终端。当你打开一个 terminal 时，操作系统会将terminal和shell关联起来，当我们在 terminal 中输入命令后，shell就负责解释命令。

### :pen\_fountain: 2.1、历史上的终端 <a href="#21-li-shi-shang-de-zhong-duan" id="21-li-shi-shang-de-zhong-duan"></a>

终端 (Terminal)，其词汇本身的意义为「终点站；末端；（电路）的端子，线接头」。而在计算机领域，终端则是一种用来让用户输入数据至计算机，以及显示其计算结果的机器。在大型机 (Mainframe) 和小型机 (Minicomputer) 的时代里，计算机曾经非常昂贵且巨大，不像现在这样人手一台。这些笨重的计算机通常被安置在单独的房间内，而操作计算机的人们坐在另外的房间里，通过某些设备与计算机进行交互。这种设备就叫做 终端 (Terminal)，也叫终端机。

![](<../.gitbook/assets/image (20).png>)

早期的终端一般是一种叫做电传打字机 (Teletype) 的设备。 Unix 的创始人 Ken Thompson 和 Dennis Ritchie 想让 Unix 成为一个多用户系统。多用户系统就意味着要给每个用户配置一个终端，每个用户都要有一个显示器、一个键盘。但当时所有的计算机设备都非常昂贵（包括显示器），而且键盘和主机是集成在一起的，根本没有独立的键盘。后来他们机智地找到了一样东西，那就是 ASR-33 电传打字机。虽然电传打字机原本的用途是在电报线路上收发电报，但是它既有可以发送信号的键盘，又能把接收到的信号打印在纸带上，完全可以作为人机交互设备使用。而且最重要的是，价格低廉。于是，他们把很多台 ASR-33 连接到计算机上，让每个用户都可以在终端登录并操作主机。就这样，他们创造了计算机历史上第一个真正的多用户操作系统 Unix，而电传打字机就成为了第一个 Unix 终端。

### :pen\_fountain: 2.2、终端的分类 <a href="#22-zhong-duan-de-fen-lei" id="22-zhong-duan-de-fen-lei"></a>

字符终端 (Character Terminal) 也叫文本终端 (Text Terminal)，是只能接收和显示文本信息的终端。早期的终端全部是字符终端。字符终端也分为 哑终端 (Dumb Terminal) 和所谓的 智能终端 (Intelligent Terminal)，因为后者可以理解转义序列、定位光标和显示位置，比较聪明，而哑终端不行。

![](<../.gitbook/assets/34 (1).png>)

DEC 公司在 1978 年制造的 `VT100`，由于其设计良好并且是第一批支持 ANSI 转义序列与光标控制的智能终端，获得了空前的成功。`VT100` 不仅是史上最流行的字符终端，更是成为了字符终端事实上的标准。

> 随着技术的进步，图形终端 (Graphical Terminal) 也开始出现在公众的视野中。图形终端不但可以接收和显示文本信息，也可以显示图形与图像。著名的图形终端有 Tektronix 4010 系列。

不过现在专门的图形终端已经极为少见，他们基本上已经被全功能显示器所取代。

### :pen\_fountain: 2.3、终端模拟器 (Terminal Emulator) <a href="#23-zhong-duan-mo-ni-qi-terminalemulator" id="23-zhong-duan-mo-ni-qi-terminalemulator"></a>

随着计算机的进化，我们已经见不到专门的终端硬件了，取而代之的则是键盘与显示器。但是没有了终端，我们要怎么与那些传统的、不兼容图形接口的命令行程序（比如说 GNU 工具集里的大部分命令）交互呢？这些程序并不能直接读取我们的键盘输入，也没办法把计算结果显示在我们的显示器上……（图形界面的原理我这里就不多说了，它们编程的时候图形接口还在娘胎里呢！）

> 这时候我们就需要一个程序来模拟传统终端的行为，即 终端模拟器 (Terminal Emulator)。严格来讲，Terminal Emulator 的译名应该是「终端仿真器」。

对于那些命令行 (CLI) 程序，终端模拟器会「假装」成一个传统终端设备；而对于现代的图形接口，终端模拟器会「假装」成一个 GUI 程序。一个终端模拟器的标准工作流程是这样的：

1. 捕获你的键盘输入；
2. 将输入发送给命令行程序（程序会认为这是从一个真正的终端设备输入的）；
3. 拿到命令行程序的输出结果（STDOUT 以及 STDERR）；
4. 调用图形接口（比如 X11），将输出结果渲染至显示器。

终端模拟器有很多，这里就举几个经典的例子：

* GNU/Linux：gnome-terminal、Konsole；
* macOS：Terminal.app、iTerm2；
* Windows：Win32 控制台、ConEmu 等。

### :pen\_fountain: 2.4、终端窗口 (Terminal Window) 与虚拟控制台 (Virtual Console) <a href="#24-zhong-duan-chuang-kou-terminalwindow-yu-xu-ni-kong-zhi-tai-virtualconsole" id="24-zhong-duan-chuang-kou-terminalwindow-yu-xu-ni-kong-zhi-tai-virtualconsole"></a>

大部分终端模拟器都是在图形用户界面 (GUI) 中运行的，但是也有例外。

> 比如在 GNU/Linux 操作系统中，按下 `Ctrl + Alt + F1,F2...F6` 等组合键可以切换出好几个全黑背景的全屏终端界面，而按下 `Ctrl + Alt + F7` 才是切换回图形界面。虽然它们并不运行在图形界面中，但其实它们也是终端模拟器的一种。

这些全屏的终端界面与那些运行在 GUI 下的终端模拟器的唯一区别就是它们是 由操作系统内核直接提供的。这些由内核直接提供的终端界面被叫做 虚拟控制台 (Virtual Console)，而上面提到的那些运行在图形界面上的终端模拟器则被叫做 终端窗口 (Terminal Window)。除此之外并没有什么差别。因为终端窗口是跑在图形界面上的，所有如果图形界面宕掉了那它们也就跟着完蛋了。这时候你至少还可以切换到 Virtual Console 去救火，因为它们由内核直接提供，只要系统本身不出问题一般都可用。

### :pen\_fountain: 2.5、终端特殊设备文件 <a href="#25-zhong-duan-te-shu-she-bei-wen-jian" id="25-zhong-duan-te-shu-she-bei-wen-jian"></a>

由于 Unix 被设计为一个多用户操作系统，所以人们会在计算机上连接多个终端（在当时，这些终端全都是电传打字机）。Unix 系统为了支持这些电传打字机，就设计了名为 tty 的子系统（因为当时的终端全都是 tty，所以这个系统也被命名为了 tty），将具体的硬件设备抽象为操作系统内部位于 /dev/tty\* 的设备文件。

#### :gem: 2.5.1、控制终端（`/dev/tty`） <a href="#251-kong-zhi-zhong-duan-devtty" id="251-kong-zhi-zhong-duan-devtty"></a>

这是个在应用程序中的一个概念，前台进程有个控制终端，就对应这个。不过它并不指任何物理意义上的终端，其实 /dev/tty 会映射到当前的设备（通过 tty 命令可以看到，也可以使用命令`ps –ax`来查看进程与哪个控制终端相连。），比如你如果在控制台界面下(即字符界面下）那么 dev/tty 就是映射到 dev/tty1-6 之间的一个（取决于你当前的控制台号），但是如果你现在是在图形界面（Xwindows），那么你会发现现在的 /dev/tty 映射到的是 /dev/pts 的伪终端上。比如你可以输入命令 #tty 那么将显示当前映射终端如：/dev/tty1 或者 /dev/pts/0 等。

#### :gem: 2.5.2、伪终端（`/dev/pty/`） <a href="#252-wei-zhong-duan-devpty" id="252-wei-zhong-duan-devpty"></a>

这个是终端的发展，为满足现在需求（比如网络登陆、xwindow窗口的管理）。伪终端（Pseudo Terminal）是成对的逻辑终端设备，例如 `/dev/ptyp3` 和 `/dev/ttyp3`（或着在设备文件系统中分 \
别是 `/dev/pty/m3` 和 `/dev/pty/s3`）。它们与实际物理设备并不直接相关。如果一个程序把 ttyp3 看作是一个串行端口设备，则它对该端口的读/写操作会反映在该逻辑终端设备对的另一个上面（ttyp3）。而 ttyp3 则是另一个程序用于读写操作的逻辑设备。这样，两个程序就可以通过这种逻辑设备进行互相交流，而其中一个使用ttyp3的程序则认为自己正在与一个串行端口进行通信。这很象是逻辑设备对之间的管道操作。 对于ttyp3（s3），任何设计成使用一个串行端口设备的程序都可以使用该逻辑设备。但对于使用ptyp3的 程序，则需要专门设计来使用 ptyp3（m3）逻辑设备。

> 例如，如果某人在网上使用telnet程序连接到你的计算机上，则telnet程序就可能会开始连接到设 \
> 备ptyp2（m2）上（一个伪终端端口上）。此时一个 getty 程序就应该运行在对应的ttyp2（s2）端口上。当telnet从远端获取了一个字符时，该字符就会通过 m2、s2 传递给 getty 程序，而 getty 程序就会通过 s2、m2 和 telnet 程序往网络上返回”login:”字符串信息。这样，登录程序与telnet程序就通过“伪终端”进行通信。通过使用适当的软件，就可以把两个甚至多个伪终端设备连接到同一个物理串行端口上。在使用设备文件系统（device filesystem）之前，为了得到大量的伪终端设备特殊文件，HP-UX AIX 等使用了比较复杂的文件名命名方式。

#### :gem: 2.5.3、控制台终端（`/dev/ttyn, /dev/console`）  <a href="#253-kong-zhi-tai-zhong-duan-devttyndevconsole" id="253-kong-zhi-tai-zhong-duan-devttyndevconsole"></a>

在UNIX系统中，计算机显示器通常被称为控制台终端（Console）。它仿真了类型为Linux的一种终端（TERM=Linux），并且有一些设备特殊文件与之相关联：tty0、tty1、tty2等。当你在控制台上登录时，使用的是tty1。使用Alt+\[F1—F6]组合键时，我们就可以切换到 tty2、tty3 等上面去。`tty1-tty6` 等称为虚拟终端，而tty0则是当前所使用虚拟终端的一个别名，系统所产生的信息会发送到该终端上。因此不管当前正在使用哪个虚拟终端，系统信息都会发送到控制台终端上。你可以登录到不同的虚拟终端上去，因而可以让系统同时有几个不同的会话期存在。只有系统或超级用户root可以向 `/dev/tty0` 进行写操作。\
&#x20;\
console是一个缓冲的概念，其实是为内核提供打印的。我们的pc，终端常用的是显示器和键盘构成，我们用户打印和内核打印都从这个终端反映给用户。所以，这里，`/dev/console` 是连接到 `/dev/tty0` 的，其实这里有2个概念，console和 tty 这2个怎么实现，其实console这个结构中有个device，这里其实就是 tty0 对应的一个虚拟终端设备。\
&#x20;\
如果，我们来个专门打印内核的设备（比如通过串口），我们把那个串口register\_console，那么 /dev/console 就到这个串口设备了。这时，内核打印就到这个串口设备了，而用户的打印还是和上面的 /dev/tty 相关，如果 /dev/tty 对应 /dev/tty0，那么用户打印还在窗口中出现。所以说 /dev/console 是用来外接控制台的。

#### :gem: 2.5.4、串行端口终端（`/dev/ttySn`） <a href="#254-chuan-hang-duan-kou-zhong-duan-devttysn" id="254-chuan-hang-duan-kou-zhong-duan-devttysn"></a>

串行端口终端(Serial Port Terminal)是使用计算机串行端口连接的终端设备。计算机把每个串行端口都看作是一个字符设备。有段时间这些串行端口设备通常被称为终端设备，因为那时它的最大用途就是用来连接终端。这些串行端口所对应的设备名称是 `/dev/tts/0`(或 `/dev/ttyS0`)、`/dev/tts/1`(或 `/dev/ttyS1`) 等，设备号分别是`(4,0)`、 `(4,1)`等，分别对应于DOS系统下的 COM1、COM2 等。若要向一个端口发送数据，可以在命令行上把标准输出重定向到这些特殊文件名上即可。例如，在命令行提示符下键入：`echo test > /dev/ttyS1` 会把单词”test”发送到连接在 `ttyS1(COM2)` 端口的设备上。可接串口来实验。\
&#x20;\
关于`/dev/console`  应该来说更像一个缓冲结果吧，来实现对内核的打印，比如说内核把要打印的内容装入缓冲区，然后由console来决定打印到哪里吧（比如是 `tty0` 还是串口等）。所以说 `/dev/console` 是用来外接控制台的。

#### :gem: 2.5.5、其他类型 <a href="#255-qi-ta-lei-xing" id="255-qi-ta-lei-xing"></a>

Linux系统中还针对很多不同的字符设备存在有很多其它种类的终端设备特殊文件。例如针对 `ISDN` 设备的 `/dev/ttyIn` 终端设备等。

### :pencil2: 2.6、Shell与终端的分工 <a href="#26shell-yu-zhong-duan-de-fen-gong" id="26shell-yu-zhong-duan-de-fen-gong"></a>

终端干的活儿是从用户这里接收输入（键盘、鼠标等输入设备），扔给 Shell，然后把 Shell 返回的结果展示给用户（比如通过显示器）。而 Shell干的活儿是从终端那里拿到用户输入的命令，解析后交给操作系统内核去执行，并把执行结果返回给终端。不过 Shell 与终端的分工有一些容易混淆的地方，这里以例子进行说明：

1. 终端将用户的键盘输入转换为控制序列（除了字符以外的按键，比如 左方向键 → `^[[D`），Shell 则解析并执行收到的控制序列（比如 `^[[D` → 将光标向左移动）；
2. 不过也有例外，比如终端在接收到 `Ctrl + C` 组合键时，不会把这个按键转发给当前的程序，而是会发送一个 `SIGINT` 信号（默认情况下，这会导致进程终止）。其他类似的特殊组合键有 `Ctrl-Z` 与 `Ctrl-\` 等，可以通过 `stty -a` 命令查看当前终端的设置。
3. Shell 发出类似「把前景色改为红色（控制序列为 `\033[31m`）」「显示 foo」等指令；终端接收这些指令，并且照着 Shell 说的做，于是你就看到了终端上输出了一行红色的 foo。
4. 除非被重定向，否则 Shell 永远不会知道它所执行命令的输出结果。我们可以在终端窗口中上下翻页查看过去的输出内容，这完全是终端提供的 feature，与 Shell 没有半毛钱关系；
5. 命令提示符 (Prompt) 是一个完全的 Shell 概念，与终端无关；
6. 行编辑、输入历史与自动补全等功能是由 Shell 提供的（比如 fish 这个 Shell 就有着很好用的历史命令与命令自动补全功能）。不过终端也能自己实现这些功能，比如说 XShell 这个终端模拟器就可以在本地写完一行命令，然后整条发送给远程服务器中的 Shell（在连接状况不佳时很有用，不然打个字都要卡半天）；
7. 终端中的复制粘贴功能（Shift + Insert 或者鼠标右键等）基本上都是由终端提供的。举个例子，Windows 默认的终端对于复制粘贴的支持很不友好，而换一个终端（例如 ConEmu）后就可以很好地支持复制粘贴。不过 Shell 以及其他命令行程序也可以提供自己的复制粘贴机制（例如 vim）。

![](<../.gitbook/assets/image (23).png>)

![](<../.gitbook/assets/image (21).png>)

## :pencil2: 3、控制台（Console）

在计算机发展的早期，计算机的外表上通常会存在一个面板，面板包含很多按钮和指示灯，可以通过面板来对计算机进行底层的管理，也可以通过指示灯来得知计算机的运行状态，这个面板就叫console。在现代计算机上，在电脑开机时（比如ubuntu）屏幕上会打印出一些日志信息，但在系统启动完成之前，terminal不能连接到主机上，所以为了记录主机的重要日志（比如开关机日志，重要应用程序的日志），系统中就多了一个名为console的设备，这些日志信息就是显示在console上。一台电脑有且只有一个console，但可以有多个terminal。

终端和控制台都不是个人电脑的概念，而是多人共用的小型中型大型计算机上的概念。一台主机连很多终端，终端为主机提供了人机接口，每个人都通过终端使用主机的资源。终端有字符哑终端和图形终端两种。\
&#x20;\
控制台是另一种人机接口，不通过终端与主机相连，而是通过显示卡-显示器和键盘接口分别与主机相连，这是人控制主机的第一人机接口。控制台与计算机主机是一体的，是计算机的一个组成部分。控制台是计算机的基本设备，而终端是附加设备。 可以认为是一种特殊的终端。\
&#x20;\
个人计算机只有控制台，没有终端。当然愿意的话，可以在串口上连一两台字符哑终端。但是Linux偏要按POSIX 标准把个人计算机当成小型机来用，那么就在控制台上通过 getty 软件虚拟了六个字符哑终端（或者叫控制台终端 `tty1-tty6`）（数量可以在 `/etc/inittab` 里自己调）和一个图型终端，在虚拟图形终端中又可以通过软件（如rxvt）再虚拟无限多个虚拟字符哑终端（pts/0....）。 记住，这全是虚拟的，用起来一样，但实际上并不是。所以在个人计算机上，只有一个实际的控制台，没有终端，所有终端都是在控制台上用软件模拟的。要把个人计算机当主机再通过串口或网卡外连真正的物理终端也可以，但由于真正的物理终端并不比个人计算机本身便宜，一般没有人这么做。\
&#x20;\
如同其他UNIX类系统，Linux本身也是基于命令行的。试试`Ctrl+Alt+Fx`。这就是控制台，算是Linux的本来面目。至于使用方法，除了多出登录注销外，其它操作和我们在 linux 图形界面（X—window）下的终端操作是一样的，在X-Window出问题或不运行X-Window的时候，操作主要在这里完成。\
&#x20;\
Linux在控制台下提供了不止一个（字符哑）终端，支持多用户同时登录，包括在本机同时登录。控制台`Alt+Fx`能够切换到第x个（字符哑）终端。如果需要从X-Window里跳到第（字符哑）终端，需要`Ctrl+Alt+Fx`。一般情况下如果要从控制台返回X-window可用`Alt+7`来返回到X-window的图形界面。（Linux发行版提供7个虚拟屏幕，1\~6号是控制台终端（（字符哑）终端），第7个上面跑X-Window。）。
