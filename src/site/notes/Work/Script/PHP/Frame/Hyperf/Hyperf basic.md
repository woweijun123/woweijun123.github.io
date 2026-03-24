---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Hyperf/Hyperf basic/","title":"Hyperf basic","tags":["flashcards"],"noteIcon":"","created":"2023-10-04T02:44:40.000+08:00","updated":"2026-03-24T17:32:05.312+08:00"}
---

# 常用命令
```php
// 创建模型
php bin/hyperf.php gen:model -F --path app/Model/Account user
```
# 集成`Symfony、Monolog`打印日志到控制台
1. /Users/weichengjun/dnmp/www/localhost/hyperf/internal/Logger/Handler/StdoutHandler.php
```php
namespace Internal\Logger\Handler;

use Monolog\Handler\AbstractProcessingHandler;
use Monolog\Logger;
use Monolog\LogRecord;
use Symfony\Component\Console\Output\ConsoleOutput;
use Symfony\Component\Console\Output\OutputInterface;

class StdoutHandler extends AbstractProcessingHandler
{
    protected OutputInterface $output;

    /**
     * @param int|string|Logger $level 错误级别
     * @param bool $bubble 是否处理其他 handler 的日志
     */
    public function __construct(int|string|Logger $level = Logger::DEBUG, bool $bubble = true)
    {
        parent::__construct($level, $bubble);
        $this->output = new ConsoleOutput();
    }

    protected function write(LogRecord|array $record): void
    {
        $this->output->write($record['formatted']);
    }
}
```
# 2.添加日志配置handler
```php
declare(strict_types=1);
use Internal\Logger\Handler\StdoutHandler;
use Monolog\Formatter\LineFormatter;
use Monolog\Handler\StreamHandler;
use Monolog\Logger;

return [
    'default' => [
        'handlers'  => [
            [
                'class'       => StdoutHandler::class, // 输出到终端
                'constructor' => [
                    'level'     => Logger::INFO, // 日志级别
                    'bubble'    => true,
                    'formatter' => [
                        'class'       => LineFormatter::class,
                        'constructor' => [
                            'format'                => "%datetime% %channel% %level_name% %message% %context% %extra%\n",
                            'dateFormat'            => 'Y-m-d H:i:s',
                            'allowInlineLineBreaks' => true,
                        ],
                    ],
                ],
            ],
            [
                'class'       => StreamHandler::class, // 输出到文件
                'constructor' => [
                    'stream' => BASE_PATH . '/runtime/logs/hyperf.log',
                    'level'  => env('LOG_LEVEL', Monolog\Logger::DEBUG),
                ],
            ],
        ],
        'formatter' => [
            'class'       => Monolog\Formatter\LineFormatter::class,
            'constructor' => [
                'format'                => null,
                'dateFormat'            => 'Y-m-d H:i:s',
                'allowInlineLineBreaks' => true,
            ],
        ],
    ]
];

```