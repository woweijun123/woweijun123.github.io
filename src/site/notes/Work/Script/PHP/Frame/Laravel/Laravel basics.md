---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Laravel/Laravel basics/","title":"Laravel basics","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:45:00.000+08:00","updated":"2026-03-24T17:34:02.270+08:00"}
---

# 安装(composer)
```shell
# 安装最新版本
composer create-project --prefer-dist laravel/laravel laravel
# 安装指定版本
composer create-project --prefer-dist laravel/laravel=5.* laravel
```
# Artisan使用
```shell
php artisan make:controller TestController # 创建控制器
php artisan make:model Models/TestModel # 创建模型
php artisan make:request TestRequest # 创建验证类
php artisan make:middleware Test # 创建中间件
php artisan cache:forget spatie.permission.cachefor # 后台框架清缓存

# 清除配置缓存文件
php artisan config:clear
# 清除应用缓存文件
php artisan cache:clear
# 清除路由缓存文件
php artisan route:clear
# 清除视图缓存文件
php artisan view:clear
# 清除优化文件（包括缓存、编译文件等）
php artisan optimize:clear

# 生成配置缓存文件，提高配置加载速度（生产环境推荐）
php artisan config:cache
# 生成路由缓存文件，提高路由注册速度（生产环境推荐）
php artisan route:cache
# 生成视图缓存文件，提高视图渲染速度
php artisan view:cache
# 优化框架加载，包括合并引导文件等（Laravel 9+ 中功能被 optimize:clear 取代或分散到其他缓存命令）
php artisan optimize
```
# Auth使用
```shell
php artisan make:auth # 生成Auth所需文件
```
# 迁移文件
```shell
# 创建迁移文件
# create_test_table(迁移文件名)  
# --table=test2 (确定迁移文件中表的名称)
# --create=test3 (迁移中创建新的数据表, 暂时没测试出来效果)
php artisan make:migration create_test_table --create=test
# 创建模型并生成迁移文件
php artisan make:model Models/Article -m 

# 迁移
php artisan migrate
# 强制迁移
php artisan migrate --force

# 回滚
php artisan migrate:rollback
# 回滚最后5个迁移
php artisan migrate:rollback --step=5
# 回滚所有迁移
php artisan migrate:reset
# 所有迁移文件, 回滚并迁移
php artisan migrate:refresh
# 最后5个迁移文件, 回滚并迁移
php artisan migrate:refresh --step=5

# 删除所有数据表和迁移
php artisan migrate:fresh
```
# 数据填充
```shell
# 创建填充文件
php artisan make:seeder StudentTableSeeder

# 填充指定文件
php artisan db:seed --class=StudentTableSeeder
# 填充所有文件
php artisan db:seed
# 强制填充所有文件
php artisan db:seed --force

# 删除所有表并重新运行所有迁移
php artisan migrate:fresh --seed
```
# 文件系统
1. Laravel的文件系统是基于Frank de Jonge的Flysystem扩展包
2. 提供了简单的接口,可以操作本地端空间、AmazonS3、Rackspace Cloud Storage
3. 可以非常简单的切换不同保存方式,但仍使用相同的API操作
# 邮件(SwiftMailer)
1. Laravel的邮件功能基于热门的SwiftMailer函数库之上,提供了一个简洁的API
2. Laravel为SMTP、Mailgun、Mandrill、AmazonSES、PHP的mail函数、以及sendmail提供了驱动从而允许你快速通过本地或云服务发送邮件
# 发送邮件
配置
```php
config/mail.php
```
发送
```php
Mail:raw() 
Mail:send()
```
# 时间函数
[Carbon - A simple PHP API extension for DateTime. (nesbot.com)](https://carbon.nesbot.com/docs/#api-addsub)
```php
Carbon::now(); // 获取当前时间对象
Carbon::parse(date('Y-m-d H:i:s', $studentInfo['first_register_time']))->diffInDays(Carbon::now()); // 计算相差日期
Carbon::now()->startOfDay()->toDateTimeString(); // 开始时间 2021-07-29 00:00:00
Carbon::now()->copy()->subDay(7)->startOfDay()->toDateTimeString(); // 7日前的结束时间 2021-07-22 23:59:59

// 获取全球唯一偏移量及其对应的 UTC 时间范围
$offsetRanges = collect(DateTimeZone::listIdentifiers())  
    ->unique()  
    ->reduce(function ($carry, $tz) use ($baseUtcTime) {  
        $timezone = new DateTimeZone($tz);  
        // 获取该时区下的 UTC 时间  
        $carbon         = $baseUtcTime->copy()->setTimezone($timezone);  
        // 计算该时区下的"昨天"  
        $localYesterday = $carbon->subDay();  
        // 使用标准格式生成时区偏移字符串 - 基于baseUtcTime确保时间一致性  
        $offsetTag = 'UTC' . $carbon->format('P');  
        $carry[$offsetTag] = [  
            'day'   => $localYesterday->toDateString(),  
            'start' => $localYesterday->copy()->startOfDay()->timestamp,  
            'end'   => $localYesterday->copy()->endOfDay()->timestamp,  
        ];  
        return $carry;  
    }, []);
```
# DBfacade
## 增
```php
// 新增一条
DB::table('users')->insert(['name' => 'riven', 'age' => 18]);
// 新增一条并且返回主键ID
DB::table('users')->insertGetId(['name' => 'riven', 'age' => 18]);
// 批量新增(适用于多条记录，不触发模型事件，不填充时间戳)
DB::table('users')->insert([
    ['name' => 'riven', 'age' => 18], ['name' => 'loren', 'age' => 18], ['name' => 'gamer', 'age' => 18]
]);
```
## 删
```php
DB::table('users')->delete();
DB::table('users')->where('votes', '>', 100)->delete();
// 清空表
DB::table('users')->truncate();
```
## 改
```php
// 更新
DB::table('users')->where('id', 1)->update(['votes' => 1]);
// 有则更、无责增
// 用第一个参数的键和值对来查找匹配的数据库记录。 如果记录存在，则使用第二个参数中的值去更新记录。 如果找不到记录，将插入一个新记录
DB::table('users')->updateOrInsert(
    ['email' => 'john@example.com', 'name' => 'John'],
    ['votes' => '2']
);

// 自增并更新其他字段
DB::table('users')->increment('votes', 1, ['name' => 'John']);
// 自增
DB::table('users')->increment('votes');
DB::table('users')->increment('votes', 5);

// 自减
DB::table('users')->decrement('votes');
DB::table('users')->decrement('votes', 5);

// 批量更新
public static function updateBatch(string $tableName = '', array $multipleData = []): int
{
    if (empty($multipleData)) {
        throw new Exception('数据不能为空');
    }
    $firstRow = reset($multipleData);
    $updateColumn = array_keys($firstRow);
    // 以ID或首个字段为条件
    $referenceColumn = isset($firstRow['id']) ? 'id' : reset($updateColumn);
    // 剔除条件字段
    unset($updateColumn['0']);
    $sets = $bindings = [];
    foreach ($updateColumn as $column) {
        $setSql = "`$column` = CASE ";
        foreach ($multipleData as $data) {
            $setSql .= "WHEN `$referenceColumn` = ? THEN ? ";
            $bindings[] = $data[$referenceColumn];
            $bindings[] = $data[$column];
        }
        $setSql .= "ELSE `$column` END";
        $sets[] = $setSql;
    }
    // 条件
    $whereIn = rtrim(str_repeat('?,', count($multipleData)), ',');
    $sets = implode(', ', $sets);
    // 传入预处理sql语句和对应绑定数据
    return DB::update(
        "UPDATE $tableName SET $sets WHERE `$referenceColumn` IN ($whereIn)",
        array_merge($bindings, array_column($multipleData, $referenceColumn))
    );
}
self::updateBatch(app(ArticleModel::class)->getTable(), [
    ['id' => 1, 'title' => '张三'],
    ['id' => 2, 'title' => '李四'],
]);
// 生成的sql语句如下
UPDATE `article`
SET `title` = CASE WHEN `id` = 1 THEN "张三" WHEN `id` = 2 THEN "李四" ELSE `title` END
WHERE `id` IN (1, 2)
```
## 查
```php
// 查询单条
$user = DB::table('users')->where('name', 'John')->first();
// 单列
$email = DB::table('users')->where('name', 'John')->value('email');
// 单列值的集合
$data = DB::table('roles')->pluck('title'); // ->lists('title') 同样效果
/* 返回结构如下
$data = [
    'title1',
    'title2',
    'title3',
];
*/

// 集合中指定字段的自定义键值
$data = DB::table('roles')->pluck('title', 'id'); // ->lists('title', 'id') 同样效果
/* 返回结构如下
$data = [
    1 => 'title1',
    2 => 'title2',
    3 => 'title3',
];
*/

// 一次获取结果集的一小块，并将其传递给闭包函数进行处理(每次取100条数据)
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    foreach ($users as $user) {
        // return false; // 返回false来终止继续获取分块结果
    }
});

// 获取多条
DB::table('student')->get();
// 获取多条，并以ID为键值
DB::table('student')->get()->keyBy('id');
```
# Eloquent ORM
## 增
```php
// 创建新对象新增
$student = new Student();
$student->name = 'riven';
$student->save();
// 新增数据, 返回student对象
$student = Student::create(['name' => 'riven', 'age' => 18]);
// 有则返, 无责增
$student = Student::firstOrCreate(['name' => 'riven', 'age' => 18]);
// 有则返, 无责new
$student = Student::firstOrNew(['name' => 'riven', 'age' => 18]);


// 假设一个 Post 模型有多个 Comment(适用于多条记录，触发模型事件，填充时间戳，推荐使用)
// 只适用于 父子关联关系（例如一个 `Post` 有多个 `Comment`）。
$post = Post::find(1);
$post->comments()->createMany([
    ['content' => '这是第一条评论', 'user_id' => 1],
    ['content' => '这是第二条评论', 'user_id' => 2],
    ['content' => '这是第三条评论', 'user_id' => 1],
]);
```
## 删
```php
// 查询删除
$student = Student::find(1);
$student->delete();
// 批量删除
Student::query()->where(['id' => [1, 2, 3]])->delete();
// 根据主键删除, 删除ID为1和2的数据
Student::destory([1, 2]);
```
## 改
```php
// 返回更新的行数
$num = Student::where(['name' => 'riven'])->update(['name' => 'loren']);
// 查询更新
$student = Student::find(1);
$student->name = 'loren';
$student->save();

// 批量更新
$student = Student::find(1);
$student->fill([
    'name' => 'loren',
    'address' => 'china'
]);
$student->save();
```
## 查
# 查询作用域
```php
/**
 * 本地查询作用域：批量设置订单动作
 * @param Builder $query
 * @param array   $orderActions
 * @return Builder
 */
#[Scope]
protected function withAction(Builder $query, array $orderActions): Builder
{
	return $query->where(function ($q) use ($orderActions) {
		$orderStatusArr = Transition::orderStatusArr(...$orderActions);
		foreach ($orderStatusArr as $orderStatus) {
			$q->orWhere(function ($subQ) use ($orderStatus) {
				$subQ->where('status', $orderStatus[0])
					 ->where('action_role', $orderStatus[1])
					 ->where('action_status', $orderStatus[2]);
			});
		}
	});
}
```
# whereHas查询慢的问题
[\[扩展包\] Laravel-wherehasin 提升 ORM 关联关系查询性能 （优化 whereHas 性能） | Laravel China 社区](https://learnku.com/articles/46696)
```shell
composer require dcat/laravel-wherehasin
```
```php
// 查询多条
Student::all();
Student::query()->get();
// 查询多条, 并以ID为键值
Student::all()->keyBy('id');
Student::query()->get()->keyBy('id');

// 查询ID=1的单条数据, 不存在返回null
Student::find(1);
// 查询ID=1的单条数据, 若不存在则抛异常
Student::findOrFail(1);
// 查询ID=1的单条数据, 不存在返回null
Student::query()->where(['id' => 1])->first()
// 查询ID=1的单条数据, 不存在返回空数组[]
collect(Student::query()->where(['id' => 1])->first())->toArray()

// 关联查询, 以关联表的字段进行筛选
$where = [];
if ($sendWordStatus = get_int($this->input, StudentCourseModel::SEND_WORD_STATUS)) {
    $where[StudentCourseModel::SEND_WORD_STATUS] = $sendWordStatus;
}
// 筛选学生用户名/姓名/备注名
$keyword = get_string($this->input, StudentCourseModel::KEYWORD);
StudentCourseModel::query()
->where($where)
->whereHas('student', function ($query) use ($keyword) {
    $query->where('student_name', 'like', "%${keyword}%")
        ->orWhere('real_name', 'like', "%${keyword}%")
        ->orWhere('teacher_remark_name', 'like', "%${keyword}%");
})
->with(['student:id,student_name,real_name,teacher_remark_name'])
->paginate($this->input['per_page'] ?? 10);
```
# 关联
```php
// 表a
// 表b
// 表c
class B
{
    public function c()
    {
        return $this->hasOne(C::class, 'c_id', 'id'); // c表的外键(c_id) 关联 b表的主键(id)
    }

    public function a()
    {
        return $this->belongsTo(A::class, 'a_id', 'id'); // b表的外键(a_id) 关联 a表的主键(id)
    }
}
```
# 判断集合是否为空
```php
$users = DB::table('users')->where('id', $id)->get();
// 若无数据，打印出来为null
if ($users->first()) {
}

if (!$users->isEmpty()) {
}

// 若无数据，打印出来为0
if ($users->count()) {
}
```
# 集合中优雅实现“退出”或“中断”逻辑
### 1. 使用 `contains` 或 `first` 模拟提前退出
如果你在寻找特定的元素，一旦找到就停止执行，`contains` 和 `first` 是天然的“短路（Short-circuit）”函数。它们在满足条件后不会继续遍历剩余元素。
```php
$users = collect([/* ... */]);

// 只要找到第一个活跃用户就会停止，不会遍历后续数据
$firstActiveUser = $users->first(function ($user) {
    return $user->active === true;
});
```
### 2. `until`：在特定条件下停止
这是处理“多层退出”逻辑最优雅的函数之一。`until` 会持续收集元素，直到闭包返回 `true` 为止（但不包括触发停止的那个元素）。
```php
$steps = collect(['step1', 'step2', 'error_occurred', 'step3']);

// 运行到 error_occurred 之前的所有步骤
$validSteps = $steps->until(function ($step) {
    return $step === 'error_occurred';
});

// 结果：['step1', 'step2']
```
### 3. `takeUntil` 与 `takeWhile`
这两个方法类似于过滤器，但它们具有**中断性**：
* **`takeWhile`**: 只要条件为真就继续取值，一旦遇到假，立即停止。
* **`takeUntil`**: 只要条件为假就继续取值，一旦遇到真，立即停止。
### 4. 处理嵌套集合：`flatMap` 的解构
当你有多层循环（例如：用户 -> 订单 -> 商品）时，使用 `flatMap` 可以将多维结构压平，从而避免在多层 `foreach` 中使用 `break 2` 这种容易出错的语法。
```php
// 优雅地寻找所有订单中是否存在某种危险商品
$hasHazardousItem = $users->flatMap->orders
    ->flatMap->items
    ->contains('type', 'hazardous');
```
### 5. 自定义“中断”信号
在非常复杂的逻辑中，如果你必须在 `each` 循环内中断，可以利用 Laravel 的特性：在 `each` 闭包中 **返回 `false**`。
```php
$collection->each(function ($item) {
    if ($item->should_stop) {
        return false; // 这相当于原生循环中的 break
    }
    
    // 逻辑处理...
});
```
### 总结与对比
| 场景          | 推荐方法                       | 效果           |
| ----------- | -------------------------- | ------------ |
| **寻找目标即停**  | `first()` / `contains()`   | 性能最优，找到即返回   |
| **获取断点前数据** | `until()` / `takeUntil()`  | 适合流式处理中断     |
| **深层嵌套查找**  | `flatMap()` + `contains()` | 消除多重循环嵌套     |
| **传统遍历中断**  | `each(fn => false)`        | 语义化的 `break` |
# 获取表前缀
```php
DB::getConfig('prefix');
DB::connection()->getTablePrefix();
Config::get('database.connections.mysql.prefix');
```
# sql语句调试
[laravel打印执行的Sql语句   -  叶落山城秋](https://www.iphpt.com/detail/75)
```php
// 1.断点sql调试
DB::enableQueryLog(); // 开启sql语句日志
$this->model->where($pk, 'in', $ids)->get(); // 调试的sql
dd(DB::getQueryLog());  // 打印sql语句

// 2.sql监听（AppServiceProvider，boot 中写入下面的代码）；log插件正则表达式：sql<(.)*>sql
if (env('APP_DEBUG', false)) {
    DB::listen(function ($query) {
        $tmp = str_replace('?', '"%s"', $query->sql);
        $qBindings = [];
        foreach ($query->bindings as $key => $value) {
            if (is_numeric($key)) {
                $qBindings[] = $value;
            } else {
                $tmp = str_replace(':' . $key, '"' . $value . '"', $tmp);
            }
        }
        $tmp = str_replace("\\", "", vsprintf($tmp, $qBindings));
        Log::info("\n{$query->time}ms;\n sql< {$tmp} >sql \n\r");
    });
}

// 3.打印包含参数的原生sql语句
Tool::enableQueryLog();
$this->model->where($pk, 'in', $ids)->get(); // 调试的sql
$sql = Tool::getQueryLog();

/**
 * 以build获取原生sql「带参数」
 * @param $query
 * @return string
 */
public static function getSql($query): string
{
    /* @var $query Builder */
    $bindings = $query->getBindings();
    $sql = $query->toSql();
    foreach ($bindings as $value) {
        $sql = Str::replaceFirst('?', "'$value'", $sql);
    }
    return $sql;
}

/**
 * 获取原生sql「带参数」
 * @param $sql
 * @param $bindings
 * @return string
 */
public static function getRawSql($sql, $bindings): string
{
    foreach ($bindings as $value) {
        $sql = Str::replaceFirst('?', "'$value'", $sql);
    }
    return $sql;
}

/**
 * 开启监听sql日志
 * @return void
 */
public static function enableQueryLog()
{
    DB::connection()->enableQueryLog();
}

/**
 * 返回监听到的sql语句
 * @return string
 */
public static function getQueryLog(): string
{
    if (
        ($queryLog = DB::connection()->getQueryLog()['1'] ?? []) &&
        !empty($queryLog['query']) &&
        !empty($queryLog['bindings'])
    ) {
        // 清除上一次原生sql语句
        DB::connection()->flushQueryLog();
        // 构建原生sql语句
        return Tool::getRawSql($queryLog['query'], $queryLog['bindings']);
    }
    return '';
}
```
# Guzzle(CURL)
文档：[Quickstart — Guzzle Documentation](https://docs.guzzlephp.org/en/stable/quickstart.html#making-a-request)
```php
// basic
$client = new Client([
    'base_uri' => 'http://www.laravel.com/api/',
    'timeout' => 5
]);

// post
$options = [
    'headers' => ['Authorization' => 'abc'],
    'form_params' => ['name' => '啦啦啦']
];
$method = 'test/add';
$data = json_decode($client->request('POST', $method, $options)->getBody()->getContents(), true);

// delete
$options = [
    'headers' => ['Authorization' => 'abc'],
    'query' => ['id' => 1]
];
$method = 'test/delete';
$data = json_decode($client->request('DELETE', $method, $options)->getBody()->getContents(), true);

// put
$options = [
    'headers' => ['Authorization' => 'abc'],
    'form_params' => ['id' => 1, 'name' => '啦啦啦']
];
$method = 'test/update';
$data = json_decode($client->request('PUT', $method, $options)->getBody()->getContents(), true);

// get
$options = [
    'headers' => ['Authorization' => 'abc'],
    'query' => ['name' => 'z3']
];
$method = 'test/read';
$data = json_decode($client->request('GET', $method, $options)->getBody()->getContents(), true);


// 并发请求代码
$route = route('api.test.concurrentOrder');

// 异步请求多条
$promises = [
    $client->postAsync($route, ['form_params' => ['num' => 1]]),
    $client->postAsync($route, ['form_params' => ['num' => 1]]),
    $client->postAsync($route, ['form_params' => ['num' => 1]]),
    $client->postAsync($route, ['form_params' => ['num' => 1]])
];
$results = Utils::unwrap($promises);
$list = [];
foreach ($results as $result) {
    $list[] = json_decode($result->getBody()->getContents(), true);
}

// 并发请求多次
$requests = function ($total) use ($client, $route) {
    for ($i = 0; $i < $total; $i++) {
        yield function () use ($client, $route) {
            return $client->postAsync($route, ['form_params' => ['num' => 1]]);
        };
    }
};

$success = $error = [];
(new Pool($client, $requests(10), [
    'concurrency' => 2,
    'fulfilled' => function (Response $response, $index) use (&$success) {
        $success[] = json_decode($response->getBody()->getContents(), true);
    },
    'rejected' => function (RequestException $reason, $index) use (&$error) {
        $error[] = json_decode($reason->getMessage(), true);
    },
]))->promise()->wait();
```
# 子查询
[聊一聊laravel query builder 使用子查询\_laravel 子查询作为表名-CSDN博客](https://blog.csdn.net/Tim_phper/article/details/78606253)
```php
// Eloquent
// 临时表子查询
$sub = UserModel::query()->whereIn(UserModel::ID, [1, 2, 3])->getQuery();
$ret = DB::query()->fromsub($sub, 'b')->groupby(['b.id'])->get();
// select * from (select * from `user` where `id` in ("1", "2", "3")) as `b` group by `b`.`id`

  
/*
    SELECT *
    FROM `supplier`
    INNER JOIN (SELECT * FROM `user` WHERE `id` IN ("1", "2", "3")) AS `b` ON `supplier`.`id` = `b`.`supplier_id`
    WHERE `supplier`.`deleted_at` IS NULL
    GROUP BY `b`.`id`
 */


// DB
$sub = UserModel::query()->selectRaw('count(id) AS count, id, supplier_id, name')
    ->where('supplier_id', '>', 0)
    ->groupBy(['id']);
$ret = DB::table(DB::raw("({$sub->toSql()}) as sub, supplier"))
    ->mergeBindings($sub->getQuery()) // 绑定参数
    ->select(['supplier.id', 'sub.name', 'count'])
    ->whereRaw('supplier.id = sub.supplier_id')
    ->get();
/*
    SELECT `supplier`.`id`, `sub`.`name`, `count`
    FROM (SELECT COUNT(`id`) AS `count`, `id`, `supplier_id`, `name`
    FROM `user`
    WHERE `supplier_id` > "0"
    AND `user`.`deleted_at` IS NULL
    GROUP BY `id`) AS `sub`,
    `supplier`
    WHERE `supplier`.`id` = `sub`.`supplier_id`
 */
```
# Eloquent 关联查询（起别名）
```php
UserModel::query()->from('user as user')->leftJoin('supplier as b', 'user.supplier_id', '=', 'b.id')->get();
// select * from `user` as `user` left join `supplier` as `b` on `user`.`supplier_id` = `b`.`id` where `user`.`deleted_at` is null
```
# JSON字段查询
```php
UserModel::query()->whereRaw('JSON_CONTAINS(`popup_ext` -> "$.*.button_attr[*]", ?)', json_encode(['jump_type' => $data['jump_type']]))->first();
```
# 对象转数组
```php
$result = json_decode(json_encode($result), true);

// 实例
$paginate = DB::table(DB::raw("({$sub->toSql()}) as sub"))
->mergeBindings($sub->getQuery())
->groupBy(['course_class_id'])
->select([
    'id', 'course_class_id', 'course_id', 'course_live_teacher_id', 'course_class_teacher_id', 'lesson_id',
    'course_live_id', 'live_start_time', 'teach_status'
])
->paginate(self::$input['per_page'])
->toArray()

// 对象转数组
$data = json_decode(json_encode(Arr::get($paginate, 'data')), true) ?? [];
// 按班级开课时间倒序排列（同日期按首个讲次开始时间倒序排序，同日期同讲次开始时间按班级ID倒序排列）
array_multisort(
    array_pluck($data, 'live_start_time'), SORT_DESC,
    array_pluck($data, 'course_class_id'), SORT_DESC, $data
);
Arr::set($paginate, 'data', $data);
```
# API资源（分页格式化数据）
```php
// collection
namespace App\Api\Resources\BackClass;

use App\Api\Resources\ResourceCollection;
use App\Services\BackClass\BackClassService;
use Illuminate\Http\Request;

/** @see \App\Models\BackClass\BackClassRule */
class BackClassRuleCollection extends ResourceCollection
{
    public $collects = BackClassRuleResource::class;
    /**
     * @var Request|mixed
     */
    private $request;

    public function toArray($request): array
    {
        $this->request = $request;
        $this->request->terms = BackClassService::terms();
        $this->collection = $this->returnPaginatedData($this->resource, $this->collection);
        return $this->collection;
    }
}

// resource
namespace App\Api\Resources\BackClass;

use App\Api\Resources\Resource;
use App\Models\BackClass\BackClassRule;

/** @mixin \App\Models\BackClass\BackClassRule */
class BackClassRuleResource extends Resource
{
    public function toArray($request)
    {
        $terms = [];
        foreach ($this->term_ids as $item) {
            $terms[] = $request->terms[$item] ?? '';
        }
        return [
            BackClassRule::ID => $this->id,
            BackClassRule::BACK_CLASS_BASE_RULE_ID => $this->back_class_base_rule_id,
            BackClassRule::TERM_IDS => $this->term_ids,
            BackClassRule::DELETED_AT => $this->deleted_at,
            BackClassRule::UPDATED_AT => $this->updated_at,
            BackClassRule::CREATED_AT => $this->created_at,
            'back_class_base_rule' => $this->backClassBaseRule,
            'terms' => $terms
        ];
    }
}

// resourceCollection
/**
 * 处理分页数据
 * @param \Illuminate\Pagination\LengthAwarePaginator $paginator
 * @param \Illuminate\Support\Collection $collection
 * @return array
 */
protected function returnPaginatedData($paginator, $collection)
{
    return [
        'meta' => [
            'pagination'=>[
                'total'=>$paginator->total(),
                'count'=>$paginator->count(),
                'per_page' => $paginator->perPage(),
                'current_page' => $paginator->currentPage(),
                'last_page' => $paginator->lastPage(),
                'total_page' => $paginator->lastPage(),
            ]
        ],
        'data' => $collection,
    ];
}
```
# 获取所有路由，用来扫描权限
```php
private static function scanRoutesBase(): array
{
    $data = [];
    $routes = Route::getRoutes()->getRoutesByName();
    foreach ($routes as $route) {
        $as = $route->action['as'] ?? '';
        if (in_array(strtolower(substr($as, 0, 4)), ['api.', 'app.'])) {
            array_push($data, [
                'name' => $route->action['name'] ?? '',
                'sign' => $route->action['as'] ?? '',
                'http_method' => $route->methods[0] ?? '',
                'http_path' => isset($route->uri) ? DIRECTORY_SEPARATOR . $route->uri: ''
            ]);
        }
    }
    return $data;
}
```
# Excel
[🚀 5 minute quick start | Laravel Excel (laravel-excel.com)](https://docs.laravel-excel.com/3.1/exports/)
## 导入
```php
if (!self::$request->file('file') || !self::$request->file('file')->isValid()) {
    $this->throwException('请上传文件');
}

// 获取上传文件扩展名
$originalName = self::$request->file('file')->getClientOriginalName();
$extension = pathinfo($originalName, PATHINFO_EXTENSION);
if (!in_array($extension, ['xlsx', 'xls'])) {
    $this->throwException('请传入正确的excel');
}

// 获取excel 信息
$excel = Excel::toArray(new stdClass(), self::$request->file('file'));
```
## 导出
```php
$excelObj = new IntegralMallOrderExport();
$excelObj->confExcelData([
    ['title' => '标题', 'content' => '内容'],
    ['title' => 'a1', 'content' => 'a1'],
    ['title' => 'a1', 'content' => 'a1'],
    ['title' => 'a1', 'content' => 'a1'],
]);
return Excel::download($excelObj, pathinfo($file->file_name, PATHINFO_FILENAME) . '.xlsx');
```
# 自定义验证类
[Laravel 自定义表单验证-自定义验证规则 | Laravel China 社区 (learnku.com)](https://learnku.com/articles/35283)
[Laravel使用笔记——完整添加自定义验证规则 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/35057722)
## 一、自定义验证类
```php
namespace App\Validators;

use Illuminate\Support\Facades\Input;

class LtValidator
{
    public function validate($attribute, $value, $parameters, $validator)
    {
        $other = Input::get($parameters['0']);
        // 替换消息
        $validator->addReplacer('lt', function ($message, $attribute, $rule, $parameters, $validator) {
            return str_replace(':end_time', $validator->getDisplayableAttribute($parameters['0']), $message);
        });
        return isset($other) && intval($value) < intval($other);
    }
}
```
## 二、注册验证 AppServiceProvider
```php
namespace App\Providers;

use App\Validators\LtValidator;
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Support\ServiceProvider;
use Log;
use Validator;

class AppServiceProvider extends ServiceProvider
{
    // 自定义验证类集合
    protected $validators = [
        'lt' => LtValidator::class
    ];

    public function boot()
    {
        $this->registerValidators();
    }
    
    // 注册自定义验证类
    protected function registerValidators(): void
    {
        foreach ($this->validators as $rule => $validator){
            Validator::extend($rule, "$validator@validate");
        }
    }
}
```
## 三、使用验证
```php
class ExamValidator extends BaseValidator
{
    /**
     * 规则
     * @var array
     */
    protected $rule = [
        'start_time'=> ['lt:end_time'],
    ];

    /**
     * 信息
     * @var array
     */
    protected $message = [
        'start_time .lt' => ':start_time 必须小于 :end_time'
    ];

    /**
     * 场景
     * @var array
     */
    protected $scene = [
        'add' => []
    ];
}
```
# 消息队列
```shell
# 1.创建队列迁移文件
# 在database/migrations 下面生成一个jobs_table.php文件
php artisan queue:table
# 2.执行迁移文件，创建jobs表
php artisan migrate
# 3.创建、编写任务类 
# 会在/app 下面生成一个Jobs文件夹 里面有个SendEmail.php类
php artisan make:job SendEmail
# 队列监听器
php artisan queue:listen

# 处理失败消息队列
# 1.创建失败队列迁移文件
# 在database/migrations 下面生成一个failed_jobs_table.php文件 
php artisan queue:failed-table
# 2.执行迁移文件，创建failed_jobs表
php artisan migrate # 如果任务失败会记录到failed_jobs表里面去
# 3.查看失败队列信息 
php artisan queue:failed
# 4.重新指定执行错误队列信息 
php artisan queue:retry 1 # 1表示失败里面的id
php artisan queue:retry all # 重新执行错误队列所有信息
# 5.删除指定错误队列信息 
php artisan queue:forget 1 # 1表示失败里面的id
# 删除所有错误队列信息 
php artisan queue:flush # 配置文件 config/queue.php
```
# MongoDB
[jenssegers/laravel-mongodb: A MongoDB based Eloquent model and Query builder for Laravel (Moloquent) (github.com)](https://github.com/jenssegers/laravel-mongodb)
laravel中打印mongodb的sql
```php
$noticeInfo = Notifications::query()  
                           ->whereIn('duos', ['playerCancel', 'customerCancel', 'playerComplete', 'customerCompleteSoon', 'adminCancel', 'cancel'])  
                           ->whereIn('payload.duoId', $duoIds)  
                           ->toMql();
```