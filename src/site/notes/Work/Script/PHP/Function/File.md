---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/File/","title":"File","tags":["flashcards"],"noteIcon":"","created":"2024-09-30T11:34:34.000+08:00","updated":"2026-03-24T17:38:05.575+08:00","dg-note-properties":{"title":"File","tags":["flashcards"],"reference linking":null}}
---

# 文件hash
## hash_file
计算指定文件的 MD5 散列值
```php
/* 创建一个要计算哈希值的文件 */
file_put_contents('example.txt', 'test');

echo hash_file('md5', 'example.txt');
// 098f6bcd4621d373cade4e832627b4f6
```
使用场景:
判断一个文件是否已经上传(避免重复上传同一文件)
## md5_file
计算指定文件的 MD5 散列值
```php
/* 创建一个要计算哈希值的文件 */
file_put_contents('example.txt', 'test');

echo md5_file('example.txt');
// 098f6bcd4621d373cade4e832627b4f6
```
使用场景:
判断一个文件是否已经上传(避免重复上传同一文件)
# 获取文件的mime类型
```php
echo finfo_file(finfo_open(FILEINFO_MIME_TYPE), 'disable.xlsx'); // application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
```
# move_uploaded_file
将上传的文件移动到新位置

本函数检查并确保由filename指定的文件是合法的上传文件(即通过 PHP 的 HTTP POST 上传机制所上传的)
如果文件合法，则将其移动为由destination指定的文件。 
这种检查显得格外重要，如果上传的文件会影响用户或本系统的其他用户的话。
```php
foreach ($_FILES["pictures"]["error"] as $key => $error) {
    if ($error == UPLOAD_ERR_OK) {
        $tmp_name = $_FILES["pictures"]["tmp_name"][$key];
        $name = $_FILES["pictures"]["name"][$key];
        move_uploaded_file($tmp_name, "/uploads/$name");
    }
}
```
# is_uploaded_file
判断文件是否是通过 HTTP POST 上传的
```php
if (is_uploaded_file($_FILES['userfile']['tmp_name'])) {
    echo "File " . $_FILES['userfile']['name'] . " uploaded successfully.\n";
    echo "Displaying contents\n";
    readfile($_FILES['userfile']['tmp_name']);
} else {
    echo "Possible file upload attack: ";
    echo "filename '" . $_FILES['userfile']['tmp_name'] . "'.";
}
```