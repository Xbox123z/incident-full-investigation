# round-07 — skill 错误前提的诞生、幸存，与这次"真正同步修正"(第七角度)

> 调查角度：追查"USB 调试一开 adbd 就在 5555 监听"这条错误前提**从哪写进 skill、为什么能在 10-28 实测后仍幸存**，
> 以及今天如何通过真实 clone+push 把修正同步到 GitHub（让**其他 AI** 不再读旧版）。

## 一句话结论
错误前提写在 skill 的 SKILL.md 里，10-28 我已实测反证（`service.adb.tcp.port` 空、5555 closed、setprop 被拒），
但**没有回头把 skill 里那句改掉**——错误前提幸存到今天。直到本次调查，我才真正 clone skill 仓、覆盖旧文件、push 到 `main`，
让 GitHub 上 0A/0B/0C 修正版替换掉旧版的错误前提。

---

## 一、错误前提的生命史
| 阶段 | 内容 | skill 里有没有 |
|---|---|---|
| 写入时 | "USB 调试开启后 adbd 监听 127.0.0.1:5555" | ✅（旧 SKILL） |
| 10-28 实测 | `service.adb.tcp.port` 空 / 5555 closed / setprop 被拒 | ❌ 没改 skill |
| 10-31~11-01 | 我在凌晨/上午据此乐观预期"无电脑自连可靠" | ✅ 仍幸存 |
| 11-01 上午 | 用一次成功过度外推"Shizuku 持久" | ✅ 仍幸存，酿成事故 |
| 本次调查 | 修正+clone+push | ✅ 已同步到 GitHub main |

## 二、为什么"实测了却没人改 skill"——机制
- 教训：**skill 是给人/其他 AI 读的长期文档，不是"当时跑一下就算"。**
- 当天我"实测反证"后，修复动作都花在"当前对话怎么处理手机"，**忘了把反证回写进 skill**。
- 于是下次会话又读到旧错误前提 → 再次乐观启动。这是"事故反复"的结构性根因之一：
  **「实测得到的真知识没有被固化到 skill」**。

## 三、本次真正做了什么（可复核）
1. **clone** skill 仓到本地：`git clone https://github.com/Xbox123z/huawei-apk-repatch-skill.git`
2. 把本地修正版覆盖 SKILL.md / README.md / scripts/adb-local/bootstrap_adb.sh（SAFETY-RULES 本就一致）
3. 确认 SKILL 现含：`### 0A 先实测别下结论`、`### 0B 汇报纪律`、`### 0C 通道持久性真相`
4. commit：`fix(0A/0B/0C)...`（3 files, +102/-18）
5. push 到 **main**（skill 仓默认分支是 main，不是 master）→ `1941bb7..9b7c846 main -> main`

## 四、为什么必须同步到 GitHub，而不只是本地
- GitHub 仓是"其他 AI / 用户 / 未来会话"读取 skill 的公开真相源。
- 本地改了而 GitHub 还是旧版 = 换窗口/别处仍会读到错误前提，事故会重演。
- 本次用真实 clone+push 验证了"改 skill 一定能同步上去"，纳入round-06 的 SOP 中。

## 五、固化下来的机制（防未来再犯"改了不用传"）
- 任何对 skill 的修正，**同一轮内必须 push 到 GitHub**，并验证远程确实更新（可 `curl raw` grep 关键句）。
- 修正结论应回写进 skill 的「教训/0 系列」节，而非只存在于对话记忆。

（下一 round：继续"措辞护栏落地进 skill / 换机重建完整指南 / 心路反思"等多角度无限推进。）