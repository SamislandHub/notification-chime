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

- 出现新的待处理交互时，通过 Web Audio API 播放内置音频（约 1.2 秒）。它不会改动、代答或延迟任何审批和提问。
- 如果音频解码失败，回退为合成的两声「叮咚」铃声。
- 每条新交互只播报一次。审批请求以关联的工具调用 id 作为去重键，所以刷新页面不会让下一条审批静音；其他卡片记住 24 小时，重新打开或查看旧卡片不会再响。
- 浏览器可能把音频挂起到用户第一次交互之后。打开 DSH 后先点一下或按一次键，否则第一声可能听不到。
- 运行时不请求网络，也不额外读取音频文件。

## 声音文件在哪

- `assets/cue.wav` 是源音频：24 kHz 单声道 16-bit PCM，已去掉首尾静音、峰值归一化到 -1.5 dBFS，并加了 5 ms 淡入淡出。
- 运行时**不使用**这个文件，而是用 `client.js` 里内嵌的 base64（常量 `CHIME_AUDIO_BASE64`），通过 `AudioContext.decodeAudioData` 解码一次后播放。原因是 DSH 的 `/plugins` 资源路由只服务 `svg / png / webp / jpg / jpeg`，`wav` 放在包里会被 404，所以音频随代码一起走。

替换音频的步骤：换掉 `assets/cue.wav`，把新文件转成 base64 填进 `client.js` 的 `CHIME_AUDIO_BASE64`，然后刷新页面即可。

## 开发

| 改动 | 生效方式 |
| --- | --- |
| `client.js`（浏览器半边） | 刷新页面 |
| `index.js`、`cordis.patch.yml`、`package.json` | 重新安装 bundle，然后重启 DSH |

`.tmp/` 里是准备音频时用的转换脚本和替换前的 `client.js` 副本，已被 git 忽略，可以随时删除。

## 许可证

MIT。内置音频由作者以同一许可证发布。

[English](README.en.md)
