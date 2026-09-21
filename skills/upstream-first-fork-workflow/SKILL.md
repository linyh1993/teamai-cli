---
name: upstream-first-fork-workflow
description: 管理快速演进的上游 fork：处理本地修复或功能、同步上游、删除已被上游覆盖的改动。适用于分支、rebase、差异归类与同步。
---

# 上游优先 Fork 工作流

共享行为以 `upstream/main` 为准，并遵循仓库的 `AGENTS.md`。

## 远程与分支

- `upstream/main`：腾讯原项目，只读。
- `origin/main`：fork 的纯上游镜像，只接收上游历史。
- `origin/main_v2`：fork 的长期集成主分支。它接收已同步的 `origin/main` 与已完成的
  fork 功能；日常开发以它为基础。
- `origin/<feature>`：从 `origin/main_v2` 创建的 fork 功能分支。它不会自动同步到
  `origin/main_v2`，只在完成、验证和确认后合并回去。

绝不向 `upstream` 推送。每次推送前都确认目标为 `origin`；若 `upstream` 的
push URL 未配置为 `DISABLED`，先停止并修正配置。不要把 fork 独有提交混入
`origin/main`。

## 改动前

1. 分别执行 `git fetch origin` 和 `git fetch upstream`，记录目标上游提交。
2. 对照本地 diff 与上游改动是否解决同一个 bug 或需求。比较可观察行为和相关测试，
   不以文件名或提交标题作为结论。
3. 不触碰无关的本地编辑。

## 上游覆盖

当上游完整解决同一个问题时：

1. 保留上游实现和测试。
2. 删除重叠的 fork 代码、测试、文档和配置。
3. rebase 时丢弃重复提交，或 amend 为只包含上游未提供的行为。

不要因为 fork 中已经开发过，就保留重复的 fallback、替代实现或兼容开关。

## 行为拆分

一个本地提交可能同时包含多个需求。删除前先逐项列出“本地行为、上游证据、覆盖结论、
处理动作”：

| 本地行为 | 上游证据 | 结论 | 动作 |
| --- | --- | --- | --- |
| Windows CLI 可发现且可启动 | 上游 diff 与 Windows 测试 | 已覆盖 | 删除 |
| 非 Git 目录允许 Codex 执行 | 上游无对应行为 | 未覆盖 | 仅在仍有需求时保留 |

只能删除已覆盖的 hunk。不要因为同一提交或同一文件的其他行为已被上游覆盖，就删除
独立的需求；也不要把独立需求伪装成已覆盖问题的一部分继续保留。

## Fork 差异

仅在上游未解决需求，或验证后仍有用户可见缺口时保留 fork 改动。在提交信息中写明
该缺口，并添加最小的定向测试。

上游部分覆盖时，提交前将本地改动缩小到未覆盖行为；再与 `upstream/main` 比对。

## 上游同步

先将腾讯上游同步到 fork 镜像，再合并到集成主分支：

```powershell
git fetch origin
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main

git switch main_v2
git merge origin/main
git push origin main_v2
```

`origin/main` 只接受上游历史。`main_v2` 是共享的长期分支，优先使用 merge 吸收
`origin/main`，不要为了线性历史强推它。

## 功能分支

```powershell
git switch main_v2
git pull --ff-only origin main_v2
git switch -c codex/<feature>

# 开发、测试、提交
git push -u origin codex/<feature>
```

功能完成后，先检查上游是否已覆盖同一需求；未覆盖的差异经过验证和确认后，才合并到
`main_v2`。功能分支绝不合并或推送到 `origin/main`。

同步或合并后报告上游提交、`main_v2`、功能分支、保留的 fork 文件和已删除的重叠
文件。确认分支没有未提交改动，并为保留的差异执行定向验证。
