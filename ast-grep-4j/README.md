# ast-grep-4j: Java 代码分析技能

## 简介

这是一个专门用于**分析 Java 代码**的 ast-grep 技能，帮助开发者快速搜索代码中的方法调用、类定义、继承关系、注解使用等模式。

**核心特点：**
- 只读分析，不会修改代码库
- 提供10+常用代码搜索模式
- 支持复杂查询的 YAML 规则
- 内置3个代码分析专用工作流

## 适用场景

- 🔍 查找方法定义和调用位置
- 🔗 分析方法调用链和依赖关系
- 🧬 分析类继承和接口实现关系
- 🏷️ 查找特定注解的使用
- 📊 代码质量扫描（空 catch、System.out.println 等）

## 快速开始

### 1. 安装 ast-grep

```bash
ast-grep --version
```

如果未安装，参考 [ast-grep 官方文档](https://ast-grep.github.io/) 安装。

### 2. 使用技能

在对话中输入以下触发词：
- "分析Java"
- "查找方法"
- "搜索代码"
- "代码分析"
- "ast-grep Java"

### 3. 选择分析场景

根据需求快速定位命令：

```
我要分析...
├── 📍 某个方法/类在哪里定义 → 第1节/第3节
├── 🔗 方法被哪些地方调用 → 第2节 + 工作流A
├── 🏷️ 使用了哪些注解 → 第4节
├── 🧬 类的继承/实现关系 → 第3节
├── 📊 代码统计/质量分析 → 工作流C
└── 🔍 复杂模式匹配 → 第5节
```

## 常用命令示例

### 查找方法定义

```bash
# 查找指定方法
ast-grep -p 'void saveDealerAppChgAccount($$$)' -l java <项目路径>

# 查找带 @Transactional 注解的方法
ast-grep scan --inline-rules "id: annotated-method
language: java
rule:
  kind: method_declaration
  has:
    kind: annotation
    pattern: '@Transactional'
    stopBy: end" <项目路径>
```

### 查找方法调用

```bash
# 查找对特定方法的调用
ast-grep -p 'saveDealerAppChgAccount($$$)' -l java <项目路径>
```

### 查找类定义

```bash
# 查找继承特定父类的类
ast-grep -p 'class $NAME extends ServiceImpl<$T> { $$$ }' -l java <项目路径>

# 查找实现特定接口的类
ast-grep -p 'class $NAME implements $INTERFACE { $$$ }' -l java <项目路径>
```

## 代码分析工作流

### 工作流 A: 分析方法调用链

追踪一个方法从定义到被调用的完整链路：

```bash
# Step 1: 找到方法定义
ast-grep -p 'void targetMethod($$$)' -l java src/

# Step 2: 查找所有调用位置
ast-grep -p 'targetMethod($$$)' -l java src/

# Step 3: 分析调用上下文
ast-grep scan --inline-rules "id: call-context
language: java
rule:
  pattern: 'targetMethod($$$)'
  inside:
    kind: method_declaration
    stopBy: end" src/
```

### 工作流 C: 代码质量扫描

快速扫描代码中的潜在问题：

```bash
# 查找空的 catch 块
ast-grep scan --inline-rules "id: empty-catch
language: java
rule:
  pattern: 'try { $$$ } catch ($E) { }'" <项目路径>

# 查找 System.out.println 调用
ast-grep -p 'System.out.println($$$)' -l java <项目路径>
```

## 目录结构

```
ast-grep-4j/
├── SKILL.md              # 技能核心定义（AI 使用）
├── README.md             # 本文件（人类阅读）
├── evals/
│   ├── evals.json        # 评估测试配置
│   ├── test-prompts.json # 测试用例
│   └── results.tsv       # 优化历史记录
└── references/
    └── rule_reference.md # ast-grep 规则语法参考
```

## 参考资料

- [ast-grep 官方文档](https://ast-grep.github.io/)
- [SKILL.md](./SKILL.md) - 完整的技能定义和工作流
- [references/rule_reference.md](./references/rule_reference.md) - 详细的规则语法参考

## 注意事项

- 所有命令均为只读操作，不会修改代码
- 搜索时建议使用 `-l java` 限制只搜索 Java 文件
- 对于大型项目，可以指定具体子目录缩小搜索范围
- 复杂的搜索建议使用 YAML 规则文件而非 inline rules
