---
title: "Git Merge Commit"
author : "tutu"
description:
date: '2025-02-26'
lastmod: '2025-02-26'
image:
math: true
hidden: false
comments: true
draft: false
categories : ['tool']
---

在 Git 中，合并多个 commit 通常使用 git rebase 或 git reset

## git rebase

- 启动交互式 rebase：

假设你想合并最近的 5 个 commit，可以运行：

git rebase -i HEAD~5

如果想要从某个commit开始-i后跟开始的commit hash

- 选择要合并的 commit：

在编辑器中，你会看到类似以下内容：

```raw
pick abc123 Commit message 1
pick def456 Commit message 2
pick ghi789 Commit message 3
pick jkl012 Commit message 4
pick mno345 Commit message 5
```

将除第一个 commit 外的其他 commit 的 pick 改为 squash（或简写 s），表示将它们合并到前一个 commit 中

- 编辑 commit 消息：

保存并关闭编辑器后，Git 会打开另一个编辑器，让你编辑合并后的 commit 消息。你可以保留或修改这些消息。

## git reset

- 重置到某个 commit：

假设你想合并最近的 5 个 commit，可以运行：

git reset --soft HEAD~5

这会将 HEAD 移动到第 5 个 commit 之前，但保留工作目录和暂存区的更改。

- 提交更改：

git commit -m "Combined commit message"