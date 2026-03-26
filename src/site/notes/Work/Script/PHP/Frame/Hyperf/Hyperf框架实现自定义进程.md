---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Frame/Hyperf/Hyperf框架实现自定义进程/","title":"Hyperf框架实现自定义进程","tags":["flashcards"],"noteIcon":"","created":"2024-09-30T17:42:08.000+08:00","updated":"2026-03-24T17:32:07.008+08:00","dg-note-properties":{"title":"Hyperf框架实现自定义进程","tags":["flashcards"],"reference linking":null}}
---

在实际项目中经常需要写一些长期运行的脚本，如基于 Redis、Kafka、RabbitMQ 实现的多进程队列消费者，多进程爬虫、定时器等等,由于hyperf的自定义进程是通过swoole\server的addProcess方式实现的，默认会跟随server一起启动，虽然可通过isEnable方法控制是否跟随服务一同启动，但是在server集群架构下，由于其耦合程度较高，在服务高可用以及横向扩容时会相当麻烦。

所以我们需要将自定义进程服务与server服务分离开了，使其可以独立部署。
# 第一步
添加配置（为了使用起来简单，配置内容跟hyperf的配置一致）
创建配置文件 `config/autoload/task_process.php`
```php
declare (strict_types = 1);
/**
 * 自定义进程 (独立启动, 非UserProcess)
 */
return [
    \App\TaskProcess\TestProcess::class,
];
```
# 第二步
创建自定义进程基本类 `app/TaskProcess/BaseProcess.php`
```php
declare (strict_types = 1);
 
namespace App\TaskProcess;
 
use Hyperf\Process\AbstractProcess;
use Swoole;
use Swoole\Process;
use Swoole\Process\Manager;
use Swoole\Process\Pool;
 
abstract class BaseProcess {
 
	/**
	 * 进程数
	 * @var integer
	 */
	public $nums = 0;
 
	/**
	 * 进程名称
	 * @var string
	 */
	public $name = '';
 
	public $enableCoroutine = true;
 
	protected $isRunning = true;
 
	protected $process;
 
    static $signal = 0;
 
	function __construct() {
		//进程自动命名
		if (empty($this->name)) {
			$this->name = trim(str_replace('\\', '.', str_replace(__NAMESPACE__, '', get_called_class())), '.');
		}
	}
 
	final public function handle(Pool $pool, int $workerId): void {
		try {
			$this->processInit($pool->getProcess());
			$this->beforeRun();
			while (true) {
				//进程结束信号
				if (BaseProcess::$signal === SIGTERM) {
					$this->onProcessExit();
					break;
				}
				$this->run();
			}
		} catch (\Throwable $e) {
			throw $e;
		}
	}
 
	protected function onProcessExit() {
		$this->isRunning = false;
	}
 
	protected function processInit($process) {
 
		$this->process = $process;
        
        echo "process {$this->name} start, pid: " . getmypid().PHP_EOL;
 
        //注册信号处理器，实现优雅重启(等待任务执行完后或者等待超时)
		pcntl_signal(SIGTERM, function () {
			BaseProcess::$signal = SIGTERM;
			$maxWaitTime = config('server.settings.max_wait_time', 5);
			$sTime = time();
			//检查进程任务状态
			Swoole\Timer::tick(500, function () use ($sTime, $maxWaitTime) {
 
				$coStat = \Swoole\Coroutine::stats();
				//如果主循环结束，且其它协程任务执行完，清理定时器以退出进程
				if (!$this->isRunning && $coStat['coroutine_num'] <= 1) {
					Swoole\Timer::clearAll();
					$this->process->exit();
				}
				//等待超时，强制结束进程
				elseif (time() - $sTime >= $maxWaitTime) {
 
					Swoole\Timer::clearAll();
					if ($this->isRunning) {
						$this->onProcessExit();
					}
					$this->process->exit();
				}
			});
		});
 
	}
 
	public static function setProcessName(string $name) {
		swoole_set_process_name(env('APP_NAME', 'app') . '.taskProcess.' . $name);
	}
 
	/**
	 * 事件循环前调用
	 * @return [type] [description]
	 */
	abstract function beforeRun();
 
	/**
	 * 事件循环，注意这里不能使用死循环
	 * @return [type] [description]
	 */
	abstract function run();
 
}
```
# 创建业务逻辑类
```php 
declare (strict_types = 1);
 
namespace App\TaskProcess;
 
/**
 * test
 */
class TestProcess extends BaseProcess {
 
	/**
	 * 进程数
	 * @var integer
	 */
	public $nums = 1;
 
	public $enableCoroutine = true;
 
 
 
	public function beforeRun() {
		//事件循环前执行，比如一些初始化工作
	}
    
	public function run() {
        //事件循环主体    
		echo date('Y-m-d H:i:s').PHP_EOL;
        sleep(1);
	}
}
```
# 第三步：
自定义进程启动命令，创建app/Command/TaskProcessCommand.php
```php 
declare (strict_types = 1);
 
namespace App\Command;
 
use App\TaskProcess\BaseProcess;
use Hyperf\Command\Annotation\Command;
use Hyperf\Command\Command as HyperfCommand;
use Psr\Container\ContainerInterface;
use Swoole;
use Swoole\Process;
use Swoole\Process\Manager;
use Swoole\Process\Pool;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputOption;
 
/**
 * @Command
 */
#[Command]
class TaskProcessCommand extends HyperfCommand
{
    /**
     * @var ContainerInterface
     */
    protected $container;
 
    protected $coroutine = false;
 
    protected static $process = [];
 
    const MANAGER_PROCESS_PID_PATH = BASE_PATH . '/runtime/taskProcess.pid';
 
    public function __construct(ContainerInterface $container)
    {
        $this->container = $container;
        parent::__construct('process');
    }
 
    public function configure()
    {
        parent::configure();
        $this->setDescription('自定义进程任务');
        $this->addOption('daemonize', 'd', InputOption::VALUE_NONE, '守护进程化');
        $this->addArgument('action', InputArgument::REQUIRED, 'start/stop/restart 启动/关闭/重启');
    }
 
    public function handle()
    {
 
        $action = $this->input->getArgument('action');
 
        if ($action === 'start') {
            $this->start();
        } elseif ($action === 'stop') {
            $this->stop();
        } elseif ($action === 'restart') {
            $this->restart();
        } else {
            echo "不支持的action, 请输入 -h 参数查看" . PHP_EOL;
        }
 
    }
 
    protected function restart()
    {
        $this->stop();
        $this->start();
    }
 
    protected function stop()
    {
 
        if (file_exists(self::MANAGER_PROCESS_PID_PATH)) {
            $managerPid = file_get_contents(self::MANAGER_PROCESS_PID_PATH);
            echo "stopping...\n";
            echo "kill pid $managerPid \n";
            $managerPid = intval($managerPid);
            $startTime = time();
            $timeout = config('server.settings.max_wait_time', 10);
            @Process::kill($managerPid);
            //等待主进程结束
            while (@Process::kill($managerPid, 0)) {
                //waiting process stop
                echo "waiting...\r";
                usleep(100000);
                echo "              \r";
                echo "waiting.\r";
                usleep(100000);
                echo "              \r";
 
                //超时 强杀所有子进程
                if ($managerPid > 0 && time() - $startTime >= $timeout) {
                    echo "wait timeout, kill -9 child process, pid: $managerPid \n";
                    echo shell_exec("ps -ef|awk '$3~/^{$managerPid}$/'") . PHP_EOL;
                    echo shell_exec("ps -ef|awk '$3~/^{$managerPid}$/ {print $2}'|xargs kill -9") . PHP_EOL;
                }
            }
            unlink(self::MANAGER_PROCESS_PID_PATH);
            echo "stopped. \n";
 
        } else {
            echo "找不到manager pid, path: " . self::MANAGER_PROCESS_PID_PATH;
        }
 
    }
 
    protected function start()
    {
        $processConfig = config('task_process');
 
        if ($processConfig) {
 
            echo "start now.\n";
 
            $daemonize = $this->input->getOption('daemonize');
            if ($daemonize) {
                //重定向标准输出到指定日志文件
                fclose(STDOUT);
                fclose(STDERR);
                $STDOUT = fopen(BASE_PATH . '/runtime/logs/taskProcess_output.log', 'ab');
                $STDERR = fopen(BASE_PATH . '/runtime/logs/taskProcess_error.log', 'ab');
 
                Process::daemon(true, true);
            }
 
            //save pid
            file_put_contents(self::MANAGER_PROCESS_PID_PATH, getmypid());
 
            BaseProcess::setProcessName('manager');
 
            $startFuncMap = [];
 
            foreach ($processConfig as $processClass) {
                $processObj = new $processClass;
                if (($processObj instanceof BaseProcess) && isset($processObj->nums) && $processObj->nums > 0) {
                    for ($i = 0; $i < $processObj->nums; $i++) {
                        $startFuncMap[] = [
                            [$processObj, 'handle'],
                            $processObj->enableCoroutine ?? false,
                            $i,
                        ];
                    }
                }
            }
 
            $pool = new Pool(count($startFuncMap), SWOOLE_IPC_UNIXSOCK, 0, false);
 
            $pool->on('workerStart', function (Pool $pool, int $workerId) use ($startFuncMap) {
                [$func, $enableCoroutine, $idx] = $startFuncMap[$workerId];
                if ($enableCoroutine) {
                    run(function () use ($func, $pool, $workerId, $idx) {
                        $pm = $func[0];
                        $idx += 1;
                        BaseProcess::setProcessName($pm->name . "[{$idx}/{$pm->nums}]");
                        call_user_func($func, $pool, $workerId);
                    });
                } else {
                    $func($pool, $workerId);
                }
            });
 
            $pool->on('Message', function (Swoole\Process\Pool $pool, string $data) {
 
            });
 
            $pool->start();
 
 
        } else {
            printf("没有可启动的自定义进程, 请在配置task_process中声明，且继承%s\n", BaseProcess::class);
        }
    }
 
    public function getProcess($pid = -1)
    {
 
        if ($pid === -1) {
            $pid = getmypid();
        }
        return static::$process[$pid] ?? null;
    }
 
    public function getAllProcess()
    {
        return static::$process;
    }
}
```
好了，一切准备就绪，通过以下命令即可启动自定义进程服务
```shell
php bin/hyperf.php process start
# 在后台运行
# php bin/hyperf.php process start -d
```
查看进程
```shell
ps -ef | grep taskProcess
```