> 配套视频地址：[https://www.bilibili.com/video/av78577184?p=2](https://www.bilibili.com/video/av78577184?p=2 "https://www.bilibili.com/video/av78577184?p=2")

----

#### **配置**

```php
取消注释  BroadcastServiceProvider,用来注册广播授权路由和闭包。 // config/app.php
配置驱动 &quot;pusher&quot;, &quot;redis&quot;, &quot;log&quot;, &quot;null&quot;          // .env
配置数据库
```

```php
npm install --save socket.io-client      // websocket 客户端
npm install --save laravel-echo          // websocket 客户端封装
npm install -g laravel-echo-server       // websocket 服务端
npm install                              // 安装所有依赖
npm run watch                            // 监控文件变化编译前端资源
laravel-echo-server init                 // 初始化 websocket 服务端
laravel-echo-server start                // 启动 websocket 服务端
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

class Prize implements ShouldBroadcast
{
    use Dispatchable, InteractsWithSockets, SerializesModels;

    public $msg;
    /**
     * Create a new event instance.
     *
     * @return void
     */
    public function __construct($msg)
    {
        $this-&gt;msg = $msg;
    }

    /**
     * Get the channels the event should broadcast on.
     *
     * @return \Illuminate\Broadcasting\Channel|array
     */
    public function broadcastOn()
    {
        return new PrivateChannel(&#039;three_good&#039;);
    }
}
```

###### **设置频道权限**

```php
Broadcast::channel(&#039;three_good&#039;, function ($user) {
//    $ids = \DB::table(&#039;three_goods&#039;)-&gt;pluck(&#039;user_id&#039;)-&gt;toArray();
    $ids = [2, 3, 4, 8];

    return in_array($user-&gt;id, $ids);
});
```



###### **设置客户端监听广播**

```php
window.Echo.private(&#039;three_good&#039;)
    .listen(&#039;Prize&#039;, (e) =&gt; {
        console.log(e);
    });
```

###### **触发事件的路由**

```php
Route::get(&#039;/event&#039;, function () {
    event(new \App\Events\Prize(&#039;到教导处来领取奖状&#039;));
});
```

###### **引入 js 到前端**

```html
&lt;!doctype html&gt;
&lt;html lang=&quot;en&quot;&gt;
&lt;head&gt;
    &lt;title&gt;Laravel&lt;/title&gt;
    &lt;meta name=&quot;csrf-token&quot; content=&quot;{{ csrf_token() }}&quot;&gt;
    &lt;script src=&quot;/js/app.js&quot;&gt;&lt;/script&gt;
&lt;/head&gt;
&lt;body&gt;
    &lt;button onclick=&quot;axios.get(&#039;/event&#039;)&quot;&gt;send&lt;/button&gt;
&lt;/body&gt;
&lt;/html&gt;
```

###### **只对其他人广播**

```php
Route::get(&#039;/event&#039;, function () {
    broadcast(new \App\Events\Prize(&#039;到教导处来领取奖状&#039;))-&gt;toOthers();
});
```

