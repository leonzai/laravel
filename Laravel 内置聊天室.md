> 配套视频地址：[https://www.bilibili.com/video/av80196918](https://www.bilibili.com/video/av80196918 "https://www.bilibili.com/video/av80196918")

----

###### **配置**

1. 打开 `config/app.php` 中 `BroadcastServiceProvider` 注释，即注册广播授权路由。
2. 在 `.env` 中配置驱动 `BROADCAST_DRIVER=redis`
3. 在 `.env` 中配置数据库
4. 取消 `config/database.php` 中的 redis 前缀

```php
&#039;redis&#039; =&gt; [
	&#039;options&#039; =&gt; [
		&#039;prefix&#039; =&gt; &#039;&#039;,
	],
];
```

###### **安装服务**

打开命令行，cd 到项目根目录下

```bash
npm install --save socket.io-client |  echo &#039;websocket 客户端&#039;
npm install --save laravel-echo     |  echo &#039;websocket 客户端封装&#039;
npm install -g laravel-echo-server  |  echo &#039;websocket 服务端&#039;
npm install                         |  echo &#039;安装所有其他依赖&#039;
npm run watch                        |  echo &#039;监控文件变化编译前端资源&#039;
laravel-echo-server init             |  echo &#039;初始化 websocket 服务端&#039;
laravel-echo-server start            |  echo &#039;启动 websocket 服务端&#039;
```

###### **初始化 websocket 客户端**

在 `resources/js/bootstrap.js` 中添加以下代码

```php
import Echo from &quot;laravel-echo&quot;

window.io = require(&#039;socket.io-client&#039;);

window.Echo = new Echo({
    broadcaster: &#039;socket.io&#039;,
    host: window.location.hostname + &#039;:6001&#039;
});
```

###### **创建聊天室**

```php
Route::get(&#039;/login/user/{id}&#039;, fn($id) =&gt; auth()-&gt;loginUsingId($id));

Route::get(&#039;/room/{roomId}&#039;, function ($roomId) {
    broadcast(new \App\Events\Hello($roomId));
    return view(&#039;welcome&#039;, [&#039;roomId&#039; =&gt; $roomId]);
});
```

###### **安装聊天室客户端**

```php
window.Echo.join(`chat.${roomId}`)
    .here((users) => {
        console.log(users);
    })
    .joining((user) => {
        console.log(user.name + ' 来了');
    })
    .leaving((user) => {
        console.log(user.name + ' 走了');
    });
```

###### **安装频道认证路由**

```php
Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
//    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
//    }
});
```
###### **定义聊天室事件**

```php
<?php

namespace App\Events;

use App\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class Hello implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $user;
    public $roomId;
    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct($roomId)
    {
        $this->user = auth()->user();
        $this->roomId = $roomId;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('room.'.$this->roomId);
    }
}
```

#### **开启聊天模式**

###### **监听聊天事件**

```php
window.Echo.join(`chat.${roomId}`)
    .here((users) => {
        console.log(users);
    })
    .joining((user) => {
        console.log(user.name + ' 来了');
    })
    .leaving((user) => {
        console.log(user.name + ' 走了');
    })
    .listen('NewMessage', (e) => {
        console.log(e.user.name + "：" + e.msg);
    });
```

###### **定义聊天室群聊事件**

```php
<?php

namespace App\Events;

use App\User;
use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Log;

class NewMessage implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $user;
    public $roomId;
    public $msg;
    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct($roomId, $msg)
    {
        $this->user = auth()->user();
        $this->roomId = $roomId;
        $this->msg = $msg;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn()
    {
        return new PresenceChannel('chat.'.$this->roomId);
    }
}
```

###### **安装聊天室群聊客户端**

```php
<!doctype html>
<html lang="en">
<head>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <script>
        var roomId = "{{ $roomId }}";
    </script>
    <script src="/js/app.js"></script>
</head>
<body>
    <input type="text" id="msg">
    <button onclick="axios.get('/room/{{ $roomId }}/'+document.getElementById('msg').value)">发送</button>
</body>
</html>
```

###### **定义聊天室群聊路由**

```php
Route::get('/room/{roomId}/{msg}', function ($roomId, $msg) {
    broadcast(new \App\Events\NewMessage($roomId, $msg))->toOthers();
});
```
