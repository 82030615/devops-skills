# AGENTS.md

This file provides guidance to CodeBuddy Code when working with code in this repository.

## 项目概述

这是一个**技能集合仓库**（devops-skills），用于存放 AI 编程助手的技能定义。每个技能是一个独立的目录，包含完整的技能定义、评估测试和参考示例。

## 当前技能列表

| 技能目录 | 技能名称 | 描述 | 触发词 |
|---------|---------|------|--------|
| `write-ears-prd/` | write-ears-prd | 基于 EARS 规范编写结构化 PRD | 编写PRD、需求文档、EARS规范 |
| `ast-grep-4j/` | ast-grep-4j | 使用 ast-grep 分析 Java 代码，支持方法查找、调用链分析、类依赖分析 | 分析Java、查找方法、搜索代码、代码分析、ast-grep Java |
| `biz-analysis-4j/` | biz-analysis-4j | 深度分析 Java 项目业务功能，生成包含类图、时序图、状态图的 Markdown 报告 | 业务分析、分析业务、Java业务分析、分析实体、分析表、业务逻辑分析 |

## 技能目录结构约定

每个技能目录应遵循以下结构：

```
skill-name/
├── SKILL.md              # 技能核心定义文档（AI 使用）
├── README.md             # 技能说明文档（人类阅读）
├── evals/
│   └── evals.json        # 技能评估测试配置
├── reference/            # 参考资料
│   └── *.md              # 相关学习文章和资源
└── test-output/          # 测试输出示例（可选）
    └── *.md              # 场景测试用例输出
```

## 添加新技能

创建新技能的步骤：

1. **创建技能目录**
   ```bash
   mkdir new-skill-name
   cd new-skill-name
   ```

2. **创建 SKILL.md**
   - 定义技能名称、描述、触发词
   - 编写详细的 AI 工作流指令
   - 包含示例和边界情况处理

3. **创建 README.md**
   - 面向人类的技能说明
   - 使用场景和示例
   - 目录结构说明

4. **创建 evals/evals.json**
   - 定义评估测试用例
   - 包含正面和负面测试场景

5. **更新 AGENTS.md**
   - 将新技能添加到技能列表表格

## SKILL.md 格式规范

```markdown
---
name: skill-name
description: 技能简短描述。触发词：关键词1、关键词2。
---

# 技能标题

## 一、适用场景判断

### ✅ 适合场景
- 场景描述

### ❌ 不适合场景
- 场景描述

## 二、执行工作流

### Step X: 步骤名称
详细指令...

## 三、输出要求
1. 格式要求
2. 结构要求
```

## 技能评估

技能测试通过 `evals/evals.json` 定义：

```json
{
  "skill_name": "skill-name",
  "evals": [
    {
      "id": 1,
      "name": "测试名称",
      "prompt": "测试输入",
      "expected_output": "预期输出描述",
      "expectations": ["验证条件1", "验证条件2"]
    }
  ]
}
```

评估类别应包括：
- 标准场景测试
- 负面测试（拒绝不合适的请求）
- 边界情况测试
- 质量检查测试

## 参考资料

- 达尔文技能框架：https://github.com/alchaincyf/darwin-skill
