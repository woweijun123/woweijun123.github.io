---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel 嵌套事务/","title":"Laravel 嵌套事务","tags":["flashcards"],"noteIcon":"","created":"2025-07-31T21:40:32.636+08:00","updated":"2026-03-24T17:33:53.453+08:00","dg-note-properties":{"title":"Laravel 嵌套事务","tags":["flashcards"],"reference linking":null}}
---

## Laravel 中嵌套事务的处理逻辑与回滚
### 1\. 数据库保存点 (Savepoints)
对于**支持保存点的数据库**（如：MySQL、PostgreSQL），Laravel 会利用这个特性来实现“嵌套事务”的行为。
- **工作原理：**
	- 当你调用 `$DB::transaction()` 方法时，如果当前**没有活跃的事务**，它会开启一个**全新的数据库事务** (`BEGIN;`)。
	- 如果在已经活跃的事务中再次调用 `$DB::transaction()`，Laravel 不会开启新的事务，而是会创建一个**保存点** (`SAVEPOINT transaction_name;`)。
	- 在保存点内部执行的所有操作都属于这个保存点。
- **异常回滚：**
	1. **最外层**事务（或任何没有保存点的事务）中发生**异常**，整个事务会**回滚到初始状态**。
	2. **内部**事务在某个**嵌套的** `$DB::transaction()` 块（即保存点）中**发生异常**，会执行**回滚到保存点**的操作 (`ROLLBACK TO SAVEPOINT transaction_name;`)。
		-  这意味着只有该保存点之后的操作会被撤销，而该保存点之前以及**父级事务中的操作依然有效**，直到最外层事务最终提交或回滚。
- **嵌套事务保证原子性**
	- **内部**事务失败向上冒泡抛异常，可以保证**整个嵌套事务**的**原子性**
**示例代码：**
```php
DB::transaction(function () {
    // 外层事务操作
    DB::table('users')->insert(['name' => 'Alice']);

    DB::transaction(function () {
        // 内层事务操作 (会创建一个保存点)
        DB::table('orders')->insert(['user_id' => 1, 'product' => 'Laptop']);

        // 假设这里发生异常
        // throw new \Exception('订单创建失败');

    }); // 如果内层有异常，会回滚到这个保存点

    // 外层事务继续 (如果内层回滚，这里的数据可能保持不变，取决于回滚到保存点)
    DB::table('logs')->insert(['message' => 'User Alice added']);

}); // 如果外层有异常，整个事务回滚
```
### 2\. 扁平化事务 (Flat Transactions)
对于**不支持保存点的数据库**（如：NoSQL 数据库），Laravel 会将所有嵌套的 `$DB::transaction()` 调用视为**单一的扁平化事务**。
* **工作原理：**
	* 在这种情况下，只有第一次调用 `$DB::transaction()` 会真正开启一个数据库事务 (`BEGIN;`)。
	* 后续的嵌套调用不会执行任何数据库命令（不会创建保存点）。它们只是增加了内部计数器，表示事务的“深度”。
* **异常回滚：**
	* **任何层级的异常都会导致整个最外层事务的完全回滚** (`ROLLBACK;`)。
	* 这意味着，无论异常发生在哪个“嵌套”层级，所有在最外层 `BEGIN` 之后的操作都会被撤销，就像只有一个事务一样。没有局部回滚到保存点这个概念。
### 3. 总结
| 特性 / 行为    | 支持保存点的数据库 (MySQL, PostgreSQL)       | 不支持保存点的数据库 (或扁平化模式)    |
| :--------- | :---------------------------------- | :--------------------- |
| **嵌套调用**   | 创建数据库保存点                            | 增加事务深度计数器，不执行数据库命令     |
| **内部异常回滚** | 回滚到对应的保存点 (`ROLLBACK TO SAVEPOINT`) | 整个事务完全回滚 (`ROLLBACK`)  |
| **外部异常回滚** | 整个事务完全回滚 (`ROLLBACK`)               | 整个事务完全回滚 (`ROLLBACK`)  |
| **提交**     | 只有最外层事务成功，最终才 `COMMIT`              | 只有最外层事务成功，最终才 `COMMIT` |
