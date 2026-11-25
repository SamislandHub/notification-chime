# Notification chime

A [DSH](https://github.com/deepseek-ai/deepseek-harness) (DeepSeek Harness)
plugin that plays a short cue the moment the agent asks for your approval or an
answer — the approval card or the question card that appears next to the
composer.

## Install

DSH Desktop or a Web build with the plugin manager. Either:

- From GitHub (needs `git`):

  ```sh
  dsh plugin --profile desktop add github:SamislandHub/notification-chime
  ```

- From a local checkout: **Settings → Plugins → Add plugin**, pick this
  directory, then restart DSH.

The plugin is a profile bundle: it applies to every session in the profile and
survives restarts. A newly added bundle is read at DSH startup.

## Behavior

- Plays the bundled clip (about 1.2 seconds by default) through the Web Audio API
  when a new pending interaction appears. It does not change, answer, or delay
  anything.
- Falls back to a synthesized two-note "ding-dong" chime if the clip cannot be
  decoded.
- Announces each new pending interaction once. Approval requests are keyed by
  their correlated tool call id, so a page reload does not silence the next
  request; other cards are remembered for 24 hours, so reopening or reviewing an
  old card does not ring again.
- Browsers can suspend audio until the first user gesture. Click or type once in
  the page after opening DSH if you do not hear the first cue.
- No audio file is fetched, and no network request is made while running.

## Where the sound lives

The cue bytes are embedded in `client.js` as the base64 constant
`CHIME_AUDIO_BASE64`. The page decodes them once with
`AudioContext.decodeAudioData`, caches the buffer, and only plays it afterwards.

Embedding is required because the `/plugins` package asset route serves only
`svg / png / webp / jpg / jpeg`; a `wav` or `mp3` shipped in the package would
answer 404. The audio therefore travels with the code and needs no external
file after installation.

The default clip is about 1.2 seconds: 24 kHz mono 16-bit PCM, leading and
trailing silence trimmed, peak normalized to -1.5 dBFS. The repository does not
ship the source audio file.

## Customizing the sound

**Check how it is installed first.** Installing from GitHub places a snapshot
copy in the profile, so editing this checkout has no effect. To customize, clone
the repository and install it **from the local directory** (the profile then
holds a `link:` to that directory), and a page reload applies your edits.

### 1. Prepare a clip

- Format: anything `decodeAudioData` accepts — WAV, MP3, OGG, or M4A; MP3 is the
  smallest.
- Length: 0.6–2 seconds works best. Do **not** leave silence at the start: the
  plugin does not trim it, so it turns into a delay.
- Loudness: normalize it so it does not sound quieter than the rest of the
  system. For example:

  ```powershell
  ffmpeg -i raw.mp3 -af loudnorm=I=-16:TP=-1.5:LRA=11 -ac 1 -ar 24000 -b:a 64k cue.mp3
  ```

### 2. Encode it and replace the constant

Run this in the plugin directory (PowerShell; replace `cue.mp3` with your file):

```powershell
$b64 = [Convert]::ToBase64String([IO.File]::ReadAllBytes('cue.mp3'))
$path = 'client.js'
$text = [IO.File]::ReadAllText($path)
$text = [regex]::Replace($text, "const CHIME_AUDIO_BASE64 = '[^']*'", "const CHIME_AUDIO_BASE64 = '$b64'")
[IO.File]::WriteAllText($path, $text)
```

On macOS or Linux, `base64 -w0 cue.mp3` (`base64 < cue.mp3 | tr -d '\n'` on
macOS) produces the same string to paste between the quotes in `client.js`.

### 3. Reload and verify

Save, reload the DSH page, then trigger an approval or a question card. Hearing
the new clip means it worked. If nothing plays, click once in the page first
(autoplay policy) and check the browser console.

### 4. Prefer the synthesized bell only

Set the constant to an empty string:

```js
const CHIME_AUDIO_BASE64 = ''
```

An empty clip fails to decode, so the plugin falls back to the two-note bell
built in `playNotes()`. To change its pitch, length, or volume, edit the `notes`
array there.

### Notes

- The clip is embedded in the script, so keep it to a few hundred KB at most
  (base64 adds about a third); larger files slow the first load.
- Every replacement starts from scratch and has no cache issue, but a page
  reload is required — the old module is not hot-swapped.
- Do not publish audio you have no right to redistribute.

## Development

| Changed | Effective after |
| --- | --- |
| `client.js` (browser half) | a page reload |
| `index.js`, `cordis.patch.yml`, `package.json` | reinstall the bundle, then restart DSH |

`.tmp/` holds the conversion script and the pre-clip copy of `client.js`; it is
ignored by git and can be deleted at any time.

## License

MIT.

[中文](README.md)
