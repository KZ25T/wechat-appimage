# 微信 AppImage

使用 AppImage 运行 Linux 原生微信，使用方法非常简单，支持多种操作系统。

[TOC]

## 1 项目特色

- [x] **使用方便**，没有配置问题，下载即可运行，删除即可卸载。
- [x] **免权限**，从下载到运行，无需 sudo 权限。
- [x] **不修改系统**，无需修改系统配置文件（`/etc/lsb-release`等）
- [x] **不乱放文件**，限制读写目录，以防在操作系统里到处乱放文件
- [x] **体积小**，最小只需 47MB，flatpak 等方案需要多达 GB 级别。
- [x] **适配多**，Debian/RHEL/Arch/Gentoo 等发行版均可使用。

其他安装方法的缺点：

- 😕 **使用麻烦**，原版需要安装 deb 包，无法直接卸载；flatpak 等方式配置复杂。
- 😕 **需要 root 权限**，需要 sudo 才能安装。
- 😕 **乱改系统**，原版微信有发行版检测，需要修改一些系统关键配置。
- 😕 **乱放文件**，家目录、`/var`目录都有放置。
- 😕 **体积大**，需要几百 MB 存储空间。
- 😕 **适配少**，只有 deb 包，没有 rpm/pkg 等安装包。

## 2 发行版适配

目前该项目已测试并支持 Debian/Arch/RHEL 系发行版。详细适配情况或其他操作见[适配目录](./distros.md)。

## 3 快速开始

本节仅针对 `x86_64` 架构，其他架构请自行打包。

### 3.1 检查运行环境

1. 检查[适配目录](./distros.md)，检查自己的发行版是否支持，或者是否需要额外的操作。未经测试的发行版可能需要自行修复一些问题。欢迎经过测试后向本仓库提起 issue，我将添加到适配目录。
2. 操作系统需要有 bwrap 命令（常见发行版都有），若没有请参考[此网站](https://command-not-found.com/bwrap)。
3. 检查操作系统是否有 XDG 用户目录（常见发行版都有）：
   - 查看 `~/.config/user-dirs.dirs`（必须有这个文件）
   - 上述文件里面要有 `XDG_DESKTOP_DIR`、`XDG_DOWNLOAD_DIR` 和 `XDG_DOCUMENTS_DIR`（桌面、下载和文档）

### 3.2 获取 AppImage 文件并运行

1. 下载：打开[Releases](https://github.com/KZ25T/wechat-appimage/releases)，依说明下载符合你的需求的最新版本的文件。若你不知道下载哪一个，请下载 `wechat-x86_64.AppImage`
2. 修改权限：找到该文件下载路径，添加可执行权限：`chmod a+x wechat-x86_64.AppImage`
3. 运行：从命令行运行 `./wechat-x86_64.AppImage`，或双击该文件运行。

> 若不能运行或需要其他帮助请参阅[5.3节](#53-常见问题解决)

### 3.3 常用文件与目录

在用户目录下，微信涉及如下目录的读写：

1. 桌面、下载、文档，参考[3.1节](#31-检查运行环境)的第三条。如果您需要上传或下载文件，那么建议下载到此目录。
2. 微信数据库文件（放在 `文档/xwechat_files` 下），除非你明确知道为什么要删除它，否则不要删除。
   - 该目录结构和 Windows 的微信相似。
   - 微信聊天记录在这里。
   - 上传或下载的文件也在这里：`文档/xwechat_files/wxid_XXXXXX/msg/file`
   - 在微信的 bwrap 环境中，它的位置是 `~/xwechat_files`
3. 微信配置文件（`~/.xwechat`），这个删除应该没太大影响。
4. 字体配置文件和（可能涉及的）显示配置文件。

## 4 从本仓库构造 AppImage

如果需要修改 AppRun 里的内容，或者架构不符，你可以按照如下方法自行打包 AppImage 运行。

> 注：本说明暂未给 x86_64 以外的架构以完整支持。

### 4.1 下载原包

1. 下载本仓库。
2. 下载一个微信 Linux 版本的 deb 包，下载地址：

   - [吾爱破解](https://www.52pojie.cn/thread-1896902-1-1.html)，或
   - [银河麒麟deb仓库](https://archive2.kylinos.cn/deb/kylin/production/PART-V10-SP1/custom/partner/V10-SP1/pool/all/)，搜索 `wechat-beta`

3. 下载“优麒麟微信”deb 包：

   - [优麒麟微信](https://www.ubuntukylin.com/applications/106-cn.html)
   - 这个包只需要提取 `libactivation.so` 备用。

### 4.2 自行打包

解包第一个 deb，移植文件：

```bash
# pwd 为仓库根目录
dpkg -X your_wechat.deb /tmp/out
cp -r /tmp/out/opt src         # 大部分文件
mkdir -p src/usr/lib           # 把第二个微信的 libactivation.so 挪进来
```

安装 appimagetool（自己上网搜）

打包 appimage：

```bash
# pwd 为仓库根目录
appimagetool ./src
```

可以取得 `wechat-x86_64.AppImage`

### 4.3 其他架构的注意事项

其他架构可能需要修改 AppRun 的第 72 行。

## 5 高级用法

### 5.1 安装运行

如果您计划长期使用本 AppImage，那么我推荐在这里安装运行。

```bash
# 安装（可以加 sudo 安装到系统目录）
./wechat-x86_64.AppImage --install
# 卸载（可以加 sudo 从系统目录卸载）
wechat --remove
```

> 普通用户直接安装会装到 `~/.local` 下，加上 `sudo` 会安装在 `/usr/local` 下。
> 卸载时是否使用 `sudo` 和安装时应保持一致。

安装后你可以在桌面添加类似的启动器图标。

安装过程只涉及三个文件（或）：

```bash
# 若使用 sudo，把 ~/.local 换成 /usr/local
~/.local/bin/wechat
~/.local/share/icons/hicolor/256x256/apps/wechat.png
~/.local/share/applications/wechat.desktop
```

如果想要在桌面上加上图标，只需要把第三个文件复制到桌面。

### 5.2 更多功能

```bash
# 帮助
./wechat-x86_64.AppImage --help
# 进入 bwrap 的 shell(bash)
./wechat-x86_64.AppImage --debug
# 关闭微信进程（早期版本无法退出，现在应该不需要这个功能了）
./wechat-x86_64.AppImage --kill
# 检查更新（显示下载链接）
./wechat-x86_64.AppImage --update
# 禁止微信读取桌面
./wechat-x86_64.AppImage --no-user-desktop
# 禁止微信读取下载
./wechat-x86_64.AppImage --no-user-download
# 禁止微信读取文档
./wechat-x86_64.AppImage --no-user-documents
# 禁止微信读取 /run 下的文件（可能会导致不稳定）
./wechat-x86_64.AppImage --no-run-file
# 安装图标、桌面文件、应用
./wechat-x86_64.AppImage --install
# 卸载图标、桌面文件、应用
wechat --remove
# 解包文件（这属于 appimage 的功能，参考 appimage 文档，其他 appimage 功能不再列出）
./wechat-x86_64.AppImage --appimage-extract
```

### 5.3 常见问题解决

#### 5.3.1 AppImage 的问题

如果您的发行版未安装 `libfuse3`，或因为权限、容器等限制无法加载 `fuse` 模块，那么本微信将不能直接以 AppImage 运行。

您可以尝试在命令后面加上 `--appimage-extract-and-run`；如有其它选项，则不需要改变。如：

```bash
./wechat-x86_64.AppImage --appimage-extract-and-run
./wechat-x86_64.AppImage --appimage-extract-and-run --help
./wechat-x86_64.AppImage --appimage-extract-and-run --no-run-file --no-user-download
```

#### 5.3.2 bwrap 的问题

在 Ubuntu 中，bwrap 默认可能不允许创建用户空间。其他发行版暂未发现该情况，如果有 bwrap 的报错，请尝试在[适配目录](./distros.md)里参考 Ubuntu 的解法。

#### 5.3.3 挂载目录的问题

参考 bwrap 报错时有哪些未成功挂载的目录，您可以尝试编辑文件重新自行打包解决：

```bash
$ cd /tmp
$ /path/to/wechat-x86_64.AppImage --appimage-extract
$ vim squashfs-root/AppRun
  # 尝试寻找并去除未成功挂载的目录
$ appimagetool squashfs-root
  # 你需要自己下载 appimagetool
  # 最好提前在 GitHub 下好 appimagetool 的 runtime-x86_64，免得每次操作都要重新下载这个文件。
  # 此时的命令为 appimagetool --runtime-file /path/to/runtime-x86_64
$ ./wechat-x86_64.AppImage
  # 尝试新打包的
```

#### 5.3.4 链接库的问题

可能有些发行版缺少链接库，你可以尝试如下方式找到未安装的库并安装：

```bash
$ ./wechat-x86_64.AppImage --debug
$ ldd /opt/wechat-beta/wechat | grep "not found"
  # 尝试寻找缺少的库并安装（自行搜索安装方法）
```

## 6 声明

本仓库仅供学习交流使用，且未经过严格测试，使用本仓库造成的一切后果由使用者负责。

## 7 参考

参考：[依云's Blog - 使用 bwrap 沙盒](https://blog.lilydjwg.me/2021/8/12/using-bwrap.215869.html)

参考：[wechat-beta-bwrap](https://github.com/lfift/wechat-beta-bwrap)
