# 通知提示音

一个 [DSH](https://github.com/deepseek-ai/deepseek-harness)（DeepSeek Harness）插件：当智能体请求审批或向你提问时，播放一段简短的提示音——也就是编辑器上方那张审批卡或提问卡出现的瞬间。

## 安装

需要 DSH Desktop，或带插件管理器的 Web 版本。两种方式：

- 从 GitHub 安装（目标机器需要 `git`）：

  ```sh
  dsh plugin --profile desktop add github:SamislandHub/notification-chime
  ```

- 从本地目录安装：**设置 → 插件 → 添加插件**，选中本目录，然后重启 DSH。

这是一个 profile bundle：装一次对该 profile 下的所有会话生效，重启后依然存在。新增 bundle 需要重启 DSH 才会被读取。

## 行为

- 出现新的待处理交互时，通过 Web Audio API 播放内置音频（默认约 1.2 秒）。它不会改动、代答或延迟任何审批和提问。
- 如果音频解码失败，回退为合成的两声「叮咚」铃声。
- 每条新交互只播报一次。审批请求以关联的工具调用 id 作为去重键，所以刷新页面不会让下一条审批静音；其他卡片记住 24 小时，重新打开或查看旧卡片不会再响。
- 浏览器可能把音频挂起到用户第一次交互之后。打开 DSH 后先点一下或按一次键，否则第一声可能听不到。
- 运行时不请求网络，也不读取外部音频文件。

## 声音是怎么来的

提示音的字节直接以 base64 内嵌在 `client.js` 的常量 `CHIME_AUDIO_BASE64` 里，页面加载后通过 `AudioContext.decodeAudioData` 解码一次、缓存，之后每次触发只做一次播放。

这样做是因为 DSH 的 `/plugins` 资源路由只服务 `svg / png / webp / jpg / jpeg`，`wav`、`mp3` 这类文件放在包里会被 404。内嵌之后音频随代码一起走，安装完就有声音，不依赖任何外部文件。

默认音频约 1.2 秒：24 kHz 单声道 16-bit PCM，去掉首尾静音，峰值归一化到 -1.5 dBFS。仓库不附带源音频文件。

## 定制声音

**先确认安装方式**：从 GitHub 安装的是快照副本，改本目录的 `client.js` 不会生效。要定制，请把仓库克隆到本地，用**从本地目录安装**的方式装（profile 里是 `link:` 指向该目录），这样改完刷新页面即可。

### 1. 准备一段音频

- 格式：浏览器 `decodeAudioData` 支持的任意格式，WAV / MP3 / OGG / M4A 都可以，MP3 体积最小。
- 长度：0.6–2 秒最合适；**开头不要留静音**，运行时不会帮你裁剪，静音会变成提示延迟。
- 响度：归一化一下，避免比系统提示音轻太多。例如：

  ```powershell
  ffmpeg -i raw.mp3 -af loudnorm=I=-16:TP=-1.5:LRA=11 -ac 1 -ar 24000 -b:a 64k cue.mp3
  ```

### 2. 转成 base64 并替换常量

在插件目录下执行（PowerShell，把 `cue.mp3` 换成你的文件）：

```powershell
$b64 = [Convert]::ToBase64String([IO.File]::ReadAllBytes('cue.mp3'))
$path = 'client.js'
$text = [IO.File]::ReadAllText($path)
$text = [regex]::Replace($text, "const CHIME_AUDIO_BASE64 = '[^']*'", "const CHIME_AUDIO_BASE64 = '$b64'")
[IO.File]::WriteAllText($path, $text)
```

macOS / Linux 可以用 `base64 -w0 cue.mp3`（macOS 去掉 `-w0`，用 `base64 < cue.mp3 | tr -d '\n'`）得到同样的字符串，再手工替换 `client.js` 里引号内的内容。

### 3. 刷新页面验证

保存后刷新 DSH 页面，然后触发一次审批或提问——听到新音频即成功。没声音时先确认是否点过页面（自动播放限制），再看浏览器开发者工具的 Console。

### 4. 只想要合成铃声

把常量设为空字符串即可：

```js
const CHIME_AUDIO_BASE64 = ''
```

空音频会解码失败，插件自动回退到 `playNotes()` 合成的「叮咚」声；想改那两声的音高、长短或音量，直接改 `playNotes()` 里的 `notes` 数组。

### 注意

- 音频整体内嵌进脚本，建议控制在几百 KB 以内（base64 会让体积增大约 1/3），否则首屏加载会变慢。
- 每次替换都是从零开始，没有缓存问题；但改完必须刷新页面，旧模块不会自动重载。
- 不要把有版权争议的音频放进公开仓库。

## 开发

| 改动 | 生效方式 |
| --- | --- |
| `client.js`（浏览器半边） | 刷新页面 |
| `index.js`、`cordis.patch.yml`、`package.json` | 重新安装 bundle，然后重启 DSH |

`.tmp/` 里是准备音频时用的转换脚本和替换前的 `client.js` 副本，已被 git 忽略，可以随时删除。

## 许可证

MIT。

[English](README.en.md)
