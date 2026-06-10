---
{"dg-publish":true,"permalink":"/Work/Tools/Composer/Composer basic/","title":"Composer basic","tags":["flashcards"],"noteIcon":"","created":"2026-05-11T23:45:07.000+08:00","updated":"2026-06-09T12:06:18.263+08:00","dg-note-properties":{"title":"Composer basic","tags":["flashcards"],"reference linking":null}}
---

文档: [作曲家 - Composer 包管理器](https://composer.php.ac.cn/doc/)
# 安装
## Linux环境
### 1.10.1
[composer.phar](https://weichengjun2.dpdns.org/d/Attachment/composer.phar?sign=eMOwz8A5XzYVc8r5_g8RexiHZGsstom2gOeUBIknDWU=:0)
### 2.1.3
[composer.phar](https://weichengjun2.dpdns.org/d/Attachment/composer2.phar?sign=ox77JfaFzb0jSG2zyky9miWhWFqtJmYIpELzbX6lTu0=:0)
```shell
# 进入用户源码目录
cd /usr/local/src

# 下载安装(二者选其一)
curl -sS https://getcomposer.org/installer | php
php -r "readfile('https://getcomposer.org/installer');" | php

# composer加入环境变量
mv composer.phar /usr/local/bin/composer
```
### composer.json 文件
```json
{
  // composer 包的名称，它包括供应商名称和项目名称，使用 `/` 分隔。
  "name": "guzzlehttp/guzzle",
  // library：这是默认类型，它会简单的将文件复制到vendor目录。
  // project：这表示当前包是一个项目，而不是一个库。
  // metapackage：当一个空的包，包含依赖并且需要触发依赖的安装，这将不会对系统写入额外的文件。
  // composer-plugin：一个安装类型为 `composer-plugin` 的包，它有一个自定义安装类型，可以为其它包提供一个 installler。
  "type": "library",
  // 一个包的简短描述。通常这个最长只有一行。
  "description": "Guzzle is a PHP HTTP client library",
  // 该包相关的关键词的数组。这些可用于搜索和过滤。
  "keywords": [
    "framework",
    "http",
    "rest",
    "web service",
    "curl",
    "client",
    "HTTP client"
  ],
  // 项目主页 homepage
  "homepage": "http://guzzlephp.org/",
  // 版本
  "minimum-stability": "dev",
  // 包的许可协议
  "license": "MIT",
  // 作者
  "authors": [
    {
      "name": "riven",
      "email": "1054487195@qq.com",
      "homepage": "https://github.com/woweijun123"
    }
  ],
  // 必须的软件包列表，除非这些依赖被满足，否则不会完成安装。
  "require": {
    "php": ">=5.5",
    "guzzlehttp/psr7": "^1.4",
    "guzzlehttp/promises": "^1.0"
  },
  // 这个列表是为开发或测试等目的，额外列出的依赖。“root 包”的 require-dev 默认是会被安装的。然而 `install` 或 `update` 支持使用 `--no-dev` 参数来跳过 `require-dev` 字段中列出的包。
  "require-dev": {
    //可以帮你指定需要的 PHP 扩展（包括核心扩展）例如在编辑器的自动提示功能
    "ext-curl": "*",
    "phpunit/phpunit": "^4.8.35 || ^5.7 || ^6.4 || ^7.0",
    "psr/log": "^1.0"
  },
  // PHP autoloader 的自动加载映射。
  "autoload": {
    "files": [
      "src/functions_include.php"
      //自动加载的函数库文件 相当于直接require该文件，直接使用其中function 数组可配置多个
    ],
    "psr-4": {
      "GuzzleHttp\\": "src/"
    }
  },
  // 一般将单元测试的路径放在这里
  "autoload-dev": {
    "psr-4": {
      "GuzzleHttp\\Tests\\": "tests/"
    }
  },
  // 建议安装的包，它们增强或能够与当前包良好的工作。这些只是信息，并显示在依赖包安装完成之后，给你的用户一个建议，他们可以添加更多的包。
  "suggest": {
    "psr/log": "Required for using the Log middleware"
  },
  // 任意的，供 `scripts` 使用的额外数据。可选项
  "extra": {
    "branch-alias": {
      "dev-master": "6.3-dev"
    }
  },
  // 该属性用于标注一组应被视为二进制脚本的文件，他们会被软链接到（config 对象中的）`bin-dir` 属性所标注的目录，以供其他依赖包调用。
  // composer安装时会将script脚本文件复制到vendor/bin中 比如 php vendor/bin/script 执行一些组件的脚本功能
  "bin": [
    "bin/script"
  ]
}
```
# 镜像
## 常用镜像源地址
```shell
# 官方镜像
https://packagist.org
# 中国镜像
https://packagist.phpcomposer.com
# 阿里云镜像
https://mirrors.aliyun.com/composer/
```
## 查询当前镜像地址
```shell
composer config -gl repo.packagist
```
## 配置
### 全局配置
#### 添加配置
将配置信息添加到 Composer 的全局配置文件 config.json「用户主目录中：/Users/weichengjun/.composer/config.json」
```shell
composer config -g repo.packagist composer https://packagist.org
```
#### 解除配置
```shell
composer config -g --unset repos.packagist
```
### 项目配置
#### 添加配置
修改当前项目的 composer.json 配置文件
```shell
composer config repo.packagist composer https://mirrors.aliyun.com/composer/
```
#### 解除配置
```shell
composer config --unset repos.packagist
```
### 调试
composer 命令增加 -vvv 可输出详细的信息，命令如下：
```javascript
composer -vvv require alibabacloud/sdk
```
### 遇到问题？
1. 建议先将Composer版本升级到最新：
```javascript
composer self-update
```
2. 执行诊断命令：
```javascript
composer diagnose
```
3. 清除缓存：
```javascript
composer clear
```
4. 若项目之前已通过其他源安装，则需要更新 composer.lock 文件，执行命令：
```javascript
composer update --lock
```
5. 配置阿里云和官方两个源，阿里云镜像站下载文件出错时再从官方源上下载：
```javascript
{
    "repositories": [
        {
            "description": "阿里云镜像",
            "type": "composer",
            "url": "https://mirrors.aliyun.com/composer/",
            "canonical": false
        }
    ]
}
```
# 命令
## [全局选项](https://composer.php.ac.cn/doc/03-cli.md#global-options)
以下选项适用于每个命令
- **--verbose (-v):** 提高消息的详细程度。
- **--help (-h):** 显示帮助信息。
- **--quiet (-q):** 不输出任何消息。
- **--no-interaction (-n):** 不询问任何交互式问题。
- **--no-plugins:** 禁用插件。
- **--no-scripts:** 跳过执行 `composer.json` 中定义的脚本。
- **--no-cache:** 禁用缓存目录的使用。与将 COMPOSER_CACHE_DIR 环境变量设置为 /dev/null（或 Windows 上的 NUL）相同。
- **--working-dir (-d):** 如果指定，则使用给定的目录作为工作目录。
- **--profile:** 显示计时和内存使用信息
- **--ansi:** 强制使用 ANSI 输出。
- **--no-ansi:** 禁用 ANSI 输出。
- **--version (-V):** 显示此应用程序版本。
```shell
# 安装验证码依赖包(不设置版本，默认加载最新版本)
composer require topthink/think-captcha
# 安装指定版本验证码依赖包 
composer require topthink/think-captcha 1.*
# 版本查看
composer --version 
# 更新composer
composer selfupdate 
# 显示所有命令
composer list
# 显示所有包信息
composer show
# 在composer.json配置中添加依赖库之后运行此命令安装
composer install
# 禁用脚本钩子并禁用缓存
composer install --no-scripts --no-cache
# 搜索包
composer search packagename
# 更新所有包
composer update
# 查看依赖包版本
composer show -all monolog/monolog
# 更新指定包
composer update monolog/monolog
# 移除指定的包
composer remove monolog/monolog
# 清理缓存
composer clearcache
# 查看配置文件信息
composer config -gl
# 设置缓存文件大小
composer config --global cache-files-maxsize 2048MiB
composer config --global data-dir /www/.composer
composer config --global cache-dir /www/.composer

# 配置 Composer 全局信任该 CA
composer config --global cafile ~/.config/composer/homelab-rootCA.pem
```
# 初始化
![](https://weichengjun2.dpdns.org/i/2023/09/21/650bfb38028f8.png)
# 版本限定
# Composer 版本限定语法

## 1. 精确版本 (Exact Version)
只安装指定版本，不接受任何更新。
`1.0.0`
## 2. 范围限定 (Range Constraints)
使用比较操作符来限定版本范围。
**语法**: `<, <=, >, >=, !=`
- `>1.0.0`
- `>=1.0.0 <2.0.0` | 大于等于1.0.0，且小于2.0.0
- `!=1.2.3` | 不等于1.2.3
## 3. `*`通配符 (Wildcard)
使用通配符 `*` 来匹配任何版本号。
- `1.0.*` | 匹配所有 1.0.x 版本 (如 1.0.0, 1.0.1 等)
- `1.*` | 匹配所有 1.x.x 版本 (如 1.0.0, 1.1.2 等)
## 4. `~`波浪号操作符 (Tilde Operator)
允许对最小版本号进行非破坏性更新。
**语法**: `~`
- `~1.2` | 等价于 `>=1.2.0 <1.3.0`
- `~1.2.3` | 等价于 `>=1.2.3 <1.3.0`
## 5. `^`插入符操作符 (Caret Operator)
允许向后兼容的更新，这是最常用的方式。
**语法**: `^`
- `^1.2.3` | 等价于 `>=1.2.3 <2.0.0`。只要主版本号不变，可以更新到任何新版本。
- `^0.2.3` | 等价于 `>=0.2.3 <0.3.0`。对于主版本号为0的版本，只允许更新次版本号。
## 6. 逻辑组合 (Logical Combinations)
可以组合多个限定条件。
**语法**: 空格或逗号为 "与" (AND)，管道符 `|` 为 "或" (OR)。
- `>=1.0 <2.0` | 等同于 `>=1.0, <2.0`
- `^1.0 |
{ #2}
.0` | 允许安装 1.x 或 2.x 版本
## 7. 预发布版本 (Stability Flags)
强制指定接受特定稳定性的版本。
**语法**: `@dev, @alpha, @beta, @RC, @stable`
- `1.0.0@dev` | 接受 1.0.0 的开发版本 (dev)
- `1.0.0@RC` | 接受 1.0.0 的候选发布版本 (RC)
## 8. 别名 (Aliasing)
将一个版本号当作另一个版本号来使用。
**语法**: `as`
- `dev-main as 1.0.0-dev` | 将 `dev-main` 分支别名为 `1.0.0-dev`
## 9. 分支/引用 (Branch/Reference)
直接引用一个分支、标签或提交哈希。
- `dev-main` | 引用 `dev-main` 分支的最新提交
- `9e9e1c4` | 引用一个特定的提交哈希
# 单元测试
```shell
# 测试指定方法
composer test -- --filter testEvent
```
# `dev-main`

### 1. `dev-main` 是什么？
- **`dev-main`** 是 Composer 的一种版本标识符，表示从包的 `main` 分支（通常是默认分支）安装最新的**未发布的开发版本**。
- 例如，如果包的 GitHub 仓库没有发布稳定版本（如 `1.0.0`），但开发者持续在 `main` 分支提交代码，Composer 会默认使用 `dev-main` 作为版本约束。
### 2. 为什么默认是 `dev-main`？
当运行 `composer require riven/laravel-amqp` 时，如果没有显式指定版本号（如 `^1.0` 或 `dev-main`），Composer 会尝试找到包的 **默认版本**。以下是可能的原因：
#### (1) 包未发布稳定版本
- 如果该包的 GitHub 仓库没有通过 `git tag` 发布过稳定版本（如 `1.0.0`），Composer 会默认使用 `dev-main`。
- 例如：
```shell
composer require riven/laravel-amqp
```
  实际等效于：
```shell
composer require riven/laravel-amqp:dev-main
```
#### (2) 包的 `composer.json` 配置了 `default-branch`
- 包作者可能在其 `composer.json` 中设置了 `default-branch` 字段，指定默认分支为 `main`：
```json
{
	"extra": {
	  "branch-alias": {
		"dev-main": "1.0.x-dev"
	  }
	}
}
```
  这会让 Composer 将 `dev-main` 映射为 `1.0.x-dev`，并作为默认版本。
### 3. 如何验证当前安装的是 `dev-main`？
运行以下命令查看包的版本：
```shell
composer show riven/laravel-amqp
```
输出示例：
```
name     : riven/laravel-amqp
described: Laravel AMQP integration
versions : dev-main
type     : library
```
如果 `versions` 显示为 `dev-main`，则表示安装的是开发版本。
### 4. 如何指定其他版本？
如果你希望安装特定版本（如稳定版 `1.0.0` 或 `dev-feature-branch`），可以显式指定版本号：
#### (1) 安装稳定版本
```shell
composer require riven/laravel-amqp:1.0.0
```
#### (2) 安装其他开发分支
```shell
composer require riven/laravel-amqp:dev-feature-branch
```
#### (3) 安装最新稳定版本
如果包有多个稳定版本（如 `1.0.0`、`1.1.0`），Composer 会自动选择最新版本：
```shell
composer require riven/laravel-amqp:^1.0
```
### 5. 使用 `dev-main` 的注意事项
- **不稳定性**：`dev-main` 是开发分支的最新代码，可能包含未测试的功能或 bug。
- **频繁更新**：每次运行 `composer update` 时，Composer 可能会拉取 `main` 分支的最新提交，导致版本频繁变动。
- **生产环境风险**：不建议在生产环境中使用 `dev-main`，应优先使用已发布的稳定版本。
### 6. 如何禁止 Composer 安装 `dev-main`？
如果你希望强制使用稳定版本，可以在 `composer.json` 中设置 `minimum-stability`：
```json
{
  "minimum-stability": "stable"
}
```
这会阻止 Composer 安装开发分支（如 `dev-main`），除非显式指定。
### 7. 示例：修改为稳定版本
假设你发现安装的是 `dev-main`，但希望改用稳定版本 `1.0.0`：
1. 修改 `composer.json`：
```json
{
	"require": {
		"riven/laravel-amqp": "^1.0"
	}
}
```
2. 运行更新命令：
```shell
composer update riven/laravel-amqp
```

# Composer 的生命周期钩子
[脚本 | Composer 中文文档 | Composer 中文网](https://docs.phpcomposer.com/articles/scripts.html)
## 事件汇总
| 事件名称                          | 详细说明                                                           |
| ----------------------------- | -------------------------------------------------------------- |
| **pre-install-cmd**           | 在 `install` 命令执行前触发。                                           |
| **post-install-cmd**          | 在 `install` 命令执行后触发。                                           |
| **pre-update-cmd**            | 在 `update` 命令执行前触发。                                            |
| **post-update-cmd**           | 在 `update` 命令执行后触发。                                            |
| **pre-status-cmd**            | 在 `status` 命令执行前触发。                                            |
| **post-status-cmd**           | 在 `status` 命令执行后触发。                                            |
| **pre-package-install**       | 在资源包安装前触发。                                                     |
| **post-package-install**      | 在资源包安装后触发。                                                     |
| **pre-package-update**        | 在资源包更新前触发。                                                     |
| **post-package-update**       | 在资源包更新后触发。                                                     |
| **pre-package-uninstall**     | 在资源包被卸载前触发。                                                    |
| **post-package-uninstall**    | 在资源包被卸载后触发。                                                    |
| **pre-autoload-dump**         | 在自动加载器被转储前触发，无论是 `install`/`update` 还是 `dump-autoload` 命令都会触发。 |
| **post-autoload-dump**        | 在自动加载器被转储后触发，无论是 `install`/`update` 还是 `dump-autoload` 命令都会触发。 |
| **post-root-package-install** | 在 `create-project` 命令期间，根包安装完成后触发。                             |
| **post-create-project-cmd**   | 在 `create-project` 命令执行后触发。                                    |
- **pre-archive-cmd**: occurs before the `archive` command is executed.
- **post-archive-cmd**: occurs after the `archive` command is executed.

> 注意：Composer 不会去执行任何依赖包中定义的 `install` 或 `update` 相关脚本。
> 因此你不应该在依赖包中申明 `pre-update-cmd` 或 `pre-install-cmd`。如果你需要在执行 `install` 或 `update` 命令前使用脚本，请确保它们已被定义在根包中。

## 定义脚本
在 `composer.json` 的根 JSON 对象中应该有一个名为 `"scripts"` 的属性，它包含有一系列的事件名称，以及对应的事件脚本。一个事件的脚本可以被定义为一个字符串（仅适用于单个脚本）或数组（单个或多个脚本）。

对于任何给定的事件：
- 脚本将按照事件和定义的**顺序**触发。
- 一个脚本数组可以包含 PHP 回调和命令行可执行命令。
- 由 PHP 类文件包含的回调，其存放的位置必须确保 Composer 能够正确的载入。
脚本定义实例：
```json
{
    "scripts": {
        "post-update-cmd": "MyVendor\\MyClass::postUpdate",
        "post-package-install": [
            "MyVendor\\MyClass::postPackageInstall"
        ],
        "post-install-cmd": [
            "MyVendor\\MyClass::warmCache",
            "phpunit -c app/"
        ]
    }
}
```
使用前面定义的例子，这里的 `MyVendor\MyClass` 类，就可以被使用来执行 PHP 的回调：
```php
<?php

namespace MyVendor;

use Composer\Script\Event;

class MyClass
{
    public static function postUpdate(Event $event)
    {
        $composer = $event->getComposer();
        // do stuff
    }

    public static function postPackageInstall(Event $event)
    {
        $installedPackage = $event->getOperation()->getPackage();
        // do stuff
    }

    public static function warmCache(Event $event)
    {
        // make cache toasty
    }
}
```
当一个事件被触发，Composer 的内部事件处理程序将接收一个 `Composer\Script\Event` 对象，这是传递给您的 PHP 回调的第一个参数。这个 `Event` 对象拥有一些 getter 方法来帮助你取得当前事件的上下文：
- `getComposer()`: 返回当前的 `Composer\Composer` 对象实例。
- `getName()`: 返回事件名称的字符串。
- `getIO()`: 返回当前的 输入\输出 流，它实现了 `Composer\IO\IOInterface` 接口，以便在控制台中使用。

## 手动运行脚本
如果你想手动运行事件脚本，可以使用下面的语法结构：
```sh
composer run-script [--dev] [--no-dev] script
```
例如 `composer run-script post-install-cmd` 将会运行所有 **post-install-cmd** 事件下定义的脚本。
## 生命周期钩子应用
### 一、Composer 的生命周期钩子
Composer 提供了多个内置的生命周期钩子（events），这些钩子会在特定阶段自动触发。常见的钩子包括：
#### 1. `post-install-cmd`
- **触发时机**：在 `composer install` 完成后执行。
- **用途**：安装依赖后执行自定义操作（如运行测试、生成代码等）。
- **配置示例**：
```json
{
	"scripts": {
	  "post-install-cmd": [
		"echo 'Dependencies installed successfully!'",
		"phpunit"
	  ]
	}
}
```
- 运行 `composer install` 后，会自动执行 `phpunit` 进行单元测试。
#### 2. `post-update-cmd`
- **触发时机**：在 `composer update` 完成后执行。
- **用途**：更新依赖后执行操作（如重新生成缓存、优化类加载等）。
- **配置示例**：
```json
{
	"scripts": {
	  "post-update-cmd": [
		"php bin/console cache:clear",
		"php bin/console doctrine:schema:update --force"
	  ]
	}
}
```
#### 3. `post-create-project-cmd`
- **触发时机**：在 `composer create-project` 完成后执行。
- **用途**：创建新项目后执行初始化操作（如生成 `.env` 文件、运行数据库迁移等）。
- **配置示例**：
```json
{
	"scripts": {
	  "post-create-project-cmd": [
		"cp .env.dist .env",
		"php bin/console doctrine:database:create"
	  ]
	}
}
```
### 二、Git 钩子与 Composer 的结合
通过 **composer-git-hooks** 工具，可以将 Git 钩子集成到 `composer.json` 中，实现更灵活的自动化流程。以下是常见用法：
#### 1. 安装 composer-git-hooks
```shell
composer require --dev brainmaestro/composer-git-hooks
```
#### 2. 在 `composer.json` 中配置 Git 钩子
```json
{
	"extra": {
		"hooks": {
			"pre-commit": [
				"php-cs-fixer fix .",  // 提交前格式化代码
				"phpunit"              // 提交前运行测试
			],
			"pre-push": [
				"php-cs-fixer fix . --dry-run",  // 推送前检查代码格式
				"phpunit --testsuite=unit"       // 推送前运行单元测试
			],
			"post-merge": [
				"composer install"  // 合并代码后自动安装依赖
			]
		}
	}
}
```
#### 3. 添加钩子到 Git
```shell
vendor/bin/cghooks add
```
- 执行此命令后，Git 会根据 `composer.json` 中的配置自动生成钩子脚本（如 `.git/hooks/pre-commit`）。
#### 4. 管理钩子
- **更新钩子**：如果 `composer.json` 中的钩子配置修改了，运行：
```shell
vendor/bin/cghooks update
```
- **移除钩子**：
```shell
vendor/bin/cghooks remove
```
#### 5. 全局钩子配置
如果希望在所有项目中统一使用相同的 Git 钩子，可以全局安装：
```shell
composer global require --dev brainmaestro/composer-git-hooks
```
然后在全局配置文件中定义钩子逻辑。
### 三、实际应用场景
#### 1. 自动化代码质量检查
- 在 `pre-commit` 钩子中运行代码格式化工具（如 `php-cs-fixer` 或 `phpcbf`）：
```json
"pre-commit": [
	"php-cs-fixer fix ."
]
```
- 确保提交的代码符合团队规范。
#### 2. 提交前运行测试
- 在 `pre-commit` 或 `pre-push` 钩子中运行单元测试：
```json
"pre-push": [
	"phpunit"
]
```
- 避免推送包含未通过测试的代码。
#### 3. 安装依赖后自动优化
- 在 `post-install-cmd` 中运行性能优化命令：
```json
"post-install-cmd": [
	"php bin/console cache:warmup",
	"php bin/console asset:install"
]
```
#### 4. 合并代码后自动同步依赖
- 在 `post-merge` 钩子中自动运行 `composer install`：
```json
"post-merge": [
	"composer install"
]
```
- 确保团队成员在合并代码后无需手动安装依赖。
### 四、注意事项
1. **权限问题**：
- 确保钩子脚本有可执行权限（如 `chmod +x .git/hooks/pre-commit`）。
2. **脚本失败处理**：
- 如果钩子脚本返回非零状态码，Git 会中止操作（如提交或推送）。
3. **跨平台兼容性**：
- 在 Windows 上使用 Git Bash 或 WSL 时，确保脚本语法兼容。
4. **版本控制**：
- 将钩子配置写入 `composer.json`，避免团队成员手动配置不一致。
### 五、总结
通过 Composer 的生命周期钩子和 Git 钩子的结合，可以显著提升开发流程的自动化水平。无论是依赖安装后的初始化操作，还是提交前的代码质量检查，这些钩子都能帮助团队保持一致性并减少人为错误。结合 `composer-git-hooks` 工具，开发者可以轻松地将 Git 钩子集成到项目中，实现更高效的协作流程。