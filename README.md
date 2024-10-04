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
   - 上述文件里面要有 `XDG_DESKTOP_DIR`、`XDG_DOWNLOAD_DIR` 和 `XDG_DOCUMENTS_DIR`
   - 运行时，在用户目录下，微信只能读写：
     - 这三个目录，也就是桌面、下载、文档（应该够用了，不够用的自己改源码）
     - 微信数据库文件（放在 `${XDG_DOCUMENTS_DIR}/xwechat_files` 下，原生微信是 `~/xwechat_files`）
     - 微信配置文件（`~/.xwechat`）
     - 字体配置文件和（可能有的）显示配置文件。

### 3.2 获取 AppImage 文件并运行

1. 下载：打开[Releases](https://github.com/KZ25T/wechat-appimage/releases)，依说明下载符合你的需求的最新版本的文件。若你不知道下载哪一个，请下载 `wechat-x86_64.AppImage`
2. 修改权限：找到该文件下载路径，添加可执行权限：`chmod a+x wechat-x86_64.AppImage`
3. 运行：从命令行运行 `./wechat-x86_64.AppImage`，或双击该文件运行。

> 若不能运行或需要其他帮助请参阅[5.3节](#53-若无法使用-fuse3)

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
# 安装
sudo ./wechat-x86_64.AppImage --install
# 卸载
sudo wechat --remove
```

安装后你可以在桌面添加类似的启动器图标。

安装过程只涉及三个文件：

```text
/usr/local/bin/wechat
/usr/local/share/icons/hicolor/256x256/apps/wechat.png
/usr/local/share/applications/wechat.desktop
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
sudo ./wechat-x86_64.AppImage --install
# 卸载图标、桌面文件、应用
sudo wechat --remove
# 解包文件（这属于 appimage 的功能，参考 appimage 文档，其他 appimage 功能不再列出）
./wechat-x86_64.AppImage --appimage-extract
```

### 5.3 若无法使用 fuse3

如果您的发行版未安装 `libfuse3`，或因为权限、容器等限制无法加载 `fuse` 模块，那么本微信将不能直接运行。

您可以尝试在命令后面加上 `--appimage-extract-and-run`；如有其它选项，则不需要改变。如：

```bash
./wechat-x86_64.AppImage --appimage-extract-and-run
./wechat-x86_64.AppImage --appimage-extract-and-run --help
./wechat-x86_64.AppImage --appimage-extract-and-run --no-run-file --no-user-download
```

## 6 声明

本仓库仅供学习交流使用，且未经过严格测试，使用本仓库造成的一切后果由使用者负责。

## 7 参考

参考：[依云's Blog - 使用 bwrap 沙盒](https://blog.lilydjwg.me/2021/8/12/using-bwrap.215869.html)

参考：[wechat-beta-bwrap](https://github.com/lfift/wechat-beta-bwrap)
