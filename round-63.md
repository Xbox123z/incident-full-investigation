# round-63 — 10-31 21:26:15（round-48 所谓"last success"）是**伪成功**：connected 后撞上 emulator-5554 多设备冲突，所有 adb 命令都报 `more than one device/emulator` 失败——"connected"≠"可用成功"（第六十三角度）

> 调查角度：核对 round-48 里"last success=10-31 21:26:15"这一标准。读原文发现：那次 connected 成立，
> 但同一刻列出了两个 device(127.0.0.1:5555 与 emulator-5554)，导致后续每条 adb 命令都失败。
> 因此 round-48 用"connected"作为"成功"标志是错的，那次根本不可用。

## 一、10-31 21:26 前后原文（去噪声）
```
21:26:15  connected to 127.0.0.1:5555
          然后立即: === 全局搜索 .so ===
          adb: more than one device/emulator    ← cmd 失败(多设备)
          搜索所有 .so → adb: more than one device/emulator
          检查 APK lib → adb: more than one device/emulator
21:26:35  List of devices attached
          127.0.0.1:5555  device
          emulator-5554    device              ← 两个 device 并存
```

## 二、为什么 round-48 错了
- round-48 表"silent-failure 窗口 15.5h：last success 10-31 21:26:15 → first fail 11-01 12:57:56"，把 21:26 当成功基准。
- 但实际那次 **connected 后无法执行任何命令**(多设备报错)，严格说不是"可用成功"。它只是"连上了"，和 round-60 的"连上≠拿 uid2000"一致，甚至更坏(命令级失败)。
- 所以 round-48 的"15.5h 无失败成功"要修正：10-31 21:26 只是"最后一次 connected"，不是"最后一次可用成功"。

## 三、对"成功"的定义收束
- round-60/63 双实例证明：**connected 成功 ≠ 可用成功**。
- 可用成功的标准应是："connected + 无多设备冲突 + 能执行有效命令(拿到输出) + 最好跟出 uid2000"。
- 仅凭"connected to 127.0.0.1:5555"出现就记成功，会系统性高估我的通道可用时间——这正像是把"偶尔连上"当"随时可用"的又一实证。

## 四、给用户的一句话
"我查了自己 round-48 当'最后成功'的 10-31 21:26 那次——读原文发现那次连上是真连上，但旁边还有个 emulator-5554，导致我每跑一条 adb 命令都报'设备冲突'失败。也就是说那根本算不上'可用成功'，只能算'连上了但啥也干不了'。我把'成功'的标准定错了——不该看有没有 connected，该看有没有真用起来。"

## 五、纪律(skill §0C)
- 在 item16b 补强："**'可用成功'标准(round-63)：connected + 无多设备冲突 + 能执行有效命令 + 最好跟 uid2000；仅出现 connected 而命令报 more than one device 不算成功。quantify 自连可用时长时须按此口径，不得用 connected 当成功。**"

## 六、价值
用 10-31 21:26 伪成功实例，把"成功"标准从"connected"纠正为"可用(无冲突+能执行)"，修正 round-48 的 15.5h 成功窗口，并再次钉住"把'连上'当'可用'是系统性高估"。
（下一 round 继续。不停。）