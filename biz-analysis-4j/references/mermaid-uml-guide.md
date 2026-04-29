# Mermaid UML 语法参考

本文档提供 biz-analysis-4j 技能中使用的 Mermaid 图表语法参考。

---

## 1. 类图（Class Diagram）

### 基本语法

```mermaid
classDiagram
    class ClassName {
        +PublicAttribute: Type
        -PrivateAttribute: Type
        #ProtectedAttribute: Type
        +PublicMethod(): ReturnType
        -PrivateMethod(): ReturnType
    }
```

### 关系类型

| 语法 | 含义 | 示例 |
|------|------|------|
| `-->` | 关联（Association） | `A --> B` |
| `-->` | 依赖（Dependency） | `A ..> B` |
| `--|>` | 继承（Inheritance） | `B --|> A` |
| `..|>` | 实现（Realization） | `B ..|> A` |
| `o--` | 聚合（Aggregation） | `A o-- B` |
| `*--` | 组合（Composition） | `A *-- B` |

### 完整示例

```mermaid
classDiagram
    direction TB
    
    class OrderController {
        -OrderService orderService
        +createOrder(OrderDTO dto) Response
        +getOrder(Long id) Response
        +updateOrderStatus(Long id, Status status) Response
    }
    
    class OrderService {
        -OrderRepository orderRepository
        +create(Order order) Order
        +findById(Long id) Order
        +updateStatus(Long id, Status status) Order
    }
    
    class Order {
        -Long id
        -String orderNo
        -OrderStatus status
        -BigDecimal amount
        +getStatus() OrderStatus
        +setStatus(OrderStatus status) void
    }
    
    class OrderRepository {
        +save(Order order) Order
        +findById(Long id) Optional~Order~
        +findAll() List~Order~
    }
    
    OrderController --> OrderService : uses
    OrderService --> Order : manages
    OrderService --> OrderRepository : persists
```

---

## 2. 时序图（Sequence Diagram）

### 基本语法

```mermaid
sequenceDiagram
    participant A as 别名A
    participant B as 别名B
    
    A->>B: 同步消息
    A-->>B: 异步消息
    A-xB: 删除消息
    B-->>A: 返回消息
```

### 常用符号

| 语法 | 含义 |
|------|------|
| `->>` | 实线箭头（同步调用） |
| `-->>` | 虚线箭头（返回消息） |
| `-x` | 实线叉（删除） |
| `--x` | 虚线叉（异步删除） |
| `activate`/`deactivate` | 激活/去激活生命线 |
| `Note over A,B` | 注释 |

### 完整示例

```mermaid
sequenceDiagram
    autonumber
    actor U as 用户
    participant C as OrderController
    participant S as OrderService
    participant V as OrderValidator
    participant R as OrderRepository
    participant DB as Database
    
    U->>C: POST /orders (创建订单)
    activate C
    
    C->>S: createOrder(dto)
    activate S
    
    S->>V: validate(dto)
    activate V
    V-->>S: validation result
    deactivate V
    
    S->>S: buildOrderEntity()
    
    S->>R: save(order)
    activate R
    R->>DB: INSERT INTO orders
    activate DB
    DB-->>R: order record
    deactivate DB
    R-->>S: saved order
    deactivate R
    
    S->>S: publishOrderCreatedEvent()
    
    S-->>C: order
    deactivate S
    
    C-->>U: 201 Created
    deactivate C
```

---

## 3. 状态图（State Diagram）

### 基本语法

```mermaid
stateDiagram-v2
    [*] --> InitialState
    State1 --> State2: 触发条件
    State2 --> [*]: 结束
```

### 状态类型

| 语法 | 含义 |
|------|------|
| `state "描述" as StateName` | 状态别名 |
| `state StateName { sub1 --> sub2 }` | 复合状态 |
| `<<choice>>` | 选择节点 |
| `<<fork>>` / `<<join>>` | 分叉/汇合 |

### 完整示例

```mermaid
stateDiagram-v2
    [*] --> CREATED: 提交订单
    
    state CREATED {
        [*] --> UNPAID
        UNPAID --> CANCELLED: 超时取消
    }
    
    CREATED --> PAID: 支付成功
    PAID --> SHIPPED: 商家发货
    PAID --> REFUNDING: 申请退款
    
    REFUNDING --> REFUNDED: 退款成功
    REFUNDING --> PAID: 拒绝退款
    
    SHIPPED --> DELIVERED: 物流送达
    DELIVERED --> COMPLETED: 确认收货
    DELIVERED --> RETURNING: 申请退货
    
    RETURNING --> RETURNED: 退货完成
    RETURNING --> COMPLETED: 取消退货
    
    COMPLETED --> [*]
    CANCELLED --> [*]
    REFUNDED --> [*]
    RETURNED --> [*]
```

---

## 4. ER图（Entity Relationship Diagram）

### 基本语法

```mermaid
erDiagram
    ENTITY1 ||--o{ ENTITY2 : relationship
```

### 关系基数

| 语法 | 含义 |
|------|------|
| `||` | 一对一（必须） |
| `o|` | 一对一（可选） |
| `}|` | 多对一 |
| `o{` | 零或多 |
| `}|{` | 一或多 |

### 完整示例

```mermaid
erDiagram
    USER ||--o{ ORDER : places
    ORDER ||--|{ ORDER_ITEM : contains
    ORDER ||--|| PAYMENT : has
    PRODUCT ||--o{ ORDER_ITEM : included_in
    
    USER {
        bigint id PK
        varchar username
        varchar email
        varchar phone
        datetime created_at
    }
    
    ORDER {
        bigint id PK
        varchar order_no UK
        int status
        decimal total_amount
        bigint user_id FK
        datetime created_at
        datetime updated_at
    }
    
    ORDER_ITEM {
        bigint id PK
        bigint order_id FK
        bigint product_id FK
        int quantity
        decimal unit_price
        decimal subtotal
    }
    
    PRODUCT {
        bigint id PK
        varchar sku UK
        varchar name
        decimal price
        int stock
    }
    
    PAYMENT {
        bigint id PK
        bigint order_id FK
        varchar transaction_no
        int status
        decimal amount
        varchar pay_channel
    }
```

---

## 5. 流程图（Flowchart）

### 基本语法

```mermaid
flowchart TD
    A[开始] --> B{判断}
    B -->|是| C[处理1]
    B -->|否| D[处理2]
    C --> E[结束]
    D --> E
```

### 节点形状

| 语法 | 形状 |
|------|------|
`A[文本]` | 矩形（过程） |
`A(文本)` | 圆角矩形（开始/结束） |
`A{文本}` | 菱形（判断） |
`A((文本))` | 圆形 |
`A[/文本/]` | 平行四边形（输入/输出） |

---

## 6. 最佳实践

### 图表方向
- 类图：`direction TB`（从上到下）或 `direction LR`（从左到右）
- 时序图：默认从左到右，参与者按调用顺序排列
- 状态图：使用 `stateDiagram-v2` 语法

### 命名规范
- 使用有意义的类名和方法名
- 中文别名使用 `as` 关键字：`participant C as Controller`
- 泛型使用 `~Type~` 语法：`List~Order~`

### 注释和说明
- 使用 `%% 注释` 添加图表注释
- 使用 `Note over A,B` 在时序图中添加说明
- 状态图中使用 `:` 标注状态流转条件

### 复杂图表拆分
当图表过于复杂时，建议：
1. 拆分为多个子图
2. 使用复合状态（state 嵌套）
3. 使用链接引用其他图表
