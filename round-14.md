# round-14 — 侦查式自纠：重启后 Shizuku「未必必没」——上午那次是真的（并修正我此前的过度绝对化）（第十四角度）

> 调查角度：用户强调"每个细节抠出来"。本 round 我**提出并审视一个前几轮被我武断略过的疑点**，
> 用原始输出求证，得到一个需要**修正**的结论。诚实自纠前的错误，正是侦探该做的。

## 一句话结论
- 事实：**11-01 上午手机重启(约12:35)后，12:59 `android_shell_exec`(Shizuku/uid2000) 真的以 shell 执行成功了 `usb-start.sh`**（亮出 PID 27848/27889）。
- 因此：**"本次重启后 Shizuku 必没"是过度绝对化**——它这次重启后是**可用的**。
- 修正后准确说法：**Shizuku 的重启可用性不稳、不可依赖**；历史上出现过重启后仍可用的实例，也出现过(15:37 后)彻底不可用的实例。**绝不能被"上午那次能用"忽悠成"永远能用"。**

---

## 一、硬证据（原始 tool/result 逐字）
```
[12:59:47] CALL android_shell_exec:  sh /sdcard/Android/data/com.wangzan101.mika/files/usb-start.sh
[12:59:50] RESULT:
   Mika helper starting... Open Mika to confirm phone control is ready.
   It keeps running after you unplug the cable. After a phone restart, run this command again.
[12:59:55] CALL android_shell_exec:  ps -A -o PID,NAME | grep mika
[12:59:55] RESULT:
   27848 com.wangzan101.mika:helper
   27889 com.wangzan101.mika
```
- 背景：`uptime` = `12:59:19 up 20 min` → 手机刚在 **12:35 重启**。
- 却`android_shell_exec`真的跑起来、出了 helper PID → **Shizuku 当时以 uid2000 真实可用**（非缓存/假就绪）。

## 二、我此前的错误与修正
| 我此前(round-11/§0C)写的 | 更准确的说法 |
|---|---|
| "重启后 Shizuku 必没/彻底真空" | "Shizuku 重启后**可能仍短时可用**，但**既不保证在，也不保证不在**；历史有在的例子，也有没的例子" |
| "任何 uid2000 都要 seed，重启必失" | "有一定概率在重启后仍可用(取决于 ROM/壳/驻留)，但**没有持久保证**，不能承诺" |
| 上午错在"把一次可用当永久" | 这个结论**仍然成立**，因为它强调的是"不可靠、不可承诺"。知晓它"有时可用"更增加了它的迷惑性 |

## 三、对事故判断的修正(诚实)
- 上午我错，**不是因为"Shizuku 一定不能重启用"**——而是：
  1) 我把"12:59 那次能用"直接升级成"重启后授权还在"这种**无时间边界的永久承诺**；
  2) 我**没有验证**"下次重启(或过几小时)还在不在"，只看了当下；
  3) 我忽视了脚本自己说的 `After a phone restart, run this command again`(=作者也不认为自动持久)。
- 换句话说，即便 Shizuku 有时能重启后在，**那个"有时"也绝不等于我承诺的"永远"**。脚本作者自己的措辞就反对"run once 就永久"。

## 四、为什么 15:37 后又确实没了(仍成立)
- 本 session round-11 实测 `android_shell_exec` 只返回"服务未运行"，Shizuku 数据不在、无进程、5555 关。
- 说明当天从"上午可用"滑到"下午不可用"，**正印证其不稳定性**：不是"必活"，也不是"必死"，而是**状态会漂移，因此不可拿来承诺**。
- 将来凡需用，必需**当下实测**(0A)+ **每次重启重测**；不能用记忆里的"上次能/上次不能"替代。

## 五、沉淀(修正 §0C 措辞)
- §0C 从"Shizuku 重启必没"改为：**"Shizuku 重启后可用性不负责任、会漂移——早上实测可用、下午实测不可用的真实案例(round-14)。任何跨重启承诺前必须重新实测(0A)；脚本'sum restart run again'本身即反对自动持久。"**
- 同时明确：**"现在能"永远不等于"以后能"**，这正是上午事故的第一性教训。

（下一 round：继续用这种"疑点→自查→修正"的侦探法，继续抠。不停。）