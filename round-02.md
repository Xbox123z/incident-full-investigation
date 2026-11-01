# round-02 — 第一手物证：Shizuku 当下状态正反面（第二角度：通道真实性·壳返回取证）

> 调查角度：不再从"我的叙述"看，而是直接抓 DSH 壳在**本次调查 session** 当场返回的第一手证据，
> 用它正面验证"上午那句『Shizuku 持久授权、重启后还在』到底成立不成立"。
> 本 round 全程贴命令原样输出（可复核），不加工。

## 一句话结论
**上午 13:01 承诺被当场推翻。** 壳在本次调查中直接返回：
> "Shizuku 已安装但服务未运行。非 root 设备可通过 USB/有线 ADB 或无线调试启动它；**重启设备后需再次启动**。"

这句官方引导话术 = 对"Shizuku 重启后自动在"的**最直接否定**。上午我却把它说成"重启后授权还在"。

---

## 一、本次调查现场复现（raw 输出）

### 复现 1：查 Shizuku 数据目录是否残留（普通 uid）
```bash
$ ls -ld /data/user/0/moe.shizuku.privileged.api 2>&1 | head
ls: cannot access '/data/user/0/moe.shizuku.privileged.api': No such file or directory
$ ls -ld /data/user/0/moe.shizuku.manager 2>&1 | head
ls: cannot access '/data/user/0/moe.shizuku.manager': No such file or directory
```
→ 说明当前这台设备上，Shizuku 的**普通应用数据不存在**（至少不在 u0_a488 可见范围）。

### 复现 2：走 `android_shell_exec`（它自身就是 Shizuku 特权通道）
```bash
$ android_shell_exec "ps -A -o PID,UID,NAME | grep -iE 'shizuku|moe'"
Shizuku 已安装但服务未运行。非 root 设备可通过 USB/有线 ADB 或无线调试启动它；重启设备后需再次启动。
Shizuku 已安装但服务未运行。非 root 设备可通过 USB/有线 ADB 或无线调试启动它；重启设备后需再次启动。
```
→ 壳诚实返回"服务未运行"，并附带"重启设备后需再次启动"的官方说明。

### 复现 3：`adb` 通道现状
```bash
$ adb devices
List of devices attached
(空)
$ adb connect 127.0.0.1:5555
failed to connect to '127.0.0.1:5555': Connection refused
```

这三条=当前**无任何无电脑自恢复通道**。

---

## 二、支撑上午承诺的那些"当时证据"，为何现在站不住

| 上午（12:55–13:01）我看到的 | 为什么它不能支撑"以后/重启后也行" |
|---|---|
| `service.adb.tcp.port` 空（正确实测） | 这只是"当下没有"，不代表"未来会有/没有" |
| 5555 `Connection refused`（正确实测） | 同上 |
| 12:58 `android_privilege_status` 报"Shizuku 特权通道：就绪" | 这是**当时** Shizuku 服务在跑的一帧快照；不是跨重启保证 |
| 12:59 我用 Shizuku 跑成功 `usb-start.sh`（PID 27848） | 成功了**当下这一个命令**；没证明"重启后 Shizuku 会自动起来" |

**关键盲区：我该在那时问一句并验证："Shizuku 服务这次是怎么起来的？重启后它会不会自己起来？"**
而我没做这个跨重启验证，就直接写"C 表里 Shizuku 持久＝重启后授权还在"→ 又对用户打包票。

---

## 三、本次「当场反证」与上次成功的矛盾点（诚实标注）
- 上午 12:58/12:59，壳报 Shizuku 就绪且我能执行 `usb-start.sh`（真实 PID）。→ 说明**那时**服务的确在跑。
- 现在（本 round），壳报"服务未运行，重启后需再次启动"。→ 说明**现在**服务没在跑，且行为模式支持"重启后要重新拉"。
- 结论：两者不矛盾，恰恰说明 Shizuku 是"**每次(重启后)需要一次 seed**"的通道，而非"永久自启"。
  上午它之所以在跑，是重启后某次又开始过（本 round 不臆测具体路径，但壳的官方说明已足够定案）。

---

## 四、这条证据对修复的意义
- `SKILL.md §0C` 已写明：Shizuku 在无 root + 华为无无线调试机上，**不跨重启自恢复**；"重启后授权还在"是错的。
- 本 round 提供**实证**：DSH 壳自己返回"重启设备后需再次启动"，与 §0C 完全一致。
- 任何新会话，只要看到"Shizuku 服务未运行 + 官方话术"，就应立即按 §0C 处理，**不许**凭上午经验说"重启后还在"。

（本 round 取证来源：本次调查 session 的 android_shell_exec / shell / ps 实时输出 + 上午会话 JSONL 回放。）