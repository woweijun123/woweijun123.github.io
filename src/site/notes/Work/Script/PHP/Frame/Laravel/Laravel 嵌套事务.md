---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel 嵌套事务/","title":"Laravel 嵌套事务","tags":["flashcards"],"noteIcon":"","created":"2025-07-31T21:40:32.636+08:00","updated":"2026-07-17T10:57:01.743+08:00","dg-note-properties":{"title":"Laravel 嵌套事务","tags":["flashcards"],"reference linking":null}}
---

# 核心结论
MySQL / Laravel / Hyperf 不支持==1;;真正==的嵌套事务；用 `transactions` 计数 +（支持时）`SAVEPOINT` 模拟。
- 只有==1;;最外==层成功才真正 `COMMIT`；
- 内层「提交」只是==1;;减==深度计数。
# 两种模式
| 特性       | 支持保存点（MySQL / PostgreSQL） | 扁平化（无保存点）     |
| :------- | :------------------------ | :------------ |
| 嵌套 begin | ==1;;SAVEPOINT X==        | 只增加深度计数器      |
| 内层异常     | ==1;;ROLLBACK TO X==      | 整单 `ROLLBACK` |
| 外层异常     | 整单 `ROLLBACK`             | 整单 `ROLLBACK` |
| 真正提交     | 仅最外层 `COMMIT`             | 仅最外层 `COMMIT` |
- 有保存点时：可局部回滚到==1;;保存点==；
- 无保存点时：任意层异常==1;;整单==回滚。
两种模式真正落盘都只在==1;;最外==层。
# 开启与提交顺序
**进入**闭包由**外到内**；`commit()` 由**内到外**减计数。
三层嵌套事件顺序：
- begin顺序为 ==1;;1→2→3==，commit顺序为 ==1;;3→2→1==。
- 真正 `PDO::commit()` 仅当 `transactions` = ==1;;1==。
- 框架内层成功不执行==1;;`RELEASE SAVEPOINT`==只 `transactions--`；与「库可 `RELEASE SAVEPOINT`」的示意不同。
### Laravel 关键源码片段
`/vendor/laravel/framework/src/Illuminate/Database/Concerns/ManagesTransactions.php`
```php
public function beginTransaction()
{
    // 先创建事务或保存点，再把层级加 1
    $this->createTransaction();
    $this->transactions++;
}

protected function createTransaction()
{
    if ($this->transactions == 0) {
        // 0 表示最外层，真正开启 PDO 事务
        $this->getPdo()->beginTransaction();
    } elseif ($this->queryGrammar->supportsSavepoints()) {
        // 内层则用保存点模拟嵌套
        $this->getPdo()->exec($this->queryGrammar->compileSavepoint('trans'.($this->transactions + 1)));
    }
}

public function commit()
{
    if ($this->transactionLevel() == 1) {
        // 只有最外层才真正提交到数据库
        $this->getPdo()->commit();
    }

    // 提交完成后，层级再减 1；最终会落到 0
    $this->transactions = max(0, $this->transactions - 1);
}

public function rollBack($toLevel = null)
{
    // 默认回退一层
    $toLevel = is_null($toLevel)
        ? $this->transactions - 1
        : $toLevel;

    if ($toLevel < 0 || $toLevel >= $this->transactions) {
        return;
    }

    // 先执行回滚动作，再把当前层级改成目标层级
    $this->performRollBack($toLevel);
    $this->transactions = $toLevel;
}
```

对应理解：
- `beginTransaction()` 先决定**怎么开事务**，再把层级 `+1`。
- `commit()` 先判断当前是否是最外层；只有 `transactions == 1` 才真正 `PDO::commit()`。
- `rollBack()` 默认回退一层；回退到 `0` 时就是整单回滚。

因此“事务等级为 `0`”是最外层 `commit()` **完成后的状态**，表示连接已经不在事务中；它不是 `PDO::commit()` 的判断条件。源码中先判断 `1`、后递减到 `0`，正好解释了两种观察的差异。

# 精简示例（事件证明顺序）
- Hyperf 单测可跑（需 DB）。
- 要点：Provider 用容器单例（勿 `make()`）；
- 动态 `on()` 前清空 `listenersCache`；`finally` 还原监听器；
- `commit()` 先减 level 再抛事件，打印刚提交层用 `level + 1`。
> [!question]- Hyperf 可运行示例
> ```php
> /** @var ListenerProvider $p */
> $p = ApplicationContext::getContainer()->get(ListenerProviderInterface::class);
> $n = count($p->listeners);
> $cache = new \ReflectionProperty(ListenerProvider::class, 'listenersCache');
> $cache->setAccessible(true);
> $cache->setValue($p, []);
>
> $p->on(TransactionBeginning::class, static fn ($e) => print 'begin ' . $e->connection->transactionLevel() . PHP_EOL);
> $p->on(TransactionCommitted::class, static fn ($e) => print 'commit ' . ($e->connection->transactionLevel() + 1) . PHP_EOL);
>
> try {
>     Db::transaction(static fn () => Db::transaction(static fn () => Db::transaction(static fn () => null)));
> } finally {
>     $p->listeners = array_slice($p->listeners, 0, $n);
>     $cache->setValue($p, []);
> }
> ```
>
> 输出：`begin 1` `2` `3` → `commit 3` `2` `1`
<!--SR:!2026-07-25,8,250-->
