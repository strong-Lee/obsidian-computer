## 1. 架构逻辑简述
- **GatewayWorker (端口 8282)**：只负责维护长连接，不处理业务，不查数据库。
- **Yii2 API (端口 80)**：负责接收前端发来的消息，鉴权，存库，然后推入队列。
- **Yii2 Queue (后台进程)**：负责异步调用 DeepSeek 接口，拿到结果后，通过 GatewayClient 推送给前端。

## 2. 安装依赖

```bash
# 1. 安装 GatewayWorker (服务端核心)
composer require workerman/gateway-worker

# 2. 安装 GatewayClient (Yii2 用于给 GatewayWorker 发命令的客户端)
composer require workerman/gatewayclient

# 3. 安装队列 (Redis) 和 HTTP 客户端 (请求 DeepSeek 用) ,这里咱们先不用redis
composer require yiisoft/yii2-queue yiisoft/yii2-redis guzzlehttp/guzzle
```

## 3. 服务端开发 (GatewayWorker)

### 3.1 创建事件处理类

这是一个纯逻辑类，用于处理“用户连上了”、“用户断开了”等事件。  
新建文件：console/components/ChatEvents.php (如果目录不存在请创建)
代码参考:  [[ChatEvents.php]]

### 3.2 创建启动命令
新建 console/controllers/ServerController.php。  
用于通过命令行启动 WebSocket 服务。
代码参考: [[ServerController.php]]

### Step 3: 启动服务 (测试)

**注意：** 如果你在本地是 Windows 环境（比如用 phpstudy 或 xampp），Workerman 在 Windows 下不支持 count > 1 且不支持 Daemon 模式。

**Linux / Mac 启动方式：**  
在项目根目录运行：
```bash
php yii server/start -d
php yii queue/listen --verbose
```

**Windows 启动方式：**
```bash
php yii server/start
```

如果看到类似 Workerman[ChatGateway] start in DEBUG mode 的绿色输出，说明 **WebSocket 服务启动成功了！** 监听端口是 8282。

### Step 4: Yii2 接口开发 (绑定与发送)

现在服务跑起来了，但它还不知道谁是谁。我们需要在 Yii2 的 API 模块里写两个接口。

假设你有一个 api/controllers/ChatController.php (或者你自己定义的控制器位置)。

**先配置组件：**  
在 common/config/main-local.php 或 params.php 中配置 GatewayClient 的注册地址，确保 Yii2 能找到 GatewayWorker。

1. **修改 common/config/main.php**  
    在 components 数组中添加 gateway 组件的配置
2. **创建 common/components/GatewayComponent.php**  
    (如果没有 components 目录请新建)
3.  **编写ChatController**  
4.  **补充ChatForm**  

### Step 5: 安装依赖 (Composer)

你需要安装两个库：

1. **yii2-queue**: 用于处理异步任务（官方扩展）。
    
2. **guzzle**: 用于发送 HTTP 请求给 DeepSeek。

```bash
composer require yiisoft/yii2-queue guzzlehttp/guzzle
```

启动队列监听器
```bash
php yii queue/listen
```

本地开发环境脚本启动
```bash
#!/bin/bash
# 杀死旧进程
killall php
# 后台启动 Server
php yii server/start -d
# 后台启动 Queue Listen
php yii queue/listen &
echo "Services Started!"

# 给个权限 chmod +x start_dev.sh
```

### 正式服务器怎么搞？(Supervisor 登场)

**生产环境部署步骤 (CentOS/Ubuntu):**

1. **安装 Supervisor**: yum install supervisor 或 apt-get install supervisor
    
2. **编写配置文件**: /etc/supervisord.d/hawk-queue.ini
	```
	[program:hawk-queue]
	command=php /path/to/your/project/yii queue/listen
	autostart=true
	autorestart=true
	user=www
	numprocs=1
	redirect_stderr=true
	stdout_logfile=/path/to/your/project/runtime/logs/queue-worker.log
	```

3. **GatewayWorker 同理**: /etc/supervisord.d/hawk-server.ini
```
[program:hawk-server]
command=php /path/to/your/project/yii server/start
autostart=true
autorestart=true
user=root
redirect_stderr=true
stdout_logfile=/path/to/your/project/runtime/logs/server.log
```

4. **启动**: supervisorctl update && supervisorctl start all