本文记录在 Arch Linux 下安装 Waydroid,其他版本请参考官方教程：[Install Instructions | Waydroid](https://docs.waydro.id/usage/install-on-desktops)

## 必要的准备

1. 开启`binder`模块：`zen`内核默认开启了
2. 安装`wayland`，weston 可以在 X11 会话中运行 Waydroid。

## 安装

```sh
$ paru -S waydroid
```


可以选择通过 AUR 安装 `waydroid-image` 或 `waydroid-image-gapps` 来提供所需的 Android 映像。
推荐先使用此镜像安装成功后更换镜像，因为要下载很久。

```sh
$ waydroid init
# 初始化支持 GApps 的 Waydroid：
# waydroid init -s GAPPS -f

# 启动服务
$ systemctl enable --now waydroid-container
```

开启会话并启动GUI：
```sh
$ waydroid session start
$ waydroid show-full-ui
```

之后你会看到LineageOS启动动画。
### 更换镜像

到这个网址下载：[WayDroid - Browse /images at SourceForge.net](https://sourceforge.net/projects/waydroid/files/images/)

解压出`system.img` 和 `vendor.img`到：`/etc/waydroid-extra/images/`：

```sh
sudo mkdir -p /etc/waydroid-extra/images
sudo unzip lineage-*-system.zip -d /etc/waydroid-extra/images
sudo unzip lineage-*-vendor.zip -d /etc/waydroid-extra/images
rm lineage-*-system.zip lineage-*-vendor.zip
```

```sh
sudo waydroid init -f
```


## 参数配置

Waydroid 利用多种属性来告诉底层的 Android 系统在一些地方如何表现。我们使用 `waydroid prop set <property> <value>` 命令来设置这些属性。要取消设置属性，可以使用 `waydroid prop set <property> ""`。大多数设置需要重启 Waydroid 会话才能生效。

### 属性列表

- `waydroid prop set persist.waydroid.multi_windows true/false`：启用/禁用与桌面的窗口集成（布尔值）。
- `waydroid prop set persist.waydroid.cursor_on_subsurface true/false`：在某些合成器上，多窗口模式下显示光标的解决方法（布尔值）。
- `waydroid prop set persist.waydroid.invert_colors true/false`：将颜色空间从 RGBA 交换为 BGRA（目前仅与我们修补的 Mutter 兼容）（布尔值）。
- `waydroid prop set persist.waydroid.height_padding 0-9999`：调整高度填充（整型）。
- `waydroid prop set persist.waydroid.width_padding 0-9999`：调整宽度填充（整型）。
- `waydroid prop set persist.waydroid.width 0-9999`：用于用户覆盖所需的分辨率（整型）。
- `waydroid prop set persist.waydroid.height 0-9999`：用于用户覆盖所需的分辨率（整型）。
- `waydroid prop set persist.waydroid.suspend true/false`：当没有应用处于活动状态时，让 Waydroid 容器休眠（在显示超时后）（布尔值，默认在 4.9 及更高版本的内核上为 true）。
- `waydroid prop set persist.waydroid.uevent true/false`：允许 Android 直接访问热插拔设备（布尔值，默认为 false）。

### 修改应用行为

- `waydroid prop set persist.waydroid.fake_touch`：将鼠标输入解释为触摸输入的包名列表，支持使用 * 作为通配符。例如，设置为 "com.rovio.*" 以匹配 Rovio 的所有游戏（字符串，限制 91 个字符）。使用后可以在部分游戏中支持拖拽，但比较黏
- `waydroid prop set persist.waydroid.fake_wifi`：系统将始终显示为已连接到 Wi-Fi 的包名列表，支持使用 * 作为通配符。例如，设置为 "com.gameloft.*" 以匹配 Gameloft 的所有游戏（字符串，限制 91 个字符）。

## 命令

### 使用方法
```bash
waydroid [-h] [-V] [-l LOG] [--details-to-stdout] [-v] [-q]
```

可选参数：
- `-h, --help`：显示帮助信息并退出。
- `-V, --version`：显示程序的版本号并退出。
- `-l LOG, --log LOG`：日志文件路径（默认：/var/lib/waydroid）。
- `--details-to-stdout`：将详细信息（例如构建输出）打印到标准输出，而不是写入日志。
- `-v, --verbose`：在日志文件中写入更多内容（这可能会降低性能）。
- `-q, --quiet`：不输出任何日志消息。
- `-f`：强制选项（与初始化命令一起使用）。
- `-i LOCATION`：从指定位置初始化（默认：/usr/share/waydroid-extra/images）*与初始化动作一起使用*。

### 子命令

- `status`：快速检查 waydroid。
- `log`：跟踪 waydroid 日志文件。
- `init`：设置 waydroid 特定配置并安装镜像。
- `upgrade`：升级镜像。
- `session`：会话控制器。
- `container`：容器控制器。
- `app`：应用程序控制器（见：安装和运行 Android 应用程序）。
- `prop`：Android 属性控制器。
- `show-full-ui`：在窗口中显示 Android 全屏。
- `shell`：运行远程 shell 命令。
- `logcat`：显示 Android 的 logcat。

### 初始化
```bash
waydroid init [-h] [-i IMAGES_PATH] [-f] [-c SYSTEM_CHANNEL] [-v VENDOR_CHANNEL] [-r ROM_TYPE] [-s SYSTEM_TYPE]
```
- `-h | --help`：显示帮助信息并退出。
- `-i IMAGES_PATH | --images_path IMAGES_PATH`：自定义 waydroid 镜像路径（默认在 /var/lib/waydroid/images）。
- `-f | --force`：在重置或使用添加到 /usr/share/waydroid-extra/images 的自定义镜像时使用。
- `-c SYSTEM_CHANNEL | --system_channel SYSTEM_CHANNEL`：自定义系统通道（选项：OTA 通道 URL；默认是官方 OTA 服务器）。
- `-v VENDOR_CHANNEL | --vendor_channel VENDOR_CHANNEL`：自定义供应商通道（选项：OTA 通道 URL；默认是官方 OTA 服务器）。
- `-r ROM_TYPE | --rom_type ROM_TYPE`：Rom 类型（选项："lineage", "bliss" 或 OTA 通道 URL；默认是 LineageOS）。
- `-s SYSTEM_TYPE | --system_type SYSTEM_TYPE`：系统类型（选项：VANILLA, FOSS 或 GAPPS；默认是 VANILLA）。

### 安装Apk

```bash
waydroid app install /path/to/apk
```
可能需要安装 [ARM转译器](Waydroid.md#Arm转译)

### 日志
```bash
waydroid log [-h] [-n LINES] [-c]
```
- `-h | --help`：显示帮助信息并退出。
- `-n LINES | --lines LINES`：初始输出行数。
- `-c | --clear`：清除日志。

### 容器选项
```bash
waydroid container [-h] {start,stop,restart,freeze,unfreeze} ...
```
- `-h | --help`：显示帮助信息并退出。
- 子动作：{start, stop, restart, freeze, unfreeze}
  - `start`：启动容器。
  - `stop`：停止容器。
  - `restart`：重启容器。
  - `freeze`：冻结容器。
  - `unfreeze`：解除容器冻结。

### 会话选项
使用方法：
```bash
waydroid session [-h] {start,stop} ...
```
- `-h | --help`：显示帮助信息并退出。
- 子动作：{start, stop}：启动会话，停止会话。

### 日志
  ```bash
  waydroid log > ~/waydroid-log.txt
  ```
- logcat：
  ```bash
  sudo waydroid logcat > ~/waydroid-logcat.txt
  ```

## Waydroid Script

```bash
git clone https://github.com/casualsnek/waydroid_script
cd waydroid_script
python3 -m venv venv
venv/bin/pip install -r requirements.txt
sudo venv/bin/python3 main.py
```


### 谷歌

在页面中选择google，复制代码，打开网址：["https://google.com/android/uncertified/?pli=1"](https://google.com/android/uncertified/?pli=1). 并填入。

如果不是很需要，可以使用`f-droid`代替商店

### Arm转译

#### AMD

libndk_translation 是来自 Guybrush 固件的库。与 libhoudini 相比，libndk 在 AMD 处理器上似乎有更好的性能。

1. 打开终端。
2. 切换到包含 "main.py" 文件的目录。
3. 执行以下命令来安装 libndk：

```bash
sudo venv/bin/python3 main.py install libndk
```

#### Intel

Intel 的 libhoudini 适用于 Intel/AMD x86 CPU，取自 Microsoft 的 WSA 11 镜像。

- libhoudini 版本：11.0.1b_y.38765.m
- libhoudini64 版本：11.0.1b_z.38765.m

1. 打开终端。
2. 切换到包含 "main.py" 文件的目录。
3. 执行以下命令来安装 libhoudini：

```bash
sudo venv/bin/python3 main.py install libhoudini
```


## 参考

- [Fetching Title#z67c](https://ivonblog.com/posts/archlinux-waydroid/)
- [安全检查](https://wiki.archlinuxcn.org/wiki/Waydroid)
- [Community Projects We Like | Waydroid](https://docs.waydro.id/faq/community-projects-we-like)
- [GitHub - casualsnek/waydroid\_script: Python Script to add OpenGapps, Magisk, libhoudini translation library and libndk translation library to waydroid !](https://github.com/casualsnek/waydroid_script)