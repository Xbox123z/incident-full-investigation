# round-30 — 修正 10-28 事故：我其实写了恢复(:0)+校验，断网真因更可能是"恢复不干净"（第三十角度）

> 调查角度：一直引用 AGENTS.md 说"10-28 收尾时没恢复(被打断)"。这轮用**命令逐字原文**核对，发现这个说法需要修正：
> 那天我其实在新命令末尾执行了 `settings put global http_proxy :0` 并做了 `echo proxy_reverted=$(settings get...)` 校验。
> 断网更可能的根因不是"忘了恢复"，而是"恢复方式(:0)不够干净/持久残留"。

## 一、一手命令原文(10-28 19:43:43，955字符命令的骨干)
```bash
settings put global http_proxy 127.0.0.1:10808      # ① 设代理指向 v2ray
...操作...
settings put global http_proxy :0                    # ② 恢复：代理指空
echo proxy_reverted=$(settings get global http_proxy) # ③ 校验并打印
```
- 紧接着 tool/result(19:44:07) 打印：`proxy_reverted=:0`
- **证明：我在同一条命令里做了 try/finally——设了代理、末尾恢复(:0)、又读回校验。并未"没写恢复"。**

## 二、与 AGENTS.md 描述的差异
| 项 | AGENTS.md 原述 | 命令逐字证据(本轮) |
|---|---|---|
| 是否写恢复 | "收尾时没恢复(被打断)" | 写了 `settings put global http_proxy :0` + `echo proxy_reverted=` |
| 恢复后值 | (未提及) | `proxy_reverted=:0` |
| 断网因果 | 直指"没恢复→重启残留" | 可能兼有"恢复用 :0 不干净" |

- **所以"没恢复"应更正为"恢复了，但恢复方式(:0)可能不够干净"**。这不是要洗掉过错，而是把过错精确到"恢复细节"这一层，因为这对"以后怎么改"才是真指导。

## 三、断网真因的最可能解释(在 JSONL 可证范围内)
1. `settings put global http_proxy :0` 把代理设为字符串 `:0`——**Android 未必把它当"清除代理"**，可能仍视为一个"不可达的代理"，导致 `ERR_PROXY_CONNECTION_FAILED`。
2. 即使命令里恢复，这些测试里 morphllm 代理本身(p10808)不通(前面测到 429/000) → 恢复成 `:0` 后全局断网仍持续，直到某次用户重启/真正清除才恢复。
3. **教训的关键不是"有没有写恢复"，而是"恢复必须用『彻底清除』且带功能验证"**：
   - 恢复：`settings delete global http_proxy`（彻底删键，而非设 `:0`）
   - 校验：`settings get global http_proxy` 为空 + `curl --noproxy '*'` 直连 200（功能级），两者都要。

## 四、沉淀(进 AGENTS 红线 §一·3 & skill §0B)
- 修订红线的"没恢复"为更精确："恢复必须 `settings delete global http_proxy`（不是设`:0`），并以 `settings get`(空) + `curl --noproxy '*'` 直连 200 双重校验。把 http_proxy 设为 `:0` 不算干净恢复(round-30)。"

## 五、这轮价值
- 用**命令逐字**更正了对 10-28 事故的流行描述，把教训从"要记得写恢复"(其实写了)推进到"恢复要彻底+功能验证"(当时没做透)。
- 也更公平：不是"完全没收拾"，而是"收拾得不够干净"。这对未来真正有用——知道真正的坑在"恢复细节"而非"有无恢复意愿"。

（下一 round 继续。不停。）