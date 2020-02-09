> 配套视频地址：[https://www.bilibili.com/video/av78577184](https://www.bilibili.com/video/av78577184 "https://www.bilibili.com/video/av78577184")

----

## 配置

```php
配置驱动 &quot;pusher&quot;, &quot;redis&quot;, &quot;log&quot;, &quot;null&quot;          // .env
```

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

```php
// bootstrap.js
import Echo from &quot;laravel-echo&quot;

window.io = require(&#039;socket.io-client&#039;);

window.Echo = new Echo({
    broadcaster: &#039;socket.io&#039;,
    host: window.location.hostname + &#039;:6001&#039;
});
```

###### **事件**

```php
&lt;?php

namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class Free implements ShouldBroadcast           // 1. 事件是要广播出去的
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $msg;

    public function __construct($msg)           // 2. 广播出去的内容
    {
        $this-&gt;msg = $msg;
    }

    public function broadcastOn()               // 3. 对哪些频道进行广播
    {
        return [new Channel(&#039;countryside&#039;), new Channel(&#039;a&#039;)];
    }
}
```

###### **设置客户端监听广播**

```php
window.Echo.channel(&#039;countryside&#039;)
    .listen(&#039;Free&#039;, (e) =&gt; {
        console.log(e);
    });
```

###### **触发事件，广播开始**

```php
Route::get(&#039;/event&#039;, function () {
    event(new \App\Events\Free(&#039;免缴农业税啦&#039;));
});
```



