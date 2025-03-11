
# Php请求示例

## 请求K线
```
<?php

// 设置API请求URL
$url = "https://api.itick.org/stock/kline?region=hk&code=700.HK&kType=1";

// 设置请求头
$headers = array(
    "accept: application/json",
    "token: you_apikey"
);

// 初始化CURL
$ch = curl_init();

// 设置CURL选项
curl_setopt($ch, CURLOPT_URL, $url);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
curl_setopt($ch, CURLOPT_HTTPHEADER, $headers);

// 执行请求
$response = curl_exec($ch);

// 检查是否有错误发生
if(curl_errno($ch)) {
    echo '请求错误: ' . curl_error($ch);
}

// 关闭CURL连接
curl_close($ch);

// 输出响应结果
echo $response;
```


## 订阅实时报价
```
<?php
require 'vendor/autoload.php';

use Ratchet\Client\WebSocket;
use React\EventLoop\Loop;

// WebSocket服务器的地址
$websocket_url = "wss://api.itick.org/sws";

// 用于鉴权的消息
$auth_message = [
    "ac" => "auth",
    "params" => "you_apikey"
];

// 用于订阅的消息格式
$subscribe_message = [
    "ac" => "subscribe",
    "params" => "AM.LPL,AM.LPL",
    "types" => "depth,quote"
];

// 创建事件循环
$loop = Loop::get();

\Ratchet\Client\connect($websocket_url)->then(function(WebSocket $conn) use ($auth_message, $subscribe_message) {
    // 连接成功时的处理
    echo "WebSocket连接已打开，正在发送鉴权消息...\n";
    
    // 发送鉴权消息
    $conn->send(json_encode($auth_message));
    
    // 发送订阅消息
    $conn->send(json_encode($subscribe_message));
    
    // 处理接收到的消息
    $conn->on('message', function($msg) {
        echo "收到消息: " . $msg . "\n";
        
        // 解析JSON数据
        $data = json_decode($msg, true);
        if (isset($data['data'])) {
            echo "数据内容: " . json_encode($data['data']) . "\n";
        }
    });
    
    // 处理错误
    $conn->on('error', function($error) {
        echo "WebSocket错误: " . $error->getMessage() . "\n";
    });
    
    // 处理连接关闭
    $conn->on('close', function($code = null, $reason = null) {
        echo "WebSocket连接已关闭，状态码: " . $code . "，消息: " . $reason . "\n";
    });
    
}, function($e) {
    // 连接失败时的处理
    echo "无法连接到WebSocket服务器: " . $e->getMessage() . "\n";
});

// 运行事件循环
$loop->run();
```
