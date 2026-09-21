---
name: upstream-first-fork-workflow
description: 管理快速演进的上游 fork：处理本地修复或功能、同步上游、删除已被上游覆盖的改动。适用于分支、rebase、差异归类与同步。
---

# 上游优先 Fork 工作流

共享行为以 `upstream/main` 为准。必须在 worktree 中工作，并遵循仓库的
`AGENTS.md`。

## 远程与分支

- `upstream/main`：腾讯原项目，只读。
- `origin/main`：fork 的纯上游镜像，只接收上游历史。
- `origin/<feature>`：fork 独有的 bug 修复或功能，rebase 到 `upstream/main`。

绝不向 `upstream` 推送。每次推送前都确认目标为 `origin`；若 `upstream` 的
push URL 未配置为 `DISABLED`，先停止并修正配置。不要把 fork 独有提交混入镜像
分支；可 rebase 时不要 merge 上游。

## 改动前

1. 分别执行 `git fetch origin` 和 `git fetch upstream`，记录目标上游提交。
2. 对照本地 diff 与上游改动是否解决同一个 bug 或需求。比较可观察行为和相关测试，
   不以文件名或提交标题作为结论。
3. 在分支 worktree 中工作，不触碰无关的本地编辑。

## 上游覆盖

当上游完整解决同一个问题时：

1. 保留上游实现和测试。
2. 删除重叠的 fork 代码、测试、文档和配置。
3. rebase 时丢弃重复提交，或 amend 为只包含上游未提供的行为。

不要因为 fork 中已经开发过，就保留重复的 fallback、替代实现或兼容开关。

## Fork 差异

仅在上游未解决需求，或验证后仍有用户可见缺口时保留 fork 改动。在提交信息中写明
该缺口，并添加最小的定向测试。

上游部分覆盖时，提交前将本地改动缩小到未覆盖行为；再与 `upstream/main` 比对。

## 同步

功能分支：

```powershell
git fetch origin
git fetch upstream
git rebase upstream/main
git diff upstream/main...HEAD
git push --force-with-lease origin <feature-branch>
```

纯上游镜像分支只推送上游提交范围。未经分支所有者同意，不要强推共享分支。

同步后报告上游提交、fork 分支、保留的 fork 文件和已删除的重叠文件。确认分支没有
未提交改动，并为保留的差异执行定向验证。
