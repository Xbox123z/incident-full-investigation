# round-06 — GitHub 上传 SOP：这次调查仓库怎么一步步推上去的（第六角度）

> 调查角度：既然用户问过"当初怎么把 skill 一步步上传 GitHub"，这一 round 正好把**本次调查仓库的真实上传过程**逐字段记下来，
> 成为一个可复现 SOP。同时也记录为什么第一次用中文仓库名失败、以及用代理/token push 的正确姿势。

## 一句话结论
用 `gh` + 一次 **10808 本地代理 + token 内嵌 URL**，可以稳定地把本机目录推成公开仓库。
坑点：① GitHub 不接受那组中文仓库名的 `--source` push；② 直接 https 直连在 VPN 环境会 SSL 断；
③ token 需以 `git:token@` 形式作为认证。下面给完整、可复现的步骤。

---

## 一、环境
- 本机工具：`git`（/usr/bin/git）、`gh`（$HOME/gh）、`curl`。
- GitHub token：在 `~/.config/gh/hosts.yml` 的 `oauth_token`（scope: repo）。
- 网络：VPN 环境的本地代理口 **127.0.0.1:10808**（进程级出站，不劫持全局，不违反红线）。

## 二、完整 SOP（我在本次真实执行+成功）
```bash
# 1) 建立本地目录与 git
mkdir -p 调查全次数 && cd 调查全次数
git init
git config user.name "dsh-mobile"
git config user.email "local@dsh"

# 2) 用 gh 建一个【英文名】空公开仓(中文名会丢，见坑1)
GH="$HOME/gh"; export GH_TOKEN="<token>"
$GH repo create incident-full-investigation --public \
     --description "ADB/Shizuku 通道事故·调查全次数（多轮多角度）"

# 3) 把本地 remote 指到英文名仓，而非用 --source(坑1)
git remote add origin https://github.com/Xbox123z/incident-full-investigation.git

# 4) 写文件、commit
echo "# ..." > README.md
git add README.md && git commit -m "init"

# 5) 用 10808 代理 + token URL push(坑2/3)
URL="https://git:${GH_TOKEN}@github.com/Xbox123z/incident-full-investigation.git"
git -c http.proxy=http://127.0.0.1:10808 -c credential.helper= \
    push -u "$URL" master
```
> 每个后续 round：`git add round-NN.md && commit && 同一 push 命令`。

## 三、本次踩的坑（如实记录）
| 坑 | 现象 | 解法 |
|---|---|---|
| 1. 中文仓库名推不上去 | `gh repo create 调查全次数 ...` 建出来 URL 变成 `/-`；`push` 报 not found | 改用**英文**仓库名 `incident-full-investigation`；中文放在 README/描述里 |
| 2. 直连 Github 被断 | push 报 `OpenSSL SSL_read: unexpected eof` | 走 10808 本地代理（进程级 `-c http.proxy=http://127.0.0.1:10808`） |
| 3. token 认证不被识别 | 内嵌 `https://token@...` → `invalid credentials`；提示读 Username | 用 **`https://git:${GH_TOKEN}@github.com/...`** 格式（user=git, pass=token） |
| 4. gh 误建空仓 `-` | 残留 `Xbox123z/-` | 删除需 `delete_repo` scope；当前 token 无此 scope，暂留（无害） |

## 四、对"如何把 skill 上传到 GitHub"的回答
- 当初 skill(`huawei-apk-repatch-skill`)已经推在 `Xbox123z/huawei-apk-repatch-skill`。
- 上传的实质步骤与上面 SOP 同一套：**本地目录 → gh 建英文名仓 → token+代理 push**。
- skill 目录本身无 `.git`（说明当初是直接把文件 transfer 进一个 clone，或新起仓、把文件拷入再推）。
- 以后"换机重建/上传任意目录"，直接按本 SOP 跑即可，三分钟内一个公开仓。

（下一 round 继续推进：skill 上传细节、无电脑自连可复现路径、措辞护栏等等，逐角度无限推进。）