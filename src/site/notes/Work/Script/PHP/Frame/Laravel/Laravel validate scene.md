---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel validate scene/","title":"Laravel validate scene","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:47:15.000+08:00","updated":"2026-03-24T17:34:07.290+08:00"}
---

参考链接: https://www.jb51.net/article/184253.htm
## 前言
在我们使用 laravel 框架的验证器，有的时候需要对表单等进行数据验证，当然 laravel 也为我们提供了Illuminate\Http\Request 对象提供的 validate 方法 以及 FormRequest 和Validator。

FormRequest 通过新建文件将我们的验证部分单独分开，来避免控制器臃肿。如果验证失败，就会生成一个让用户返回到先前的位置的重定向响应。这些错误也会被闪存到 Session 中，以便这些错误都可以在页面中显示出来。

如果传入的请求是 AJAX，会向用户返回具有 422 状态代码和验证错误信息的 JSON 数据的 HTTP 响应。如果是接口请求或 ajax, 那么我们可能还需要将返回的 json 数据修改成我们想要的格式。

当我们实际开发中，可能一个模块需要有多个验证场景，如果为每一个验证场景都新建一个 FormRequest 不就太过繁琐了。那么给 laravel 加上一个验证场景通过一个验证类一个模块或多个模块来适应不同的场景不就方便很多了。

## 开始
首先 我们封装了一个基类 BaseValidate.php 并将其放在 app\Validate 下，当然你也可以放在其他地方，只要修改好命名空间就好。
```php
/**
 * 基础验证类(支持验证场景)
 * @Author   Wcj
 * @email 1054487195@qq.com
 * @DateTime 2021/2/7 16:58
 */

namespace App\Api\Validator;

use App\Exceptions\CustomException;
use Illuminate\Support\Facades\Validator;

/**
 * 扩展验证器
 */
class BaseValidator
{
    /**
     * 当前验证规则
     * @var array
     */
    protected $rule = [];
    /**
     * 验证提示信息
     * @var array
     */
    protected $message = [];
    /**
     * 验证场景定义
     * @var array
     */
    protected $scene = [];
    /**
     * 设置当前验证场景
     * @var array
     */
    protected $currentScene = null;
    /**
     * 验证失败错误信息
     * @var array
     */
    protected $error = [];
    /**
     * 场景需要验证的规则
     * @var array
     */
    protected $only = [];

    /**
     * 设置验证场景
     * @access public
     * @param string $name 场景名
     * @return $this
     */
    public function scene($name)
    {
        // 设置当前场景
        $this->currentScene = $name;

        return $this;
    }

    /**
     * 数据验证
     * @Author Wcj
     * @email 1054487195@qq.com
     * @DateTime 2021/2/7 17:22
     * @param array $data 数据
     * @param array $rules 验证规则
     * @param array $message 自定义验证信息
     * @param string $scene 验证场景
     * @return array|bool
     * @throws CustomException
     */
    public function check($data = [], $rules = [], $message = [], $scene = '')
    {
        if (empty($data)) {
            $data = request()->all(); // 若没传数据则取请求的数据
        }
        $this->error = [];
        if (empty($rules)) {
            // 读取验证规则
            $rules = $this->rule;
        }
        if (empty($message)) {
            $message = $this->message;
        }
        // 读取场景
        if (!$this->getScene($scene)) {
            throw new CustomException('请先设置场景');
        }
        // 如果场景需要验证的规则不为空
        if (!empty($this->only)) {
            $new_rules = [];
            foreach ($this->only as $value) {
                if (array_key_exists($value, $rules)) {
                    $new_rules[$value] = $rules[$value];
                }
            }
            $rules = $new_rules;
        }
        $validator = Validator::make($data, $rules, $message);
        // 验证失败
        if ($validator->fails()) {
            throw new CustomException($validator->errors()->first());
        }
        return !empty($this->error) ? false : include_keys($validator->getData(), array_keys($rules));
    }

    /**
     * 获取数据验证的场景
     * @Author Wcj
     * @email 1054487195@qq.com
     * @DateTime 2021/2/7 17:00
     * @param string $scene
     * @return bool
     */
    protected function getScene($scene = '')
    {
        if (empty($scene)) {
            // 读取指定场景
            $scene = $this->currentScene;
        }
        $this->only = [];

        if (empty($scene)) {
            return true;
        }
        if (!isset($this->scene[$scene])) {
            // 指定场景未找到写入error
            $this->error = "scene:" . $scene . 'is not found';
            return false;
        }
        // 如果设置了验证适用场景
        $scene = $this->scene[$scene];
        if (is_string($scene)) {
            $scene = explode(',', $scene);
        }
        // 将场景需要验证的字段填充入only
        $this->only = $scene;
        return true;
    }

    // 获取错误信息
    public function getError()
    {
        return $this->error;
    }
}
```

## 使用
接下来我们来验证一个文章的提交信息，首先我们新建一个文章验证类 ArticleValidate.php 并填充一些内容
```php
namespace App\Validate;
use App\Validate\BaseValidate;
/**
 * 文章验证器
 */
class ArticleValidate extends BaseValidate
{
    // 验证规则
    protected $rule = [
        'id' => 'required',
        'title' => 'required|max:255',
        'content' => 'required',
    ];
    // 自定义验证信息
    protected $message = [
        'id.required' => '缺少文章id',
        'title.required' => '请输入title',
        'title.max' => 'title长度不能大于 255',
        'content.required' => '请输入内容',
    ];
    // 自定义场景
    protected $scene = [
        'add' => "title,content",
        'edit' => ['id', 'title', 'content'],
    ];
}
```

如上所示，在这个类中我们定义了验证规则 rule，自定义验证信息 message，以及验证场景 scene

## 非场景验证
我们只需要定义好规则
```php
public function update()
{
    $ArticleValidate = new ArticleValidate;
    $request_data = [
        'id' => '1',
        'title' => '我是文章的标题',
        'content' => '我是文章的内容',
    ];
    if (!$ArticleValidate->check($request_data)) {
        var_dump($ArticleValidate->getError());
    }
}
```

如果验证未通过我们调用getError()方法来输出错误信息,getError()暂不支持返回所有验证错误信息 。

## 场景验证
我们需要提前在验证类中定义好验证场景

如下，支持使用字符串或数组，使用字符串时，要验证的字段需用，隔开
```php
// 自定义场景
protected $scene = [
    'add' => "title,content",
    'edit' => ['id', 'title', 'content'],
];
```

然后在我们的控制器进行数据验证
```php
public function add()
{
    $ArticleValidate = new ArticleValidate;
    $request_data = [
        'title' => '我是文章的标题',
        'content' => '我是文章的内容',
    ];
    if (!$ArticleValidate->scene('add')->check($request_data)) {
        var_dump($ArticleValidate->getError());
    }
}
```

## 控制器内验证
当然我们也允许你不创建验证类来验证数据，
```php
public function add()
{
    $Validate = new BaseValidate;
    $request_data = [
        'title' => '我是文章的标题',
        'content' => '我是文章的内容',
    ];
    $rule = [
        'id' => 'required',
        'title' => 'required|max:255',
        'content' => 'required',
    ];
    // 自定义验证信息
    $message = [
        'id.required' => '缺少文章id',
        'title.required' => '请输入title',
        'title.max' => 'title长度不能大于 255',
        'content.required' => '请输入内容',
    ];
    if (!$Validate->check($request_data, $rule, $message)) {
        var_dump($Validate->getError());
    }
}
```

通过验证场景，既减少了控制器代码的臃肿，又减少了 FormRequest 文件过多，还可以自定义 json 数据是不是方便多了呢，