---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/MVC框架原理/","title":"MVC框架原理","tags":["flashcards"],"noteIcon":"","created":"2026-03-10T22:33:54.000+08:00","updated":"2026-03-25T17:33:58.045+08:00","dg-note-properties":{"title":"MVC框架原理","tags":["flashcards"],"reference linking":null}}
---

# 1.什么是MVC
MVC模式（Model-View-Controller）是软件工程中的一种软件架构模式。
MVC把软件系统分为三个基本部分：模型（Model）、视图（View）和控制器（Controller）。
PHP中MVC模式也称Web MVC，从上世纪70年代进化而来。
MVC的目的是实现一种动态的程序设计，便于后续对程序的修改和扩展简化，并且使程序某一部分的重复利用成为可能。
除此之外，此模式通过对复杂度的简化，使程序结构更加直观。
MVC各部分的职能：
- 模型Model – 管理大部分的业务逻辑和所有的数据库逻辑。模型提供了连接和操作数据库的抽象层。
- 控制器Controller - 负责响应用户请求、准备数据，以及决定如何展示数据。
- 视图View – 负责渲染数据，通过HTML方式呈现给用户。
![web_mvc.gif](https://weichengjun2.dpdns.org/i/2023/09/21/650be5d333e94.gif)

一个典型的Web MVC流程：
1. Controller截获用户发出的请求；
2. Controller调用Model完成状态的读写操作；
3. Controller把数据传递给View；
4. View渲染最终结果并呈献给用户。

# 2.为什么要自己开发MVC框架
网络上有大量优秀的MVC框架可供使用，本教程并不是为了开发一个全面的、终极的MVC框架解决方案。
我们将它看作是一个很好的从内部学习PHP的机会。
在此过程中，你将学习面向对象编程和MVC设计模式，并学习到开发中的一些注意事项。
更重要的是，通过自制MVC框架，每个人都可以完全控制自己的框架，将你的想法融入到你的框架中。
这不是很美妙的事情吗~~~

# 3.准备工作
## 3.1 环境准备
这里我们需要最基本的PHP环境：
- Nginx或Apache
- PHP5.4+
- MySQL
推荐使用phpStudy或docker一键安装这样的LNMP环境。
## 3.2 代码规范
在目录设置好以后，我们接下来规定代码的规范：
1. MySQL的表名需小写或小写加下划线，如：item，car_orders。
2. 模块名（Models）需用大驼峰命名法，即首字母大写，并在名称后添加Model，如：ItemModel，CarModel。
3. 控制器（Controllers）需用大驼峰命名法，即首字母大写，并在名称后添加Controller，如：ItemController，CarController。
4. 方法名（Action）需用小驼峰命名法，即首字母小写，如：index，indexPost。
5. 视图（Views）部署结构为控制器名/行为名，如：item/view.php，car/buy.php。
上述规则是为了程序能更好地相互调用。
接下来就开始真正的PHP MVC编程了。

## 3.3 目录准备
在开始开发前，我们给这个框架先起个名字吧，就叫：Fastphp框架。
然后根据需要来把项目的目录创建。
假设我们建立的项目为 project，目录结构就这样：
```shell
project                 根目录
├─app                   应用目录
│  ├─controllers        控制器目录
│  ├─models             模块目录
│  ├─views              视图目录
├─config                配置文件目录
├─fastphp               框架核心目录
├─static                静态文件目录
├─index.php             入口文件
```
然后按照下一步，把Nginx或者Apache的站点根目录配置到project目录，。

## 3.4 重定向
重定向的目的有两个：设置根目录为project所在位置，以及将所有请求都发送给 index.php 文件。
如果是Apache服务器，在 project 目录下新建一个 .htaccess 文件，内容为：
```shell
<IfModule mod_rewrite.c>
    # 打开Rerite功能
    RewriteEngine On

    # 如果请求的是真实存在的文件或目录，直接访问
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteCond %{REQUEST_FILENAME} !-d

    # 如果访问的文件或目录不是真事存在，分发请求至 index.php
    RewriteRule . index.php
</IfModule>
```
如果是Nginx服务器，修改配置文件，在server块中加入如下的重定向：
```shell
location / {
    # 重定向所有非真实存在的请求到index.php
    try_files $uri $uri/ /index.php?$args;
}
```
这样做的主要原因是：
（1）静态文件能直接访问。
如果文件或者目录真实存在，则直接访问存在的文件/目录。
比如，静态文件static/css/main.css真实存在，就可以直接访问它。

（2）程序有单一的入口。
这种情况是请求地址不是真实存在的文件或目录，这样请求就会传到 index.php 上。
例如，访问地址：localhost/item/detail/1，在文件系统中并不存在这样的文件或目录。
那么，Apache或Nginx服务器会把请求发给index.php，并且把域名之后的字符串赋值给REQUEST_URI变量。
这样在PHP中用`$_SERVER['REQUEST_URI']就能拿到/item/detail/1`；

（3）可以用来生成美化的URL，利于SEO。
# 4 PHP MVC核心文件
### 4.1 入口文件
接下来，在 project 目录下新建 index.php 入口文件，文件内容为：
```php
// 应用目录为当前目录
define('APP_PATH', __DIR__ . '/');
// 开启调试模式
define('APP_DEBUG', true);
// 加载框架文件
require(APP_PATH . 'fastphp/Fastphp.php');
// 加载配置文件
$config = require(APP_PATH . 'config/config.php');
// 实例化框架类
(new fastphp\Fastphp($config))->run();
```
注意，上面的PHP代码中，并没有添加PHP结束符号?>。
这么做的主要原因是：
对于只有 PHP 代码的文件，最好没有结束标志?>，
PHP自身并不需要结束符号，不加结束符让程序更加安全，很大程度防止了末尾被注入额外的内容。
### 4.2 配置文件
在入口文件中，我们加载了config.php文件的内容，那它有何作用呢？
从名称不难看出，它的作用是保存一些常用配置。
config.php 文件内容如下，作用是定义数据库连接参数参数，以及配置默认控制器名和操作名：
```php
<?php
// 数据库配置
$config['db']['host'] = 'localhost';
$config['db']['username'] = 'root';
$config['db']['password'] = '123456';
$config['db']['dbname'] = 'project';
// 默认控制器和操作名
$config['defaultController'] = 'Item';
$config['defaultAction'] = 'index';
return $config;
```
入口中的$config变量接收到配置参数后，再传给框架的核心类，也就是Fastphp类。
### 4.3 框架核心类
入口文件对框架类做了两步操作：实例化，调用run()方法。
实例化操作接受$config参数配置，并保存到对象属性中。
run()方法则调用用类自身方法，完成下面几个操作：
1. 类自动加载
2. 环境检查
3. 过滤敏感字符
4. 移除全局变量的老用法
5. 路由处理
在fastphp目录下新建核心类文件，名称Fastphp.php，代码：
```php
<?php
namespace fastphp;
// 框架根目录
defined('CORE_PATH') or define('CORE_PATH', __DIR__);

/**
 * fastphp框架核心
 */
class Fastphp
{
    // 配置内容
    protected $config = [];
    public function __construct($config)
    {
        $this->config = $config;
    }
    // 运行程序
    public function run()
    {
        spl_autoload_register(array($this, 'loadClass'));
        $this->setReporting();
        $this->removeMagicQuotes();
        $this->unregisterGlobals();
        $this->setDbConfig();
        $this->route();
    }
    // 自动加载类
    public function loadClass($className)
    {
        $classMap = $this->classMap();

        if (isset($classMap[$className])) {
            // 包含内核文件
            $file = $classMap[$className];
        } elseif (strpos($className, '\\') !== false) {
            // 包含应用（application目录）文件
            $file = APP_PATH . str_replace('\\', '/', $className) . '.php';
            if (!is_file($file)) {
                return;
            }
        } else {
            return;
        }
        include $file;
        // 这里可以加入判断，如果名为$className的类、接口或者性状不存在，则在调试模式下抛出错误
    }
    // 内核文件命名空间映射关系
    protected function classMap()
    {
        return [
            'fastphp\base\Controller' => CORE_PATH . '/base/Controller.php',
            'fastphp\base\Model' => CORE_PATH . '/base/Model.php',
            'fastphp\base\View' => CORE_PATH . '/base/View.php',
            'fastphp\db\Db' => CORE_PATH . '/db/Db.php',
            'fastphp\db\Sql' => CORE_PATH . '/db/Sql.php',
        ];
    }
    // 检测开发环境
    public function setReporting()
    {
        if (APP_DEBUG === true) {
            error_reporting(E_ALL);
            ini_set('display_errors', 'On');
        } else {
            error_reporting(E_ALL);
            ini_set('display_errors', 'Off');
            ini_set('log_errors', 'On');
        }
    }
    // 检测敏感字符并删除
    public function removeMagicQuotes()
    {
        if (get_magic_quotes_gpc()) {
            $_GET = isset($_GET) ? $this->stripSlashesDeep($_GET) : '';
            $_POST = isset($_POST) ? $this->stripSlashesDeep($_POST) : '';
            $_COOKIE = isset($_COOKIE) ? $this->stripSlashesDeep($_COOKIE) : '';
            $_SESSION = isset($_SESSION) ? $this->stripSlashesDeep($_SESSION) : '';
        }
    }
    // 删除敏感字符
    public function stripSlashesDeep($value)
    {
        $value = is_array($value) ? array_map(array($this, 'stripSlashesDeep'), $value) : stripslashes($value);
        return $value;
    }
    // 检测自定义全局变量并移除。因为 register_globals 已经弃用，如果
    // 已经弃用的 register_globals 指令被设置为 on，那么局部变量也将
    // 在脚本的全局作用域中可用。 例如， $_POST['foo'] 也将以 $foo 的
    // 形式存在，这样写是不好的实现，会影响代码中的其他变量。 相关信息，
    // 参考: http://php.net/manual/zh/faq.using.php#faq.register-globals
    public function unregisterGlobals()
    {
        if (ini_get('register_globals')) {
            $array = array('_SESSION', '_POST', '_GET', '_COOKIE', '_REQUEST', '_SERVER', '_ENV', '_FILES');
            foreach ($array as $value) {
                foreach ($GLOBALS[$value] as $key => $var) {
                    if ($var === $GLOBALS[$key]) {
                        unset($GLOBALS[$key]);
                    }
                }
            }
        }
    }
    // 配置数据库信息
    public function setDbConfig()
    {
        if ($this->config['db']) {
            define('DB_HOST', $this->config['db']['host']);
            define('DB_NAME', $this->config['db']['dbname']);
            define('DB_USER', $this->config['db']['username']);
            define('DB_PASS', $this->config['db']['password']);
        }
    }
    // 路由处理
    public function route()
    {
        $controllerName = $this->config['defaultController'];
        $actionName = $this->config['defaultAction'];
        $param = array();
        $url = $_SERVER['REQUEST_URI'];
        // 清除?之后的内容
        $position = strpos($url, '?');
        $url = $position === false ? $url : substr($url, 0, $position);
        // 删除前后的“/”
        $url = trim($url, '/');
        if ($url) {
            // 使用“/”分割字符串，并保存在数组中
            $urlArray = explode('/', $url);
            // 删除空的数组元素
            $urlArray = array_filter($urlArray);
            // 获取控制器名
            $controllerName = ucfirst($urlArray[0]);
            // 获取动作名
            array_shift($urlArray);
            $actionName = $urlArray ? $urlArray[0] : $actionName;
            // 获取URL参数
            array_shift($urlArray);
            $param = $urlArray ? $urlArray : array();
        }
        // 判断控制器和操作是否存在
        $controller = 'app\\controllers\\' . $controllerName . 'Controller';
        if (!class_exists($controller)) {
            exit($controller . '控制器不存在');
        }
        if (!method_exists($controller, $actionName)) {
            exit($actionName . '方法不存在');
        }
        // 如果控制器和操作名存在，则实例化控制器，因为控制器对象里面
        // 还会用到控制器名和操作名，所以实例化的时候把他们俩的名称也
        // 传进去。结合Controller基类一起看
        $dispatch = new $controller($controllerName, $actionName);
        // $dispatch保存控制器实例化后的对象，我们就可以调用它的方法，
        // 也可以像方法中传入参数，以下等同于：$dispatch->$actionName($param)
        call_user_func_array(array($dispatch, $actionName), $param);
    }
}
```
下面重点讲解主请求方法 route()，它也称路由方法。
路由方法的主要作用是：截取URL，并解析出控制器名、方法名和URL参数。
假设我们的 URL 是这样：
```php
yoursite.com/controllerName/actionName/queryString
```
当浏览器访问上面的URL，route()从全局变量 `$_SERVER['REQUEST_URI']中获取到字符串/controllerName/actionName/queryString`。
然后，会将这个字符串分割成三部分：controllerName、actionName 和 queryString。
例如，URL链接为：yoursite.com/item/detail/1/hello，那么route()分割之后，
- ControllerName名就是：item
- actionName名就是：detail
- URL参数就是：array(1, hello)

分割完成后，路由方法再实例化控制器：itemController，并调用其中的detail方法 。
### 4.4 Controller基类
接下来，就是在 fastphp 中创建MVC基类，包括控制器、模型和视图三个基类。
在fastphp/base/目录下新建控制器基类，文件名 Controller.php，功能就是总调度，内容如下：
```php
<?php
namespace fastphp\base;

/**
 * 控制器基类
 */
class Controller
{
    protected $_controller;
    protected $_action;
    protected $_view;

    // 构造函数，初始化属性，并实例化对应模型
    public function __construct($controller, $action)
    {
        $this->_controller = $controller;
        $this->_action = $action;
        $this->_view = new View($controller, $action);
    }

    // 分配变量
    public function assign($name, $value)
    {
        $this->_view->assign($name, $value);
    }

    // 渲染视图
    public function render()
    {
        $this->_view->render();
    }
}
```
Controller 类用assign()方法实现把变量保存到View对象中。
这样，在调用$this->render() 后视图文件就能显示这些变量。
### 4.5 Model基类
新建模型基类，继承自数据库操作类Sql类。
因为数据库操作比较复杂，所以SQL操作我们单独创建一个类。
Model基类涉及到3个类：Model基类本身，它的父类SQL，以及提供数据库连接句柄的Db类。
在fastphp/base/目录下新建模型基类文件，名为 Model.php，代码如下：
```php
<?php
namespace fastphp\base;

use fastphp\db\Sql;

class Model extends Sql
{
    protected $model;

    public function __construct()
    {
        // 获取数据库表名
        if (!$this->table) {

            // 获取模型类名称
            $this->model = get_class($this);

            // 删除类名最后的 Model 字符
            $this->model = substr($this->model, 0, -5);

            // 数据库表名与类名一致
            $this->table = strtolower($this->model);
        }
    }
}
```
在fastphp/db/目录下建立一个数据库基类 Sql.php，代码如下：
```php
<?php
namespace fastphp\db;

use \PDOStatement;

class Sql
{
    // 数据库表名
    protected $table;

    // 数据库主键
    protected $primary = 'id';

    // WHERE和ORDER拼装后的条件
    private $filter = '';

    // Pdo bindParam()绑定的参数集合
    private $param = array();

    /**
     * 查询条件拼接，使用方式：
     *
     * $this->where(['id = 1','and title="Web"', ...])->fetch();
     * 为防止注入，建议通过$param方式传入参数：
     * $this->where(['id = :id'], [':id' => $id])->fetch();
     *
     * @param array $where 条件
     * @return $this 当前对象
     */
    public function where($where = array(), $param = array())
    {
        if ($where) {
            $this->filter .= ' WHERE ';
            $this->filter .= implode(' ', $where);

            $this->param = $param;
        }

        return $this;
    }

    /**
     * 拼装排序条件，使用方式：
     *
     * $this->order(['id DESC', 'title ASC', ...])->fetch();
     *
     * @param array $order 排序条件
     * @return $this
     */
    public function order($order = array())
    {
        if($order) {
            $this->filter .= ' ORDER BY ';
            $this->filter .= implode(',', $order);
        }

        return $this;
    }

    // 查询所有
    public function fetchAll()
    {
        $sql = sprintf("select * from `%s` %s", $this->table, $this->filter);
        $sth = Db::pdo()->prepare($sql);
        $sth = $this->formatParam($sth, $this->param);
        $sth->execute();

        return $sth->fetchAll();
    }

    // 查询一条
    public function fetch()
    {
        $sql = sprintf("select * from `%s` %s", $this->table, $this->filter);
        $sth = Db::pdo()->prepare($sql);
        $sth = $this->formatParam($sth, $this->param);
        $sth->execute();

        return $sth->fetch();
    }

    // 根据条件 (id) 删除
    public function delete($id)
    {
        $sql = sprintf("delete from `%s` where `%s` = :%s", $this->table, $this->primary, $this->primary);
        $sth = Db::pdo()->prepare($sql);
        $sth = $this->formatParam($sth, [$this->primary => $id]);
        $sth->execute();

        return $sth->rowCount();
    }

    // 新增数据
    public function add($data)
    {
        $sql = sprintf("insert into `%s` %s", $this->table, $this->formatInsert($data));
        $sth = Db::pdo()->prepare($sql);
        $sth = $this->formatParam($sth, $data);
        $sth = $this->formatParam($sth, $this->param);
        $sth->execute();

        return $sth->rowCount();
    }

    // 修改数据
    public function update($data)
    {
        $sql = sprintf("update `%s` set %s %s", $this->table, $this->formatUpdate($data), $this->filter);
        $sth = Db::pdo()->prepare($sql);
        $sth = $this->formatParam($sth, $data);
        $sth = $this->formatParam($sth, $this->param);
        $sth->execute();

        return $sth->rowCount();
    }

    /**
     * 占位符绑定具体的变量值
     * @param PDOStatement $sth 要绑定的PDOStatement对象
     * @param array $params 参数，有三种类型：
     * 1）如果SQL语句用问号?占位符，那么$params应该为
     *    [$a, $b, $c]
     * 2）如果SQL语句用冒号:占位符，那么$params应该为
     *    ['a' => $a, 'b' => $b, 'c' => $c]
     *    或者
     *    [':a' => $a, ':b' => $b, ':c' => $c]
     *
     * @return PDOStatement
     */
    public function formatParam(PDOStatement $sth, $params = array())
    {
        foreach ($params as $param => &$value) {
            $param = is_int($param) ? $param + 1 : ':' . ltrim($param, ':');
            $sth->bindParam($param, $value);
        }

        return $sth;
    }

    // 将数组转换成插入格式的sql语句
    private function formatInsert($data)
    {
        $fields = array();
        $names = array();
        foreach ($data as $key => $value) {
            $fields[] = sprintf("`%s`", $key);
            $names[] = sprintf(":%s", $key);
        }

        $field = implode(',', $fields);
        $name = implode(',', $names);

        return sprintf("(%s) values (%s)", $field, $name);
    }

    // 将数组转换成更新格式的sql语句
    private function formatUpdate($data)
    {
        $fields = array();
        foreach ($data as $key => $value) {
            $fields[] = sprintf("`%s` = :%s", $key, $key);
        }

        return implode(',', $fields);
    }
}
```
应该说，Sql基类是框架的核心部分。为什么？
因为通过它，我们创建了一个 SQL 抽象层，可以大大减少了数据库的编程工作。
虽然 PDO 接口本来已经很简洁，但是抽象之后框架的可灵活性更高。
Sql类里面有用到Db:pdo()方法，这是我们创建的Db类，它提供一个PDO单例。
在fastphp/db/目录下创建Db.php文件，内容：
```php
<?php
namespace fastphp\db;

use PDO;
use PDOException;

/**
 * 数据库操作类。
 * 其$pdo属性为静态属性，所以在页面执行周期内，
 * 只要一次赋值，以后的获取还是首次赋值的内容。
 * 这里就是PDO对象，这样可以确保运行期间只有一个
 * 数据库连接对象，这是一种简单的单例模式
 * Class Db
 */
class Db
{
    private static $pdo = null;

    public static function pdo()
    {
        if (self::$pdo !== null) {
            return self::$pdo;
        }

        try {
            $dsn    = sprintf('mysql:host=%s;dbname=%s;charset=utf8', DB_HOST, DB_NAME);
            $option = array(PDO::ATTR_DEFAULT_FETCH_MODE => PDO::FETCH_ASSOC);

            return self::$pdo = new PDO($dsn, DB_USER, DB_PASS, $option);
        } catch (PDOException $e) {
            exit($e->getMessage());
        }
    }
}
```
### 4.6 View基类
在fastphp/base/目录下新建视图基类 View.php 内容如下：
```php
<?php
namespace fastphp\base;

/**
 * 视图基类
 */
class View
{
    protected $variables = array();
    protected $_controller;
    protected $_action;

    function __construct($controller, $action)
    {
        $this->_controller = strtolower($controller);
        $this->_action = strtolower($action);
    }
 
    // 分配变量
    public function assign($name, $value)
    {
        $this->variables[$name] = $value;
    }
 
    // 渲染显示
    public function render()
    {
        extract($this->variables);
        $defaultHeader = APP_PATH . 'app/views/header.php';
        $defaultFooter = APP_PATH . 'app/views/footer.php';

        $controllerHeader = APP_PATH . 'app/views/' . $this->_controller . '/header.php';
        $controllerFooter = APP_PATH . 'app/views/' . $this->_controller . '/footer.php';
        $controllerLayout = APP_PATH . 'app/views/' . $this->_controller . '/' . $this->_action . '.php';

        // 页头文件
        if (is_file($controllerHeader)) {
            include ($controllerHeader);
        } else {
            include ($defaultHeader);
        }

        //判断视图文件是否存在
        if (is_file($controllerLayout)) {
            include ($controllerLayout);
        } else {
            echo "<h1>无法找到视图文件</h1>";
        }
        
        // 页脚文件
        if (is_file($controllerFooter)) {
            include ($controllerFooter);
        } else {
            include ($defaultFooter);
        }
    }
}
```
这样，核心的PHP MVC框架核心就完成了。
下面我们编写应用来测试框架功能。
# 5 应用
### 5.1 数据库部署
在 SQL 中新建一个 project 数据库，增加一个item 表、并插入两条记录，命令如下：
```mysql
CREATE DATABASE `project` DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
USE `project`;
CREATE TABLE `item`
(
  `id`        INT(11)      NOT NULL auto_increment,
  `item_name` VARCHAR(255) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE = INNODB
  AUTO_INCREMENT = 1
  DEFAULT CHARSET = utf8;
INSERT INTO `item` VALUES (1, 'Hello World.');
INSERT INTO `item` VALUES (2, 'Lets go!');
```
### 5.2 部署模型
然后，我们还需要在app/models/目录中创建一个 ItemModel.php 模型，内容如下：
```php
<?php
namespace app\models;

use fastphp\base\Model;
use fastphp\db\Db;

/**
 * 用户Model
 */
class ItemModel extends Model
{
    /**
     * 自定义当前模型操作的数据库表名称，
     * 如果不指定，默认为类名称的小写字符串，
     * 这里就是 item 表
     * @var string
     */
    protected $table = 'item';

    /**
     * 搜索功能，因为Sql父类里面没有现成的like搜索，
     * 所以需要自己写SQL语句，对数据库的操作应该都放
     * 在Model里面，然后提供给Controller直接调用
     * @param $title string 查询的关键词
     * @return array 返回的数据
     */
    public function search($keyword)
    {
        $sql = "select * from `$this->table` where `item_name` like :keyword";
        $sth = Db::pdo()->prepare($sql);
        $sth = $this->formatParam($sth, [':keyword' => "%$keyword%"]);
        $sth->execute();

        return $sth->fetchAll();
    }
}
```
因为 Item 模型继承了 Model基类，所以它拥有 Model 类的所有功能。
### 5.3 部署控制器
在 app/controllers/ 目录下创建一个 ItemController.php 控制器，内容如下：
```php
<?php
namespace app\controllers;

use fastphp\base\Controller;
use app\models\ItemModel;
 
class ItemController extends Controller
{
    // 首页方法，测试框架自定义DB查询
    public function index()
    {
        $keyword = isset($_GET['keyword']) ? $_GET['keyword'] : '';

        if ($keyword) {
            $items = (new ItemModel())->search($keyword);
        } else {
            // 查询所有内容，并按倒序排列输出
            // where()方法可不传入参数，或者省略
            $items = (new ItemModel)->where()->order(['id DESC'])->fetchAll();
        }

        $this->assign('title', '全部条目');
        $this->assign('keyword', $keyword);
        $this->assign('items', $items);
        $this->render();
    }

    // 查看单条记录详情
    public function detail($id)
    {
        // 通过?占位符传入$id参数
        $item = (new ItemModel())->where(["id = ?"], [$id])->fetch();

        $this->assign('title', '条目详情');
        $this->assign('item', $item);
        $this->render();
    }
    
    // 添加记录，测试框架DB记录创建（Create）
    public function add()
    {
        $data['item_name'] = $_POST['value'];
        $count = (new ItemModel)->add($data);

        $this->assign('title', '添加成功');
        $this->assign('count', $count);
        $this->render();
    }
    
    // 操作管理
    public function manage($id = 0)
    {
        $item = array();
        if ($id) {
            // 通过名称占位符传入参数
            $item = (new ItemModel())->where(["id = :id"], [':id' => $id])->fetch();
        }

        $this->assign('title', '管理条目');
        $this->assign('item', $item);
        $this->render();
    }
    
    // 更新记录，测试框架DB记录更新（Update）
    public function update()
    {
        $data = array('id' => $_POST['id'], 'item_name' => $_POST['value']);
        $count = (new ItemModel)->where(['id = :id'], [':id' => $data['id']])->update($data);

        $this->assign('title', '修改成功');
        $this->assign('count', $count);
        $this->render();
    }
    
    // 删除记录，测试框架DB记录删除（Delete）
    public function delete($id = null)
    {
        $count = (new ItemModel)->delete($id);

        $this->assign('title', '删除成功');
        $this->assign('count', $count);
        $this->render();
    }
}
```
### 5.4 部署视图
在 app/views/目录下新建 header.php 和 footer.php 两个页头页脚模板，如下。
header.php 内容：
```php
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title><?php echo $title ?></title>
    <link rel="stylesheet" href="/static/css/main.css" type="text/css" />
</head>
<body>
    <h1><?php echo $title ?></h1>
```
footer.php 内容：
```php
</body>
</html>
```
页头文件用到了main.css样式文件，内容：
```css
html, body {
    margin: 0;
    padding: 10px;
    font-size: 20px;
}
input {
    font-family: georgia, times;
    font-size: 24px;
    line-height: 1.2em;
}
a {
    color: blue;
    font-family: georgia, times;
    line-height: 1.2em;
    text-decoration: none;
}
a:hover {
    text-decoration: underline;
}
h1 {
    color: #000000;
    font-size: 41px;
    border-bottom: 1px dotted #cccccc;
}
td {
    padding: 1px 30px 1px 0;
}
```
然后，在 app/views/item 目录下创建以下几个视图文件。
index.php，浏览数据库内 item 表的所有记录，内容：
```php
<form action="" method="get">
    <input type="text" value="<?php echo $keyword ?>" name="keyword">
    <input type="submit" value="搜索">
</form>

<p><a href="/item/manage">新建</a></p>

<table>
    <tr>
        <th>ID</th>
        <th>内容</th>
        <th>操作</th>
    </tr>
    <?php foreach ($items as $item): ?>
        <tr>
            <td><?php echo $item['id'] ?></td>
            <td><?php echo $item['item_name'] ?></td>
            <td>
                <a href="/item/manage/<?php echo $item['id'] ?>">编辑</a>
                <a href="/item/delete/<?php echo $item['id'] ?>">删除</a>
            </td>
        </tr>
    <?php endforeach ?>
</table>
```
add.php，添加记录后的提示，内容：
```php
<a class="big" href="/item/index">成功添加<?php echo $count ?>条记录，点击返回</a>
```
manage.php，管理记录，内容：
```php
<form action="<?php echo $postUrl; ?>" method="post">
    <?php if (isset($item['id'])): ?>
        <input type="hidden" name="id" value="<?php echo $item['id'] ?>">
    <?php endif; ?>
    <input type="text" name="value" value="<?php echo isset($item['item_name']) ? $item['item_name'] : '' ?>">
    <input type="submit" value="提交">
</form>

<a class="big" href="/item/index">返回</a>
```
update.php，更改记录后的提示，内容：
```php
<a class="big" href="/item/index">成功修改<?php echo $count ?>项，点击返回</a>
```
delete.php，删除记录后的提示，内容：
```php
<a href="/item/index">成功删除<?php echo $count ?>项，点击返回</a>
```
# 6 应用测试
这样，在浏览器中访问 project程序：http://localhost/item/index/，就可以看到效果了。
![](https://weichengjun2.dpdns.org/i/2023/09/21/650be5d53bbe8.jpeg)

以上代码已经全部发布到 github 上，关键部分加了注释：
- 仓库地址：https://github.com/yeszao/fastphp
- 源码打包：https://github.com/yeszao/fastphp/archive/master.zip
- [最好的框架](https://gitee.com/rivenwei/bestphp)
要设计更好的MVC，或使用得更加规范，请看《MVC架构的职责划分原则》。
参考资料：
- [手把手编写PHP MVC框架实例教程 - 歪麦博客](https://www.awaimai.com/128.html)
- [write your own php mvc framework](http://anantgarg.com/2009/03/13/write-your-own-php-mvc-framework-part-1/)