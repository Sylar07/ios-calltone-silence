# 归档说明

本目录存放**开发过程中产生但已不属于当前方案**的内容。
保留它们是为了可追溯——记录我们试过什么、为什么放弃，避免后来者重复踩坑。

**这些内容不进入当前仓库的 git 历史**（`.gitignore` 已排除），只在本机保留。

---

## 为什么会有这么多归档

本项目最初的目标方案是 [AirBeep](https://github.com/LinkPass888/AirBeep)——
一个需要装在 iPhone 上的 App。我们花了大量时间研究它，直到发现了一条**短得多的路径**：
macOS 本身就带有 AirTraffic 逃逸所需的框架，**一根数据线即可，不需要任何 App**。

所以下面这些"旧方案"不是失败，而是**被更短路径取代**。

---

## archive/airbeep-upstream/

**AirBeep 的完整 iOS 工程源码**（`LinkPass888/AirBeep` 的克隆）。

| 内容 | 说明 |
|---|---|
| `airbeep/` | Swift 源码（约 2400 行） |
| `Vendor/` | 预编译 Rust 静态库（**127 MB**，仓库体积的主要来源） |
| `airbeep.xcodeproj/`、`scripts/`、`Appicon.icon/` | 工程与构建配置 |
| `CHANGELOG.md`、`LICENSE`（GPL-3.0） | 上游文件 |

**为什么放弃**：它需要 IPA 重签名、免费 Apple ID 的 7 天限制、LocalDevVPN 本地回环隧道、
美区 Apple ID，而且 iPhone 上要装 App。我们最终一个都不需要。

**顺带发现的一个 bug**（已记录在 `docs/HANDOFF.md`）：仓库里的 `Vendor/AirliftFFI.xcframework`
是**过期版本**（CI 重建后没有提交回仓库），新克隆直接用 Xcode 构建会因
`-Wl,-u,_al_exploit_read_file` 等强制符号标志而**链接失败**。

## archive/airliftsilence-windows/

**Windows 路线**的全部材料。

| 内容 | 说明 |
|---|---|
| `AirliftSilence-v1.0.0.exe` | 官方 Windows GUI（37 MB） |
| `src/` | 上游原版源码（含那个 `product_type` bug） |
| `source-patched/` | 我们修过 bug 的版本（Windows 用） |
| `silent-ios-format/` | 作者 dump 的音频与配套静音件 |
| `预检.bat` | Windows 前置检查脚本 |
| `silent/` | Windows 版自带静音件 |

**为什么放弃**：Windows 路线**本身是可行的**，但我们做出了 macOS 移植，
于是不再需要一台 Windows 电脑。留着是给"手边只有 Windows"的情况备用。

> ⚠️ 注意 `silent-ios-format/` 里的静音件时长是**另一台设备**的（1.940000 s），
> **不要用**。原因见根目录 README 的「第一个坑」。

## archive/device-backup-duplicates/

同一台设备原件的**重复副本**（`backup-*` 与 `device-backup-*`，哈希两两一致）。
正式副本在仓库根的 `device-backup-24A446/`。

## docs/archive/

已放弃路线的文档：

| 文件 | 说明 |
|---|---|
| `AirBeep-技术方案与实施文档.md` | AirBeep 方案的完整分析（含 7 条替代路线的排除理由） |
| `AirliftSilence-Windows操作手册.md` | Windows 路线操作手册 |
| `2026-10-11-核查补充.md` | 对上述文档的外部核查与修正 |
| `README.md`、`README_zh-CN.md` | AirBeep 原版 README |
| `airbeep-reference/` | AirBeep 的补丁、头文件、上游 README 等核查素材 |
| `macos-port-README-obsolete.md` | macOS 移植版早期说明（路径已过时） |

---

## 完整提交历史

精简前的 git 仓库共有 **110 个提交**（含上游 AirBeep 的 97 个）。
本次研究新增的 12 个提交如下：

| 提交 | 说明 |
|---|---|
| `333a9bc` | 在真机 iPhone 19,7 / iOS 27.0.1 上完成替换并验证 |
| `424534b` | 用 mock 设备端到端演练救援流程（20 项断言） |
| `4c005fa` | macOS 移植——让 iPhone 不再需要 Windows |
| `862270e` | 记录该 bug 上游尚未有人报告 |
| `bdff2bb` | 修复导致误报的设备识别 bug |
| `47d603b` | 静音件生成器（按设备实际格式） |
| `fb5babd` | 失败签名对照表 |
| `138ac14` | 只读设备体检工具 |
| `6dd5362` | Windows 预检脚本 + 记录 27.2 截止日期 |
| `b33dd03` | 记录 remote 布局 |
| `f314992` | 交接材料 |
| `1a8ae4c` | AirBeep 核查记录 |

**旧的 git 历史（70 MB，含那两个 64 MB 静态库）已移出本工作区保留**，
未纳入当前精简后的仓库。
