```php
<?php
namespace console\controllers;

use yii\console\Controller;
use Workerman\Worker;
use GatewayWorker\Register;
use GatewayWorker\Gateway;
use GatewayWorker\BusinessWorker;

/**
 * 聊天服务器管理脚本
 * 启动命令: php yii server/start
 * 停止命令: php yii server/stop
 * 重启命令: php yii server/restart
 */
class ServerController extends Controller
{
    public function actionStart()
    {
        $this->runServer('start');
    }

    public function actionStop()
    {
        $this->runServer('stop');
    }

    public function actionRestart()
    {
        $this->runServer('restart');
    }

    public function actionReload()
    {
        $this->runServer('reload');
    }

    private function runServer($action)
    {
        // 检查是不是在 Windows 下运行 (Workerman 在 Windows 下有局限性)
        if (strpos(strtolower(PHP_OS), 'win') === 0) {
            echo "Windows下建议使用 WSL 或 Docker 运行 Workerman，或者直接运行 start_for_win.bat (需单独配置)\n";
            // Windows 简单开发环境可以直接运行，但只能单进程
        }

        // 定义服务参数
        // 1. Register 服务 (服务注册中心)
        $register = new Register('text://0.0.0.0:1238');
        $register->name = 'ChatRegister';

        // 2. Gateway 服务 (网关，负责前端连接)
        // 注意：小程序正式上线强制要求 WSS (SSL)。
        // 这里的 websocket://0.0.0.0:8282 是非加密的，适合本地开发。
        // 生产环境建议用 Nginx 反向代理 wss 到 8282 端口，或者在这里配置 SSL 证书。
        $gateway = new Gateway("websocket://0.0.0.0:8282");
        $gateway->name = 'ChatGateway';
        $gateway->count = 2; // 进程数
        $gateway->lanIp = '127.0.0.1'; // 内网IP
        $gateway->startPort = 2900; // 内部通讯起始端口
        $gateway->registerAddress = '127.0.0.1:1238'; // 注册中心地址
        // 心跳设置：每50秒检测一次，如果不回复则断开
        $gateway->pingInterval = 50;
        $gateway->pingNotResponseLimit = 1; 
        $gateway->pingData = '{"type":"pong"}';

        // 3. BusinessWorker 服务 (业务逻辑)
        $worker = new BusinessWorker();
        $worker->name = 'ChatBusinessWorker';
        $worker->count = 2; // 进程数
        $worker->registerAddress = '127.0.0.1:1238';
        // 设置处理业务逻辑的类
        $worker->eventHandler = 'console\components\ChatEvents';

        // 设置 Workerman 运行参数
        global $argv;
        $argv[0] = 'yii server'; // 伪装脚本名
        $argv[1] = $action; // start, stop, etc.
        
        // 运行所有服务
        Worker::runAll();
    }
}
```