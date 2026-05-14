# Skills

这是我的个人 Skills 仓库，参考 `anthropics/skills` 的目录组织方式，统一收集可复用的 Agent Skills。

## About This Repository

仓库按以下结构组织：

- `skills/`：实际可用的 skill
- `template/`：创建新 skill 的最小模板
- `spec/`：本仓库采用的简单约定与说明

当前第一个 skill 是：

- `skills/awesome-html-note-agent`：把主题或现有内容做成交互式单文件 HTML 笔记，并按“生成 -> 巡检 -> 迭代 -> 经验沉淀”的闭环执行。

## Repository Layout

```text
.
├── README.md
├── spec/
│   └── README.md
├── template/
│   └── SKILL.md
└── skills/
    └── awesome-html-note-agent/
        └── SKILL.md
```

## Using These Skills

每个 skill 都是一个独立目录，核心文件是 `SKILL.md`。

最常见的使用方式是：

1. 把 skill 目录同步到本地 skills 目录；
2. 让 Agent 根据 `name` 和 `description` 识别触发；
3. 由 `SKILL.md` 中的 instructions 指导后续执行。

## Creating a Basic Skill

创建新 skill 时，可以直接复制 `template/SKILL.md`，并至少填写：

- `name`
- `description`

然后在正文中写清楚：

- 适用场景
- 执行步骤
- 输出格式
- 注意事项

## Notes

- 这是一个个人仓库，重点是沉淀我自己的工作流与高频能力。
- 每个 skill 优先保持“一个目录 + 一个 `SKILL.md`”的简单结构；如果需要脚本或资源，再按需扩展。
