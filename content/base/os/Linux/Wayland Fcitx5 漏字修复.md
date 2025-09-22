---
tags:
  - wayland
  - fcitx5
---
在Arch下切换了KDE-Wayland显示，但是有一些奇怪的漏字现象。

当输入一句中文之后，会有一些拼音的字母跳过输入法直接进入程序，伴随着UI的卡死，形成类似于 `你好`=>`a拟合哦` 这样的奇怪句子。

解决方法是编辑Chrome的 `.desktop` 文件，在命令后添加以下参数即可。 `--enable-features=UseOzonePlatform --ozone-platform=wayland --enable-wayland-ime`

类似的解决方法也适用于所有Electron开发的应用程序，例如`vscode`、`typora`、`obsidian`、`哔哩哔哩`等