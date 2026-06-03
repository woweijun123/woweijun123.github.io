---
{"dg-publish":true,"permalink":"/Work/Linux/Network/Protobuf/","title":"Protobuf","tags":["flashcards"],"noteIcon":"","created":"2026-04-10T09:45:58.442+08:00","updated":"2026-04-10T11:02:00.621+08:00","dg-note-properties":{"title":"Protobuf","tags":["flashcards"],"reference linking":null}}
---

### 文件与结构规范
* **文件命名：** 使用 **小写加下划线**，例如 `room_service.proto`。
* **文件结构：** 建议按以下顺序排列：
    1.  License (如有)
    2.  版本声明 (`syntax = "proto3";`)
    3.  Package 定义
    4.  Import 语句
    5.  Option 定义（如 `option php_namespace = "...";`）
    6.  消息、枚举和服务定义

### 命名约定 (核心)
| 对象                   | 格式                     | 示例                            |
| :------------------- | :--------------------- | :---------------------------- |
| **消息 (Message)**     | **大驼峰** (PascalCase)   | `message RoomState { ... }`   |
| **字段 (Field)**       | **小写下划线** (snake_case) | `string room_name = 1;`       |
| **枚举名 (Enum)**       | **大驼峰** (PascalCase)   | `enum UserRole { ... }`       |
| **枚举值 (Enum Value)** | **全大写下划线** (CAPS)      | `USER_ROLE_ADMIN = 1;`        |
| **服务 (Service)**     | **大驼峰** (PascalCase)   | `service RoomManager { ... }` |
| **RPC 方法**           | **大驼峰** (PascalCase)   | `rpc GetRoomInfo(...)`        |
### 枚举 (Enums) 特殊规则
* **零值占位：** 枚举的**第一个值必须是 `0`**，且通常命名为 `UNKNOWN` 或 `UNSPECIFIED`。
* **前缀处理：** 为了避免命名冲突（尤其是 C++ 或 Go），建议在枚举值前加上枚举名作为前缀。
> 示例：`enum Status { STATUS_UNSPECIFIED = 0; STATUS_OPEN = 1; }`
### 字段编号 (Field Numbers) 的艺术
* **重要性：** 编号是二进制协议的灵魂，**发布后绝不可更改**。
* **热点优化：** 将**最常用**的字段分配在 **1 到 15** 之间（编码仅占 1 字节）。
* **预留编号：** 如果删除了某个字段，必须使用 `reserved` 关键字标记该编号或名称，防止未来被误用导致旧版本客户端解析崩溃。
> 示例：`reserved 2, 15, 9 to 11;`
### 集合与重复项
* **Repeated 字段：** 使用**复数**命名，清晰表达这是一个列表。
> 示例：`repeated string member_ids = 1;`
* **Map 字段：** 如果需要键值对，优先考虑 `map<key_type, value_type>`，但注意 map 无法保证顺序。
### Optional 特性 (字段存在性)
- **原始特性**：显式开启“存在性追踪”。
- **核心价值**：区分 **“默认值”** 与 **“未赋值”**。
    - **默认**：`0/false/""` 不传输，接收方无法判断是发送方赋值还是缺失补齐。
    - **Optional**：即使是零值也会传输，接收方可通过 `has_<field>()` 判断是否被显式赋值。
- **存储开销**：通过位图 (Bitmask) 标记，对体积影响极小。
### 最佳工程实践建议
* **避免字段冲突：** **不要修改已有的字段类型或编号**。如果需求变了，增加新字段，废弃（Deprecated）旧字段。
* **包名定义：** 建议包含版本号，如 `package room.v1;`。这样当你以后进行重大架构调整（v2）时，新旧协议可以并存。
* **注释：** 使用 `//` 进行单行注释，这些注释会被提取并保留在生成的代码中，对 App 端同事非常友好。