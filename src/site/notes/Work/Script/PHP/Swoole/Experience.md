---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Swoole/Experience/","title":"Experience","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:16:17.000+08:00","updated":"2026-03-24T17:50:03.758+08:00","dg-note-properties":{"title":"Experience","tags":["flashcards"],"reference linking":null}}
---

TP框架继承swoole问题
1. 在swoole的http服务下, 基类(父类)只会执行一次
2. 公共函数文件写函数会报错
3. 使用swoole语法重启swoole和kill -usr1 均对swoole自身类不生效, 仅对框架内容生效
4. 别用index方法名, 导致不管执行哪个方法都会执行index方法