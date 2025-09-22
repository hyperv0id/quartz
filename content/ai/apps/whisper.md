---
tags:
  - ai
  - ai/text-to-speech
aliases:
  - opanai-whisper
---
# 语音转文字

- openai 仓库地址：[GitHub - openai/whisper: Robust Speech Recognition via Large-Scale Weak Supervision](https://github.com/openai/whisper)
- 示例代码：[Colab Example](https://colab.research.google.com/github/openai/whisper/blob/master/notebooks/LibriSpeech.ipynb)
- 博客地址：[Introducing Whisper](https://openai.com/blog/whisper)
- 论文地址：[[2212.04356] Robust Speech Recognition via Large-Scale Weak Supervision](https://arxiv.org/abs/2212.04356)
- 李沐老师whisper精度论文视频：[OpenAI Whisper 精读【论文精读·45】\_哔哩哔哩\_bilibili](https://www.bilibili.com/video/BV1VG4y1t74x/?spm_id_from=333.999.0.0)

# 环境搭建

准备conda虚拟环境
```sh
# 创建虚拟环境
conda create -n whisper python=3.9
# 进入虚拟环境
# conda init [shell名字] 笔者此处使用 zsh
conda init zsh

source activate whisper
# conda activate whisper
```

安装python仓库
```sh
pip install git+https://github.com/openai/whisper.git

pip install jiwer
```

安装ffmpeg，大多数包管理器都可以提供该工具：

```shell
# on Ubuntu or Debian
sudo apt update && sudo apt install ffmpeg

# on Arch Linux
sudo pacman -S ffmpeg

# on MacOS using Homebrew (https://brew.sh/)
brew install ffmpeg

# on Windows using Chocolatey (https://chocolatey.org/)
choco install ffmpeg

# on Windows using Scoop (https://scoop.sh/)
scoop install ffmpeg
```


# 开始使用

## 命令行

这个命令会使用small模型将音频生成字幕，并同时生成包括`srt`，`json`，`txt`在内的多种文本文件
```sh
$ whisper micro-machines.wav --model small
100%|███████████████████████████████████████| 461M/461M [01:22<00:00, 5.86MiB/s]
Detecting language using up to the first 30 seconds. Use `--language` to specify the language
Detected language: English
[00:00.000 --> 00:03.140]  This is the Micro Machine Man, presenting the most midget miniature motorcade of Micro Machine.
[00:03.140 --> 00:06.640]  Each one has dramatic details, terrific trims, precision paint jobs, plus incredible Micro Machine Pocket Playsets.
[00:06.640 --> 00:08.760]  There\'s a police station, fire station, restaurant, service station, and more.
[00:08.760 --> 00:10.260]  Perfect pocket portables to take any place.
[00:10.260 --> 00:15.200]  And there are many miniature play sets to play with, and each one comes with its own special edition Micro Machine Vehicle and fun, fantastic features that miraculously move.
[00:15.200 --> 00:19.160]  Raise the boat lift at the airport, marina, man, the gun turret at the army base, clean your car at the car wash, raise the toll bridge.
[00:19.160 --> 00:21.160]  And these play sets fit together to form a Micro Machine World.
[00:21.160 --> 00:25.220]  Micro Machine Pocket Playsets are tremendously tiny, so perfectly precise, so dazzlingly detailed, you\'ll want to pocket them all.
[00:25.220 --> 00:27.700]  Micro Machines and Micro Machine Pocket Playsets sold separately from Golube.
[00:27.700 --> 00:29.700]  The smaller they are, the better they are.
```

## python脚本

```python
import whisper

model = whisper.load_model("small")
result = model.transcribe("micro-machines.wav")
print(result["text"])
```

输出如下
```sh
$ python3 main.py
 This is the Micro Machine Man presenting the most midget miniature motorcade of Micro Machine. Each one has dramatic details, terrific trims, precision paint jobs, plus incredible Micro Machine Pocket Places. There\'s a police station, fire station, restaurant, service station, and more. Perfect pocket portables to take any place. And there are many miniature play sets to play with, and each one comes with its own special edition. Micro Machine Vehicle and fun, fantastic features that miraculously move. Raise the boat lift at the airport, Marina Man, the gun turret at the Army Base, clean your car at the car wash. Raise the toll bridge. And these play sets fit together to form a Micro Machine World. Micro Machine Pocket Places, so tremendously tiny, so perfectly precise, so dazzlingly detailed. You all want to pocket them all. Micro Machines and Micro Machine Pocket Places sold separately from Golube. The smaller they are, the better they are.
```

一个更丰富的案例
```python
import whisper

model = whisper.load_model("base")

# load audio and pad/trim it to fit 30 seconds
audio = whisper.load_audio("audio.mp3")
audio = whisper.pad_or_trim(audio)

# make log-Mel spectrogram and move to the same device as the model
mel = whisper.log_mel_spectrogram(audio).to(model.device)

# detect the spoken language
_, probs = model.detect_language(mel)
print(f"Detected language: {max(probs, key=probs.get)}")

# decode the audio
options = whisper.DecodingOptions()
result = whisper.decode(model, mel, options)

# print the recognized text
print(result.text)
```

# 延伸应用

- [faster-whisper](faster-whisper.md)