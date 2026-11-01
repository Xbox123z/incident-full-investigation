# round-08 — 换同款新手机：如何一步步重建全部通道（第八角度）

> 调查角度：用户需求"换同款新手机如何一步步重建通道"。本 round 给**诚实可用**的完整流程，
> 明确标注哪一步需要电脑/USB seed、哪一步之后不需要电脑。绝不为了"方便"而藏起要电脑的一步。

## 一句话结论
换新机后，**第一次仍必须插一根电脑 USB 做 seed**（无 root + 华为删无线调试的机型无法绕开）。
seed 一次之后，到下次重启前，我可以全程本机自连、不需要电脑。要"重启后也自动"须另有持久化措施(见 §四)。

---

## 〇、前置事实（这台机型的硬限制）
- 华为 CLS-AL00 / HarmonyOS 4.3：删了「无线调试」类 → 无 `adb_wifi_enabled` 路径。
- 无 root：无法 `setprop service.adb.tcp.port`，无法改 adbd TCP 模式。
- 结论：唯一能"种下本机 TCP/Shizuku 通道"的，是**一次 USB 电脑 adb**。

## 一、重建分步（含"哪步要电脑"标注）
> 准备：新手机打开「开发人员选项 → USB 调试」；一根数据线；电脑装了 adb（或任意终端 adb）。

### Step 1（🧷 需要电脑·只这一次）
```
USB 连电脑
电脑执行: adb devices          # 应看到 device
           adb tcpip 5555      # 让 adbd 在 5555 监听(本次开机内)
           adb connection 状态ok
# (可选) 若要用 Shizuku: 电脑 adb shell sh /data/local/tmp/*.sh 或装 Shizuku manager 由 adb 启动
```

### Step 2（不需要电脑）
```
手机(或我的 shell 环境)执行:
  adb -P 5037 connect 127.0.0.1:5555     # 本机自连(因为 seed 已种)
  adb -P 5037 -s 127.0.0.1:5555 shell id # 应能得 uid=2000(shell)
```

### Step 3（不需要电脑）
```
走已通的 channel 跑安装/启动：
  sh /sdcard/Android/data/<app>/files/usb-start.sh   # 例：拉起 helper
```

### 说明
- 到此"不用电脑"的部分 = **本次开机内、种子已种**。
- **任何一次重启**后：5555 消失、Shizuku 服务不自动 → 需要**再插一次电脑 seed**（除非做了持久化）。

## 二、各能力的最短重建路径（换机后）
| 能力 | 最短路径 | 要电脑? |
|---|---|---|
| 本机 adb 自连(5555) | 电脑 `adb tcpip 5555` 一次 → 本机 connect | 首次要 |
| Shizuku(uid2000) | 电脑 adb 启动一次 → 之后 session 内可用 | 首次要 |
| 装 APK(pm install) | 走已通 channel 用 `pm install` | 有通道即免 |
| 拉 Mika/app helper | 走已通 channel 跑 usb-start.sh | 有通道即免 |

## 三、诚实边界提醒（这次事故的同款雷区）
- **"拔线没事"只在"本次开机、种子已种"下成立；重启后绝不默认成立。**
- 对用户打包票前，先问：**"我们要承诺的是『现在拔线』还是『以后重启都行』?"** 后者必须先有持久化件。

## 四、想要"重启后也自动"（可选，不承诺未验证）
- 需要一条**开机自启 seed**：如 root 设备、或用带开机自启的 device-owner 类工具、或 USB-C 供电的 adb 硬件。
- 这些**必须先实测验证**，没验证绝不写进承诺。默认如实说"重启后要再插一次电脑"。

（下一 round：从措辞护栏落地进 skill、以及"不该再犯的说法清单"继续多角度推进。）