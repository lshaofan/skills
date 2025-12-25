# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

这是一个 Claude Code 技能包（Skills）集合仓库，用于存放和管理各类开发辅助技能。

## 目录结构

```
skills/
├── skills/           # 技能包存放目录
│   └── <skill-name>/ # 单个技能包
│       ├── skill.md  # 技能定义（必需）
│       └── README.md # 技能说明
├── templates/        # 技能开发模板
└── docs/             # 项目文档
```

## 技能包开发

### 创建新技能

1. 在 `skills/` 目录下创建技能目录（使用 kebab-case 命名）
2. 创建 `skill.md` 定义技能逻辑
3. 创建 `README.md` 说明技能用途

### skill.md 格式

```markdown
---
name: 技能名称
description: 技能描述
triggers:
  - 触发条件
---

# 技能执行逻辑

具体的提示词和执行步骤...
```

## 编码规范

- 技能包目录名：kebab-case（如 `code-review`）
- 文件名：小写
- 文档语言：中文
