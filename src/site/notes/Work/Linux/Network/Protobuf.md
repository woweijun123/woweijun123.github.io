---
{"dg-publish":true,"permalink":"/Work/Linux/Network/Protobuf/","title":"Protobuf","tags":["flashcards","#network","#protobuf"],"noteIcon":"","created":"2026-07-18T21:32:46.000+08:00","updated":"2026-10-31T18:46:49.289+08:00","dg-note-properties":{"title":"Protobuf","tags":["flashcards","#network","#protobuf"],"reference linking":null}}
---

# 文件与结构规范
* **文件命名：** 使用 **小写加下划线**，例如 `room_service.proto`。
* **文件结构：** 建议按以下顺序排列：
    1.  License (如有)
    2.  版本声明 (`syntax = "proto3";`)
    3.  Package 定义
    4.  Import 语句
    5.  Option 定义（如 `option php_namespace = "...";`）
    6.  消息、枚举和服务定义
# 命名约定 (核心)
| 对象                   | 格式                     | 示例                            |
| :------------------- | :--------------------- | :---------------------------- |
| **消息 (Message)**     | **大驼峰** (PascalCase)   | `message RoomState { ... }`   |
| **字段 (Field)**       | **小写下划线** (snake_case) | `string room_name = 1;`       |
| **枚举名 (Enum)**       | **大驼峰** (PascalCase)   | `enum UserRole { ... }`       |
| **枚举值 (Enum Value)** | **全大写下划线** (CAPS)      | `USER_ROLE_ADMIN = 1;`        |
| **服务 (Service)**     | **大驼峰** (PascalCase)   | `service RoomManager { ... }` |
| **RPC 方法**           | **大驼峰** (PascalCase)   | `rpc GetRoomInfo(...)`        |
# 枚举 (Enums) 特殊规则
* **零值占位：** 枚举的**第一个值必须是 `0`**，且通常命名为 `UNKNOWN` 或 `UNSPECIFIED`。
* **前缀处理：** 为了避免命名冲突（尤其是 C++ 或 Go），建议在枚举值前加上枚举名作为前缀。
> 示例：`enum Status { STATUS_UNSPECIFIED = 0; STATUS_OPEN = 1; }`
# 字段编号 (Field Numbers) 的艺术
* **重要性：** 编号是二进制协议的灵魂，**发布后绝不可更改**。
* **热点优化：** 将**最常设置**的字段分配在 **1 到 15** 之间，此时字段的 **tag/key** 占 1 字节；字段值仍按自身类型编码，长度分隔类型还包含长度前缀。
* **字段号硬边界：** 合法范围是 `1～536,870,911`（$2^{29}-1$）；其中 `19,000～19,999` 由 Protobuf 实现保留，业务字段不可使用。这里限制的是消息字段编号，不是枚举值范围。
* **预留编号：** 删除字段后必须用 `reserved` 保留其字段编号，避免重新使用同一 wire tag；使用 TextFormat 或 ProtoJSON 时还应保留字段名。编号复用可能造成歧义解析、数据损坏、隐私泄露或解析失败，不只是“客户端崩溃”。
> 示例：`reserved 2, 15, 9 to 11;`
# 集合与重复项
* **Repeated 字段：** 使用**复数**命名，清晰表达这是一个列表。
> 示例：`repeated string member_ids = 1;`
* **Map 字段：** 如果需要键值对，优先考虑 `map<key_type, value_type>`，但注意 map 无法保证顺序。
# Optional 特性 (字段存在性)
- **基本类型字段**：proto3 中，未标记 `optional` 的单一数值、枚举、字符串和字节字段使用隐式存在性；`optional` 为这些字段显式开启存在性追踪。
- **核心价值**：区分 **“默认值”** 与 **“未赋值”**。
    - **隐式存在性**：基本类型的 `0/false/""` 默认不序列化，接收方无法区分“显式设置默认值”与“未设置”。
    - **显式存在性**：已设置的 `optional` 字段即使是默认值也会序列化；未设置则不序列化。生成 API 可通过对应的 presence 方法判断是否设置，具体方法名因语言而异。
- **适用边界**：消息字段和 `oneof` 本来就追踪存在性；`repeated` 与 `map` 不区分“空”与“未设置”。
- **存储开销**：存在性由生成 API 与运行时状态维护，具体内存表示因语言和实现而异；部分实现使用位图，但 Protobuf wire format 没有独立的存在性位图。显式默认值在 wire 上仍以普通 tag + value 编码。
# 最佳工程实践建议
* **避免字段冲突：** **不要修改已有的字段类型或编号**。如果需求变了，增加新字段，废弃（Deprecated）旧字段。
* **包名定义：** 建议包含版本号，如 `package room.v1;`。这样当你以后进行重大架构调整（v2）时，新旧协议可以并存。
* **注释：** 使用 `//` 进行单行注释，这些注释会被提取并保留在生成的代码中，对 App 端同事非常友好。
