# Silence iOS Call-Recording Disclosure Tones

> **No app to install. No jailbreak. No signing. No VPN. No pairing file — just a cable.**
>
> Plug an iPhone into a Mac over USB and replace the system's call-recording start/stop
> disclosure tones with silence of identical length. Full backup kept, fully reversible.

[简体中文](README.md) · **English**

---

## What this is

Since iOS 18.1, Apple ships call recording — but it plays a **spoken announcement** when
recording starts and stops ("This call will be recorded", or 此通话将被录音 on Chinese
devices). **Both parties hear it**; that is its purpose: to notify the other person.

This project replaces those two audio files with **silence of exactly the same duration**,
so recording itself keeps working and the start/stop timing does not shift.

### How it differs from the app-based approaches

| | App-based (e.g. AirBeep) | **This project** |
|---|---|---|
| App installed on the iPhone | required | **not needed** |
| IPA signing / 7-day free-account limit | required | **not needed** |
| Loopback VPN (LocalDevVPN et al.) | required | **not needed** |
| Pairing file | required | **not needed** |
| What you actually need | a pile of prerequisites | **one USB cable** |

The reason is a **more direct entry point** — see *How it works*.

---

## 🙏 Credits: this was built with DSH + DeepSeek

**Huge thanks to [DeepSeek](https://www.deepseek.com/) and [DSH (DeepSeek Harness)](https://github.com/deepseek-ai/deepseek-harness).**

This did not start from a finished idea and then get coded up — it was **worked out in
collaboration with an AI agent**, step by step:

- The original plan was a different project (install an app on the iPhone, sign it, run a
  loopback VPN). The AI progressively ruled out **seven** more convoluted routes.
- The key insight — **"macOS already ships `AirTrafficHost.framework`"** — came out of
  reading the upstream code. It was Windows-only purely because it looks for a `.dll`.
- The `product_type` bug that sends everyone down the wrong path was found by reading the
  source line by line.
- Even "the silent file must match the device's actual duration" only surfaced after dumping
  the real originals on a device and comparing hashes — a 400 ms difference that is
  invisible if you only read the docs.

**So I genuinely encourage you to just use DSH yourself.** You can reproduce this path:

```sh
# install DSH, then in your working directory:
dsh
> clone https://github.com/jiutinglifeng/ios-calltone-silence
> read the README and walk me through removing the call-recording tone from my iPhone
```

Plug the iPhone into a Mac over USB and let it drive. The README is written to be specific
enough — including **which file to open when something fails, and which error message maps
to which action** (see `docs/失败签名对照表.md`).

> If you hit something new, have it fold the finding back into the docs and open a PR.
> **The next improvement to this may well come from you.**
>
> This project's own contribution is genuinely small: porting `airlift.py` to macOS, fixing
> three misleading bugs, and establishing that the silent file must match the device's
> duration. **The rest is the upstream researchers' work** — see the credits table below.

---

## Verified on

**Fully verified end to end (read → write → restore → reboot retest):**

| | |
|---|---|
| Device | iPhone 19,7 |
| OS | **iOS 27.0.1 (24A446)** |
| Host | Mac (Apple Silicon) |

What was verified: the replacement itself, an identical hash on read-back,
**a real recorded call confirming the announcement is gone**, and that it
**still holds after rebooting the phone**.

> One known leftover: the other party can still hear three very faint lead-in
> beeps ("doo-doo-dee"). iOS labels them 即将播放此通话正在录音的提示
> ("about to play the call-recording notice"). They are a **separate audio
> element** from the announcement and are not covered by this replacement.

### ⚠️ About iOS versions

| iOS | Status |
|---|---|
| iOS 18.0 – 27.0.1 | considered supported upstream (we only verified 27.0.1) |
| **iOS 27.2 beta 3 and later** | ❌ **patched by Apple — this no longer works** |

> **If you are on 27.0.1 or lower, turn off automatic updates**
> (Settings › General › Software Update › Automatic Updates) before you start.
> Once patched, it does not come back.

---

## How it works

This builds on the **AirTraffic / ATAirlock sandbox escape** researched by the upstream
[airlift](https://github.com/0xjohnnydev/airlift) project.

AirTraffic is Apple's official channel for syncing media (books, audiobooks) from a Mac to
iOS. It has a path-validation flaw:

1. Upload a crafted archive via `com.apple.streaming_zip_conduit` containing a **relative
   symlink** `p0/p1/p2/link → ../../../<target path>`
2. Overwrite `Media/Books/Sync/Books.plist` over AFC, with `..` sequences in the AssetID
3. Drive the AirTraffic state machine (`HostInfo → RequestingSync → ReadyForSync →
   AssetManifest → FileComplete`) to trigger the move
4. **The flaw** is in `-[ATAirlock processCompletedAsset:]`:
   - the AssetID (identifier) is **not path-validated** — `..` escapes
     `/var/mobile/Media/Airlock/Book`
   - the destination gets only a **string prefix check** `hasPrefix:@"/var/mobile/Media/"`,
     not a check of the symlink-resolved real path

So files can be written outside the sandbox — including the directory holding the two
disclosure tones.

### The key finding: macOS already has what you need

Upstream `AirliftSilence` ships **Windows-only**, on the grounds that "the exploit needs
`AirTrafficHost.dll`".

**That reasoning only applies to that particular Windows port.** macOS has
`/System/Library/PrivateFrameworks/AirTrafficHost.framework` with **identical symbols**.
Measured on this machine:

| Library | Symbols required | Available on macOS |
|---|---|---|
| `AirTrafficHost.framework` | 9 (`ATHostConnection*`, `ATCFMessage*`) | ✅ **9 / 9** |
| `CoreFoundation.framework` | 8 (`CFString*`, `CFData*`, `CFPropertyList*`, `CFRelease`) | ✅ **8 / 8** |

Because the ctypes signatures in `airlift.py` were already platform-independent,
**only the library loading had to change** — no logic was touched. That is exactly what
`macos/enablement.patch` does (~70 lines).

---

## Usage

### Setup

```sh
python3 -m venv .venv
.venv/bin/pip install pymobiledevice3
```

> **If you hit `PermissionError: ~/.pymobiledevice3`** — that is just an unwritable pairing
> cache directory, not a device problem. The commands below already set the workaround.

### Step 1 — Connect and inspect (zero risk)

Plug in the iPhone over USB and unlock it. Then:

```sh
cd macos
AIRLIFT_PAIRING_CACHE=/tmp/pd3-pairing ../.venv/bin/python ../tools/设备体检.py scan
```

This is **read-only** against the device's media directory. It does not trigger the escape
and is safe to run any number of times. It checks for leftover files from any previous
failed operation.

### Step 2 — Dump the originals (do not skip)

```sh
AIRLIFT_PAIRING_CACHE=/tmp/pd3-pairing .venv/bin/python airlift.py read \
  /var/mobile/Library/CallServices/Greetings/default/StopDisclosure.caf \
  backup-StopDisclosure.caf --logging

AIRLIFT_PAIRING_CACHE=/tmp/pd3-pairing .venv/bin/python airlift.py read \
  /var/mobile/Library/CallServices/Greetings/default/StartDisclosureWithTone.m4a \
  backup-StartDisclosureWithTone.m4a --logging
```

`read` is a **round trip**: it moves the file into a staging area, reads it, then writes it
back to its original location. On success the file is unchanged.

> **Why this must come first** — see *Pitfall #1*. Skipping it is how people end up with a
> silent file of the wrong duration, or lose their only copy of the original.

### Step 3 — Generate silence matching *your* device (critical)

```sh
cd ..
./tools/静音件生成.sh macos/backup-StopDisclosure.caf macos/silent-Stop.caf
./tools/静音件生成.sh macos/backup-StartDisclosureWithTone.m4a macos/silent-Start.m4a
```

The script reads the original's **codec / sample rate / channel count / duration**,
generates a matching silent file, and verifies all four.

> ⚠️ **Do not reuse any pre-made silent file** — including the one in `macos/`.
> **Its duration was generated for a different device.** Reason below.
>
> Note: `ffmpeg` **cannot mux Opus into CAF**, and `StopDisclosure.caf` is exactly Opus/CAF.
> The generator therefore uses macOS's built-in `afconvert` for CAF and `ffmpeg` for m4a.

### Step 4 — Replace

```sh
cd macos
AIRLIFT_PAIRING_CACHE=/tmp/pd3-pairing .venv/bin/python airlift.py replace \
  /var/mobile/Library/CallServices/Greetings/default/StopDisclosure.caf \
  silent-Stop.caf --backup device-backup-Stop.caf --logging

AIRLIFT_PAIRING_CACHE=/tmp/pd3-pairing .venv/bin/python airlift.py replace \
  /var/mobile/Library/CallServices/Greetings/default/StartDisclosureWithTone.m4a \
  silent-Start.m4a --backup device-backup-Start.m4a --logging
```

**Every time, check that `previousSize` in the output is not `null`:**

```json
{"ok": true, "size": 4327, "previousSize": 41944, "backup": "device-backup-Stop.caf"}
```

- a number → ✅ the original was safely read back and backed up; continue
- `null` → 🔴 **the original could not be read back** and is sitting in the device's
  `Media/airlift-replaced-*`. **Stop immediately** and recover it with
  `tools/设备体检.py scan`.

### Step 5 — Verify

```sh
AIRLIFT_PAIRING_CACHE=/tmp/pd3-pairing .venv/bin/python airlift.py read \
  /var/mobile/Library/CallServices/Greetings/default/StopDisclosure.caf verify.caf
shasum -a 256 verify.caf silent-Stop.caf   # hashes must match
```

Then **place a real call and record it** to confirm the announcement is gone,
and **reboot the phone and test again**.

---

## 🔴 Two pitfalls you need to know

### 1. The silent file's duration must be generated for *your* device

The announcement is a **spoken notification**, so its content varies with system language —
which means **the file durations differ between devices**.

Measured on a 24A446 device:

| File | This device | Upstream author's dump | Upstream's bundled silence |
|---|---|---|---|
| `StopDisclosure.caf` | Opus 48k **1.520000 s** | Opus 48k 1.920000 s | 1.940000 s |
| `StartDisclosureWithTone.m4a` | AAC 44.1k 1.618639 s | AAC 44.1k 1.618639 s | 1.671837 s |

**The stop tone differs by 400 ms.** Using the upstream bundled silence would make it
420 ms longer than the original — visibly stretching the playback window.

> Corroborating detail: AirBeep's author treats **changing the system language** as a way to
> force the original tones to be regenerated. That trick only makes sense if the tones are
> language-dependent.

**So: dump first, then generate against the real duration.** This is a required step, not an
optimization.

### 2. An upstream bug makes the tool report "phone not connected"

This line is in `airlift.py`:

```python
pt = ld.product_type or ""
```

But the **lockdown client in `pymobiledevice3` 11.x has no `product_type` attribute**
(it belongs to the irecv / DFU subsystem). The line raises `AttributeError`, and the
`except Exception: continue` right below **swallows it silently** — every device is skipped,
and the tool finally reports a completely unrelated **`no paired iPhone found`**.

**You will suspect your cable, the trust prompt, or the driver. The fault is in the code.**

The `macos/airlift.py` in this repo fixes it (and makes the error message carry the real
reason). Three such bugs were fixed in total — see `docs/HANDOFF.md`.

---

## Restoring the originals

Write your own dump back:

```sh
cd macos
AIRLIFT_PAIRING_CACHE=/tmp/pd3-pairing .venv/bin/python airlift.py write \
  /var/mobile/Library/CallServices/Greetings/default/StopDisclosure.caf \
  <path to your backup> --logging
```

> `write` is for when the target path is empty; if it still exists, use `replace`.

**Note:** `device-backup-24A446/` holds another device's originals. **Do not use them to
restore your own device** (durations differ). Use the backup you dumped in Step 2.

**Another fallback:** the upstream author notes that changing the system language forces the
system to regenerate the default tones. We have not independently verified this.

---

## Layout

```
macos/                     the actual solution
  airlift.py                 AirTraffic escape (3 fixes + macOS support)
  airlift_gui.py             optional GUI
  enablement.patch           every change relative to upstream
  silent-Stop.caf            silent file used on our device (SAMPLE — regenerate for yours)
  silent-Start.m4a           idem

tools/
  静音件生成.sh              generate silence matching the device (essential)
  设备体检.py                read-only scanner / rescuer for leftover originals
  演练-救援流程.py           end-to-end rescue drill, 20 assertions, no device needed

docs/
  HANDOFF.md                 full technical record: procedure, pitfalls, all measurements
  失败签名对照表.md          12 real error strings → what to do

device-backup-24A446/      our device's originals (for format reference only)
```

*(Tool filenames are in Chinese; each is documented above.)*

---

## ⚠️ Legal and safety

- This tone exists to **let the other party know the call is being recorded**. Removing it
  means the system no longer informs them.
- Consent requirements for call recording vary by jurisdiction. **Some countries and US
  states require all parties to consent**; using this there may be unlawful.
  **Check your local law and obtain any required consent.**
- This relies on undocumented Apple behaviour and **may stop working after any OS update**.
- The mechanism only **moves and writes files** — it does not flash firmware or touch system
  partitions, so there is no bricking mechanism. Even so, **back up the originals first**
  (Step 2).

---

## Credits

All of the underlying work is upstream:

| Project | Author | Contribution |
|---|---|---|
| [airlift](https://github.com/0xjohnnydev/airlift) | Johnny Franks | original AirTraffic / ATAirlock sandbox escape research (macOS PoC) |
| [airlift-windows](https://github.com/DahanGuy/airlift-windows) | Guy Dahan | Windows port driving `AirTrafficHost.dll` through ctypes |
| [AirliftSilence](https://github.com/YiHoooong/AirliftSilence) | YiHoooong | packaging for this exact goal, the Grappa session fix, backup + verification flow |
| [AirCard-iOS](https://github.com/Mak5er/AirCard-iOS) | Mak5er | Grappa client tokens and the fix approach |
| [bl_sbx](https://github.com/h2zi/bl_sbx) | h2zi | an earlier attempt at replacing these two files |

**This project's own contribution is three things:** porting `airlift.py` to macOS
(library loading only), fixing three bugs that cause misleading failures, and establishing
that **the silent file must be generated against the device's actual duration**.

## License

Upstream projects are **MIT** (Copyright © 2026 Johnny Franks); see `LICENSE`.
This project modifies them and is released under MIT as well.
