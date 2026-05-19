---
name: skill-test-lab
description: 为一个已有 skill 搭建 /tmp 最小测试仓库，使用 coco 非交互或续跑方式真实触发该 skill，抓取 stream-json 日志验证 skill 是否加载、流程是否正确、是否存在 worktree/cwd/验证/交互问题，并输出结论与证据路径。
metadata:
  author: cody
  version: "1.0.0"
  requires:
    bins: ["coco", "git", "python3"]
---

# Skill Test Lab

你负责把“测试一个 skill 是否真的可用”这件事标准化执行，而不是只读 `SKILL.md` 做静态分析。

你的目标是：

1. 搭出一个最小但真实可运行的测试环境
2. 通过 `coco` 真正触发目标 skill
3. 抓取证据，确认 skill 是否被加载、是否完成预期动作
4. 如果发现问题，定位到具体步骤与日志证据
5. 修正后再次复测，直到结果清晰

## 适用场景

当用户表达以下任意意图时使用本 Skill：

- “帮我测试这个 skill”
- “验证这个 skill 能不能真的跑通”
- “给这个 skill 做一次真实演练”
- “不要只看提示词，直接跑起来试试”
- “检查这个 skill 在 coco 里是否真的会被触发”

## 核心原则

- **必须实测，不做纸面验证。**
- **优先在 `/tmp` 创建最小仓库**，避免污染真实项目。
- **优先使用 `coco -p` 或 `traecli -p` 做非交互测试**，因为这更容易自动化、抓日志、复现实验。
- **如果 skill 需要二次确认**（例如“是否合并”“是否发布”），要使用 `--resume` 继续同一 session 模拟用户选择。
- **必须保留证据路径**：临时仓库、日志文件、关键输出。

## 输入理解

如果用户只给了一个 skill 名，没有给测试任务内容，你要自动补齐一个最小可验证任务。

### 默认输入项

- 目标 skill 名：从用户输入中识别，例如 `/worktree-safe-merge`
- 测试仓库路径：默认 `/tmp/<skill-name>-repo` 或更具体名字
- 日志路径：默认 `/tmp/<skill-name>-stream.jsonl`
- 测试任务：用最小任务验证该 skill 的核心承诺

## 标准流程

### Step 1：先读目标 skill

优先读取：

```bash
/Users/bytedance/.trae/skills/<skill-name>/SKILL.md
```

先提取以下信息：

- skill 的承诺能力是什么
- skill 需要什么类型的项目/目录/文件
- skill 是否依赖交互式追问
- skill 是否要求测试、构建、提交、merge、部署等动作

### Step 2：设计最小可验证场景

按“够小、够真、够能暴露问题”的标准造数据。

示例：

- 如果 skill 是代码修改类：创建最小可运行仓库 + 一个明确会失败的测试
- 如果 skill 是 worktree 类：创建最小 git 仓库并给出一条具体修改任务
- 如果 skill 是部署类：准备一个最小 HTML 或最小应用

不要一开始就用复杂真实仓库，除非用户明确要求。

### Step 3：在 `/tmp` 创建测试仓库

优先创建一个最小仓库，包括：

- `AGENTS.md`（如果需要给 skill 额外约束）
- 最小源文件
- 最小测试文件
- `package.json` / 其他运行配置

然后：

```bash
git init -b main
git add .
git commit -m "Create test repository baseline"
```

### Step 4：非交互触发 skill

优先用：

```bash
coco -p "/<skill-name> <task>" -y --query-timeout 10m --bash-tool-timeout 5m
```

如果需要抓完整行为证据，优先用：

```bash
coco -p "/<skill-name> <task>" -y --output-format stream-json > "/tmp/<skill-name>-stream.jsonl"
```

### Step 5：确认 skill 是否真的被加载

不要只看最终结果文本。必须在 stream-json 或会话日志中确认以下任一证据：

1. 出现：

```text
<command-name>/<skill-name></command-name>
```

2. 或者出现：

```text
Base directory for this skill: ...
```

3. 或者在上下文中看到完整 skill 内容被注入

如果没有这些证据，就不要轻易宣称“skill 已触发成功”。

### Step 6：检查 skill 是否完成核心动作

按该 skill 的承诺逐项验证，例如：

- 是否真的创建了 worktree
- 是否真的改了目标文件
- 是否真的运行了测试 / build
- 是否真的做了 merge 演练
- 是否真的停在“等待用户确认”的位置，而不是越权继续

验证时必须结合：

- 文件内容
- git 状态
- worktree 列表
- 命令输出
- stream-json 日志证据

### Step 7：如果 skill 需要后续选择，继续续跑会话

如果 skill 最终要求用户选择 `1 / 2 / 3` 一类选项，则继续：

```bash
coco -p "1" --resume -y
```

或者用户要求的其他输入。

这样可以验证“确认后的后半段流程”是否也正确。

### Step 8：发现问题时必须记录“问题 → 证据 → 修正 → 复测”闭环

报告里至少包含：

- 问题是什么
- 哪一轮测试发现的
- 证据路径是什么
- 修正了什么
- 修正后哪一轮复测通过

### Step 9：重点关注这些高频问题

#### 1. skill 被误识别但未真正加载

- 只有文本回答，没有 skill 注入证据

#### 2. 命令在错误 cwd 执行

- 例如改的是 worktree 文件，但 `npm test` 在原始仓库跑了

#### 3. 只做了修改，没有真实验证

- 没有实际执行测试命令，只口头说“应该可以”

#### 4. 交互/非交互行为不一致

- 在 `coco -p` 下无法弹出 `AskUserQuestion`
- 此时应退化为文本选项等待后续输入，而不是自动执行危险操作

#### 5. 临时资源未清理或主仓库被污染

- merge-check worktree 未删
- 主仓库被直接修改

### Step 10：最终汇报模板

最终用简洁中文输出：

- 测试的 skill 名
- 测试仓库路径
- 日志路径
- skill 是否真的被加载
- 核心流程是否跑通
- 发现的问题
- 修正内容
- 最终是否通过
- 如有必要，给出关键证据文件路径

## 质量标准

- 必须至少有一轮真实触发
- 如果第一次失败，优先修正后再复测
- 至少保留一个日志文件作为证据
- 最终结论必须基于实际运行结果，而不是主观判断

## 特别注意

- 不要只阅读 skill 然后给“理论上可用”的结论
- 不要跳过日志验证
- 不要在需要续跑时直接结束测试
- 不要省略对副作用的检查（如 git 状态、worktree 污染、自动 merge）

如果用户没有提供测试任务内容，就自动选择一个最小但足以覆盖 skill 核心承诺的任务继续执行。
