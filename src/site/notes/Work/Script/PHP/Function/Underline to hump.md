---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Function/Underline to hump/","title":"Underline to hump","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:25:46.000+08:00","updated":"2026-04-07T15:02:19.962+08:00","dg-note-properties":{"title":"Underline to hump","tags":["flashcards"],"reference linking":null}}
---

```php
// 💡推荐：下划线转驼峰(首字母小写)
function underlineToHump($str)
{
    return lcfirst(strtr(
        ucwords(strtr($str, ['_' => ' '])),
        [' ' => '']
    ));
}
echo underlineToHump('hello_world'); // helloWorld

// 下划线转驼峰
function underlineToHumpTwo($str)
{
    return preg_replace_callback(
        '/[-_]+([a-z]{1})/i',
        function ($e) {
            return strtoupper($e[1]);
        },
        $str
    );
    // return ucfirst($result); // 首字母大写
}
echo underlineToHumpTwo('hello_world'); // helloWorld

// 驼峰转下划线
function hump_to_underline($str)
{
    return preg_replace_callback('/([A-Z]{1})/', function ($m) {
        return '_' . strtolower($m[0]);
    }, $str);
}

// 驼峰转下划线(数组处理)
function hump_to_underline_recursive($data = [])
{
    $result = [];
    foreach ($data as $k => $v) {
        if (is_array($v) || is_object($v)) {
            $result[hump_to_underline($k)] = hump_to_underline_recursive((array)$v);
        } else {
            $result[hump_to_underline($k)] = hump_to_underline($v);
        }
    }
    return $result;
}

var_export(convertHump([
    'helloWorld' => 'helloWorld',
    'helloWorldTwo' => [
        'helloWorldThree' => 'helloWorldThree
    ']
]));
/*
array (
  'hello_world' => 'hello_world',
  'hello_world_two' => 
  array (
    'hello_world_three' => 'hello_world_three
    ',
  ),
)
*/
```

