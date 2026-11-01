# round-23 — 「当初到底怎么做到的」最精确还原：10-27 首次本机自连的失败→修复逐字序列（第二十三角度）

> 调查角度：用户反复要"把当初如何做到自连 adb 的每个细节完全还原"。前几轮(round-13)讲了"成功"，
> 但没抠出**真正决定成败的那个机制**。这轮把 10-27 首次本机自连的完整记录逐字还原，定位到那个成败关键。

## 一句话结论
**本机自连成功的决定性机制不是"有没有 adb 二进制"，而是"'adb server'进程必须用 `env LD_LIBRARY_PATH=$LIB` 启动"。**
第一次 `nohup $ADB server`（没带 env）直接 `CANNOT LINK EXECUTABLE: libabsl_log_internal_message.so not found`；
第二次 `nohup env LD_LIBRARY_PATH=$LIB $ADB server` 才 `connected to 127.0.0.1:5555`，并拿到 `uid=2000(shell)`。

---

## 一、10-27 13:10-13:11 完整逐字序列（一手记录）
### 第 1 次尝试：失败
```
LD_LIBRARY_PATH=$LIB $ADB -P 5037 connect 127.0.0.1:5555
→ * failed to start daemon
  error: cannot connect to daemon
  Server had pid: 30807
```
### 诊断
```
5037 closed
5555 OPEN        ← adbd 在，缺的只是 adb client/server 这端
```
### 第 2 次尝试：显式失败原因
```
nohup $ADB -P 15037 server nodaemon ...
→ CANNOT LINK EXECUTABLE ".../usr/bin/adb":
  library "libabsl_log_internal_message.so" not found:
  needed by ...
```
**→ 关键失败点：adb server 进程缺 LD_LIBRARY_PATH，找不到依赖库。**
### 第 3 次（成功）
```
nohup env LD_LIBRARY_PATH=$LIB $ADB -P 5037 server nodaemon > adb-s4.log 2>&1 &
sleep 4
LD_LIBRARY_PATH=$LIB $ADB -P 5037 connect 127.0.0.1:5555
→ 10-27 13:11:34 ... W adb: usb_linux.cpp:633 Failed to open bus ... Permission denied
  (无害:它在 usb/无关,仍可经 tcp)  
  connected to 127.0.0.1:5555
```
### 成双 + uid2000(4秒后)
```
adb devices -l:
  127.0.0.1:5555   device product:CLS-AL00 model:CLS_AL00 device:HWCLS transport_id:2
  emulator-5554    device product:CLS-AL00 model:CLS_AL00 device:HWCLS transport_id:1
adb -s 127.0.0.1:5555 shell id
→ uid=2000(shell) gid=2000(shell) groups=2000(shell),1004(input),1007(...)
```
### 固定用法
```
export ANDROID_SERIAL=127.0.0.1:5555
run(){ LD_LIBRARY_PATH=$LIB $ADB shell "$@"; }   ← 此后所有 shell 操作走这里
```

## 二、成败关键（为什么是 env）
- adb client 用 `LD_LIBRARY_PATH=$LIB $ADB ...` 能跑（client 不需要那套 so），
  但**spawn 的 `adb server` 是个新进程，若不显式 `env LD_LIBRARY_PATH=$LIB`，它继承不到** → 链接阶段就死。
- 所以正确配方 = **server 用 `env LD_LIBRARY_PATH=$LIB` 起 + 之后 connect/client 也用同一 LD_LIBRARY_PATH**。
- 这也解释了 round-13 里"`execl returned -1`"那类报错的真相：LD_LIBRARY_PATH 没正确传对。

## 三、对"当初怎么做到"的最完整答案
```
1. 手机 adbd 处于 USB 调试/无线调试启用状态 → 自带 127.0.0.1:5555 + emulator-5554 (round-17:非tcpip命令所开)
2. 本机有 adb 二进制 + 其依赖 so(已向量下载自举)  → adbget/merged2/.../usr
3. 用 env LD_LIBRARY_PATH=$LIB 起 adb server (缺这一步就 CANNOT LINK, 本轮实证)
4. adb -s 127.0.0.1:5555 shell → 拿 uid=2000(shell)
5. 封装 run(){ LD_LIBRARY_PATH=$LIB $ADB shell "$@"; } 长期复用
```
- 缺任一项都失败；尤其第 3 步的 `env` 是本轮新增、此前未写入 round 的成败关键。

## 四、沉淀(skill §0A 加一条)
- 新增：**"自举 adb 时，`adb server` 必须 `nohup env LD_LIBRARY_PATH=$LIB $ADB server nodaemon` 起，否则 `CANNOT LINK EXECUTABLE: ... not found`(round-23)。client 侧同样带 LD_LIBRARY_PATH。这是当初自连成功的第一性命令细节。"**

## 五、这轮的价值
补完了"当初怎么做到"的**最后一颗螺丝**：不是"有二进制就成"，而是"server 进程必须正确继承库路径"。这是用户想在每一个细节上看到的还原。
（下一 round 继续。不停。）