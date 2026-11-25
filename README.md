# claudespeak

**Hear Claude Code's replies out loud.**

claudespeak is a Claude Code plugin. When Claude finishes a reply, it speaks a one-to-three sentence summary of it in a natural voice from [Fish Audio](https://fish.audio). You can step away from the screen, or work in another window, and still hear what Claude did and whether it needs anything from you.

> "I fixed the login bug and the tests pass now. You'll want to check the new cookie timeout before you deploy."

It runs on **macOS** and **Linux**.

---

## How it works

1. **Claude finishes a reply.** claudespeak waits for the final answer. It ignores sub-agents and tool calls.
2. **The reply is turned into speech.**
   - A short, plain reply ("Done. All tests pass.") is spoken exactly as written.
   - A longer reply goes to a small, fast model (Haiku by default), which writes a 1–3 sentence spoken summary. That summary never contains code, file paths, URLs or markdown. If the summary model doesn't answer in time, the first sentences of the reply are used instead.
3. **Fish Audio speaks it.** The text is sent to Fish Audio's text-to-speech service. The audio streams straight into `ffmpeg` and out of your speakers, so it starts playing as soon as the first chunk arrives instead of waiting for the whole clip.

### It knows when to be quiet

- **One voice at a time.** Even with several Claude Code sessions open, only one voice plays. A new reply anywhere cuts off the one before it.
- **Typing stops it.** Sending a new prompt cuts off the previous reply's speech.
- **Only sessions you're watching speak.** It stays silent in sub-agents, `claude -p`, scheduled routines, Cowork, and any `claude` started from inside another session.

---

## Requirements

| | |
| --- | --- |
| **Operating system** | macOS, or Linux with PulseAudio, PipeWire or ALSA. Windows is not supported. |
| **ffmpeg** | macOS: `brew install ffmpeg` · Linux: `sudo apt install ffmpeg` (or your distro's package) |
| **Fish Audio API key** | Create one at <https://fish.audio/app/api-keys/> |
| **Claude Code** | A build that loads plugin hooks modules. Tested on 2.1.309 (macOS) and 2.1.331 (Linux). |

**Where sound goes**

- **macOS:** your default output device, through ffmpeg's `audiotoolbox` output.
- **Linux:** PulseAudio when a sound server is running. PipeWire, the default on current Ubuntu and Fedora desktops, also counts. Otherwise it uses ALSA's default device. Run `ffmpeg -hide_banner -devices` to check that your ffmpeg has the `pulse` or `alsa` output; distro builds include both.

---

## Install

```bash
claude plugin marketplace add albertshiney/claudespeak
claude plugin install claudespeak@claudespeak
```

For the Linux version, install from this fork's `linux-support` branch until the change is merged upstream:

```bash
git clone -b linux-support https://github.com/gmorkis/claudespeak ~/claudespeak
claude plugin marketplace add ~/claudespeak
claude plugin install claudespeak@claudespeak
```

Then restart Claude Code, or run `/reload-plugins` in an open session.

### Add your Fish Audio key

Run `/speak key`. It creates `~/.claude/claudespeak/.env` and opens it. Paste your key after `FISH_API_KEY=` and save. The change takes effect on the next reply, with no restart needed.

- The file is created readable only by you (`chmod 600`).
- It lives outside the plugin folder, so plugin updates never wipe it.
- A `FISH_API_KEY` environment variable takes priority over the file.
- The key is passed to `curl` through standard input, never on the command line, so other programs on your machine can't read it from the process list.
- On Linux the file is opened with `xdg-open`. If that's missing, open the printed path in any editor.

If you skip this step, your first finished reply creates the file and tells you where it is.

---

## Commands

claudespeak is **on by default**. `/speak on` and `/speak off` apply to every session on the machine at once, including ones already open.

| Command | What it does |
| --- | --- |
| `/speak` | Says whether spoken replies are on, and whether this session can speak |
| `/speak on` | Reads replies aloud in every session |
| `/speak off` | Stops reading aloud in every session, and cuts off anything playing |
| `/speak stop` | Cuts off the current speech in any session, without changing the setting |
| `/speak key` | Creates the key file if needed and opens it |

---

## Settings

Three options appear in the plugin's settings:

| Option | Default | What it is |
| --- | --- | --- |
| `voiceId` | Sawyer (`fa4c9eb3dccc4806b382b40d61c6b10a`) | The Fish Audio voice. Any `reference_id` from Fish's voice library works. |
| `ttsModel` | `drama-3` | Fish's speech model: `drama-3`, `s2.1-pro` or `s2-pro` |
| `summaryModel` | `haiku` | The model that writes spoken summaries of long replies |

Speaking speed is the `SPEECH_SPEED` constant at the top of `hooks/register.ts`. The default is `1.1`, and Fish accepts `0.5` to `2.0`.

---

## Cost and privacy

- **Fish Audio credits:** every spoken reply uses some. Because claudespeak is on by default, that means every reply in every session you're watching.
- **Claude usage:** each long reply makes one small Haiku call on your Claude account to write the summary. Short replies make no model call.
- **What leaves your machine:** the spoken text (a summary of the reply, or the short reply itself) is sent to Fish Audio. The full reply goes only to the summary model on your own Claude account.

`/speak off` stops all of it.

---

## Files it keeps

All of these are in `~/.claude/claudespeak/`:

| File | Purpose |
| --- | --- |
| `.env` | Your Fish Audio key |
| `setting.json` | Whether spoken replies are on or off |
| `current-speech` | Which session is speaking right now, so only one voice plays |

---

## Development

```bash
claude plugin validate .
claude plugin test .
```

Everything is in `hooks/register.ts`. The tests in `tests/speak.test.ts` replace the network, the speakers and the model with fakes, so running them costs no credits and makes no sound.

## Status

Tested on macOS with scripted Claude Code sessions. On Linux, the unit tests pass and the real playback pipeline has been run into a PulseAudio sink, but nobody has yet listened to it on a Linux desktop. Sessions the desktop app starts in the background may behave differently from terminal sessions.
