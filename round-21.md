# round-21 — 建设性实证：DSH 壳确有开机自启(验证成功)，但它只自启自己的 3080 服务，不含 uid2000 通道（第二十一角度）

> 调查角度：这轮回答一个此前只反面总结、没正面实证的问题——"这手机/壳到底能不能开机自己把通道拉起来？"
> 我直接解包壳 APK 查 Manifest，并读了壳自己的开机诊断日志，得到**一手证据**：壳有 `BOOT_COMPLETED` 自启且多次成功，但自启的只是 HTTP 服务，不是 uid2000 shell 通道。

## 一句话结论
- **有**：DSH 壳注册了 `android.intent.action.BOOT_COMPLETED` + 权限 `RECEIVE_BOOT_COMPLETED`，有 `com.dsharnessmobile.shell.BootReceiver`，开机后被系统拉起。
- **证据**：壳 `boot-segments.log`/`boot-diag.log` 记录了多次完整自举：`boot-start → listen(约8s) → first-http(约10s)`，多次成功。
- **但**：它自举的是**自己的 Web 服务(127.0.0.1:3080)**，不负责 adb/Shizuku 的 **uid2000 shell 能力**。所以"壳开机能自启"≠"开机后通道自动恢复"。

---

## 一、一手证据
### 1) APK Manifest（解包 base.apk 提取字符串）
```
android.permission.RECEIVE_BOOT_COMPLETED     ← 声明开机自启权限
android.intent.action.BOOT_COMPLETED           ← 监听开机广播
com.dsharnessmobile.shell.BootReceiver         ← 开机广播接收器
com.dsharnessmobile.shell.AdbKeyboardReceiver  ← 另有 ADB 键盘接收器
```
### 2) 开机诊断日志（boot-segments.log，多次成功）
```
dsh-boot-segments t_boot_start=... t_listen=... t_listen_ms=8123
                   t_first_http=... t_first_http_ms=9949 ... note=first-http
（另有多个 listen≈8s / first-http≈10s 的成功序列）
```
→ 壳在**开机后被拉起并且完整起来了（能 listen 127.0.0.1:3080 并响应 HTTP）**。

## 二、关键区分（避免建设性自我误导）
- 壳开机自启的是 **DSH 自己的 Web/推理运行时**(端口 3080)——它让账号/Agent 能重启后继续工作。
- **但它不需要也不能**提供「uid2000 shell / adb / Shizuku 特权」给本机命令通道。
- 这两个是独立能力：
  | 能力 | 开机后是否自恢复 |
  |---|---|
  | DSH Web 服务(聊天/任务) | ✅ 自启(本轮实证) |
  | adb 5555 / Shizuku uid2000(本机 shell 特权) | ❌ 需 seed；与壳自启无关 |

## 三、对"以后怎么办"的建设性答案(诚实，不粉饰)
- 若要"开机后连 uid2000 通道也自动恢复"，壳层面的 BOOT_COMPLETED **目前不含**这项工作；要加，需满足之一：
  - 有 root / device-owner(可注册更全权限广播) → 让壳开机后也跑 `adb tcpip` 或拉起 Shizuku(若设备支持自启)。
  - 或依赖"壳以特权 uid 启动"的机型(本机无，需验证)。
- 对本机：**没有上述能力，开机后 uid2000 通道仍须 seed(电脑/USB 或某次无线调试)**。壳自启只能保证"聊天服务回来"，不保证"shell 通道回来"。
- 这是对用户"难道一点办法没有？"的诚实正面答：**办法存在(壳已能自启)，但只覆盖服务层，不覆盖 shell 通道层；覆盖通道需 root/device-owner 能力，本机暂无。**

## 四、沉淀(skill §0C/SAFETY)
- 加一句：**"壳(本机 BOOT_COMPLETED)能开机自启自己的 3080 服务(实证 round-21)，但自启 ≠ 通道；uid2000 shell 通道无自启，仍须 seed。"**
- 避免将来把"壳能自启"误当成"通道能自启"来安抚用户。

## 五、这轮的价值
填补了"建设性"维度：不再只是"不能/漂移"，而是**精确划出"哪部分能自启(DSH服务)、哪部分不能(uid2000通道)"**。
给用户一个明确可核查的边界：**开机后聊天能用≠开机后手机命令能用**，两者要分开判断、分开恢复。

（下一 round：继续侦探。不停。）