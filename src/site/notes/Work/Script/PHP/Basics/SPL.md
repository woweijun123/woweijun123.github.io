---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Basics/SPL/","title":"SPL","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:26:14.000+08:00","updated":"2026-03-24T17:22:36.346+08:00"}
---

[PHP: SPL - Manual](https://www.php.net/manual/zh/book.spl.php)
# 普通队列
## 先入先出(FIFO)
```php
$queue = new SplQueue;
// 入队
$queue->enqueue(1); // 此时队列中的节点为 1
$queue->enqueue(2); // 此时队列中的节点为 2 - 1
$queue->enqueue(3); // 此时队列中的节点为 3 - 2 - 1

// 队列节点的个数
$size = $queue->count();

// 得到最后入列元素的值
$top = $queue->top(); // 此时结果为3，队列节点不变

// 得到先入列元素的值
$bottom = $queue->bottom(); // 此时结果为1，队列节点不变

// 出队
$queue->dequeue(); // 此时队列中的节点为 3 - 2
$queue->dequeue(); // 此时队列中的节点为 3
$queue->dequeue(); // 此时队列中的节点为 空

// 判断队列是否为空
$result = $queue->isEmpty();  // 结果为true
```
## 先入后出(FILO)
```php
$queue = new SplQueue;

// 入队
$queue->push(1); // 此时队列中的节点为 1
$queue->push(2); // 此时队列中的节点为 2 - 1
$queue->push(3); // 此时队列中的节点为 3 - 2 - 1
// 出列
$queue->pop();      // 次数队列中的节点为 2 - 1
$queue->pop();      // 次数队列中的节点为 1
$queue->pop();      // 次数队列中的节点为 空

// 入列
$queue->enqueue(1); // 此时队列中的节点为 1
$queue->enqueue(2); // 此时队列中的节点为 2 - 1
$queue->enqueue(3); // 此时队列中的节点为 3 - 2 - 1
// 出列
$queue->dequeue();  // 此时队列中的节点为 3 - 2
$queue->dequeue();  // 此时队列中的节点为 3
$queue->dequeue();  // 此时队列中的节点为 空
```
## 迭代器指针
```php
$queue = new SplQueue;

// 将元素压入队列
$queue->enqueue(1); // 此时队列中的节点为 1
$queue->enqueue(2); // 此时队列中的节点为 2 - 1
$queue->enqueue(3); // 此时队列中的节点为 3 - 2 - 1

// 将迭代器指针退回到第一个节点（按入列顺序）
$queue->rewind();

// 得到当前节点的值，如果直接使用，得到的结果是NULL，必须在rewind之后使用
$current = $queue->current();

// 当前节点的序号（按入列顺序，从0开始）
$queue->key();

// 当前迭代器指向的节点是否为空
$valid = $queue->valid();

// 迭代器指向下一个节点（按入列顺序）
$queue->next();
```
# 优先级队列
[PHP: SplPriorityQueue - Manual](https://www.php.net/manual/zh/class.splpriorityqueue.php)

| **概念**          | **描述**                                           |
| --------------- | ------------------------------------------------ |
| **定义**          | PHP SPL 提供的基于**堆 (Heap)** 的数据结构，用于存储带优先级的元素。     |
| **核心特性**        | **最大堆 (Max Heap)**：总是先提取**优先级最高**（数字最大）的元素。      |
| **关键方法**        | `insert(data, priority)`                         |
|                 | `extract()`                                      |
|                 | `isEmpty()`                                      |
|                 | `count()`                                        |
| **提取模式**        | `setExtractFlags()`                              |
| **稳定性**         | **非稳定**：如果两个元素的优先级相同，它们出队的顺序是不确定的（通常是后插入的先出）。    |
| **Min Heap 实现** | 如果需要最小堆（优先级数字最小的先出），可以在插入时将优先级取负值 (`-priority`)。 |
```php
// 创建并设置提取模式：只返回数据
$queue = new SplPriorityQueue();
$queue->setExtractFlags(SplPriorityQueue::EXTR_DATA);

// 插入任务（数据, 优先级）
$queue->insert('Task A (P100)', 100);
$queue->insert('Task B (P10)', 10);
$queue->insert('Task C (P80)', 80);
$queue->insert('Task D (P5)', 5);
$queue->insert('Task E (P100)', 100);

echo "任务处理顺序 (高优先级优先):\n";

// 循环提取并处理任务
while (!$queue->isEmpty()) {
    echo $queue->extract() . "\n";
}
/*
任务处理顺序 (高优先级优先):
Task A (P100)
Task E (P100)
Task C (P80)
Task B (P10)
Task D (P5)
*/
```