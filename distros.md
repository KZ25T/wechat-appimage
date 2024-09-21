# 适配目录

目前已测试 Debian/Arch/RHEL 系发行版。

## Debian 系

- [x] **Debian** 12 可直接运行。
- [x] **Kali Linux** 可直接运行。
- [x] **Ubuntu** 24.04 按如下操作后可运行：
  - 把下面内容写入 `/etc/apparmor.d/bwrap`：

    ```text
    abi <abi/4.0>,
    include <tunables/global>

    profile bwrap /usr/bin/bwrap flags=(unconfined) {
      userns,

      # Site-specific additions and overrides. See local/README for details.
      include if exists <local/bwrap>
    }
    ```

  - 然后运行 `sudo systemctl reload apparmor` 之后即可。
  - 注：Ubuntu 这个操作系统很奇怪，相比于标准 Debian，魔改的地方太多，不见得所有人都能按照以上方法运行好。如果不行请自己查询修改。

## Arch 系

- [x] **Arch Linux** 可直接运行。
  - 请检查是否有 `/usr/lib/libnss3.so`，否则 `sudo pacman -S nss`
  - 请检查是否有基础 Qt 库：`/usr/lib/qt`，否则 `sudo pacman -S qt5-base`
  - 因为 Arch Linux 比较灵活，所以是否还有其他默认不带的库我不太了解。如有遇到请自行修复。
- [x] **Manjaro** 可直接运行。

## RHEL 系

- [x] **Fedora** 40 可直接运行。
- [ ] **CentOS Stream** 过气发行版，还没给 livecd，没测
