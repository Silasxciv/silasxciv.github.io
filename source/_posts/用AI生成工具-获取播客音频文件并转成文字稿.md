---
layout: post
title: 用AI生成工具-获取播客音频文件并转成文字稿
comments: true
tags:
  - 教程
date: 2026-07-28 23:01:14
---
播客如今是我获取信息的重要途经，有很多高质量的播客，谈论的内容听一遍比较难懂，有些播客没做时间轴，重复听某一段内容还是较为困难的，所以打算用AI生成一个工具，我给它一个播客链接，它能自动获取播客的音频文件，并转换为文字稿，方便我后期再用AI工具整理。
<!--more-->
## 1. 先用AI帮我生成prompt
在DeepSeek上键入如下内容

>我是个编程小白，我打算用python做一个程序，我给这个程序一个小宇宙播客的链接，它会自己扒这个播客的音频文件，之后自己转成逐字稿，给我输出这个音频和逐字稿就行，以播客单集名称命名，请帮我写一个prompt

这样就能得到AI帮我写的prompt

>角色：你是一个精通 Python 的爬虫与音频处理专家。
>背景：我是一个完全不懂编程的小白，使用 Windows/Mac 系统。
>任务：请为我生成一个完整的 Python 脚本。
>输入：用户复制粘贴的一个“小宇宙”播客单集链接（例如：https://www.xiaoyuzhoufm.com/episode/xxxxx）。
>核心功能要求：
>
>防屏蔽下载：小宇宙有反爬机制，必须使用 yt-dlp 库，并携带 Referer 请求头来下载音频（MP3/M4A 格式）。
>智能命名：自动从网页标题中提取该播客的单集名称，下载的音频文件和输出的逐字稿必须都用这个名字命名（例如：《故事FM第100期.mp3和故事FM第100期.txt`）。
>离线转写：使用本地 OpenAI Whisper 模型（加载 small 或 base 模型）将音频转成带时间戳的逐字稿（纯文本即可）。
>输出位置：所有文件保存在代码所在的目录下。
>依赖清单：请明确列出我需要安装的 Python 库，以及电脑必须安装的 ffmpeg 程序。
>容错：添加简单的报错暂停逻辑，如果下载失败，提示我检查网络，不要直接闪退。

## 2. 把AI帮我生成prompt输入到AI里
在Gmini里键入上面的prompy，AI就会按依次生成步骤，参照AI的步骤进行操作就可以，如果哪一步执行错误，将报错再粘贴给AI，它就会告诉你解决方法

### 2.1. 安装 ffmpeg
Windows 系统：
最简单的安装方式是打开“命令提示符”（CMD），输入以下命令按回车（Windows 10/11 适用）：

```
winget install Gyan.FFmpeg
```

事实上Gmini第一次告诉我键入上面的命令即可，但我执行时报错winget无法识别，告诉它后，它重新给我下面的方法

1. 下载 ffmpeg 压缩包
打开浏览器，访问 ffmpeg 官方推荐的 Windows 下载地址：
👉 https://www.gyan.dev/ffmpeg/builds/

在网页中找到 ffmpeg-git-full.7z 或 ffmpeg-release-full.7z 链接点击下载（如果你的电脑解压不了 .7z 格式，可以下载 ffmpeg-release-essentials.zip）。

2. 解压并提取文件
下载完成后，把压缩包解压出来。

进入解压后的文件夹，找到名为 bin 的文件夹。

进入 bin 文件夹，你会看到里面有 ffmpeg.exe、ffplay.exe 和 ffprobe.exe 这三个文件。

3. 将 ffmpeg 放到脚本同一个目录下（最省事的招数！）
直接把 bin 文件夹里的 ffmpeg.exe 复制一份，粘贴到你存放 Python 脚本（xyz_transcribe.py）的同一个文件夹里。

原因：Python 在执行程序时，会自动优先检查当前文件夹。只要 ffmpeg.exe 和你的 Python 脚本在同一个目录下，Whisper 和 yt-dlp 就能直接识别并使用它，不需要配置任何复杂的系统环境变量！

### 2.2. 安装 Python 依赖库
打开你的终端（Mac）或命令提示符（Windows CMD），依次运行以下两条命令来安装所需的 Python 扩展库：

```
pip install yt-dlp openai-whisper
```

提示：openai-whisper 在第一次运行转写时，会自动下载需要的 PyTorch 依赖和 Whisper 模型库（约几百兆到 1 GB），请确保网络稳定。

### 2.2. 完整的 Python 脚本代码
将下面的完整代码保存为一个名为 xyz_transcribe.py 的文件，放在你喜欢的任意文件夹中。

```
import os
import sys
import re
import yt_dlp
import whisper

def sanitize_filename(name):
    """清理文件名中的非法字符，防止保存失败"""
    return re.sub(r'[\\/*?:"<>|]', '_', name)

def download_audio(url):
    """使用 yt-dlp 下载小宇宙播客音频，并添加 Referer 防屏蔽"""
    print("正在解析并下载音频，请稍候...")
    
    # 配置 yt-dlp 下载选项
    ydl_opts = {
        # 强制设置 Referer 请求头以应对反爬
        'http_headers': {
            'Referer': 'https://www.xiaoyuzhoufm.com/',
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/120.0.0.0 Safari/537.36',
        },
        'format': 'm4a/bestaudio/best',  # 优先下载高质量音频
        'outtmpl': '%(title)s.%(ext)s',  # 默认使用单集标题命名文件
        'quiet': False,
        'no_warnings': True,
    }

    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            # 获取单集元信息
            info = ydl.extract_info(url, download=True)
            # 提取原始标题并清理非法字符
            raw_title = info.get('title', 'podcast_episode')
            clean_title = sanitize_filename(raw_title)
            
            # 获取实际下载的文件扩展名
            ext = info.get('ext', 'm4a')
            downloaded_file = f"{clean_title}.{ext}"

            # 兼容处理：如果文件名有细微差异，寻找最新生成的音频文件
            if not os.path.exists(downloaded_file):
                # 尝试寻找包含标题的文件
                for f in os.listdir('.'):
                    if clean_title in f and f.endswith(('.m4a', '.mp3', '.aac')):
                        downloaded_file = f
                        break

            print(f"✅ 音频下载成功：{downloaded_file}")
            return downloaded_file, clean_title

    except Exception as e:
        print("\n❌ 音频下载失败！")
        print(f"错误信息: {e}")
        print("💡 建议排查：1. 检查输入的链接是否正确；2. 检查网络连接是否正常。")
        return None, None

def transcribe_audio(audio_path, output_title, model_size="base"):
    """使用本地 OpenAI Whisper 模型转写音频并生成带时间戳的 txt 文件"""
    if not audio_path or not os.path.exists(audio_path):
        print("未找到可用的音频文件，跳过转写步骤。")
        return

    print(f"\n正在加载 Whisper ({model_size}) 模型（首次运行需下载模型文件，请保持网络畅通）...")
    try:
        model = whisper.load_model(model_size)
    except Exception as e:
        print(f"❌ Whisper 模型加载失败: {e}")
        return

    print("🎙️ 开始转写音频，这可能需要几分钟（取决于电脑配置与音频时长）...")
    try:
        # 执行转写
        result = model.transcribe(audio_path, verbose=False)

        # 构造输出的文本文件名
        txt_filename = f"{output_title}.txt"

        print(f"正在保存逐字稿到：{txt_filename}")
        with open(txt_filename, "w", encoding="utf-8") as f:
            for segment in result.get("segments", []):
                # 格式化时间戳 (例如 00:01:23)
                start_sec = int(segment['start'])
                start_time = f"{start_sec // 3600:02d}:{(start_sec % 3600) // 60:02d}:{start_sec % 60:02d}"
                text = segment['text'].strip()
                
                # 写入 [HH:MM:SS] 内容 格式
                f.write(f"[{start_time}] {text}\n")

        print(f"🎉 转写完成！逐字稿已保存为：《{txt_filename}》")

    except Exception as e:
        print(f"❌ 音频转写过程中出错: {e}")

def main():
    print("=" * 50)
    print("      小宇宙播客单集音频下载与离线转写工具      ")
    print("=" * 50)

    # 提示用户粘贴链接
    url = input("请输入小宇宙单集链接 (例如 https://www.xiaoyuzhoufm.com/episode/xxxxx):\n").strip()

    if not url:
        print("⚠️ 输入链接为空，程序退出。")
        input("\n按回车键退出...")
        sys.exit()

    # 1. 下载音频
    audio_file, clean_title = download_audio(url)

    # 2. 如果下载成功，开始离线转写
    if audio_file and clean_title:
        # 可选模型大小: "tiny", "base", "small", "medium", "large"
        # 推荐使用 "base" 或 "small"，在速度与准确率之间取得平衡
        transcribe_audio(audio_file, clean_title, model_size="base")

    print("\n任务已结束。")
    # 防闪退逻辑：等用户按回车键后再关闭窗口
    input("按回车键退出程序...")

if __name__ == "__main__":
    main()
```

### 2.3. 运行程序
Windows：在保存 xyz_transcribe.py 的文件夹空白处按住 Shift 键点击鼠标右键，选择 “在此处打开 PowerShell 窗口” 或 “在终端中打开”。

输入以下命令并按回车：

```
python xyz_transcribe.py
```

操作流程：
按照提示粘贴你的小宇宙单集 URL，例如：[https://www.xiaoyuzhoufm.com/episode/606c3a](https://www.xiaoyuzhoufm.com/episode/606c3a)...

程序会自动开始下载音频。

下载完成后， Whisper 模型会自动开始离线转写。

处理结束后，同级目录下会生成两个文件：

播客单集名称.m4a（或 .mp3）

播客单集名称.txt（包含 [00:01:23] 逐字稿内容... 带时间戳的文件）

## 3. 试用代码
我在Hyper V虚拟机里，参照AI给的步骤，尝试AI给我的步骤

在命令行里，用Python运行这段代码，随意复制一段小宇宙播客的链接给到命令行，程序正常开始执行音频下载和转写

命令行界面:
![插入图片](/assets/images/260728_1.jpg "命令行界面")

程序执行的结果：
![插入图片](/assets/images/260728_2.jpg "程序执行的结果")

## 4. 把代码转成非Python环境可执行的程序
事实证明AI生成的工具有效，之后我通过下面的指令，打算把py转换成exe程序

```
pyinstaller --onefile your_script.py
```

在外面宿主计算机里新建文件夹，单独放置ffmpeg.exe和生成的xyz_transcribe.exe，在没有安装Python环境的计算机里测试程序，程序报错
![插入图片](/assets/images/260728_3.jpg "程序执行报错")

把报错粘贴给AI，描述情况，AI给了答案和新方法，因为Whisper 库在转写音频时，需要用到它内部自带的一个音频滤镜文件 mel_filters.npz。
当使用 PyInstaller 打包成 EXE 时，PyInstaller 默认只打包了 .py 代码，忽略了这个非 Python 代码的数据资源文件。所以软件解压运行到临时目录（_MEI221882）时，找不到这个文件就报错了。

```
pyinstaller -F --add-data "C:\Users\Administrator\AppData\Local\Programs\Python\Python314\Lib\site-packages\whisper;whisper" xyz_transcribe.py
```

执行前需要手动先确认下路径，因为AI不知道当前Python版本，路径可能不匹配

重新执行的效果
![插入图片](/assets/images/260728_4.jpg "程序执行正常")

程序可以正常执行，那么之后听到需要重复收听的播客，再通过AI整理文字稿的时候，在没有配置Python环境的宿主计算机上处理也会比较方便。