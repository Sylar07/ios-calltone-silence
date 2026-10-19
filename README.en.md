# Silence iOS Call-Recording Disclosure Tones

> **No app to install. No jailbreak. No signing. No VPN. No pairing file — just a cable.**
>
> Plug an iPhone into a Mac over USB and replace the system's call-recording start/stop
> disclosure tones with silence of identical length. Full backup kept, fully reversible.

[简体中文](README.md) · **English**

---

## ⚠️ Read this before you plug anything in

I understand the concern: **running a stranger's code against your phone is a real risk.**
So this section is not reassurance — it is facts, plus how to verify them yourself.

### What it does

- Replaces **two audio files** on your iPhone (the call-recording start/stop announcements) with
  **silence of exactly the same duration**:
  - `/var/mobile/Library/CallServices/Greetings/default/StopDisclosure.caf`
  - `/var/mobile/Library/CallServices/Greetings/default/StartDisclosureWithTone.m4a`
- Before replacing, it reads the originals back **onto your computer** as a backup

### What it does not do

- ❌ **No network access.** I grepped the whole of `airlift.py`: **no HTTP / urllib / requests /
  socket calls**. Everything goes over USB, between your phone and your computer only
- ❌ **No jailbreak**, no privilege escalation, no kernel changes
- ❌ **No scanning.** The paths it touches are **hard-coded in the command**, the two above
- ❌ **It only ever touches the media / user-data layer** — see *Risk and limits*

### Why it cannot brick your phone

This is a mechanical argument, not a promise:

> Bricking requires modifying the **boot chain (SecureROM / iBoot), the system volume
> (APFS snapshot) or the SEP**. On a **non-jailbroken** device those are protected by
> **Secure Boot and code signing** and **cannot be written to at all**.
>
> This project goes through **AirTraffic** — Apple's own media-sync protocol. That channel's
> **ceiling is the media partition**. In other words: it is not that we carefully avoid the
> system volume — **the channel has no permission to touch it.**

### What I will not promise you

- Not that **Apple will never restore these files in a future update** (they have in fact been
  silently updated before)
- Not that it works on **any version other than iOS 27.0.1 (24A446)**
- Not that `pymobiledevice3` stays compatible across versions — **that library moves fast and
  we have already been bitten by it** (hence the pinned version in `requirements.txt`)

### Three things you can verify yourself (in order)

**① Read the code.** `macos/airlift.py` is **764 lines**. 90% of it is not ours — it comes from
the upstream [airlift](https://github.com/0xjohnnydev/airlift) /
[AirliftSilence](https://github.com/YiHoooong/AirliftSilence) projects.
**We only changed how the system libraries are loaded — about 70 lines**, all visible in
`macos/enablement.patch`.

**② Start with the read-only step.** `tools/设备体检.py scan` **only reads** the device's media
directory. It triggers no write and is safe to run any number of times.
**Run it first and see whether it connects cleanly.**

**③ Then the `read` step.** It is a round trip — **on success the file returns to where it was.**
If it works, the whole chain works. If it fails you find out immediately, **and you have not
written anything yet.**

> **If you are not going to read the code and not going to run the read-only step — please
> don't use this.** That is not a formality; I mean it.

### 🚫 When not to use this

- **iOS 27.2 beta 3 or later** — patched by Apple. **It will not work; don't bother**
- **It's your only phone and any hiccup would be a disaster** — then it isn't worth it.
  Honestly, this is not a tool for everyone
- **You are not comfortable with a terminal** — please go through it with someone who is

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
| What you actually need | a pile of prerequisites | **one Mac and one USB cable** |

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

### Other platforms

The core tool is already cross-platform — `macos/airlift.py` keeps **both** branches,
`_load_macos()` and `_load_windows()`, selected by `sys.platform`. `设备体检.py`,
`演练-救援流程.py` and `requirements.txt` have no macOS-specific dependencies either.

But **we only verified it on macOS**. Here is the honest state:

| Platform | Status |
|---|---|
| **macOS** | ✅ fully verified (Apple Silicon) |
| **Windows** | 🟡 code complete, **not verified on a real device**. The only missing piece is an equivalent of `静音件生成.sh` (it is bash + macOS `afconvert`) |
| **Linux** | ❌ very likely infeasible — this exploit depends on Apple's own `AirTrafficHost` framework, which does not exist there |

> **Want to help?** Silent-tone generation on Windows is the one concrete gap.
> The open question: **does a PCM CAF with the correct duration also work?**
> If it does, Windows users can generate it with `ffmpeg` and the gap closes immediately.
> See **[issue #1](https://github.com/jiutinglifeng/ios-calltone-silence/issues/1)** — PRs welcome.

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

## 🔴 Risk and limits

### What it actually touches on the device

| Operation | Location | Note |
|---|---|---|
| Temporarily rewrites the sync index `Books/Sync/Books.plist`, **then restores it** | media partition | required to trigger the escape; saved and put back |
| Drops temporary archives and symlinks under `Media/`, **then deletes them** | media partition | intermediate artifacts of the escape |
| `read` / `replace` move the **target audio file** into `Media/` and write it back | media partition | only the two paths you passed on the command line |

**All of it is in the media / user-data layer.**

### The worst case

Honestly: **the worst we actually observed is a failed write leaving the target audio file
missing or damaged.** The symptom is the announcement being gone, or playing oddly.
**It does not affect booting, calling, or anything else.**

And that state **has a defined recovery path** (see *Restoring the originals*), because:

1. On failure the original **stays in the device's `Media/` directory** — it is not gone;
   `tools/设备体检.py` exists to retrieve it
2. The dump you made in Step 2 is a **copy independent of the device**
3. Final fallback: **reinstalling / restoring the OS** will make Apple re-deliver these files

### What it cannot do

- ❌ **Cannot reach the boot chain / system volume / SEP** — protected by Secure Boot and code
  signing on a non-jailbroken device, and the AirTraffic channel **has no such permission**
- ❌ Will not touch your photos, messages, accounts, or other apps
- ❌ Will not void your warranty (no jailbreak, no system-volume changes)

### What it does not guarantee

- **Not that it stays working**: Apple may restore these files in any update. They **have been
  silently updated before**
- **Not cross-version compatibility**: verified only on **iOS 27.0.1 (24A446)**
- **Not third-party dependency stability**: `pymobiledevice3` moves fast, **we have already been
  bitten once**, hence the pinned version
- **Not that we exhausted every failure mode** — anyone claiming "nothing else can possibly go
  wrong" is bluffing. What we can say: **every failure we observed is recoverable, and recovery
  does not depend on the device itself**

### The one part of the supply chain we cannot vouch for

```
pymobiledevice3
```

It is a third-party library, not ours. We **pin the version we verified** in `requirements.txt`,
but be aware this is **the one part of this project we cannot fully audit for you**.
Take a look at its repository before installing (it is a well-known open-source iOS tooling
library).

> **What we did audit:** `macos/airlift.py`, 764 lines, read line by line; no network calls;
> about 70 lines of change versus upstream, comparable line by line in `macos/enablement.patch`.

---

## Usage

### Setup

```sh
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
```

> `requirements.txt` **pins the `pymobiledevice3` version we verified.**
> Why pin: that library iterates extremely fast (700+ releases on PyPI) and reshuffles its
> internal API — **we were bitten by exactly that during development**, and the tool reported a
> completely misleading error. Use the pinned version; if you want to upgrade, run the read-only
> `设备体检.py scan` first to check compatibility.

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

## 🔴 Two common pitfalls

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

## FAQ

**Q: I'm not technical — can I still do this?**
Not on your own, no. Have someone technical read the code and the steps with you, or just use
DSH and let it walk you through (see the credits section above).

**Q: Will this break my phone?**
See *Risk and limits*. The worst case we observed is a missing/damaged announcement file, which
**does not affect booting or calls** and has a defined recovery path. But **nobody, including us,
can give you a 100% guarantee** — so make the Step 2 backup first.

**Q: Does it read my private data?**
No. There are **no network calls anywhere in the source**; everything goes over USB between your
phone and your computer. It only touches the two audio paths hard-coded in the command, plus the
temporary files the AirTraffic escape needs.

**Q: Why not just use one of the existing apps?**
Because those require installing an app on the iPhone, IPA signing (free accounts expire after 7
days), a LocalDevVPN loopback tunnel and a US Apple ID. Once we found a more direct entry point,
**none of that is needed**.

**Q: Will it survive an OS update?**
**iOS 27.2 beta 3 and later are patched by Apple** — it will not work after upgrading. If you are on
27.0.1 or lower, **turn off automatic updates now**.

**Q: Will the announcement come back after an update or restore?**
Possibly. These audio files **have been silently updated by Apple before**. If the announcement
returns, just run the same steps again.

**Q: I have a different iPhone model — will it work?**
In principle yes, but **we only verified on one iPhone 19,7 / iOS 27.0.1 (24A446)**. The important
part: **the silent file must be generated against your device's actual duration** (see Step 3).
Do not reuse the one in this repo — it was generated for a different device.

---

## Layout

```
requirements.txt           pinned dependency version (the one we verified)
README.md                  中文版（primary）
README.en.md               this file

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

## ⚖️ Legal

- This tone exists to **let the other party know the call is being recorded**. Removing it
  means the system no longer informs them.
- Consent requirements for call recording vary by jurisdiction. **Some countries and US
  states require all parties to consent**; using this there may be unlawful.
- **Check your local law and obtain consent where required.** That responsibility is the
  user's, not the author's.

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
