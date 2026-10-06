<div align="center">

<img src="Icon/mangomidi.png" width="140" alt="Mango MIDI icon">

# 🥭 Mango MIDI

**Piano audio in, MIDI out — from a file, or from whatever your Mac is playing. Chords and all.**

Made by Mingyu 🧑‍💻

<br>

![macOS](https://img.shields.io/badge/macOS-15%2B-202020?style=for-the-badge&logo=apple&logoColor=white)
![Swift](https://img.shields.io/badge/Swift-SwiftUI%20%2B%20CoreML-FA7343?style=for-the-badge&logo=swift&logoColor=white)
![Version](https://img.shields.io/badge/version-2.1.0-7C5CFF?style=for-the-badge)
![Accuracy](https://img.shields.io/badge/notes%20found-98.5%25-2EA043?style=for-the-badge)
![Price](https://img.shields.io/badge/price-free-2EA043?style=for-the-badge)

</div>

---

> [!NOTE]
> **Everything happens on your Mac.** 🔒 The model runs locally through CoreML, the audio never leaves
> the machine, and there is no account and no server.

---

## 📖 Contents

| | | |
| --- | --- | --- |
| [🎹 What it does](#-what-it-does) | [📥 Install](#-install) | [👀 Using it](#-using-it) |
| [📊 How accurate](#-how-accurate) | [🎼 Chords](#-chords) | [⌨️ The command line](#️-the-command-line) |
| [🤝 DynamicMango](#-handing-scores-to-dynamicmango) | [🕶️ Privacy](#️-privacy) | [🆕 What's new](#-whats-new-in-210) |
| [⚖️ Licence](#️-licence) | | |

---

## 🎹 What it does

- **Convert a file** 🎵 — drop a piano recording on the window (or the Dock icon), or choose one.
- **Listen to what's playing** 👂 — press Start Listening, a **3 · 2 · 1** countdown ticks (Cancel
  any time), then whatever your Mac is playing is transcribed as it plays.
- **Save** 💾 — when it's done, it asks where to put the `.mid`: notes, velocities and sustain pedal,
  readable by any DAW or notation app.

---

## 📥 Install

Download **`mango-midi-2.1.0.dmg`** from the
[latest release](https://github.com/mannnnnnnngo/MangoMIDI/releases/latest), drag the mango onto
Applications, then **right-click Mango MIDI → Open** the first time (the app isn't signed with a paid
Apple developer account; right-click → Open is Apple's own way past the warning). macOS 15+.

---

## 👀 Using it

**Convert a File** is the big card. Drop audio on it, or press **Choose File…**. A progress bar and the
roll fill in as it goes; **Cancel** stops it. When it finishes, a save panel asks where the MIDI goes.

**Listen to What's Playing** is the card beside it:

1. Start a song in Spotify, Music, a browser — anything.
2. Press **Start Listening**. A 3-2-1 countdown ticks; **Cancel** stops it.
3. With Spotify or Music, it can rewind the song to 0:00 first (on by default), hears **only that
   app**, and stops by itself when the song ends. From anything else it hears everything your Mac
   plays except itself — press **Stop & Convert** when you're done.
4. Most of the song is transcribed while it plays, so finishing takes seconds. Then the save panel.

The result screen plays the original under the falling notes so you can check it by ear, and
**Save Again…** writes another copy anywhere.

---

## 📊 How accurate

Measured, not estimated: 12 MAESTRO v3 **test** pieces — real concert-grand recordings with the exact
notes played, ~30,000 notes — scored with mir_eval's standard note metric (right key, onset within
50 ms). None of the models below trained on these pieces.

| | precision | recall | **F1** |
|---|---|---|---|
| Mango MIDI 1.x (basic-pitch + search) | 72.3% | 66.2% | 67.9% |
| Mango MIDI 2.0 (Transkun v2) | 99.6% | 97.5% | 98.5% |
| **Mango MIDI 2.1 (retrained for dense music)** | **99.8%** | **97.2%** | **98.4%** |

### 🔥 Hard songs (Rush E and friends)

Ten minutes of Rush E-style music — hammered repeated notes, 10-20-note chords in both hands,
runs at 30+ notes a second, clusters; **49 notes a second on average** — generated from MIDI so every
note is known, and played on three sampled pianos. The third piano was never used in training.

| | 2.0 | **2.1** |
|---|---|---|
| piano 1 | 77.5% | **94.1%** |
| piano 2 | 78.1% | **93.7%** |
| piano 3 (never heard in training) | 77.0% | **93.5%** |

2.1 retrained the model on 30 hours of music like this, mixed with real concert recordings so it
did not forget how a normal piano piece goes.

And on harder audio — the same concert pieces, degraded:

| condition | F1 |
|---|---|
| 128 kbps MP3 | 98.5% |
| big reverberant room | 98.2% |
| a different, sampled piano | 99.1% |

> [!NOTE]
> **Why not 100%?** Nothing transcribes real recordings perfectly. What it still misses are mostly
> very soft notes struck at the same instant as louder ones — the quietest tone of a chord — where the
> loud notes physically mask it. Those numbers are what it measured, not rounded up.

---

## 🎼 Chords

Both hands, several notes each — including tight clusters like **C-D-F♯** under **E-G♯-A♯** — come
through as separate notes, because every one of the 88 keys is decided on its own: the model scores
every possible note on each key and picks the best set for that key alone.

| on the test pieces | 1.x | 2.0 |
|---|---|---|
| notes that are part of a chord | 51.9% | **95.7%** |
| chords of 6+ notes | 45.4% | **93.4%** |
| clusters (neighbours ≤ 2 semitones apart) | 29.1% | **83.7%** |

---

## ⌨️ The command line

```bash
/Applications/Mango\ MIDI.app/Contents/MacOS/MangoMIDI --help
```

| Flag | What it does |
|---|---|
| `--transcribe=FILE` | Transcribe an audio file and write a `.mid` beside it |
| `--out=FILE.mid` | …or here |
| `--mono` | Mix to mono first, as a live recording is |
| `--now-playing` | Show what Mango MIDI can see playing, and what it can listen to |
| `--self-test` | MIDI writer, decoder, handoff and model checks |

---

## 🤝 Handing scores to DynamicMango

Songs transcribed by listening are also written, with an index, into the folder
[DynamicMango](https://github.com/mannnnnnnngo/DynamicMango) reads
(`~/Library/Application Support/DynamicMango/Scores`), and DynamicMango shows them instead of its own
live transcription.

---

## 🕶️ Privacy

**Nothing leaves your Mac.** No account, no analytics, no server.

- The transcriber runs here, in CoreML inside the app.
- Listening only happens after you press Start Listening and the countdown ends, and stops when you
  press Stop or Cancel. The recording is kept in a temporary file so Play works, and nowhere else.
- Your MIDI files go wherever you save them; settings are a JSON file in `~/.config/mangomidi`.

The only request Mango MIDI makes by itself is reading one small text file on GitHub to find out
whether a newer version exists. Settings → Updates switches even that off.

---

## 🆕 What's new in 2.1.0

| | |
|---|---|
| 🔥 **Hard songs** | Retrained for dense, fast, impossible-to-play music: Rush E-style pieces went from 77% to **94%** of notes. |

### 2.0.0

| | |
|---|---|
| 🎯 **A new transcriber** | Transkun v2 replaces basic-pitch: 67.9% → **98.5%** of notes on real recordings. |
| 🎼 **Real chords** | Every key decided independently — clusters went from 29% to 84%. |
| 🎵 **Convert a File first** | The main screen is a drop zone. Nothing listens by itself any more. |
| ⏱️ **3-2-1 to listen** | Start Listening counts down with a tick, and Cancel works at every step. |
| 💾 **Asks where to save** | A save panel at the end of every conversion. |
| 🎹 **Velocity and pedal** | The `.mid` carries how hard each note was played, and the sustain pedal. |

---


## ⚖️ Licence

Mango MIDI is **free to use** but **not open source** — the source is not published. The full terms
are in [`LICENSE`](LICENSE).

The transcription model is [Transkun](https://github.com/Yujia-Yan/Transkun) v2 by Yujia Yan, used
under the MIT licence, retrained for Mango MIDI, and it ships inside the app.

Copyright © 2026 Mingyu. All rights reserved.

---

<div align="center">

**Made with 🥭 by Mingyu**

🔒 Everything runs on your Mac · 🎹 Feeds [DynamicMango](https://github.com/mannnnnnnngo/DynamicMango)

Part of [🥭 MangoApps](https://github.com/mannnnnnnngo/MangoApps)

</div>
