<div align="center">

<img src="Icon/mangomidi.png" width="140" alt="Mango MIDI icon">

# 🥭 Mango MIDI

**Turns a song into a piano MIDI score by transcribing it over and over *while it plays*, keeping whichever attempt best explains the recording.**

*An app for piano players who want the sheet music but are broke — lwk me 🥲*

Made by Mingyu 🧑‍💻

<br>

![macOS](https://img.shields.io/badge/macOS-15%2B-202020?style=for-the-badge&logo=apple&logoColor=white)
![Version](https://img.shields.io/badge/version-1.0.0-7C5CFF?style=for-the-badge)
![Status](https://img.shields.io/badge/status-in%20development-F59E0B?style=for-the-badge)
![Price](https://img.shields.io/badge/price-free-2EA043?style=for-the-badge)
![Privacy](https://img.shields.io/badge/data%20sent%20anywhere-none-0EA5E9?style=for-the-badge)

</div>

---

> [!WARNING]
> **Mango MIDI is in development.** 🚧 The search itself is built and measured — the numbers below are
> from real runs, not estimates — but this is not a finished app you install and forget. Expect rough
> edges in the window, and expect the command line to be the honest interface for a while yet.

> [!NOTE]
> **Everything happens on your Mac.** 🔒 The model runs locally through CoreML, the audio never leaves
> the machine, and there is no account and no server.

---

## 📖 Contents

| | | |
| --- | --- | --- |
| [🎹 What it does](#-what-it-does) | [📥 Install](#-install) | [👀 Using it](#-using-it) |
| [⌨️ The command line](#️-the-command-line) | [🔁 Why the loop exists](#-why-the-loop-exists) | [🥁 Background noise is not piano](#-background-noise-is-not-piano) |
| [📊 What it scores](#-what-it-scores) | [♻️ Never doing a song twice](#️-never-doing-a-song-twice) | [🤝 Handing scores to DynamicMango](#-handing-scores-to-dynamicmango) |
| [🗂️ Where things live](#️-where-things-live) | [🔔 Updates](#-updates) | [⚖️ Licence](#️-licence) |

---

## 🎹 What it does

A neural transcriber run once is a guess with settings baked into it. Different thresholds recover
different notes: one setting finds the quiet inner voice and invents a harmonic, another is clean but
sparse. **There is no way to know which was right from the transcription alone.**

There is a way to know from the **audio**. Render a candidate score back to sound and ask how well it
explains what was actually played. That turns transcription from a guess into a search with a
measurable objective — and the loop simply keeps the best answer it has found, for as long as the song
is playing.

---

## 📥 Install

Grab the `.dmg` from **[the Releases page](https://github.com/mannnnnnnngo/MangoMIDI/releases)**, open
it, and drag **Mango MIDI** onto Applications. Needs macOS 15 or newer.

> [!NOTE]
> Mango MIDI is still in development, so there may not be a build on that page yet. 🚧

> [!IMPORTANT]
> The first time you open it, macOS blocks it — the app isn't signed with a paid Apple developer
> account. Double-click Mango MIDI, press **Done** on the warning, then go to
> **&#63743; → System Settings → Privacy & Security**, scroll to the bottom, and press **Open Anyway**.
> Press **Open Anyway** once more to confirm. You only do this once. 🔓

Mango MIDI needs permission to record audio, since listening to what is playing is the entire point.
macOS asks the first time you press play. 🎧

---

## 👀 Using it

Play something. Mango MIDI finds the player, taps **that process** rather than the whole system where
it can, and starts searching. The window shows a piano roll filling in as the search improves, the
agreement figure it has reached, and the transport for what it has built so far. Export writes a
`.mid`.

When the player cannot be identified it falls back to tapping everything making sound — and it
**says so in the window**, because muting the other app is something only you can do.

---

## ⌨️ The command line

The command line is not a convenience. It is how the transcription is measured, and every number in
this README came out of it.

```bash
/Applications/Mango\ MIDI.app/Contents/MacOS/MangoMIDI --help
```

| Flag | What it does |
|---|---|
| `--transcribe=FILE` | Transcribe an audio file and write a `.mid` beside the library entry |
| `--live=FILE` | Run the continuous search over a file *as if it were playing* — no player, no tap, no one watching |
| `--truth=FILE.json` | Score the result against known notes (`[{midi,start,dur}, ...]`) |
| `--seconds=N` · `--target=N` | Time budget, and the agreement to stop at |
| `--speed=N` | Playback speed for `--live`; 1 is real time, 0 is as fast as it decodes |
| `--support=N` | How clearly a note's pitch must stand out of the recording |
| `--support-sweep=FILE` | Decode once, then show what each note-support threshold keeps — and costs |
| `--now-playing` | Show what Mango MIDI can see playing, and what it can tap |
| `--self-test` | Check the MIDI writer, the objective and the excerpt search with no audio device at all |
| `--library` | List everything transcribed, and what is left to do on it |
| `--deliver` | Re-deliver the whole library to DynamicMango |
| `--quick` | A fast, low-effort pass — for checking the pipeline, not for results |

---

## 🔁 Why the loop exists

One search runs over a finished recording against a time budget. The other runs *during* the song and
never stops of its own accord — the song ending is what ends it, not a budget. Every round makes
another batch of candidates, scores them, and keeps whichever explains the recording best.

Two things make an attempt cheap enough to repeat indefinitely:

- **Inference runs once per moment of audio, ever.** The audio heard so far is kept along with the
  model's opinions about it; a candidate is a re-decode of those activations, which is microseconds.
- **An attempt is scored over one excerpt**, twenty seconds or so, rather than over the whole piece —
  and the excerpt chosen is the stretch the current transcription explains *worst*. Attempts cost a
  twentieth as much, and are spent where the transcription is actually wrong.

> [!IMPORTANT]
> An excerpt winner is never adopted directly. Settings that win over one excerpt can lose over the
> whole piece, and adopting excerpt winners is how a search convinces itself it is improving while
> getting worse. So the winner is a **challenger**: periodically both it and the incumbent are measured
> over everything heard, and the challenger is adopted only if it wins there. The excerpt search
> proposes; the full-length measurement disposes. ⚖️

Measured on a 70-second fixture with known notes, played at real time: **288 attempts, agreement
0.969, and 100% precision / 98% recall against ground truth.** Both numbers are printed, because the
agreement is the search's opinion of itself and only the second one says whether that opinion means
anything.

---

## 🥁 Background noise is not piano

The transcription model is a *pitch* model, not a piano model: anything with energy at a pitch becomes
a note. Worse, a struck transient has energy at **every** pitch at once, which is precisely what an
onset detector fires on — so one snare hit can produce a dozen notes spread across the keyboard.

The spectral objective cannot fix this, and it looks as though it should. It asks how much of the
recording a candidate explains, and a note placed on a drum hit *does* explain energy that is genuinely
there.

So a second test asks a different question: does the recording contain this note's **harmonic series**
where the note is claimed? For the first few partials it measures what fraction of the surrounding
fifth that partial is louder than — a rank, so it is scale-free, and a broadband strike lands at ~0.5
by construction because it raises every bin together. Notes below the threshold are dropped *before*
scoring, so the search optimises the transcription that will actually be saved.

Calibrated with `--support-sweep`, which decodes once and applies each threshold to the same notes so
the search cannot confound the sweep. Both fixtures peak at 0.92:

| fixture (70s, 453 played notes) | | precision | recall | F1 |
|---|---|---|---|---|
| piano alone | off | 84% | 100% | 91% |
| piano alone | **0.92** | 95% | 100% | **97%** |
| piano + kit, pad and hiss | off | 51% | 97% | 67% |
| piano + kit, pad and hiss | **0.92** | 75% | 94% | **83%** |

End to end, with the search on top, the noisy fixture goes from **69% precision** to **83%** offline
and **91%** live. Clean audio is unchanged at 100% — the filter costs no recall there, which is what
makes it safe to leave on.

**What it deliberately cannot do** is remove another *instrument's* notes. A sustained string pad has a
real, prominent fundamental, and nothing in the spectrum distinguishes it from a piano playing the same
note. That is source separation: a different problem, needing a different model. Most of the remaining
9% on the noisy fixture is the pad.

---

## 📊 What it scores

> [!NOTE]
> **On "99.999% identical".** The target is 0.99999 and it means **"do not stop early"**. Nothing can
> establish that a transcription is exactly the notes a pianist played — the audio does not contain
> that fact unambiguously, and two different scores can produce near-identical sound.

What the loop reports is its **agreement score**: how much of the original piano content the candidate
explains, measured on the audio itself. Against a real recording that number does not reach five nines,
because a synthetic render never matches a real piano's timbre, room and mastering exactly. So in
practice the search runs for as long as the song does, and then reports the figure it actually
measured — to five decimal places, so an afternoon of searching does not read the same as a minute of
it.

A run that plateaus at 0.94 says so, rather than rounding itself up.

---

## ♻️ Never doing a song twice

Every track gets a folder under `~/Documents/Mango MIDI/`, keyed by a fingerprint of the *audio* rather
than by title, so the same piece found under a different name is still recognised. The record holds the
best score found, the agreement it reached, the target it was aiming at, the settings it got to, and
the **cumulative** attempts and search time across every session.

Coming back to a song has three outcomes, and one of them is doing nothing:

| the record says | what happens |
|---|---|
| at the target | show it; no work at all |
| heard end to end, short of the target | carry on from the stored capture — no tap, no player needed |
| only part of it was heard | listen again, with the search *seeded* from the settings it had reached |

"Complete" means **the whole piece was heard**, not that the search is over. Those are separate facts,
and conflating them loses the ability to resume. Changing songs mid-search writes the partial result
out first, so work is never thrown away.

---

## 🤝 Handing scores to DynamicMango

Each finished `.mid` and an index are written into a folder
[DynamicMango](https://github.com/mannnnnnnngo/DynamicMango) reads
(`~/Library/Application Support/DynamicMango/Scores` by default, plus any extra folders in the config).
DynamicMango reads it on a track change, and **a delivered score replaces DynamicMango's own
transcription rather than joining it** — the model is not run at all for that track.

That is deliberate, and it reverses the first design. A delivered score is the end of a search that
rendered candidates back to audio and kept the best over the whole piece; a live pass is one two-second
window judged once and never revised. Merging them drew the live pass's wrong notes on top of a
transcription that had already rejected them — permanently, because the cache never forgets. Better
evidence has to win outright, or there was no point fetching it.

---

## 🗂️ Where things live

| Where | What |
|---|---|
| `~/Documents/Mango MIDI/<track>/` | 🎼 One folder per track: the `.mid`, the record, the stored capture |
| `~/Library/Application Support/DynamicMango/Scores` | 🤝 The handoff folder DynamicMango reads |

Nothing lives in this repository. Transcriptions are recordings of what you were listening to, so they
stay on your Mac. 🔒

---

## 🔔 Updates

Mango MIDI checks [`updates/latest.json`](updates/latest.json) on this repository and tells you when a
newer version is out. That is the only network request it makes, it carries nothing about you, and the
download is whatever is attached to the matching release. 📡

---

## ⚖️ Licence

Mango MIDI is **free to use** but **not open source**. It may not be redistributed, modified, resold,
reverse engineered, or presented as anyone else's work. The full terms are in [`LICENSE`](LICENSE).

Mango MIDI uses [basic-pitch](https://github.com/spotify/basic-pitch) for note detection. That is not
mine and is not covered by the above — it carries its own licence.

Copyright © 2026 Mingyu. All rights reserved.

---

<div align="center">

**Made with 🥭 by Mingyu**

🆓 Free forever · 🔒 Nothing leaves your Mac · 🎹 For broke pianists

</div>
