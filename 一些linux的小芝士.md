# 一些linux终端常用命令

## 查询磁盘的一些命令

fdisk

lsblk

blkid

free -h

df -h

使用modinfo命令查询模块详细内容

### **查看目录的大小**

要检查某个目录的大小，你可以使用以下命令：

```bash
du -sh /path/to/directory
```

- `-s`：显示总大小（summary），而不是每个文件的大小。
- `-h`：以人类可读的形式显示大小（例如，显示为 KB、MB 或 GB，而不是字节）。

如果想查看目录中每个文件和子目录的大小，可以使用以下命令：

```bash
du -h /path/to/directory
```

也可以使用一些组合选项来更精确地查看目录的大小。例如：

```bash
du -ah /path/to/directory
```

- `-a`：包括所有文件（不仅仅是子目录）。



## 查看各种系统相关的函数用法

```bash
man 1  function #(库函数)
man 2  function #（syscall）
man 3  function #（C函数）
```



```bash
./mk -j32 -KFLAGS="Werror" #开启这个选项能将Warning变为Error提示
```



### 下载ubuntu源码

https://packages.ubuntu.com/zh-cn/noble/linux-source-6.8.0



### Linux内核源码的位置

/usr/src/linux



## linux内核完整步骤：从编译到安装与配置

### **步骤 1：准备环境**

1. **安装必要依赖**

   ```bash
   sudo dnf install -y gcc gcc-c++ make perl kernel-devel kernel-headers \
   elfutils-libelf-devel ncurses-devel bison flex openssl-devel dracut
   ```

2. **获取内核源码**

   - 从 [kernel.org](https://www.kernel.org) 或发行版仓库下载内核源码。

   - 解压源码并进入源码目录：

     ```bash
     tar -xf linux-x.x.x.tar.xz
     cd linux-x.x.x
     ```

3. **复制当前系统的内核配置**

   - 使用当前系统的配置作为基础：

     ```bash
     cp /boot/config-$(uname -r) .config
     ```

4. **可选：自定义内核配置**

   - 使用图形化配置工具：

     ```bash
     make menuconfig
     ```

------

### **步骤 2：编译内核与模块**

1. **清理旧构建文件**

   ```bash
   make clean
   ```

2. **编译内核**

   - 使用多线程加速编译：

     ```bash
     make -j$(nproc)
     ```

3. **编译内核模块**

   ```bash
   make modules -j$(nproc)
   ```

------

### **步骤 3：安装内核与模块**

#### **方法一：使用 `make install` 自动安装**

默认情况下：

- `make install` 会将内核镜像文件复制到 `/boot` 目录，并生成符号链接 `vmlinuz` 和 `System.map`。

- 命令：

  ```bash
  sudo make modules_install
  sudo make install
  ```

------

#### **方法二：解决 `make install` 失败（手动安装）**

如果 `make install` 失败，可以手动完成安装步骤：

1. **手动复制内核镜像文件**

   - 根据架构选择正确路径（例如 AArch64 使用 `Image`）：

     ```bash
     sudo cp arch/arm64/boot/Image /boot/vmlinuz-<custom-name>
     ```

   - 示例：

     ```bash
     sudo cp arch/arm64/boot/Image /boot/vmlinuz-5.10.0-custom
     ```

2. **复制配置和符号表文件**

   ```bash
   sudo cp System.map /boot/System.map-5.10.0-custom
   sudo cp .config /boot/config-5.10.0-custom
   ```

3. **生成 `initramfs` 文件**

   - 使用 `dracut` 创建初始内存盘文件：

     ```bash
     sudo dracut --force /boot/initramfs-5.10.0-custom.img 5.10.0-custom
     ```

4. **安装内核模块**

   - 如果 `make modules_install` 失败，可以手动复制模块：

     ```bash
     sudo cp -r ./lib/modules/5.10.0-custom /lib/modules/
     ```

------

### **步骤 4：更新 GRUB 配置**

1. **生成新的 GRUB 配置**

   - **UEFI 系统**：

     ```bash
     sudo grub2-mkconfig -o /boot/efi/EFI/<发行版>/grub.cfg
     ```

   - **传统 BIOS 系统**：

     ```bash
     sudo grub2-mkconfig -o /boot/grub2/grub.cfg
     ```

2. **验证 GRUB 是否正确添加新内核**

   ```bash
   grep "vmlinuz-5.10.0-custom" /boot/efi/EFI/<发行版>/grub.cfg
   ```

------

### **步骤 5：重启并选择新内核**

1. **重启系统**

   ```bash
   sudo reboot
   ```

2. **在 GRUB 菜单中选择新内核**

   - 开机时按 **`Esc`** 或 **`Shift`** 键，进入 GRUB 菜单。
   - 选择新编译的内核。

3. **验证当前运行的内核版本**

   ```bash
   uname -r
   ```

### **常见问题及解决方案**

#### 1. `make install` 失败的原因及解决

- **原因**：权限不足、路径问题或 Makefile 配置错误。

- **解决方案**：

  1. 手动复制文件到 `/boot` 和 `/lib/modules`（见 **步骤 3 方法二**）。

  2. 确保有足够的磁盘空间和权限：

     ```bash
     sudo df -h /boot
     sudo chmod -R 755 /boot
     ```

------

#### 2. 如何自定义 `vmlinuz` 文件名？

1. **通过 Makefile 自定义**：

   - 修改 `Makefile` 中的安装规则：

     ```makefile
     install:
         cp $(objtree)/arch/arm64/boot/Image /boot/vmlinuz-custom
     ```

2. **手动指定文件名**（推荐）：

   - 跳过 `make install`，直接复制文件到 `/boot`，使用您希望的文件名。

------

#### 3. GRUB 默认内核未更改

- 确保新内核已添加到 GRUB 配置：

  ```bash
  sudo grub2-mkconfig -o /boot/efi/EFI/<发行版>/grub.cfg
  ```

- 如果希望保留旧内核为默认启动：

  ```bash
  sudo grub2-set-default <menu-index>
  ```

完整流程回顾：

1. 准备环境和源码：安装依赖、解压源码、复制配置。
2. 编译内核和模块：`make` 和 `make modules`。
3. 自动安装或手动安装：
   - `make install` 自动安装。
   - 手动复制文件到 `/boot` 和 `/lib/modules`。
4. 更新 GRUB 配置：`grub2-mkconfig`。
5. 重启并验证：选择新内核，确认版本。



