---
{"dg-publish":true,"permalink":"/Work/Tools/Jetbrains/PhpStorm 函数注释设置/","title":"PhpStorm 函数注释设置","tags":["jetbrains","phpstorm"],"noteIcon":"","created":"2026-06-26T18:50:21.107+08:00","updated":"2026-06-26T18:51:01.890+08:00","dg-note-properties":{"title":"PhpStorm 函数注释设置","tags":["jetbrains","phpstorm"],"reference linking":"[PhpStorm函数注释的设置](https://www.cnblogs.com/Steven-shi/p/8136081.html)"}}
---

# 概述
PhpStorm 方法注释默认模板在 **Settings → Editor → File and Code Templates → Includes → PHP Function Doc Comment**。若直接把文件头模板里的 `${USER}`、`${DATE}`、`${TIME}` 复制进方法注释模板，占位符**不会被替换**，会原样输出。
# 默认模板
## 方法注释
```php
/**
${PARAM_DOC}
#if (${TYPE_HINT} != "void") * @return ${TYPE_HINT}
#end
${THROWS_DOC}
*/
```
## 文件头（占位符可用）
```php
/**
 * Created by ${PRODUCT_NAME}.
 * User: ${USER}
 * Date: ${DATE}
 * Time: ${TIME}
 */
```
# 解决步骤
1. **Settings → Editor → Live Templates**，点 `+` 新建 **Live Template**（按向导配置变量与适用范围）
2. 回到 **File and Code Templates → Includes → PHP Function Doc Comment**，改为：
```php
/**
 * Notes:
 * User: ${USER}
 * Date: ${DATE}
 * Time: ${TIME}
${PARAM_DOC}
#if (${TYPE_HINT} != "void") * @return ${TYPE_HINT}
#end
${THROWS_DOC}
*/
```
3. Apply → OK；在方法前输入 `/**` 回车即可生成
# 生成效果
```php
/**
 * Notes: 下单接口
 * User: Steven
 * Date: 2017/12/28
 * Time: 15:19
 * @return array
 * @throws \yii\db\Exception
 */
```
# 一句话总结
方法注释要自动填充 User/Date/Time，需先建 Live Template 再改 PHP Function Doc Comment 模板，不能仅靠 File and Code Templates 直接复用文件头占位符。
