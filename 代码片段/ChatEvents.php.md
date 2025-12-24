```php
<?php
namespace console\components;

use GatewayWorker\Lib\Gateway;

class ChatEvents
{
    /**
     * 当客户端连接时触发
     * @param int $client_id 全局唯一的客户端ID
     */
    public static function onConnect($client_id)
    {
        // 连接成功，向客户端发送一个初始化消息，包含 client_id
        // 这一步很重要，前端拿到 client_id 后，需要请求 Yii2 接口进行“绑定”
        Gateway::sendToClient($client_id, json_encode([
            'type' => 'init',
            'client_id' => $client_id
        ]));
    }

    /**
     * 当客户端发来消息时触发
     * (注意：我们主要推荐走 HTTP 接口发消息，这个回调主要用于心跳检测等简单逻辑)
     */
    public static function onMessage($client_id, $message)
    {
        // 可以在这里处理心跳
        // $data = json_decode($message, true);
        // if($data['type'] === 'ping') return;
    }

    /**
     * 当客户端断开连接时触发
     */
    public static function onClose($client_id)
    {
        // GatewayWorker 会自动清理组关系，通常这里不需要写代码
    }
}
```