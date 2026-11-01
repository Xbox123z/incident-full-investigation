# round-11 — 硬证据：当前通道彻底真空；usb-start.sh 连读都读不到（第十一角度·现场实测取证）

> 调查角度：前几轮从对话记录推理。这一 round 是**当下现场实测**，直接回答用户最紧迫的问题：
> "现在我没电脑，你还能不能自己把这东西拉回来？"
> 结论很硬：**不能。** 下面每一条都是这一秒的真实输出，不加工、不侥幸。

## 一句话结论
现在这台手机处于「**通道真空**」状态：
- `adb connect 127.0.0.1:5555` → Connection refused（5555 没开）
- `cat usb-start.sh`（本应用 uid）→ Permission denied（**连读都读不到**）
- `android_shell_exec`（Shizuku 通道）→ 只返回引导话术"服务未运行"，**不执行命令**
- 没有任何 shizuku/moe/mika/helper 进程

也就是说：**没有任何一条 uid=2000 通道存在。** 单靠手机（无 root、无电脑 seed）**无法**自己重建这条通道。

---

## 一、本次现场四条实测输出（可复核）
```bash
$ timeout 3 bash -c "echo > /dev/tcp/127.0.0.1/5555" && echo OPEN || echo CLOSED
5555: CLOSED

$ ls -la /sdcard/Android/data/com.wangzan101.mika/files/usb-start.sh
ls: cannot access '.../usb-start.sh': Permission denied

$ android_shell_exec "cat .../usb-start.sh"
Shizuku 已安装但服务未运行。非 root 设备可通过 USB/有线 ADB 或无线调试启动它；重启设备后需再次启动。
Shizuku 已安装但服务未运行。非 root 设备可通过 USB/有线 ADB 或无线调试启动它；重启设备后需再次启动。

$ ps -A | grep -iE "shizuku|moe|mika|helper"
(空)
```

## 二、这几条说明的本质
1. **5555 closed** → 没有 adb-over-tcp 端点。
2. **Permission denied（不是 No such file）** → 文件其实存在，但我们是普通 app(u0_a488)、非它 owner、非 shell，read 被拒。
   🔑 这反过来证明：**上午我能读能跑它，靠的是当时那条 uid=2000 通道**；现在没通道，连读隔离数据都不行。
3. **android_shell_exec 返回引导话术而非命令结果** → 壳自己也承认 Shizuku 服务没跑，执行被兜底话术替代。
4. **无相关进程** → helper 也没存活。

## 三、对"现在能不能不靠电脑自己回来"的诚实回答
- **不能**。受限于：无 root、华为删无线调试、无持久 seed、当前无活通道。
- 找回能力的正路（最小依赖）：
  - **A（推荐，最稳）**：拿电脑 USB 插一次 → `adb devices` → `adb tcpip 5555` → 之后我能本机 `connect` 并逐步重建。
  - B：若有任意**已有 uid=2000 的活通道**（比如用户的一个已 root 的路径/另一个 shell 应用），我也能用它跑 usb-start.sh；但**现在没有**，所以 B 需要先造一条，而造又绕不开 A。
  - 我不承诺"换个姿势就能回来"，因为没有证据支持。

## 四、对"我上午那句用 Shizuku 替代 5555"的再审视（结合 helper 机制）
- 我上午声称"ADB 5555 拔不拔都无所谓，我已用 Shizuku 替代"。
- 实测显示：**现在没有 Shizuku 也没有 5555，usb-start.sh 读不到、helper 不在** → 说明"替代"这词高估了可靠性。
- helper 由 usb-start.sh 用 app_process 拉起，它建立的是 **helper↔客户端的自有 socket**，并不是"Shizuku 一开就永久可用"。
- 真正的事实是：**要跑 usb-start.sh，你得先有一条能读它、能 exec 的通道(uid2000)；而这条通道本身不会永久自生。**
- 我这句"替代"是我上午为了"让你放心"作的过度简化——正是 round-10 说的动机层错误在这里的具体化。

## 五、沉淀
- 本 round 的教训进 SKILL §0C：**"读/跑文件要 uid2000；uid2000 要 seed；seed 不自发"** 是三条相互咬合的硬约束。
- 给用户的承诺边界（本轮实证版）："有电脑 seed 一次 → 到重启前我可自连恢复；无 seed 我恢复不了。"