# round-27 — 一键式恢复 SOP：断种后如何把 uid2000 通道喂回来（用户可照做分步指南）（第二十七角度）

> 调查角度：把前面多 round(05/13/15/17/23)零散的"代码/seed 方法"汇成**一份可照做、可执行的恢复手册**——
> 从"通道断了"到"拿到 uid2000 shell"的完整分步，前提与命令都核实过。

## 一、前提（先核实，别跳）
- 本机无 root、无线调试入口被系统移除(round-17)、无 DPM(round-21) → **唯一可靠 seed = 电脑侧 adb 传一次**。
- 本机自举元件已确认在场(round-27 实测)：`adb` 二进制 + `libabsl_log_internal_message.so`(round-23 成败库)。
- 所以**恢复分两段**：①电脑/*USB seed 喂出一个可连端点 → ②之后纯本机自连(round-23 配方)。

## 二、第一步：电脑侧 seed（一次性，需你插线/无线配一次）
> 前提：手机 USB 调试开(开发人员选项)，插电脑。
```bash
# ① 电脑确认能看到设备
adb devices
# ② 把 adbd 切到 TCP 模式（一次性喂端点，重启即失效，本意就是"喂一次"）
adb tcpip 5555
#    若手机提示需要"无线调试"配对，按弹窗用临时的 pairing 端口 + 码配对一次也行(它同样重启失效)
# ③ 丢下电脑。现在手机 adbd 应在 127.0.0.1:5555 和 emulator-XXXX 上监听
```

## 三、第二步：本机自连(插线后可拔,电脑可拿走) — round-23 配方
```bash
# 在你的手机(DSH/Mika/任意终端)里：
export GHOME=/data/user/0/com.dsharnessmobile.shell/files/home
export LIB="$GHOME/adbget/merged2/data/data/com.termux/files/usr/lib"
export ADB="$GHOME/adbget/merged2/data/data/com.termux/files/usr/bin/adb"
# ① 起 server(必须 env LD_LIBRARY_PATH，否则 CANNOT LINK round-23)
pgrep -f "env LD_LIBRARY_PATH.*adb.*server" >/dev/null || \
  nohup env LD_LIBRARY_PATH="$LIB" "$ADB" -P 5037 server nodaemon > "$GHOME/adb-s.log" 2>&1 &
sleep 4
# ② connect
env LD_LIBRARY_PATH="$LIB" "$ADB" -P 5037 connect 127.0.0.1:5555
# ③ 验证 uid
env LD_LIBRARY_PATH="$LIB" "$ADB" -P 5037 -s 127.0.0.1:5555 shell id   # 应 = uid=2000(shell)
# ④ 封装复用
run(){ env LD_LIBRARY_PATH="$LIB" "$ADB" -P 5037 -s 127.0.0.1:5555 shell "$@"; }
```

## 四、第三步（可选/进阶）：让电脑端完全消失后仍能本机恢复的条件
- 若想让"无电脑也能再恢复"，需**另存一个能自连接的端点**的 seed——但本机无 root/无线，做不到"持久免电脑"(round-21/17)。
- 所以**最诚实**：本机"无电脑自连"用于【种子在时复用】，不用于【从零建种子】；从零建种子必须电脑一次。

## 五、常见失败与处理
| 现象 | 原因 | 处理 |
|---|---|---|
| `CANNOT LINK EXECUTABLE ... not found` | server 没带 LD_LIBRARY_PATH | 用 `env LD_LIBRARY_PATH=$LIB` 起 server(round-23) |
| `connect 127.0.0.1:5555` refused | adbd 没切 TCP/或已重启 | 重新电脑 `adb tcpip 5555`(第一步) |
| `failed to start daemon` | server 端口被占/上次残留 | kill 残留 adb server 再起 |
| Shizuku 返回"服务未运行" | seed 后 Shizuku 没起 | (可选)电脑 `adb shell sh /path/usb-start.sh` 起 helper;或走自连 shell 自已手动起 |

## 六、沉淀(skill §0A 加"一键恢复手册")
- 把上述第一~三步作为 `验证seed并恢复uid2000通道` 的标准段落，第一个选项 = 电脑 `adb tcpip 5555`，第二段 = 本机自连 recipe(round-23)。**凡说"需要 seed"，就给出这两段可照做命令，别只说"需要电脑"四个字。**

## 七、这轮价值
把"需要 seed"从一句空话变成**可一键复制的恢复手册**——用户以后断种不再慌，照做第一步(电脑一次)+第二步(本机自连)即可回到 uid2000。这就是"解决方法"最落地的样子。
（下一 round 继续。不停。）