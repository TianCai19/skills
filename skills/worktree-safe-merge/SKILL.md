---
name: worktree-safe-merge
description: 为任意开发任务创建 git worktree，在独立 worktree 中完成实现、验证是否可安全合并回原始分支，并在真正合并前询问用户是否执行合并。
metadata:
  author: cody
  version: "1.0.0"
  requires:
    bins: ["git"]
---

# Worktree Safe Merge Skill

你负责执行一套通用的 **Git worktree 开发 + 安全回合并** 工作流：

1. 找到目标仓库与原始分支
2. 创建独立 worktree 和功能分支
3. 在 worktree 中完成用户要求的工作
4. 实际运行验证命令
5. 非破坏性判断是否可以安全合并回原始分支
6. **必须先询问用户是否合并**，只有在用户明确同意后才执行真正的 merge

这个 Skill 适合“先开 worktree 做事，完成后再决定是否合并回原分支”的任务，而不是直接在当前分支裸改。

## 触发场景

当用户表达以下任意意图时使用本 Skill：

- “开一个 worktree 做这个任务”
- “不要污染当前分支，单独开分支改”
- “帮我先在 worktree 里完成，再看看能不能合回去”
- “做完后判断能不能安全 merge 回原来的分支”
- “创建 worktree 完成修改，然后问我要不要合并”

## 目标与边界

### 目标

- 不直接在原始工作目录里做改动
- 通过 worktree 隔离开发任务
- 完成后给出明确的“是否可安全合并”结论
- 合并前必须征求用户决定

### 边界

- 默认 **不 push、不创建 PR**，除非用户额外要求
- 默认 **不删除原始 worktree、不删除 feature branch、不清理 worktree**，除非用户额外要求
- 默认 **不自动合并**；只有在用户明确确认后才 merge

## 输入推断规则

如果用户没有把参数说全，按下面规则自动补齐：

### 1. 仓库路径

- 如果当前工作目录就在 Git 仓库内，优先使用：

```bash
git rev-parse --show-toplevel
```

- 如果当前目录不在 Git 仓库内，再询问用户仓库路径

### 2. 原始分支（目标分支）

- 默认使用用户发起任务时所在仓库的当前分支：

```bash
git -C "$REPO" branch --show-current
```

- 这个分支就是后续要判断是否能安全 merge 回去的 **原始分支 / 目标分支**

### 3. feature 分支名

- 根据任务自动生成简洁的 kebab-case slug
- 推荐格式：

```text
wt/<task-slug>-<YYYYMMDD-HHMMSS>
```

- 例如：`wt/fix-login-timeout-20260519-153000`

### 4. worktree 路径

- 默认使用仓库父目录下的集中 worktree 目录：

```text
<repo-parent>/.worktrees/<repo-name>/<feature-branch-sanitized>
```

- 例如：

```text
/Users/me/codes/.worktrees/my-app/wt-fix-login-timeout-20260519-153000
```

## 强制执行规则

### 1. 永远不要直接在原始工作目录中实现需求

- 原始工作目录只用于读取状态、记录原分支、做最终 merge（如果用户同意）
- 实际开发、测试、修复都必须在新建的 worktree 中完成

### 2. 在开始改代码前先读取仓库规范

进入目标仓库后，优先检查并读取以下文件（如果存在）：

- 仓库根目录 `AGENTS.md`
- 仓库根目录 `README*`
- 与当前任务直接相关的配置或开发说明文件

如果仓库里还有更具体的子目录规范文件，也要优先遵守。

### 3. 必须做真实验证

- 不能只修改代码不验证
- 必须根据仓库实际情况运行合适的测试 / lint / build / typecheck / smoke test
- 如果某个验证失败，要继续排查并尽量修复，而不是直接停止在第一次失败
- **不要依赖当前 shell 的工作目录碰巧正确。** 只要命令针对 feature worktree 或 merge-check worktree 执行，就必须显式使用 `git -C "$WORKTREE_PATH" ...`、`npm --prefix "$WORKTREE_PATH" ...`，或者 `cd "$WORKTREE_PATH" && ...` 这类写法，避免命令误在原始工作目录中运行。

### 4. 合并安全性判断必须是“非破坏性”的

- 不能直接在原始分支上盲目 merge 试试
- 必须先做一次 **临时验证 worktree / 非破坏性 merge 演练**
- 只有确认无冲突、关键验证可通过，才可判定为“可安全合并”

### 5. 合并前必须询问用户

无论安全检查结果如何，**都不能跳过用户确认直接 merge**。

## 标准执行流程

### Step 1：识别仓库、原始分支、基线提交

先获取以下信息：

```bash
REPO=$(git rev-parse --show-toplevel)
TARGET_BRANCH=$(git -C "$REPO" branch --show-current)
BASE_COMMIT=$(git -C "$REPO" rev-parse HEAD)
```

然后同步远端信息：

```bash
git -C "$REPO" fetch --all --prune
```

记录：

- 仓库根目录
- 原始分支名
- 创建 worktree 时对应的基线提交

### Step 2：创建 feature branch 与 worktree

在不污染当前工作目录的前提下创建新分支与 worktree：

```bash
git -C "$REPO" worktree add -b "$FEATURE_BRANCH" "$WORKTREE_PATH" "$TARGET_BRANCH"
```

要求：

- 分支名不能与现有分支冲突
- worktree 路径不能与现有目录冲突
- 如果冲突，自动加时间戳或递增后缀再重试

### Step 3：在 worktree 中理解任务并实施修改

在 `WORKTREE_PATH` 中：

1. 读取仓库规范文件
2. 搜索与任务相关的代码
3. 使用 Todo 工具跟踪复杂任务
4. 完成实现、修复或重构
5. 只做与任务直接相关的改动

如果用户需求略有歧义，但整体方向明确，默认主动完成，而不是停在分析阶段。

### Step 4：运行验证命令

必须根据项目类型选择最合适的验证命令，常见包括：

- `npm test`
- `npm run build`
- `npm run lint`
- `pnpm test`
- `pytest`
- `go test ./...`
- `cargo test`

原则：

- 至少运行与本次修改最相关的验证
- 如果项目有 build，优先补充一次 build
- 如果涉及 UI、交互、页面流程，优先考虑浏览器实测或自动化验证
- 对于依赖当前目录的命令（如 `npm test`、`pnpm test`、`pytest` 中的相对导入场景），必须显式在 `WORKTREE_PATH` 下执行，不要默认沿用主会话 cwd

示例：

```bash
cd "$WORKTREE_PATH" && npm test
```

或：

```bash
npm --prefix "$WORKTREE_PATH" test
```

### Step 5：整理改动并形成 feature branch 成果

完成开发后，检查：

```bash
git -C "$WORKTREE_PATH" status --short
git -C "$WORKTREE_PATH" diff --stat
```

如果需要提交，遵循仓库提交规范；若用户未禁止提交，且后续要进行 merge 判断，通常应先把 worktree 中的修改整理为 feature branch 上的提交。

提交时优先使用安全格式：

```bash
git -C "$WORKTREE_PATH" commit -m "$(cat <<'EOF'
Implement <task summary>

Briefly explain why this change is needed.
EOF
)"
```

### Step 6：非破坏性判断是否可安全合并回原始分支

这一环节必须独立执行，不能直接在原始工作目录上试错。

#### 6.1 创建临时 merge 验证 worktree

基于 **当前最新的目标分支 tip** 创建一个临时 worktree。由于目标分支通常已经在原始工作目录中检出，这里优先使用 `--detach`，避免因为“同一分支已在其他 worktree 中检出”而失败。例如：

```bash
git -C "$REPO" fetch --all --prune
TARGET_TIP=$(git -C "$REPO" rev-parse "$TARGET_BRANCH")
git -C "$REPO" worktree add --detach "$MERGE_CHECK_PATH" "$TARGET_TIP"
```

#### 6.2 在临时 worktree 中做 merge 演练

```bash
git -C "$MERGE_CHECK_PATH" merge --no-commit --no-ff "$FEATURE_BRANCH"
```

判断规则：

- 如果出现冲突：**不能安全合并**
- 如果 merge 可成功进入 staged 状态：继续做验证

#### 6.3 在模拟 merge 结果上再运行关键验证

在 `MERGE_CHECK_PATH` 中，对合并后的结果至少运行最关键的验证命令。

同样要求：验证命令必须显式在 `MERGE_CHECK_PATH` 中执行，不能误用原始工作目录。

判定建议：

- **安全可合并**：无 merge conflict，且关键验证通过
- **暂不安全**：有冲突，或合并后关键验证失败

#### 6.4 清理临时 merge 演练状态

无论成功还是失败，都要清理验证现场：

```bash
git -C "$MERGE_CHECK_PATH" merge --abort
git -C "$REPO" worktree remove "$MERGE_CHECK_PATH"
```

如果 merge 已成功但 `merge --abort` 不再适用，则使用 `git reset --hard HEAD` 恢复，再移除临时 worktree。

### Step 7：把安全评估结论明确告诉用户

至少说明以下内容：

- 原始分支 / 目标分支名
- feature branch 名
- worktree 路径
- 是否检测到 merge conflict
- 模拟 merge 后的验证命令与结果
- 结论是“可安全合并”还是“暂不安全”

### Step 8：必须询问用户是否执行真实合并

安全检查完成后，使用 `AskUserQuestion` 询问用户。

如果当前运行环境是 **非交互模式**（例如 `coco -p` / `traecli -p`），无法弹出交互式提问框，则降级为：

- 在最终回复里明确给出编号选项
- 等待用户在后续消息中回复 `1` / `2` / `3` 或等价文本
- 不要因为无法弹出 `AskUserQuestion` 就自动 merge

#### 如果结论为“可安全合并”

优先给出如下选项：

1. `立即合并回原分支（Recommended）`
2. `先保留 feature branch，不合并`
3. `仅查看结果，稍后再决定`

#### 如果结论为“暂不安全”

优先给出如下选项：

1. `先不合并，保留当前结果（Recommended）`
2. `继续处理冲突或失败项`
3. `仅查看结果，稍后再决定`

### Step 9：只有在用户确认后才执行真实 merge

如果用户明确要求合并：

1. 回到目标分支对应仓库
2. 确认目标分支仍然是预期分支
3. 确认原始工作目录是干净的；如果不干净，不要强行 merge
4. 再次同步远端信息（如有必要）
5. 执行真实 merge
6. 对真实 merge 结果再跑一次最关键验证
7. 报告 merge 完成状态

优先策略：

- 能 fast-forward 就优先 fast-forward
- 不能 fast-forward 时再使用普通 merge

示例：

```bash
git -C "$REPO" status --short
git -C "$REPO" checkout "$TARGET_BRANCH"
git -C "$REPO" merge --ff-only "$FEATURE_BRANCH"
```

如果不能 fast-forward，再考虑：

```bash
git -C "$REPO" merge --no-ff "$FEATURE_BRANCH"
```

如果原始工作目录不干净，或当前并不适合直接在原目录执行 merge，需要先向用户说明阻塞点，再决定是否改为用户指定的干净工作区处理；不要默默绕过这个前置条件。

### Step 10：结果汇报

最终至少汇报：

- 仓库路径
- 原始分支名
- feature branch 名
- worktree 绝对路径
- 主要修改文件
- 已执行的验证命令与结果
- 安全合并判断结果
- 是否已询问用户
- 如果用户同意合并：补充实际 merge 结果

## 决策细则

### 什么叫“安全可合并”

至少满足以下条件：

1. feature branch 改动已经完整落盘
2. 模拟 merge 没有冲突
3. 模拟 merge 后关键验证通过

如果其中任一项不满足，就不要宣称“可安全合并”。

### 什么情况下需要继续修复，而不是立刻询问合并

如果发现以下情况，先修再进入询问环节：

- feature worktree 中还有明显未完成实现
- 相关测试或构建还没跑
- 已知错误是本次修改引入且可以继续修

只有在“实现完成 + 已验证 + 已做 merge 演练”后，再询问是否合并。

## 重要注意事项

- 不要假设用户一定想 push 或提 PR
- 不要因为“看起来改动不大”就省略 merge 演练
- 不要在原始工作目录直接改文件来省事
- 不要在未询问用户前直接 merge
- 不要在结论不明确时说“应该没问题”；要给出基于实际演练的判断
- 如果真实 merge 失败或分支状态变化，要明确说明失败点和下一步建议

## 推荐输出模板

完成安全评估后，优先用简洁中文汇报：

```text
已在独立 worktree 中完成本次任务。

- 仓库：<repo>
- 原始分支：<target-branch>
- feature 分支：<feature-branch>
- worktree：<worktree-path>
- 主要修改：<files>
- 验证：<commands and results>
- merge 演练：<conflict-free / conflicts / validation failed>
- 结论：<可安全合并 / 暂不安全>

是否现在合并回 <target-branch>？
```

如果用户同意，再继续执行真实 merge。
