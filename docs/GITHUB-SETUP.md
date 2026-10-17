# GitHub 接入说明（本沙箱）

配置时间：本次会话
GitHub 账号：**yyl637**

## 已完成

| 项目 | 状态 |
|---|---|
| SSH 密钥 | `~/.ssh/id_ed25519`（ed25519，无口令） |
| 公钥 | 已添加到 GitHub → Settings → SSH keys |
| 主机指纹 | `~/.ssh/known_hosts` 已写入 github.com 的 rsa/ecdsa/ed25519 公钥 |
| 认证验证 | `ssh -T git@github.com` → `Hi yyl637! You've successfully authenticated` |
| git 身份 | `yyl637 <yyl637@users.noreply.github.com>` |
| 默认分支 | `main` |
| URL 重写 | `https://github.com/` → `git@github.com:`（贴 https 地址也会走 SSH） |
| 读路径验证 | 已成功 clone `octocat/Hello-World` |
| 写路径验证 | ✅ 已向 `yyl637/dsh` 推送 commit `a5acaae`，服务端 ref 一致 |

## ⚠️ 已知环境坑：git 命令必须加 `GIT_CONFIG_COUNT=0`

沙箱给每个 shell 注入了三个环境变量：

```
GIT_CONFIG_COUNT=2
GIT_CONFIG_VALUE_0=git@github.com:
GIT_CONFIG_VALUE_1=ssh://git@github.com/
```

但配套的 `GIT_CONFIG_KEY_0/1` 缺失。Git 见到 `COUNT>0` 就去读对应 key，读不到直接：

```
error: missing config key GIT_CONFIG_KEY_0
fatal: unable to parse command-line config
```

**任何** git 命令都会因此失败（`git init`、`git status`、`git clone` 全挂），与本仓库无关。

### 解法

每条 git 命令前加 `GIT_CONFIG_COUNT=0`：

```bash
GIT_CONFIG_COUNT=0 git clone git@github.com:yyl637/<repo>.git
GIT_CONFIG_COUNT=0 git push
```

或等价的：

```bash
env -u GIT_CONFIG_COUNT -u GIT_CONFIG_VALUE_0 -u GIT_CONFIG_VALUE_1 git <子命令>
```

这个注入来自沙箱运行时而非 DSH 源码（`dsh-web-app` 里搜不到 `GIT_CONFIG_COUNT`），所以无法在仓库层面修掉，只能每次命令前缀规避。

## 常用命令

```bash
# 克隆
GIT_CONFIG_COUNT=0 git clone git@github.com:yyl637/<repo>.git

# 提交
GIT_CONFIG_COUNT=0 git add -A
GIT_CONFIG_COUNT=0 git commit -m "message"

# 推送
GIT_CONFIG_COUNT=0 git push            # 首次： GIT_CONFIG_COUNT=0 git push -u origin main

# 验证连通性
ssh -T git@github.com
```

## 产出总仓：yyl637/dsh

工作目录 `/home/agentuser/liubai world` 已初始化为 git 仓库，origin 指向 `git@github.com:yyl637/dsh.git`，跟踪 `main`。

**之后任何任务产出的 md 和代码，直接在根目录 `git add -A` 就会进这个仓库**，不用再 clone 别的目录。

目录约定：

```
.
├── docs/           # 文档产出
├── projects/       # 代码产出，每个任务一个子目录
└── _trash/         # 临时与测试残留，被 .gitignore 排除
```

历史里 `a5acaae`（连通性测试）和 `b3f1203`（delivery-test 测试包）是最初的验证提交，内容已由 `d0ebab4` 清理，仅留作历史。

## 撤销接入


1. GitHub → https://github.com/settings/keys → 删除标题为 `dsh-agent@liubai-world` 的 key
2. 沙箱内：`rm -rf ~/.ssh/id_ed25519 ~/.ssh/id_ed25519.pub`

删除 key 立即生效，无需重启任何东西。

## 安全边界

- 私钥仅在沙箱内，不外传；用户侧只有公钥。
- 权限边界＝该 GitHub 账号自身的仓库权限，没有额外授权。
- 任何 `git push` 都以 `yyl637` 身份进行，建议先用测试仓库验证。
