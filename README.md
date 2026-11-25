# claudespeak

A mod for Claude Code. When Claude finishes a reply, it reads a 1-3 sentence summary out loud with [Fish Audio](https://fish.audio). Short plain replies are read as they are. It streams, so speech starts as the audio arrives.

Only one voice plays at a time, even with several sessions open: a new reply cuts off the one before it. It stays quiet in sub-agents, `claude -p`, scheduled desktop routines, and a `claude` started from inside another session.

## Requirements

- **macOS or Linux.** On macOS it plays through ffmpeg's `audiotoolbox` output. On Linux it plays through PulseAudio (which PipeWire also serves, as on current Ubuntu and Fedora desktops) when a sound server is running, and through ALSA's default device otherwise. Windows is not supported.
- **ffmpeg**: `brew install ffmpeg` on macOS, `sudo apt install ffmpeg` (or your distro's package) on Linux. On Linux it needs the `pulse` or `alsa` output, which distro builds include; `ffmpeg -hide_banner -devices` lists them.
- **A Fish Audio account and API key.** Get a key at <https://fish.audio/app/api-keys/>.
- **Claude Code** with a build that loads hooks modules. Tested on 2.1.309 (macOS) and 2.1.331 (Linux).

On Linux, `/speak key` opens the key file with `xdg-open`; if that is missing, open the path it prints in any editor.

## Install

```
claude plugin marketplace add albertshiney/claudespeak
claude plugin install claudespeak@claudespeak
```

Then restart Claude Code, or run `/reload-plugins` in an open session.

### Add your Fish Audio key

Open the .env at `~/.claude/claudespeak/.env`, paste your Fish Audio API key after `FISH_API_KEY=`, save.

The file does not exist until claudespeak makes it, so run `/speak key` to create it and open it. If you skip this, your first finished reply does the same thing and tells you the path. The file is created readable only by you, and it lives outside the plugin folder so updates don't wipe it. A `FISH_API_KEY` environment variable takes priority over the file.

## Commands

It is **on by default**. `/speak on` and `/speak off` apply to every session on the machine.

| Command | What it does |
| --- | --- |
| `/speak` | Says whether it is on, and whether this session can speak |
| `/speak on` | Reads replies aloud in every session |
| `/speak off` | Stops reading aloud in every session |
| `/speak stop` | Cuts off the current speech in any session, and leaves the setting alone |
| `/speak key` | Creates the .env if needed and opens it |

## Voice and speed

Both are constants at the top of `hooks/register.ts`:

- `DEFAULTS.voiceId`: the Fish voice, Sawyer by default. It is also an option in the plugin's settings.
- `SPEECH_SPEED`: 1.1 by default, Fish's `prosody.speed` (0.5 to 2.0).

## Cost

Every spoken reply uses Fish Audio credits, and because it is on by default that means every reply in every session a person is attached to. Long replies also use one Haiku call on your Claude account to write the summary. Use `/speak off` to stop both.

## Status

Tested on macOS with scripted Claude Code sessions. The Linux path is covered by the unit tests and by playing the real pipeline into a PulseAudio sink; it has not yet been tried by ear on a Linux desktop. Not yet confirmed by hand in the desktop app's Code tab, and sessions the app starts in the background may behave differently there.
