# round-13 — 完整还原「当初怎么自己给自己拉 adb」+ 那个错误前提的真正诞生地（第十三角度·逐字实证）

> 调查角度：用户要"当初到底是怎么做到无电脑自连 adb 的，一步步还原"。这一 round **逐字回放原始命令与输出**，
> 并挖出 skill 里"USB调试一开就 5555"错误前提的**确切来源**——当初它是"当时实测真开的"，不是完全凭空编的。
> 真相比"单纯撒谎"更微妙：**错误来自把"当时某种状态下实测开"写成"永远、任何状态都开"。**

## 一句话结论
当初确实**在纯手机上**自连成功了：用 apt 下载 android-tools 的 deb → 手工解包拿到 adb 二进制和它依赖的全部 .so → 设 `LD_LIBRARY_PATH` → 本机 `adb connect 127.0.0.1:5555` → 成功对 `127.0.0.1:5555` 和 `emulator-5554` 都拿到 device。
而 skill 那句"USB 调试开启后 adbd 监听 5555"不是无中生有——当时它实测就是这么通的。**真正错在：把一个"当时恰好成立"的状态写成了"永远成立、不依赖额外条件"，于是重启后(5555 没开、emulator 也没了)这套就失效，而我上午却拿它当"持久"承诺。**

---

## 一、当初自连 adb 的原始步骤（逐字回放，来自会话 %05B记录）
### ① 下载并解包 android-tools(拿到 adb 二进制)
```bash
cd "$HOME/adbget" && apt-get download android-tools
# 得到 android-tools_37.0.0p1-3_aarch64.deb
# 用 python 手工解 ar/deb，抽出 data.tar 里的:
#   merged/data/data/com.termux/files/usr/bin/adb
```
### ② 下载 adb 依赖的动态库并合到 merged2
```bash
apt-get download libabsl-* liblog libprotobuf libfmt libbrotli1 liblz4 libzstd libc++ 
for d in *.deb; do
  ar p "$d" data.tar.xz | tar xJ -C merged2 ...   # 解出所有 .so 到 merged2/.../usr/lib
done
```
### ③ 用 LD_LIBRARY_PATH 跑 adb 客户端(纯本机)
```bash
L=$HOME/adbget/merged2/data/data/com.termux/files/usr/lib
B=$HOME/adbget/merged2/data/data/com.termux/files/usr/bin/adb
LD_LIBRARY_PATH=$L $B -P 5037 connect 127.0.0.1:5555
# → "connected to 127.0.0.1:5555"
LD_LIBRARY_PATH=$L $B -P 5037 devices -l
# → 127.0.0.1:5555  device product:CLS-AL00 model:CLS_AL00 device:HWCLS
#   emulator-5554   device product:CLS-AL00 model:CLS_AL00 device:HWCLS
```
**这就是"纯手机自己给自己拉 adb"的全部秘密：手机里装一个 adb 客户端，连回手机自己的 adbd。**

### ④ 当时为什么能连上？——关键(看 16:41:37 当时的 reasoning)
> "**127.0.0.1:5555 is open!** This is the adbd TCP port. ... adbd is **also listening on port 5555** (possibly because the user enabled wireless debugging, **or because EMUI's adbd listens on 5555 by default**)."

当时**实测** 127.0.0.1:5555 是开的、还有 `emulator-5554`。所以 22:46:53 写 bootstrap_adb.sh 时，我把这个实测固化成注释：
> "前置：开发者选项里「USB 调试」开启（adbd 会在 5555 监听，**即使没插线**）"

---

## 二、错误前提到底错在哪（与"纯粹撒谎"不一样）
- **对**的部分：当时纯本机 USB 调试开着的状态下，5555 确实通（实测）。
- **错**的部分：写成"USB 调试一开就一定有 5555、且**永远**这样、不依赖无线调试/其它条件"。
- 事实上：
  - 那次`emulator-5554`在列表= adbd 跑在**emulator 形态**的附加 TCP 监听上；
  - 那次能连很可能还叠加了当时的"无线调试"/某次配对/`persist` 状态；
  - **重启后**：`service.adb.tcp.port`空、5555 refused、emulator也没了 → 说明它本来就不是"USB 调试自带、稳定、持久"。
- 所以正确写法是：**"初始在特定状态下 5555 自连成功过；它依赖的并不仅是'USB调试开启'，重启后会消失；要用必须先重建该监听状态(tcpip/无线调试/其它seed)"**。skill 当时省掉了这个限定 → 埋雷。

## 三、这如何直接解释 11-01 事故
- skill 声称"USB 调试开启→5555 在"，我就以为这套稳定。
- 11-01 重启后我用它做思维惯性：既然"USB 调试开了"，理应 5555 在 → 要不是 12:59 亲眼看到 refused，我可能就这么糊弄过去了。
- 真正该做的：**每次都用 0A「先实测」确认 5555 到底开没开**，而不是拿 skill 的"应该开"当结论。这就是 round-01/§0A 的核心价值。

## 四、可复现的"本机自连 adb"正确方法（更新后）
1. 先确保已有 adbd 的 TCP 监听端点(seed)：重启后需一次 `adb tcpip 5555` 或无线调试/其它 seed。
2. 再用下方"客户端自举"连：
```bash
# (一次性) 用 apt 下载 android-tools + 依赖 deb，解包到 adbget/merged2
# 之后每次:
L=$HOME/adbget/merged2/data/data/com.termux/files/usr/lib
B=$HOME/adbget/merged2/data/data/com.termux/files/usr/bin/adb
LD_LIBRARY_PATH=$L $B -P 5037 connect 127.0.0.1:5555 && LD_LIBRARY_PATH=$L $B -P 5037 devices -l
```
3. **先测**：`echo > /dev/tcp/127.0.0.1/5555` 通不通，通再 connect；不通别假设。没有无线调试的华为重启后大概率不通 → 需要 seed。

## 五、沉淀
- 修订 bootstrap_adb.sh 注释，把"USB 调试→5555 自动在"改成正确定位：**"需要已存在 TCP 端点；重启后通常要重新 seed，先 0A 实测"**。
- skill §0A 明确：skill 里"应该怎么"≠"当前就是这么"；一切以本轮实测为准。
- 这个"当时实测对、写成永久对"的两段式错误，正是 round-12(直接读警告却反着写)的技术根源补全。

（下一 round：继续抠 11-01 为何重启后 5555 彻底消失，及"可持续自启通道"是否真实存在。）