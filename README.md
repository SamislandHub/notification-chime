# Notification chime

A [DSH](https://github.com/deepseek-ai/deepseek-harness) plugin that plays a
short cue the moment the agent asks for your approval or an answer — the
approval card or the question card that appears next to the composer.

## Install

DSH Desktop or a Web build with the plugin manager. Either:

- From GitHub (needs `git`):

  ```sh
  dsh plugin --profile desktop add github:SamislandHub/notification-chime
  ```

- From a local checkout: **Settings → Plugins → Add plugin**, pick this
  directory, then restart DSH.

The plugin is a profile bundle: it applies to every session in the profile and
survives restarts.

## Behavior

- Plays the bundled clip (about 1.2 seconds) through the Web Audio API when a new
  pending interaction appears. It does not change, answer, or delay anything.
- Falls back to a synthesized two-note "ding-dong" chime if the clip cannot be
  decoded.
- Announces each new pending interaction once. Approval requests are keyed by
  their correlated tool call id, so a page reload does not silence the next
  request; other cards are remembered for 24 hours, so reopening or reviewing an
  old card does not ring again.
- Browsers can suspend audio until the first user gesture. Click or type once in
  the page after opening DSH if you do not hear the first cue.
- No audio file is fetched, and no network request is made while running.

## How the clip is stored

The `/plugins` package asset route serves only image extensions, so the cue is
embedded in `client.js` as base64 (`CHIME_AUDIO_BASE64`) and decoded once per
page with `AudioContext.decodeAudioData`.

The embedded audio is a 24 kHz mono 16-bit PCM WAV: leading and trailing silence
trimmed, peak normalized to -1.5 dBFS, and given 5 ms fades to keep the edges
click-free.

## Development

| Changed | Effective after |
| --- | --- |
| `client.js` (browser half) | a page reload |
| `index.js`, `cordis.patch.yml`, `package.json` | reinstall the bundle, then restart DSH |

The conversion script and the pre-clip copy of `client.js` live in `.tmp/`,
which is ignored by git; delete it whenever you like.

## License

MIT. The bundled audio clip is released under the same license by its author.
