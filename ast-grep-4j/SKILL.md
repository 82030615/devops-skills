---
name: ast-grep-4j
description: 专门用于分析 Java 代码的 ast-grep 技能。帮助用户搜索代码中的方法调用、类定义、继承关系、注解使用等常见模式。触发词：分析Java、查找方法、搜索代码、代码分析、ast-grep Java
---

# ast-grep-4j: Java 代码分析技能

## 概述

本技能基于 ast-grep，专门用于**只读分析** Java 代码库。提供常用的代码搜索模式，包括方法查找、调用关系分析、类结构探索等。

> **只读说明**：本技能所有命令均为只读操作，不会修改代码库。如需重构，请参考 ast-grep 官方文档使用 rewrite 功能。

## 参考文档

本技能目录下包含 `references/` 文件夹，其中有详细的规则语法参考文档：

- **`references/rule_reference.md`** - ast-grep 规则的完整语法参考

**何时参考该文档：**
- 需要编写复杂的 YAML 规则文件时
- 想了解 `kind`、`has`、`inside`、`precedes`、`follows` 等关系规则的详细用法
- 需要了解 `all`、`any`、`not` 等复合规则的组合方式
- 想了解元变量（metavariables）的高级用法
- 需要调试规则不生效的问题

**建议：** 先从本 SKILL.md 的常用命令开始，遇到复杂场景时再查阅 rule_reference.md。

---

## 快速开始（3步工作流）

### Step 1: 确认 ast-grep 已安装
```bash
ast-grep --version
```
如果未安装，参考 [ast-grep 官方文档](https://ast-grep.github.io/) 安装。

### Step 2: 选择搜索模式
根据你的需求选择对应章节：
- 找方法 → 查看「1. 查找方法定义」
- 找调用 → 查看「2. 查找方法调用」
- 找类 → 查看「3. 查找类定义」
- 复杂条件 → 查看「高级用法」

### Step 3: 执行并验证
1. 先用 `-l java` 限制只搜索 Java 文件
2. 如果结果太多，添加更具体的 pattern
3. 对于大型项目，先指定具体子目录缩小搜索范围

---

## 分析场景决策树

根据你的分析目标，快速定位合适的命令：

```
我要分析...
├── 📍 某个方法/类在哪里定义
│   └── → 第1节「查找方法定义」/ 第3节「查找类定义」
├── 🔗 方法被哪些地方调用（调用链）
│   └── → 第2节「查找方法调用」+ 第6节「分析方法调用链」
├── 🏷️ 使用了哪些注解
│   └── → 第4节「查找注解」
├── 🧬 类的继承/实现关系
│   └── → 第3节「查找类定义」
├── 📊 代码统计/质量分析
│   └── → 第7节「代码分析专用工作流」
└── 🔍 复杂模式匹配
    └── → 第5节「高级用法」+ references/rule_reference.md
```

---

## 常用命令速查

### 1. 查找方法定义

```bash
# 查找指定方法的定义
ast-grep -p 'void saveDealerAppChgAccount($$$)' -l java <项目路径>

# 查找包含特定注解的方法
ast-grep scan --inline-rules "id: annotated-method
language: java
rule:
  kind: method_declaration
  has:
    kind: annotation
    pattern: '@Transactional'
    stopBy: end" <项目路径>
```

### 2. 查找方法调用

```bash
# 查找对特定方法的调用
ast-grep -p 'saveDealerAppChgAccount($$$)' -l java <项目路径>

# 查找 this.xxx() 调用
ast-grep -p 'this.$METHOD($$$)' -l java <项目路径>
```

### 3. 查找类定义

```bash
# 查找类定义
ast-grep -p 'class $NAME { $$$ }' -l java <项目路径>

# 查找继承特定父类的类
ast-grep -p 'class $NAME extends ServiceImpl<$T> { $$$ }' -l java <项目路径>

# 查找实现特定接口的类
ast-grep -p 'class $NAME implements $INTERFACE { $$$ }' -l java <项目路径>
```

### 4. 查找注解

```bash
# 查找使用特定注解的类/方法/字段
ast-grep -p '@Autowired $$$' -l java <项目路径>

# 查找 @RequestMapping 及其变体
ast-grep scan --inline-rules "id: request-mapping
language: java
rule:
  any:
    - pattern: '@RequestMapping($$$)'
    - pattern: '@GetMapping($$$)'
    - pattern: '@PostMapping($$$)'
    - pattern: '@PutMapping($$$)'
    - pattern: '@DeleteMapping($$$)'" <项目路径>
```

### 5. 查找字段和变量

```bash
# 查找特定类型的字段
ast-grep -p 'private $TYPE $NAME;' -l java <项目路径>

# 查找使用 @Autowired 的字段
ast-grep -p '@Autowired private $TYPE $NAME;' -l java <项目路径>
```

### 6. 查找条件语句

```bash
# 查找 if 语句中的特定模式
ast-grep -p 'if ($COND) { $$$ }' -l java <项目路径>

# 查找 try-catch 块
ast-grep -p 'try { $$$ } catch ($E) { $$$ }' -l java <项目路径>
```

### 7. 查找日志语句

```bash
# 查找各种日志调用
ast-grep scan --inline-rules "id: log-statements
language: java
rule:
  any:
    - pattern: 'log.debug($$$)'
    - pattern: 'log.info($$$)'
    - pattern: 'log.warn($$$)'
    - pattern: 'log.error($$$)'
    - pattern: 'LOGGER.debug($$$)'
    - pattern: 'LOGGER.info($$$)'
    - pattern: 'LOGGER.warn($$$)'
    - pattern: 'LOGGER.error($$$)'" <项目路径>
```

### 8. 查找 MyBatis Mapper

```bash
# 查找 Mapper 接口
ast-grep -p '@Mapper public interface $NAME { $$$ }' -l java <项目路径>

# 查找 XML 映射文件中的 SQL 语句（需要在 XML 文件中搜索）
ast-grep -p '<select id=\"$ID\" $$$>$$$</select>' -l xml <项目路径>/src/main/resources/mapper
```

### 9. 查找 Service 层调用

```bash
# 查找 Service 方法调用链
ast-grep scan --inline-rules "id: service-call-chain
language: java
rule:
  kind: method_invocation
  pattern: '$SERVICE.$METHOD($$$)'
  inside:
    kind: method_declaration
    stopBy: end" <项目路径>
```

### 10. 查找数据库操作

```bash
# 查找 Repository 调用
ast-grep scan --inline-rules "id: repository-operations
language: java
rule:
  any:
    - pattern: '$REPO.save($$$)'
    - pattern: '$REPO.findById($$$)'
    - pattern: '$REPO.findAll($$$)'
    - pattern: '$REPO.delete($$$)'
    - pattern: '$REPO.update($$$)'" <项目路径>
```

---

## 高级用法

### 使用 YAML 规则文件

对于复杂查询，建议创建 YAML 规则文件：

```yaml
# find-unauthorized-methods.yml
id: unauthorized-methods
language: java
severity: warning
rule:
  kind: method_declaration
  pattern: 'public void $NAME($$$)'
  not:
    has:
      kind: annotation
      any:
        - pattern: '@PreAuthorize($$$)'
        - pattern: '@Secured($$$)'
        - pattern: '@RolesAllowed($$$)'
      stopBy: end
message: Public method without security annotation
```

运行：
```bash
ast-grep scan --rule find-unauthorized-methods.yml <项目路径>
```

### 调试模式

```bash
# 查看代码的 AST 结构
ast-grep run --pattern 'class Example { }' --lang java --debug-query=cst

# 查看 pattern 如何被解析
ast-grep run --pattern 'void $NAME()' --lang java --debug-query=pattern
```

---

## 代码分析专用工作流

### 工作流 A: 分析方法调用链

适用于：追踪一个方法从定义到被调用的完整链路

```bash
# Step 1: 找到方法定义位置
ast-grep -p 'void targetMethod($$$)' -l java src/

# Step 2: 查找所有调用位置
ast-grep -p 'targetMethod($$$)' -l java src/

# Step 3: 分析调用上下文（在哪个方法中被调用）
ast-grep scan --inline-rules "id: call-context
language: java
rule:
  pattern: 'targetMethod($$$)'
  inside:
    kind: method_declaration
    stopBy: end" src/
```

### 工作流 B: 分析类依赖关系

适用于：分析一个类被哪些其他类依赖/引用

```bash
# 查找 import 了某类的文件
ast-grep -p 'import com.example.$CLASS;' -l java src/

# 查找使用了某类的字段
ast-grep -p '$TYPE $NAME = new TargetClass($$$)' -l java src/

# 查找方法参数中使用了某类
ast-grep -p '$RET methodName(TargetClass $PARAM, $$$)' -l java src/
```

### 工作流 C: 代码质量扫描

适用于：快速扫描代码中的潜在问题模式

```bash
# 查找空的 catch 块
ast-grep scan --inline-rules "id: empty-catch
language: java
rule:
  pattern: 'try { $$$ } catch ($E) { }'" <项目路径>

# 查找 System.out.println 调用（应该使用日志框架）
ast-grep -p 'System.out.println($$$)' -l java <项目路径>

# 查找 TODO 注释
ast-grep -p '// TODO: $$$' -l java <项目路径>
```

---

## Java 特定的 AST Kind 参考

常用 Java AST 节点类型：

| Kind | 说明 |
|------|------|
| `class_declaration` | 类定义 |
| `interface_declaration` | 接口定义 |
| `method_declaration` | 方法定义 |
| `field_declaration` | 字段定义 |
| `method_invocation` | 方法调用 |
| `annotation` | 注解 |
| `import_declaration` | import 语句 |
| `package_declaration` | package 声明 |

## 通配符说明

- `$VAR` - 匹配单个 AST 节点
- `$$$` - 匹配多个 AST 节点（贪婪匹配）

## 边界条件与故障排除

### 常见问题处理

**搜索不到结果？**
- 检查方法名/类名拼写是否正确（Java 区分大小写）
- 尝试使用更宽松的 pattern，如 `$NAME` 代替具体名称
- 使用 `--debug-query=cst` 查看 AST 结构，确认 kind 类型

**结果太多？**
- 添加更具体的上下文，如 `class $C { $$$ void $METHOD() $$$ }`
- 使用 `inside` 或 `has` 限定搜索范围
- 指定具体的子目录而非整个项目

**命令执行失败？**
- 确认 ast-grep 版本 >= 0.40（`ast-grep --version`）
- 检查项目路径是否正确
- 对于 Windows 路径，使用正斜杠 `/` 或加引号

**YAML 规则不生效？**
- 检查缩进（YAML 对缩进敏感）
- 确认 `language: java` 已指定
- 关系规则（inside/has）添加 `stopBy: end`
- 使用 `ast-grep scan --rule rule.yml --debug-query` 调试

### 大型项目分析建议

对于大型 Java 项目：

1. **分层搜索**：先搜索 Controller 层，再搜索 Service 层，最后搜索 Mapper/Repository 层
2. **限定范围**：使用具体子目录缩小搜索范围，如 `src/main/java/com/example/service`
3. **使用 `--json` 参数**：获取结构化输出便于后续处理
4. **保存结果**：将输出重定向到文件 `> results.txt` 方便查看
