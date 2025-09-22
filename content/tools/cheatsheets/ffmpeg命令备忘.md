---
tags:
  - cheatsheet
  - tool/ffmpeg
---

## 基本转换

```sh
ffmpeg -i in.mp4 out.avi
```
### 将 MKV 文件重新混合为 MP4

```sh
ffmpeg -i in.mkv -c:v copy -c:a copy out.mp4
```

### 高质量编码
使用`crf`（恒定速率因子）参数来控制输出质量。cRF越低，质量越高（范围：0-51）。默认值为23，视觉上无损压缩对应于`-crf 18`. 使用该`preset`参数来控制压缩过程的速度。附加信息： https: [//trac.ffmpeg.org/wiki/Encode/H.264](https://trac.ffmpeg.org/wiki/Encode/H.264)

```sh
ffmpeg -i in.mp4 -preset slower -crf 18 out.mp4
```

## 修剪

无需重新编码：

```sh
ffmpeg -ss [start] -i in.mp4 -t [duration] -c copy out.mp4
```

- [`-ss`](http://ffmpeg.org/ffmpeg-all.html#Main-options)指定开始时间，例如`00:01:23.000`或`83`（以秒为单位）
- [`-t`](http://ffmpeg.org/ffmpeg-all.html#Main-options)指定剪辑的持续时间（相同格式）。
- 最近`ffmpeg`还有一个标志来提供结束时间`-to`。
- [`-c`](http://ffmpeg.org/ffmpeg-all.html#Main-options)copy 将第一个视频、音频和字幕比特流从输入复制到输出文件，而不对其进行重新编码。这不会损害质量并使命令在几秒钟内运行。

重新编码：

如果您省略该`-c copy`选项，`ffmpeg`将根据您选择的格式自动重新编码输出视频和音频。对于高质量视频和音频，请分别阅读[x264 编码指南](https://ffmpeg.org/trac/ffmpeg/wiki/x264EncodingGuide)和[AAC 编码指南](http://ffmpeg.org/trac/ffmpeg/wiki/AACEncodingGuide)。

例如：

```sh
ffmpeg -ss [start] -i in.mp4 -t [duration] -c:v libx264 -c:a aac -strict experimental -b:a 128k out.mp4
```

## 复用另一个视频中的视频和音频

要从 `in0.mp4` 复制视频并从 `in1.mp4` 复制音频：

```sh
ffmpeg -i in0.mp4 -i in1.mp4 -c copy -map 0:0 -map 1:1 -shortest out.mp4
```

- 使用[-c copy](http://ffmpeg.org/ffmpeg.html#Stream-copy)流将被`stream copied`重新编码，因此不会有质量损失。如果要重新编码，请参阅[FFmpeg Wiki：H.264 编码指南](https://trac.ffmpeg.org/wiki/Encode/H.264)。
- 该`-shortest`选项将使输出持续时间与最短输入流的持续时间相匹配。
- 有关详细信息，请参阅[`-map`选项文档。](http://ffmpeg.org/ffmpeg.html#Advanced-options)

## 提取音频

```sh
ffmpeg -i test.mp4 -f mp3 -vn test.mp3
```

## 连接解复用器

首先，制作一个文本文件。

```sh
file 'in1.mp4'
file 'in2.mp4'
file 'in3.mp4'
file 'in4.mp4'
```

然后，运行`ffmpeg`：

```sh
ffmpeg -f concat -i list.txt -c copy out.mp4
```

## 延迟音频/视频

视频延迟 3.84 秒：

```sh
ffmpeg -i in.mp4 -itsoffset 3.84 -i in.mp4 -map 1:v -map 0:a -vcodec copy -acodec copy out.mp4
```

音频延迟 3.84 秒：

```sh
ffmpeg -i in.mp4 -itsoffset 3.84 -i in.mp4 -map 0:v -map 1:a -vcodec copy -acodec copy out.mp4
```

## 刻录字幕

使用[libass](http://ffmpeg.org/ffmpeg.html#ass)库（确保您的 ffmpeg 安装在配置中包含该库`--enable-libass`）。

首先将字幕转换为.ass格式：

```sh
ffmpeg -i sub.srt sub.ass
```

然后使用视频过滤器添加它们：

```sh
ffmpeg -i in.mp4 -vf ass=sub.ass out.mp4
```

## 从视频中提取帧

要提取 1 到 5 秒以及 11 到 15 秒之间的所有帧：

```sh
ffmpeg -i in.mp4 -vf select='between(t,1,5)+between(t,11,15)' -vsync 0 out%d.png
```

每秒仅提取一帧：

```sh
ffmpeg -i in.mp4 -fps=1 -vsync 0 out%d.png
```

## 旋转视频

顺时针旋转90度：

```sh
ffmpeg -i in.mov -vf "transpose=1" out.mov
```

对于转置参数，您可以传递：

```sh
0 = 90CounterCLockwise and Vertical Flip (default)
1 = 90Clockwise
2 = 90CounterClockwise
3 = 90Clockwise and Vertical Flip
```

使用`-vf "transpose=2,transpose=2"`180度。

## 下载“传输流”视频流

1. 找到播放列表文件，例如使用 Chrome > F12 > Network > Filter: m3u8
2. 下载并连接视频片段：

```sh
ffmpeg -i "path_to_playlist.m3u8" -c copy -bsf:a aac_adtstoasc out.mp4
```

如果您收到“协议‘https 不在白名单‘文件，加密’！” 错误，添加`protocol_whitelist`选项：

```sh
ffmpeg -protocol_whitelist "file,http,https,tcp,tls" -i "path_to_playlist.m3u8" -c copy -bsf:a aac_adtstoasc out.mp4
```

## 将部分音频静音

要将前 90 秒的音频替换为静音：

```sh
ffmpeg -i in.mp4 -vcodec copy -af "volume=enable='lte(t,90)':volume=0" out.mp4
```

要将 1'20" 到 1'30" 之间的所有音频替换为静音：

```sh
ffmpeg -i in.mp4 -vcodec copy -af "volume=enable='between(t,80,90)':volume=0" out.mp4
```

## 去隔行

使用“另一个去隔行过滤器”去隔行。

```sh
ffmpeg -i in.mp4 -vf yadif out.mp4
```

## 从图像创建视频幻灯片

参数：`-r`标记图像帧率（每幅图像的反比时间）；`-vf fps=25`标记输出的真实帧率。

```sh
ffmpeg -r 1/5 -i img%03d.png -c:v libx264 -vf fps=25 -pix_fmt yuv420p out.mp4
```

## 从视频中提取图像

- 提取所有帧：`ffmpeg -i input.mp4 thumb%04d.jpg -hide_banner`
- 每秒提取一帧：`ffmpeg -i input.mp4 -vf fps=1 thumb%04d.jpg -hide_banner`
- 仅提取一帧：`ffmpeg -i input.mp4 -ss 00:00:10.000 -vframes 1 thumb.jpg`

## 显示每帧的帧编号

```sh
ffmpeg -i in.mov -vf "drawtext=fontfile=arial.ttf: text=%{n}: x=(w-tw)/2: y=h-(2*lh): fontcolor=white: box=1: boxcolor=0x00000099: fontsize=72" -y out.mov
```

## 元数据：更改标题

```sh
ffmpeg -i in.mp4 -map_metadata -1 -metadata title="My Title" -c:v copy -c:a copy out.mp4
```

## 工具

https://ffmpeg.lav.io/是一个用于编写 FFmpeg 操作的交互式资源。