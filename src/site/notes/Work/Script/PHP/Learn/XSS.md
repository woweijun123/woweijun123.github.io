---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/XSS/","title":"XSS","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:26:36.000+08:00","updated":"2026-03-24T17:47:57.684+08:00","dg-note-properties":{"title":"XSS","tags":["flashcards"],"reference linking":null}}
---

```php
htmlspecialchars() // 将特殊字符转换为 HTML 实体
htmlspecialchars_decode() // 将特殊的 HTML 实体转换回普通字符

htmlentities() // 将字符转换为 HTML 转义字符
html_entity_decode() // 将HTML实体转换为它们对应的字符
```

```php
$str = '<script>alert(\'€\')</script>';

// 特殊字符转换为HTML实体「不转义 单引号」
$htmlEntities = htmlentities($str);
echo $htmlEntities, PHP_EOL; // &lt;script&gt;alert(&#039;&euro;&#039;)&lt;/script&gt;
// HTML实体转换为特殊字符
echo html_entity_decode($htmlEntities), PHP_EOL; // <script>alert('€')</script>

// 特殊字符转换为HTML实体「转义单引号」
echo htmlentities($str, ENT_QUOTES, 'UTF-8'), PHP_EOL; // &lt;script&gt;alert(&#039;&euro;&#039;)&lt;/script&gt;


// 特殊字符转换为HTML实体「不转义 单引号、€」
$htmlSpecialChars = htmlspecialchars($str);
echo $htmlSpecialChars, PHP_EOL; // &lt;script&gt;alert(&#039;€&#039;)&lt;/script&gt;
// HTML实体转换为特殊字符
echo htmlspecialchars_decode($htmlSpecialChars), PHP_EOL; // <script>alert('€')</script>

// 特殊字符转换为HTML实体「不转义 €」
echo htmlspecialchars($str, ENT_QUOTES, 'UTF-8'), PHP_EOL; // &lt;script&gt;alert(&#039;€&#039;)&lt;/script&gt;
```
结论: 做一般表单提交的时候完全可以用strip_tags去除html标签，如果涉及到富文本编辑器需要保留html标签，可以用htmlspecialchars对提交数据进行过滤。