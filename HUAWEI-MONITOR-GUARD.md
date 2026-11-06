# 华为监控/拦截调查 — 现场记录（待续）

> 主题：用户想知道华为手机（HONOR? 实为 Mate70 / HarmonyOS 4.3.0，Kirin 9010）上类
> `systemhelper` 的系统组件是否监视安装/流量，以及**能否"干掉"这些组件**。
> 本文件是跨会话的「记忆锚点」：用户过几天回来，凭文件名即可接上。

## 一、用户原始诉求（贯穿多条）
1. 测华为会不会**拦截安装 OKX Wallet / Nicegram**（真实安装拦不拦）。
2. 刚才又转问：类 `systemhelper` 的组件会不会**监视**（安装/流量/文件）。
3. 想**"干掉"这些组件**（像关杀毒那样），尤其点名：
   - HwHiAIDSEngine
   - HwBehaviorAuth
   - HwIAware
   - AntimalPlugin
   - AiSecurityPlugin.apk

## 二、已实锤的组件清单（本次在本机实测 `ls /system/{app,priv-app}`）
华为上**没有**字面叫 `systemhelper` 的组件，但**职能对应**的是这一批（全在系统分区）：

### 安全/杀毒/行为检测（"监视安装/流量"主力）
| 组件 | 位置 | 职责 |
|---|---|---|
| **HwSystemManager** | `priv-app/HwSystemManager/` | 华为"手机管家"主程序（40MB），聚合下面插件 |
| ├─ **AntimalPlugin.apk** | 同上 | 杀毒/恶意软件检测（扫 APK / 安装时查毒） |
| ├─ **AiSecurityPlugin.apk** | 同上 | AI 安全检测（安装时风险/行为判定 → "拦不拦"那层） |
| ├─ **WlanSecurePlugin.apk** | 同上 | Wi-Fi 安全检测 |
| **HwSecurityServer** | `priv-app/` | 安全服务（权限/认证/敏感操作） |
| **HwSecurityPrivacyCenter(+Hm)** | `priv-app/` | 安全隐私中心 |
| **HwIAware** | `app/HwIAware` | 智能行为感知（AI 调度/后台行为识别） |
| **HwBehaviorAuth** | `app/` | 行为认证 |

### 监控/诊断/上报
| 组件 | 职责 |
|---|---|
| **HwDiagnosis** | 系统诊断（可能上报） |
| **HwChrService**（app 里） | 崩溃/错误收集上报（会把堆栈传华为） |
| **HwHiAIDSEngine** | AI 服务引擎（数据语义/敏感词识别） |
| **HwDDMP** | 设备管理/策略 |
| **HwSystemServer** | 系统级常驻服务 |

### 关键 package path（实测）
- `com.huawei.systemmanager` → `/data/app/.../base.apk` + `AiSecurityPlugin.apk`（manifest 里带插件 APK）
- `com.huawei.systemserver` → `/system/priv-app/HwSystemServer/HwSystemServer.apk` + `Booster.apk`

## 三、「能不能干掉」的实测结论（我用自己权限试过）
- 我的 uid：`u0_a488`（untrusted_app 沙盒，`u:r:untrusted_app`）。**无系统权限**
- `pm disable-user --user 0 <pkg>` 对全部目标 → **`SecurityException` / `not allowed to disable this package`**
- HwSystemManager 是**平台受保护包**（`not allowed to disable`），比一般系统应用权限更高
- **结论：普通 app 身份绝对干不掉，纯命令/换词无效。要走高权限通道。**

## 四、别人（cyber AI）给的高级方案（已翻成人话，本会话给用户讲过）
> 前提：必须在「无线调试 + PC adb」或 Shizuku 接上后才有权限。用户手机目前没接。

| 方案 | 做法 | 风险/可回滚 |
|---|---|---|
| **A（建议先做）** | 不拆组件，**掐联网**：NetGuard/Rethink（本地VPN，免root）按 uid 断 Hw* 上网 + 关后台。组件活着、系统不崩、数据出不去 | 零风险，可逆 |
| **B（能成的一刀）** | `pm suspend --user 0 <pkg>` 挂起（图标变灰、不能跑、不能自启），不是 disable 所以成功率更高。回滚 `pm unsuspend` | 中，可回滚 |
| **C** | `pm uninstall -k --user 0 <pkg>` 卸用户版（有时能过 protected 查）。**必记回滚** `pm install-existing --user 0 <pkg>`。先拆插件（Antimal/AiSecurity/HwChrService），宿主最后动。先用 `pm list packages -s` 找全 | 中高，可回滚 |
| **D** | 组件级拆除：`dumpsys package <pkg> | grep -E "Service|Receiver|Provider"` 挑上报/采集组件 `pm disable-user --user 0 <pkg>/.XXXService`，逐个试、报错跳过，改坏 `pm enable` 回滚 | 中，要耐心 |
| **E** | **root**：华为 2018 停发解镸码，Mate70 只能第三方付费解锁，丢保修/变砖风险。root 后别改 /system（dm-verity+AVB 拦截），用 Magisk 模块 systemless 覆盖；或 LSPosed hook / iptables 按 uid 丢包 | 高，不推荐 |

### 副作用（会先讲清）
- 手机管家一停，**「应用启动管理」设置界面也废了**（它由管家提供）
- 部分机型**安装校验、权限弹窗、电池优化列表**受牵连
- 所以 B/C/D 都要先记好回滚命令，E 必须做成模块

### cyber AI 的排序：A 先做（10分零风险）→ B 补刀（挂起宿主）→ D 拆组件 → E 不值当

## 五、我的大白话建议（给用户讲过）
- 用户想防的八成是"数据往外传" → **A（掐联网）正好对症、零风险**，比硬拆安全一百倍。
- 硬拆容易搞坏/搞残系统，半残比不拆更烦。
- 不要再对核心 Hw* 组件硬碰（会被拒且危险）。

## 六、待办/用户回来后怎么接
1. 用户说过几天回来，报仓库名 → 我扫描这个文件即可想起。
2. 用户已说"今天太累" → 本文件就是**嘱托上传的档案**。
3. 若继续：先让用户选方案字母（A/B/C/D/E），再落地。
   - 选 A：装 NetGuard，按 uid 掐 Hw* 联网。
   - 选 B：需先接无线调试 + PC adb（用户手机当前未接）。
   - 选 D：先 dump 再生成批量 disable 命令脚本（自动跳过失败+留回滚清单）。
   - 选 E：提供 Magisk 模块目录结构。

## 七、经验教训（固化）
1. **探测系统组件：`ls /system/app /system/priv-app` 直接看**，比 ps/pm 可靠（uid 沙盒下 ps/pm 都盲）。
2. **u0_a488 沙盒碰 /system 一律 SecurityException** — 先验 `id`，输出必须是 `uid=2000(shell)` 或 `uid=0(root)` 才有下一步，否则全部白打。
3. **禁用系统组件的正确姿势是高权限通道**（wireless debug + adb / Shizuku / root），不是换 shell 命令。
4. 华为不放 `systemhelper` 名字，职能在 HwSystemManager/AiSecurity/Antimal 里。
5. **远程仓库 remote.url 内嵌 token** 会明文漏到工具输出里，处理 remote 必须 `grep -oE` 或剥 token，禁止整行打印。
6. push 到 `<repo>.git 的 token 已在本机 `<checkout>/home/.dsh/channel-pack/token-ledger.json` 有登记。