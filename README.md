# 2026 强网杯线上初赛 WriteUp 集

官网：https://aihaiqi.cn/helium-ctf/

这是 2026 年第十届"强网杯"全国网络安全挑战赛线上初赛的题目复盘归档，按 Web / Pwn / Reverse / Crypto / Misc 五个方向整理，共 41 题。

每道题一个独立目录，按下面的约定组织：

- `题目.md` —— 平台上原始题面
- `WriteUp.md` —— 解题思路与过程
- 其余脚本（`solve.py` / `exp.py` / `exp.c` / `extract.py` / `exploit.sh` / `build.sh` 等）—— WriteUp 里用到的解题代码
- `附件/` —— 平台下发的原始附件（zip 等）及 `附件说明.md`；只有平台提供了附件的题目才有这个目录

> 个别题目暂未整理出完整题解，在下方列表里单独标注；Misc 方向的 `宇宙微波背景辐射` 附件体积较大，未随仓库分发，见文末网盘说明。

## Web（9 题）

| 题目 | 题解 | 备注 |
|---|---|---|
| BlindReview | [WriteUp](Web/BlindReview/WriteUp.md) | 待补 |
| cwp | [WriteUp](Web/cwp/WriteUp.md) | |
| Crack a Little Smile | [WriteUp](Web/Crack%20a%20Little%20Smile/WriteUp.md) | |
| ezcms | — | 待补 |
| ezphp | [WriteUp](Web/ezphp/WriteUp.md) | |
| ezstar | [WriteUp](Web/ezstar/WriteUp.md) | |
| fault_web_and_fault_hash | [WriteUp](Web/fault_web_and_fault_hash/WriteUp.md) | |
| vaultlink | [WriteUp](Web/vaultlink/WriteUp.md) | |
| vaultlink_revenge | [WriteUp](Web/vaultlink_revenge/WriteUp.md) | |

## Pwn（18 题）

| 题目 | 题解 |
|---|---|
| babel | [WriteUp](PWN/babel/WriteUp.md) |
| babyark | [WriteUp](PWN/babyark/WriteUp.md) |
| babysilent | [WriteUp](PWN/babysilent/WriteUp.md) |
| babystar | [WriteUp](PWN/babystar/WriteUp.md) |
| Babyv8 | [WriteUp](PWN/Babyv8/WriteUp.md) |
| bad_puppet | [WriteUp](PWN/bad_puppet/WriteUp.md) |
| Blackout | [WriteUp](PWN/Blackout/WriteUp.md) |
| CanonicoPWN | [WriteUp](PWN/CanonicoPWN/WriteUp.md) |
| drywell | [WriteUp](PWN/drywell/WriteUp.md) |
| easytl | [WriteUp](PWN/easytl/WriteUp.md) |
| ez_qemu | [WriteUp](PWN/ez_qemu/WriteUp.md) |
| HESEUS | [WriteUp](PWN/HESEUS/WriteUp.md) |
| hexachord | [WriteUp](PWN/hexachord/WriteUp.md) |
| Ironclad | [WriteUp](PWN/Ironclad/WriteUp.md) |
| oldsvr | [WriteUp](PWN/oldsvr/WriteUp.md) |
| shadowledger | [WriteUp](PWN/shadowledger/WriteUp.md) |
| What The Fox! | [WriteUp](PWN/What%20The%20Fox%21/WriteUp.md) |

## Reverse（3 题）

| 题目 | 题解 |
|---|---|
| ELFStudy | [WriteUp](Reverse/ELFStudy/WriteUp.md) |
| firmware | [WriteUp](Reverse/firmware/WriteUp.md) |
| license | [WriteUp](Reverse/license/WriteUp.md) |

## Crypto（6 题）

| 题目 | 题解 |
|---|---|
| crp-11741 | [WriteUp](Crypto/crp-11741/WriteUp.md) |
| crp-12025 | [WriteUp](Crypto/crp-12025/WriteUp.md) |
| frozen | [WriteUp](Crypto/frozen/WriteUp.md) |
| hoRRibleRing | [WriteUp](Crypto/hoRRibleRing/WriteUp.md) |
| vmshare | [WriteUp](Crypto/vmshare/WriteUp.md) |
| zkRSA | [WriteUp](Crypto/zkRSA/WriteUp.md) |

## Misc（6 题）

| 题目 | 题解 | 备注 |
|---|---|---|
| 0perator | [WriteUp](Misc/0perator/WriteUp.md) | |
| Dreambound | [WriteUp](Misc/Dreambound/WriteUp.md) | |
| NeuronShuffle | [WriteUp](Misc/NeuronShuffle/WriteUp.md) | |
| 宇宙微波背景辐射 | [WriteUp](Misc/宇宙微波背景辐射/WriteUp.md) | 附件见网盘 |
| 谍影重重7 | [WriteUp](Misc/谍影重重7/WriteUp.md) | |
| 问卷调查 | [WriteUp](Misc/问卷调查/WriteUp.md) | |

## 大附件下载

`Misc/宇宙微波背景辐射` 的原始附件没有放进仓库（体积太大），需要的话通过网盘自取：

- 夸克网盘：<https://pan.quark.cn/s/70a404ae2835?pwd=jrPj>，提取码 `jrPj`
