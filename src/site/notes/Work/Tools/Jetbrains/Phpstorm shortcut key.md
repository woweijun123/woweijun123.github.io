---
{"dg-publish":true,"permalink":"/Work/Tools/Jetbrains/Phpstorm shortcut key/","title":"Phpstorm shortcut key","tags":["flashcards"],"noteIcon":"","created":"2023-10-03T05:39:37.000+08:00","updated":"2026-03-26T11:07:57.371+08:00","dg-note-properties":{"title":"Phpstorm shortcut key","tags":["flashcards"],"reference linking":null}}
---

# IntelliJ IDEA 中有什么让你相见恨晚的技巧？ 
- https://www.zhihu.com/question/300830746
- https://github.com/xiaoxiunique/tool-tips
- https://novnan.github.io/PhpStorm/phpstorm-useful-shortcuts/
- https://www.jetbrains.com/help/phpstorm/setting-background-image.html
# 快捷键
```shell
ctrl+j         # 插入活动代码提示
ctrl+q         # 查看代码注释
ctrl+h         # 显示类结构图
ctrl+p         # 显示默认参数
ctrl+F12       # 当前文件的方法列表
ctrl+shift+F12 # 显示/隐藏工具条
ctrl+shift+i   # 查看变量或方法定义源, 有的时候我们不想进入方法内部, 或者进入类的内部查看细节, 想要在外面就探查清楚, 就可以使用此种方法
ctrl+shift+/   # 块注释
ctrl+[]        # 匹配 {}[]
ctrl+shift+[]  # 选中块代码<table>....</table>
ctrl+alt+←/→   # 返回上次编辑的位置
ctrl+alt+[/]   # 切换上一个或下一个项目窗口
ctrl+alt+o     # 删除类中没有用到的package
ctrl+alt+u     # 显示类之间的关系

alt+left/right # 标签切换
alt+f1         # 在项目中定位当前文件所在位置, ctrl+鼠标左键 点击标签栏也可查询文件位置
alt+f7         # 查找所有使用当前方法的代码位置地址
alt+shift+\    # 树视图中的详细信息
alt+shift+ins  # 纵向光标选择
alt+j          # 依次多选编辑, 同sublime的ctrl+d反方向用alt+shift+j
ctrl+shift+alt+j # 全局选中, alt+j 加强版

shift+del      # 删除到行尾 
shift+F6       # 批量重命名变量，或修改文件名
F2             # 快速定位错误或警告，反向用shift+f2
F3             # 向下查找关键字出现位置
F4             # 查找变量来源/源码
F5             # 复制文件/文件夹
F6             # 移动
ctrl+1         # 将所选表达式的结果放入局部变量中
ctrl+2         # 用常量(静态final字段)替换选定的表达式
ctrl+3         # 将所选表达式的结果放入类属性中
ctrl+4         # 将选择的表达式转换为方法参数
ctrl+alt+m     # 将选择的代码段转换为方法

shfit+F9       # 运行 debug 模式
alt+shfit+F9   # 从多项目中启动一个 debug 模式
alt+shift+u     # 切换命名规范(驼峰、下换线等切换), 需要装CamelCase插件

ctrl+n           # 使用类名来查找即可
ctrl+shift+n     # 使用文件名来查找即可
ctrl+shift+alt+n # 使用symbol(符号)来查找即可
ctrl+shift+alt+m # 方法{}[]区域范围切换
ctrl+shift+enter # 快速补全行末分号
shift*2          # 查找所有文件和目录

ctrl+shift+a 输入 open recent   # 快速打开最近打开过的项目
ctrl+shift+a 输入 import setting export setting # 快速导入导出 idea 设置
ctrl+shift+a 输入 git clone # 快速克隆项目
```
# Setting Repository 常见问题
[phpstorm使用Setting Repository报错Authentication failed – 行星带 (beltxman.com)](https://beltxman.com/2244.html?replytocom=7505)
[(11条消息) 使用idea配置Settings Repository时报错Commit on repo without HEAD currently not supported_秋楓的博客-CSDN博客](https://blog.csdn.net/qq_34869990/article/details/103398653)
