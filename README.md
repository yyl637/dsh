# dsh

DSH agent 的产出总仓。由 agent 在会话中生成的文档与代码统一汇总到这里。

## 目录结构

```
.
├── docs/           # 文档产出（md / 报告 / 说明）
├── projects/       # 代码产出，每个任务一个子目录
│   └── <任务名>/
└── _trash/         # 临时与测试残留，被 .gitignore 排除，不入库
```

## 约定

- 新任务在 `projects/` 下建独立子目录，避免不同任务的依赖互相污染。
- 提交信息用 `<类型>: <说明>`，类型取 `feat` / `fix` / `docs` / `chore` / `test`。
- 大文件（>50MB）不要进这个仓库，GitHub 单文件硬上限 100MB。
- 任何密钥、token、`.env` 一律不入库，见 `.gitignore`。

## 接入方式

本仓库通过 SSH（ed25519 密钥）推送，账号 `yyl637`。配置细节见 `docs/GITHUB-SETUP.md`。

## 注意

本沙箱注入了残缺的 `GIT_CONFIG_*` 环境变量，所有 git 命令需加前缀：

```bash
GIT_CONFIG_COUNT=0 git status
```
