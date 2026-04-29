# biz-analysis-4j: Java 业务功能分析技能

## 简介

**biz-analysis-4j** 是一个专门用于深度分析 Java 项目业务功能的 AI 技能。它以业务实体名称或数据库表名作为输入，自动生成包含完整业务逻辑的标准 Markdown 分析报告。

**核心价值**：
- 🚀 **快速理解业务**：新成员可以通过分析报告快速了解业务逻辑
- 📊 **可视化业务**：自动生成类图、时序图、状态图、ER图
- 🔍 **深度分析**：从入口点到数据库的完整调用链路追踪
- 📝 **标准文档**：输出可直接用于技术文档或架构评审

---

## 适用场景

- **新成员 onboarding**：快速了解项目业务架构
- **代码审查准备**：梳理业务逻辑，提高评审效率
- **技术文档编写**：自动生成可复用的架构图
- **系统重构评估**：分析业务依赖关系，评估重构影响
- **业务知识沉淀**：将代码中的业务逻辑转化为文档

---

## 快速开始

### 1. 安装依赖

确保已安装 `ast-grep`：

```bash
ast-grep --version
```

### 2. 使用技能

在对话中输入触发词，后跟实体名称或表名：

**触发词**：
- "业务分析"
- "分析业务"
- "Java业务分析"
- "分析实体"
- "分析表"
- "业务逻辑分析"

**示例**：
```
分析一下 Order 实体
分析表 t_user 的业务逻辑
对 Product 实体进行业务分析
```

---

## 分析内容说明

### 1. 入口点识别

自动识别以下类型的业务入口：

| 入口类型 | 识别方式 | 示例 |
|----------|----------|------|
| **Controller** | `@RestController`、`@Controller` 类中的方法 | 用户 HTTP 请求入口 |
| **MQ 监听器** | `@RabbitListener`、`@KafkaListener` | 消息队列消费入口 |
| **定时任务** | `@Scheduled` 注解方法 | 定时业务处理 |

### 2. 调用链路追踪

从入口方法开始，递归追踪方法调用链：

```
Controller.createOrder()
    ↓
OrderService.create()
    ↓
OrderValidator.validate()
    ↓
OrderRepository.save()
    ↓
Database
```

**追踪边界**：
- 到达 Repository 层（数据库操作）
- 到达外部接口调用（HTTP/RPC）
- 到达第三方 JAR 包
- 最大深度 7 层（防止循环依赖）

### 3. 实体关系分析

自动提取实体类及其关联关系：

- **关联实体**：`@OneToOne`、`@OneToMany`、`@ManyToOne`、`@ManyToMany`
- **值对象**：`@Embedded`、`@Embeddable`
- **继承关系**：类继承、接口实现

### 4. 状态机分析

识别业务状态字段，分析状态流转：

- 自动识别枚举类型的状态字段
- 分析代码中的状态变更逻辑
- 生成状态流转图

### 5. 文档输出

最终生成的 Markdown 文档包含：

1. **业务概述**：实体对应业务描述
2. **入口点分析**：所有 Controller/MQ/定时任务列表
3. **调用链路图**：Mermaid 类图
4. **核心时序图**：Mermaid 时序图
5. **实体关系图**：Mermaid ER图
6. **状态机分析**：Mermaid 状态图 + 状态说明
7. **方法清单**：Service/Repository 层方法汇总

---

## 使用示例

### 示例 1：分析订单实体

**输入**：
```
分析一下 Order 实体
```

**分析过程**：
1. 识别为实体类名，定位 `Order` 类
2. 查找 `OrderController`，发现 `createOrder`、`getOrder`、`updateStatus` 等方法
3. 追踪 `createOrder` 方法：
   - `OrderController.createOrder()` → `OrderService.create()` → `OrderRepository.save()`
4. 分析 `Order` 实体字段，发现 `OrderStatus` 状态字段
5. 识别状态值：CREATED、PAID、SHIPPED、COMPLETED、CANCELLED
6. 生成包含类图、时序图、状态图的完整报告

**输出文档结构**：
```markdown
# Order 业务分析报告

## 一、业务概述
- 实体名称：Order
- 对应表名：t_order
- 业务描述：电商订单核心业务实体

## 二、入口点分析
### 2.1 Controller 接口
| 类名 | 方法 | URL | 描述 |
|------|------|-----|------|
| OrderController | createOrder | POST /orders | 创建订单 |
| OrderController | getOrder | GET /orders/{id} | 查询订单 |
...

## 三、调用链路图
[Mermaid 类图]

## 四、核心时序图
[Mermaid 时序图]

## 五、实体关系图
[Mermaid ER图]

## 六、状态机分析
[Mermaid 状态图]

## 七、方法清单
...
```

### 示例 2：通过表名分析

**输入**：
```
分析表 t_user
```

**分析过程**：
1. 识别为表名，通过 MyBatis XML 查找对应实体类
2. 发现 `resultMap type="User"`，定位到 `User` 实体
3. 继续分析 `User` 实体的业务逻辑

---

## 目录结构

```
biz-analysis-4j/
├── SKILL.md              # 技能核心定义（AI 使用）
├── README.md             # 本文件（人类阅读）
├── evals/
│   └── evals.json        # 评估测试配置
└── references/
    └── mermaid-uml-guide.md  # Mermaid UML 语法参考
```

---

## 依赖说明

本技能依赖以下工具和能力：

1. **ast-grep-4j 技能**：用于底层 Java 代码分析
2. **ast-grep 工具**：必须安装 ast-grep >= 0.40
3. **Mermaid 语法**：图表使用 Mermaid 语法渲染

---

## 注意事项

1. **命名约定**：技能依赖标准的 Java 命名规范和常用框架注解
2. **递归限制**：调用链分析最大深度为 7 层
3. **性能考虑**：大型项目建议先限定到具体模块目录
4. **边界识别**：追踪到 Repository 层或外部接口即停止

---

## 参考资料

- [SKILL.md](./SKILL.md) - 完整的技能定义和工作流
- [references/mermaid-uml-guide.md](./references/mermaid-uml-guide.md) - Mermaid UML 语法参考
- [ast-grep-4j 技能](../ast-grep-4j/) - 底层代码分析技能

---

## 许可证

MIT License
