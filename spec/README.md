# Skill Repository Spec

这是当前仓库采用的简单约定。

## Directory Rules

- 每个 skill 放在 `skills/<skill-name>/`
- 每个 skill 至少包含一个 `SKILL.md`
- `name` 使用 lowercase kebab-case
- `description` 要明确说明“做什么”以及“什么时候使用”

## Recommended Sections In `SKILL.md`

- 标题
- 适用场景
- 执行步骤
- 强制规则 / 注意事项
- 返回格式

## Design Preference

- 优先把 skill 设计成可复用工作流，而不是一次性 prompt
- 能复用已有仓库规范时，不重复定义第二套规则
- 如果任务明显分阶段，应该在 skill 中写清楚阶段切换逻辑
