# 微信 AppImage

使用 AppImage 运行 Linux 原生微信

优点：

- [x] **使用方便**，无需像 flatpak 等那样存在诸多配置问题。下载即可运行，删除即可卸载。
- [x] **免权限**，从下载到运行，无需 sudo 权限。
- [x] **不修改系统**，无需修改系统配置文件（`/etc/lsb-release`等）
- [x] **不乱放文件**，限制读写目录，以防在操作系统里到处乱放文件
- [x] **体积小**，最小只需 47MB，flatpak 等方案需要多达 GB 级别。
- [x] **适配多**，Debian/RHEL/Arch/Gentoo 等发行版均可使用。

直接安装原版微信，不使用本 AppImage 的缺点：

- 😕 **使用麻烦**，需要安装 deb 包，无法直接卸载。
- 😕 **需要 root 权限**，需要 sudo 才能安装。
- 😕 **乱改系统**，原版微信有发行版检测，需要修改一些系统关键配置。
- 😕 **乱放文件**，家目录、`/var`目录都有放置。
- 😕 **体积大**，需要几百 MB 存储空间。
- 😕 **适配少**，只有 deb 包，没有 rpm/pkg 等安装包。

适配操作系统（已测试）：

- [x] **Debian** Debian 12 可直接运行。
- [x] **Kali Linux** Kali Linux Rolling 可直接运行。
- [x] **Ubuntu** 22.04、24.04 进行下面操作后可以运行。（建议先尝试能否直接运行，不能的再按照下面操作）
  - 需要安装：`sudo apt install libfuse2`
  - 需要运行：`sudo rm /var/lib/dbus/machine-id && sudo cp /etc/machine-id /var/lib/dbus/machine-id`
  - 需要把下面内容写入 `/etc/apparmor.d/bwrap`：

    ```text
    abi <abi/4.0>,
    include <tunables/global>

    profile bwrap /usr/bin/bwrap flags=(unconfined) {
      userns,

      # Site-specific additions and overrides. See local/README for details.
      include if exists <local/bwrap>
    }
    ```

    然后运行 `sudo systemctl reload apparmor`

## 使用前提

- 需要安装 bwrap（常见发行版都有）
- 需要有 XDG 用户目录（常见发行版都有）
  - 查看 `~/.config/user-dirs.dirs`（必须有这个文件）
  - 里面要有 `XDG_DESKTOP_DIR`、`XDG_DOWNLOAD_DIR` 和 `XDG_DOCUMENTS_DIR`
  - 运行时，在用户目录下，微信只能读写：
    - 这三个目录，也就是桌面、下载、文档（应该够用了，不够用的自己改源码）
    - 微信数据库文件（放在 `${XDG_DOCUMENTS_DIR}/xwechat_files` 下，原生微信是 `~/xwechat_files`）
    - 微信配置文件（`~/.xwechat`）
    - 字体配置文件。

## 获取 AppImage 文件

### 从 release 下载

在本仓库 release 下载 x86_64 版，两个 release 基于 238 和 241 版构建，建议使用新的 241 版。

下载后记得给执行权限。

### 自行构建

以下构建方式仅测试过 238 和 241 的 x86_64 版。

下载本仓库，同时下载一个微信 Linux 版本的 deb 包，下载地址：

- [吾爱破解](https://www.52pojie.cn/thread-1896902-1-1.html)，或
- [银河麒麟deb仓库](https://archive2.kylinos.cn/deb/kylin/production/PART-V10-SP1/custom/partner/V10-SP1/pool/all/)，搜索 `wechat-beta`

还需要下载“优麒麟微信”deb 包：

- [优麒麟微信](https://www.ubuntukylin.com/applications/106-cn.html)

在后者里提取 libactivation.so 备用。

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

## 运行方法

注：实际测试中有极少时候第一次扫码之后没反应，此时可以关闭重新启动扫码。

### 简单运行

直接双击此文件或者命令行启动即可。

### 安装运行

如果您计划长期使用本 AppImage，那么我推荐在这里安装运行。如果只是简单体验，那么选择上一条即可。

```bash
# 安装
sudo ./wechat-x86_64.AppImage --install
# 卸载
sudo wechat --remove
```

安装后你可以在桌面添加类似的启动器图标。

安装过程只涉及三个文件：

```text
/usr/bin/wechat
/usr/share/icons/hicolor/256x256/apps/wechat.png
/usr/share/applications/wechat.desktop
```

如果想要在桌面上加上图标，只需要把第三个文件复制到桌面。

### 更多功能

```bash
# 帮助
./wechat-x86_64.AppImage --help
# 进入 bwrap 的 shell(bash)
./wechat-x86_64.AppImage --debug
# 关闭微信进程（早期版本无法退出，现在应该不需要这个功能了）
./wechat-x86_64.AppImage --kill
# 安装图标、桌面文件、应用
sudo ./wechat-x86_64.AppImage --install
# 卸载图标、桌面文件、应用
sudo ./wechat-x86_64.AppImage --remove
# 解包文件（这属于 appimage 的功能，参考 appimage 文档，其他 appimage 功能不再列出）
./wechat-x86_64.AppImage --appimage-extract
```

## 声明

本仓库仅供学习交流使用，且未经过严格测试，使用本仓库造成的一切后果由使用者负责。

## 参考

参考：[依云's Blog - 使用 bwrap 沙盒](https://blog.lilydjwg.me/2021/8/12/using-bwrap.215869.html)

参考：[wechat-beta-bwrap](https://github.com/lfift/wechat-beta-bwrap)
