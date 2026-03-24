---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Learn/file_get_contents vs POST/","title":"file_get_contents vs POST","tags":["flashcards"],"noteIcon":"","created":"2024-02-22T01:01:02.000+08:00","updated":"2026-03-24T17:47:31.761+08:00"}
---

### 1. 发送数据的文件
```php
$data = ['username' => '周伯通', 'password' => '123456', 'sign'=>'asdfg123456'];
$url = 'http://xxx.com/fpost.php';//fpost.php是接受数据的文件，代码在下面
$ch = curl_init(); //初始化curl
curl_setopt($ch, CURLOPT_URL, $url);//设置链接
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);//设置是否返回信息
curl_setopt($ch, CURLOPT_POST, 1);//设置为POST方式
curl_setopt($ch, CURLOPT_POSTFIELDS, $data);//POST数据
curl_setopt($ch, CURLOPT_USERAGENT, "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; .NET CLR 1.1.4322; .NET CLR 2.0.50727)");
$response = curl_exec($ch);//接收返回信息
if(curl_errno($ch)){//出错则显示错误信息
    print curl_error($ch);
}
curl_close($ch); //关闭curl链接
echo $response;//显示返回信息
```
### 2. 接受数据的文件
```php
$res = file_get_contents('php://input');
var_dump('file_get_contents 数据是:'.$res);
echo'<br> post 数据是:';
var_dump($_POST);
/*
string(97) "file_get_contents 数据是:username=%E5%91%A8%E4%BC%AF%E9%80%9A&password=123456&sign=asdfg123456" 
post 数据是:array(3) { 
    ["username"]=> string(9) "周伯通" 
    ["password"]=> string(6) "123456" 
    ["sign"]=> string(11) "asdfg123456" 
}
*/
```
运行后我们会发现：**`file_get_contents('php://input')`不能接收curl post过来的数组**。
### 解释：
- `$_POST`获取：原始数据是**一维数组**或**键值对字符串(&拼接)**。
- `file_get_contents`获取：原始数据是**json字符串**或使用**http_build_query**：比如上面修改如下：
```php
curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query($data)); // raw数据
```