https://ywnz.com/linux/3522.html
如果在 Linux 系统中没有 GUI（图形界面），还能玩吗？当然能，以下跟着本文的方法一起操作没有 X Window 桌面的 Linux，它就是 Linux 字符模式，甚至是在进入 Linux 系统之前的 Grub 命令行模式。以下介绍 GRUB 及其实战、Linux 纯字符模式和 Framebuffer、字符界面下联网、使用 fbterm、显示 Framebuffer 的信息、在 Framebuffer 下截图、在 Framebuffer 下查看图片、在纯字符界面下上网、视频播放、使用屏保等内容。

**前言**

没有桌面环境的 Linux，并不一定就不是图形界面，因为 Linux 图形界面无处不在。以前我使用 Linux 桌面的时候，总是有一个误区：认为只有 XServer 启动后，才能够访问到图形系统，否则只能访问字符界面。随着对 Linux 的认识逐步加深，才发现即使在 XServer 启动之前，图形界面也是无处不在的。例如，Grub 的系统启动菜单，可以是图形化的，还可以通过改背景和主题进行美化。再例如在 Linux 初始化过程中，有一个 PlyMouth 软件，可以直接通过内核的 DRM 模块访问图形硬件，从而显示一个图形化的启动界面和进度条，同理，PlyMouth 也是可以通过更改主题进行美化的。最后，当 Linux 初始化完成后，会给我们显示一个让我们登录的图形界面，这就是 DM（Display Manager），这个 DM 既是 XServer 的父进程，负责启动 XServer，又是一个 XClient，给出图形化的登录接口。登录成功后，它又是 Gnome Shell 的父进程，负责启动 Gnome Shell。还有，即使在纯字符界面下，也是可以使用 FrameBuffer 获得图形功能的，甚至可以截图和播放视频。唯一的区别，就是在这些模式下，在没有桌面环境的情况下，我们和计算机的交互，往往只能通过 CLI 进行。

参考：[X-Window_Linux常用命令大全](https://ywnz.com/linux/xwindow/)

**一、逆天的 GRUB**

Linux 系统启动的过程是这样的：先由 BIOS 启动一个系统引导程序；然后系统引导程序负责把 Linux 的内核加载到内存，同时把 initrd 加载到内存，然后把控制权交给 Linux 的内核；Linux 的内核初始化完成后，将控制权交给 init 程序；init 程序负责启动各种服务。如果要启动图形桌面系统，则 init 先启动一个窗口管理器，由窗口管理器负责用户的登录和验证；用户登录和验证成功后，窗口管理器负责启动 X 服务器和客户端，进入桌面系统。如果是不需要图形桌面系统的 Linux，则 init 启动 login 程序，login 程序负责用户的登录和验证，验证成功后，启动一个 shell。

GRUB 就是目前 Linux 系统使用的系统引导程序，是计算机启动后运行的第一个程序（当然，BIOS除外）。它在将 Linux 内核加载到内存的时候，还可以向内核传递各种参数。目前的 Linux 发行版使用的 GRUB 都已经是第 2 版了，它的功能和配置都和以前的版本不一样。网上很多文章都是基于以前的 GRUB Legacy 版本进行的讲解，已经不能适应现在新的形势了。

GRUB 是计算机启动后运行的第一个程序，这个时候 Linux 的内核还没有加载，其它的程序也都不可能运行。这时有人就会想了，这个 GRUB 的功能应该相当有限吧。我刚开始也是这么想的。但是当我读完前面给出的 GRUB 文档后，我的思想被彻底颠覆了。GRUB 的功能太 TM 强了，简直逆天。

那么这个一开机就启动的简单程序究竟具有哪些让人意想不到的功能呢？请看我列举几条：

1.能够访问任何设备上的数据，不管你是硬盘、软盘还是光盘；

2.能够探测到所有的内存；

3.能够识别大部分的文件系统，不管你是 FAT32、NTFS 还是 ext2/ext3/ext4；

4.能够识别文件系统中的文件，文档中说它可识别大部分可执行文件格式，ELF什么的根本不在话下；

5.能够使用 .png、.jpg 格式的图片作为背景，说明它能够识别一些图片格式；

6.对字体的支持稍微差一点，好像只能使用 PFF2 格式的字体；

7.当然可以读取和输出硬盘上的文本文件；

8.据说还能播放乐曲；

9.支持联网，可以从网络上启动操作系统；

10.可以支持串口输入输出。

这些功能真的是已经超强了，就快赶上一个操作系统了。重要的是，它还提供了一个非常好用的命令行界面，该命令行界面的使用方法和 Linux 系统中的 Shell 极其接近，也支持编程、支持环境变量，其编程的语法也和 Bash 差不多。再加上 GRUB 提供的丰富的命令，该界面使用起来爽得不要不要的。

**二、GRUB 实战**

实践出真知，下面以 Ubuntu 为例开始实战。

1.GRUB的界面

刚安装好的 Ubuntu 启动时不显示 GRUB 界面，因为它在设置中把它隐藏了。它的启动画面是这样的：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21422C1.JPG)

必须按一下 ESC 键，我们才能够看到 GRUB 的菜单，它是这样的：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21431406.JPG)

上面这个界面想必大家已经很熟悉了。在这个界面中，如果按下 c 键，就会切换到 GRUB 的命令行界面，如下：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21441161.JPG)

还有一种情况就是，如果大家在使用 Linux 过程中不小心删除了 /boot/grub/grub.cfg，或者配置错误，或者删除了 Linux 系统所在的硬盘分区的数据，使得 GRUB 无法正确加载 Linux 系统，也会自动进入到这个命令行界面。

2.GRUB 支持的命令

GRUB 的命令补全功能非常方便，只要按一下 TAB 键，就可以显示它支持的所有命令。命令之后按 TAB 键，可以自动补全文件名。

使用ls命令可以列出目录和文件，使用cat命令可以输出文本文件的内容。在 GRUB 中，使用(hd0, msdos1)或者(hd0, gpt1)识别硬盘分区，使用(hd0, gptN)/boot/grub/grub.cfg这样的形式识别文件。由于 GRUB 能自动识别根分区，所以我下面的命令中省略掉了指定硬盘分区的部分。如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21452X2.JPG)

在上图中，我使用cat /etc/fstab命令显示了我系统中硬盘分区的情况。可以看到，我使用的是 GPT 分区格式和 EFI 固件，硬盘分了三个去，第一个分区的挂载点是/boot/efi，并且是 vfat 格式的文件系统，第二个分区的挂载点是根目录/，第三个分区是 swap 空间。按照 GRUB 的术语，则分区(hd0, gpt1)是挂载的/boot/efi，分区(hd0, gpt2)是根目录，分区(hd0, gpt3)是交换分区。可以看到，GRUB 中硬盘是从 0 开始计数的，而分区是从 1 开始计数的。

3.GRUB 的环境变量

命令行参数、环境变量、配置文件是对软件进行配置的三驾马车，GRUB 也不例外，它的很多行为也受环境变量控制。下面看一个例子，当我想查看 GRUB 的启动配置文件/boot/grub/grub.cfg时，使用cat命令查看该文件的内容，但是由于该文件太长，一个屏幕显示不完，所以只能看到最后几行，如下两图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R215022E.JPG)

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21511136.JPG)

这是非常蛋疼的，但还不是最郁闷的，毕竟/boot/grub/grub.cfg是系统中的一个文件，大不了我进 Linux 后用 vim 看。最蛋疼的是某些命令的输出，只能看到最后几行，又不能保存下来，真的让人捉急。就像下面这个例子，我使用videoinfo命令查看我的 GRUB 支持哪些图形分辨率：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21522G3.JPG)

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21531940.JPG)

这个时候，就只能通过设置环境变量的方法来解决问题了。使用set pager=1命令设置环境变量pager，让 GRUB 的输出启用分页，如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21549312.JPG)

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R2160c49.JPG)

我们还可以通过不带参数的set命令显示所有可用的环境变量，如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21624616.JPG)

也可以使用echo命令输出某一个环境变量，如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21634I2.JPG)

4.更改分辨率

我们可以控制 GRUB 显示界面的分辨率，还可以通过 GRUB 控制 Linux 启动进入字符模式后的分辨率。前提条件是要看我们的 BIOS 和显卡支持哪些模式。可以通过 videoinfo命令查看，如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21A3340.JPG)

我使用的是虚拟机，因为玩 GRUB 不使用虚拟机无法截图啊。如果采取的是 EFI 固件，则其输出如下：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21FE34.JPG)

如果采取的是 Legacy BIOS，则其输出如下：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21GR21.JPG)

可以看到，如果使用的是 Legacy BIOS，它的显示模式是由 ' VESA BIOS Extension Video Driver ' 提供支持的。如果使用的是 EFI，则其显示模式是由 ' EFI GOP driver ' 提供支持的。在我的虚拟机中，它们能提供的最高分辨率也只有 1152 x 864，远远达不到 1920 x 1080。但是在我的物理机中，都是可以达到 1920 x 1080。而且貌似只能进入 1920 x 1080，想改小还改不了。在物理机上，想通过改小分辨率，然后利用显示器的放大功能来放大字体的梦想是破灭了的。

在虚拟机中，我要做的是把分辨率改大，至少让我完全进入 Linux 字符界面的时候有个 1024 x 768 的分辨率吧，不然字符界面用起来岂不是太憋屈。可以通过修改 /etc/default/grub文件，然后调用sudo update-grub命令更新 GRUB。如下图，使用sudo vim /etc/default/grub修改配置文件：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21I2C8.JPG)

在上面的这个文件中的注释里，也写得很明白了，要修改 GRUB 和 Linux 字符界面的分辨率，可以通过修改 GRUB_GFXMODE 和 GRUB_GFXPAYLOAD_LINUX 参数来设置，而且千万不要设置GRUB_TERMINAL=console，不然就真的进入只有文字的文字模式了，没有 Graphic 的支持，还谈啥分辨率呢。

然后重启系统，可以看到我们的 GRUB 界面变大了一圈，如下两图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21J3605.JPG)

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21K4944.JPG)

下面进入 Linux 的字符界面，进入 Linux 字符界面的方式是启动进入 Linux 后，使用 sudo systemctl set-default multi-user.target，然后重启，在 1024 x 768 的分辨率下开一个 vim 看看，字体感觉有点小，如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R222029D.JPG)

又找到了怀旧的感觉，不是吗？唯一的缺憾是字太小。如果是在我的物理机上，15.6寸 1920 x 1080 的屏幕，字会小得根本无法看。下面，我们再来探讨 Framebuffer 的正确打开姿势。

**三、Linux 纯字符模式和 Framebuffer**

1.Linux 纯字符界面的用途

高手必备、省资源，服务器一般不安装图形界面、图形界面崩溃后紧急救援。

2.进入字符界面的正确方式

目前新的 Linux 发行版基本上都使用 Systemd 作为 init 程序，不再使用 SysV init 和 Upstart init。所以如果想让系统启动后直接进入字符界面，应该使用如下命令：

sudo systemctl set-default multi-user.target

反过来，要让系统启动后直接进入图形界面，应该使用如下命令：

sudo systemctl set-default graphical.target

另外，Linux 本身提供有虚拟控制台的功能，使用Ctrl + Alt + F1到Ctrl + Alt + F7进行切换，其中有一个是图形界面，剩下的是字符界面。图形界面玩崩溃了，就不得不使用Ctrl + Alt + F3切换到字符界面进行救援。

3.关于 Framebuffer

字符界面分两种，一种是不开启 Framebuffer 的，另一种是开启 Framebuffer 的。Framebuffer 是一种图形驱动，不开启 Framebuffer 就是真的全字符，不能改变分辨率，不能显示图像，不能截图。目前最新的 Linux 发行版默认开启 Framebuffer。控制 Framebuffer 开启和关闭，以及分辨率的方法，是设置 Grub2 的参数。修改 /etc/default/grub文件，添加如下参数可以设置分辨率：

GRUB_GFXPAYLOAD_LINUX=1024x768x32

然后使用如下命令更新 GRUB2 配置：

sudo update-grub

其中的分辨率必须是我们的硬件支持的。可以通过 GRUB2 命令行中的videoinfo命令查看我们的硬件支持的分辨率。

如果要关闭 Framebuffer，则这样更改 GRUB2 的配置：

GRUB_GFXPAYLOAD_LINUX=text

同样，需要：

sudo update-grub

然后重启。

**四、字符界面下联网**

以前在图形界面的时候，设置个网络、连接个 wifi 非常简单，玩儿似的，结果一进入纯字符界面就抓瞎。不联网，就不能下载和安装软件包，后面就玩不下去了。所以进入纯字符界面后，要解决的第一件事就是怎么联网。说到管理网络的工具，大家可以列举一大堆，什么 ipconfig、iwconfig、ip 等等。但是，在最新的 Linux 发行版中，已经是使用 NetManager 管理网络了。通过阅读 NetManager 的文档，可以知道它提供一个功能很强大的命令行工具，那就是 nmcli。通过man nmcli可以查看该工具的用法。

使用如下命令可以查看可用的 wifi 热点以及连接 wifi：

nmcli device status　　#查看网络连接的状态，可以看到各网卡的名称

nmcli device wifi list iface 网卡名称　　#查看可用的 wifi 热点

nmcli device wifi connect **** password ****　　#连接 wifi，需要提供 wifi 的名称和密码

如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R21950M8.JPG)

连上了网，Ubuntu 就可以在纯字符界面下起飞了。

**五、使用 fbterm**

通过上面的截图，发现两个问题：

1.在我的 1920x1080 的笔记本电脑屏幕上，默认的字实在是太小；

2.不能显示中文，上图中 wifi 热点名称含有中文的，都显示不出来。

解决办法是使用 fbterm。一举解决字体大小问题和中文显示问题。先安装 fbterm，使用如下命令：

sudo apt-get install fbterm

先使用sudo fbterm启动 fbterm 一次，再用exit命令退出，这样，fbterm 会自动生成一个默认的配置文件~/.fbtermrc，然后修改~/.fbtermrc配置文件中的两行，设置使用的字体和字体大小，如下：

font-names=DejaVu Sans Mono

font-size=16

然后使用sudo fbterm命令启动 fbterm，就可以了。下面是看对比图，使用 fbterm 之前，Vim 的启动界面是不能显示中文的：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R22220101.JPG)

使用 fbterm 之后，中文可以正常显示：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R22231J0.JPG)

使用 fbterm 之前，阅读代码是这样的：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R22241L4.JPG)

字非常的小，NERDTree 和 Tagbar 里面的符号显示也有问题。使用 fbterm 之后，就很漂亮了，如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R2225H00.JPG)

这才是全高清屏该有的显示效果嘛。关于在 fbterm 下输入中文，我尝试过 fbterm-ucimf，也尝试过 fcitx-frontend-fbterm，都没有成功。后来我就不试了，反正我也没有在全字符界面下输入中文的需求。

**六、显示 Framebuffer 的信息**

使用 fbset 可以查看 Framebuffer 的信息，包括 Framebuffer 是否开启，分辨率是多少，由哪个内核模块提供支持等。使用如下命令安装 fbset：

sudo apt-get install fbset

使用sudo fbset -i命令查看 Framebuffer 的信息，如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R2230G94.JPG)

**七、在 Framebuffer 下截图**

使用 fbgrab 可以在 Framebuffer 下进行截图。使用如下命令安装 fbgrab：

sudo apt-get install fbgrab

使用 fbgrab 命令的方式如下：

sudo fbgrab -c N filename.png　　#对 /dev/ttyN 对应的终端进行截图

sudo fbgrab -C N filename.png　　#先跳转到 /dev/ttyN 对应的终端，再进行截图

sudo fbgrab -s N filename.png　　#先等待 N 秒，再进行截图

我前面的图片都是使用 fbgrab 截的，这里就不贴图了。

**八、在 Framebuffer 下查看图片**

使用 fbi 可以在 Framebuffer 下查看图片，同样使用sudo apt-get install fbi安装这个软件。在看图界面下按 H 键，还会显示帮助信息。如下图：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R2232EZ.JPG)

参考：[Linux：优化和压缩JPEG和PNG图片的命令行工具](https://ywnz.com/linuxml/202.html)

**九、在纯字符界面下上网**

使用老牌的上网工具 w3m，安装方式sudo apt-get install w3m，然后使用w3m https://www.kernel.org/就可以访问Linux内核网站了，只有文字哦，图片就不要想了。

当然，必须在 fbterm 下执行才能显示中文。

**十、视频播放**

使用 mplayer 播放器可以播放视频，通过mplayer -vo help命令可以看到，mplayer 支持很多种视频驱动，而 Framebuffer 正是其中一种。使用sudo mplayer -vo fbdev2 badapple.mp4播放 Bad Apple 的 PV 视频，效果如下：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R2233LW.JPG)

因为这里要录制 gif 动画，所以我使用了虚拟机运行 Linux，Framebuffer 的分辨率设置为 1024x768。同样，通过上面的mplayer -vo help命令，还可以看到 mplayer 支持使用 libaa 库，将视频播放为字符画。据我观察，只有在图形界面下效果才可以，纯字符界面不行。使用mplayer -vo aa -moniterpixelaspect 0.5 badapple.mp4播放。

**十一、使用屏保**

当然是 cmatrix 啦，在终端中执行 cmatrix 命令就行了。效果如下：

![在Linux下没有GUI一起玩得转，附操作方法](https://ywnz.com/uploads/allimg/18/1-1Q10R22414929.JPG)

**结语**

通过以上的介绍，可以得出一个结论，没有图形界面的 Linux 系统一样能操作，能完成很多工作，而且实现起来并不困难。

**相关主题**

[常见Linux命令用途一展（共32个分类）](https://ywnz.com/linuxml/2599.html)