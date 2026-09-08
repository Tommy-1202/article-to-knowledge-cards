# 知识卡片: Git Rebase 与 Merge 的区别

> 来源: skills/make-knowledge-cards/test_fixtures/test_tech.md · 共 6 张卡片

### 1. Merge 的工作机制

**核心知识**: `git merge` 在两个分支的汇合点创建一个新的 "merge commit",该 commit 拥有两个父提交,从而保留双方的真实历史。

**解释**: 执行 merge 之后,提交图会呈现"两条线汇合"的形态,你能清楚看到分支在哪里分叉、谁在什么时候合入了什么。历史没有被改写,所有原始 commit 及其作者、时间戳都完整保留。

**例子 / 自测**:
- 自测: 当你在 main 分支执行 `git merge feature-x` 时,产生的新 merge commit 有几个父提交?

### 2. Merge 的取舍

**核心知识**: merge 保留真实历史但视觉上更乱,适合需要追溯"这行代码是谁、什么时候引入"的场景。

**解释**: 多人频繁合并 feature 分支时,主干上会出现大量分叉与汇合节点,看起来不如 rebase 干净。但因为历史未被改写,出现 bug 时可以通过 `git blame` / `git bisect` 精确定位责任 commit,这是它不可替代的优势。

**例子 / 自测**:
- 自测: 如果团队半年后需要定位一个线上 bug 的引入 commit,merge 历史相比 rebase 的优势是什么?

### 3. Rebase 的工作机制

**核心知识**: `git rebase` 把当前分支的所有提交"摘下来",在目标分支的最新提交之上依次重新应用,使历史呈现为一条直线。

**解释**: 应用 rebase 之后,你分支上的 commit 看起来就像是在目标分支的最新状态之后直接写的,中间的分叉在视觉上消失。代价是所有被重放的 commit 会获得新的 hash,因为它们的父提交已经改变。

**例子 / 自测**:
- 自测: 把 feature 分支 rebase 到 main 之后,feature 分支上的 commit hash 会发生变化还是保持不变?为什么?

### 4. Rebase 的铁律

**核心知识**: 永远不要对已经推送到公共共享仓库的提交执行 rebase。

**解释**: 因为 rebase 会重写 commit hash,你的本地历史会与远程历史"分叉"。当其他协作者基于旧的 commit 继续工作时,他们拉取你的新历史后会出现大量冲突和重复 commit,排查起来极其痛苦。这条规则没有例外。

**例子 / 自测**:
- 自测: 假设你把 feature 分支推到了 origin 后做了一次 rebase 再 force push,你的同事 Alice 此时正基于旧 origin 写新代码,会发生什么?

### 5. 何时用 Merge、何时用 Rebase

**核心知识**: 简单经验法则——本地未推送的 feature 分支用 rebase,已推送或多人的分支用 merge。

**解释**: rebase 适合"只属于我、还没和别人共享"的提交,可以让我的本地历史保持干净线性。merge 适合"已经公开、需要保留真实协作过程"的提交,因为它不改写历史。团队最好统一约定一种风格,避免每个人的历史长得都不一样,增加 code review 难度。

**例子 / 自测**:
- 自测: 你刚 clone 下来的本地 feature 分支还没 push 过,此时同步 main 的最新改动,你应该用 merge 还是 rebase?

### 6. Rebase + Merge 组合工作流

**核心知识**: 工业界常见的做法是先 rebase 自己的本地分支到最新的 main,再把整理过的分支 merge 进 main。

**解释**: 这个组合让 main 上的历史保持干净(只有 merge commit 形成的简单分叉),同时让你在本地开发时享受线性历史的便利。它把"保持个人历史清爽"和"保留团队协作真实过程"两个目标分阶段达成,是目前相当主流的工作流。

**例子 / 自测**:
- 自测: 这种 rebase-then-merge 工作流为什么既能保持 main 历史干净,又不会违反"不要 rebase 公共提交"的铁律?
