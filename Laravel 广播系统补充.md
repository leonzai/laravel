> 配套视频地址：[https://www.bilibili.com/video/av80196918?p=2](https://www.bilibili.com/video/av80196918?p=2 "https://www.bilibili.com/video/av80196918?p=2")

----

###### **设置广播名称**

```php
// 默认是事件的类名
public function broadcastAs()
{
    return &#039;server.created&#039;;
}

//     .listen(&#039;.server.created&#039;, (e) =&gt; {
//         console.log(e);
//     });
```

###### **增加广播数据**

```php
// 默认只包含 public 属性
public function broadcastWith()
{
    return [&#039;id&#039; =&gt; $this-&gt;user-&gt;id];
}
```

###### **自定义授权端点**

```php
// 默认 broadcasting/auth
window.Echo = new Echo({
    broadcaster: &#039;socket.io&#039;,
    host: window.location.hostname + &#039;:6001&#039;
    authEndpoint: &#039;/custom/endpoint/auth&#039;
});
```

###### **定义频道授权**

```php
Broadcast::channel(&#039;channel&#039;, function() {
    // ...
}, [&#039;guards&#039; =&gt; [&#039;web&#039;, &#039;admin&#039;]]);
```

###### **定义频道类**

```php
php artisan make:channel OrderChannel
```

```php
use App\Broadcasting\OrderChannel;

Broadcast::channel(&#039;order.{order}&#039;, OrderChannel::class);
```

```php
&lt;?php

namespace App\Broadcasting;

use App\Order;
use App\User;

class OrderChannel
{
    /**
     * Create a new channel instance.
     *
     * @return void
     */
    public function __construct()
    {
        //
    }

    /**
     * Authenticate the user&#039;s access to the channel.
     *
     * @param  \App\User  $user
     * @param  \App\Order  $order
     * @return array|bool
     */
    public function join(User $user, Order $order)
    {
        return $user-&gt;id === $order-&gt;user_id;
    }
}
```

###### **获取当前连接的 socketId**

```php
var socketId = Echo.socketId();
```

###### **监听多个事件**

```php
Echo.private(&#039;orders&#039;)
    .listen(...)
    .listen(...)
    .listen(...);
```

###### **退出频道**

在 Echo 实例上调用 `leaveChannel` 方法，可以退出公有频道：

```php
Echo.leaveChannel(&#039;orders&#039;);
```

调用 `leave` 方法，可以退出公有频道、私有频道和在线频道：

```php
Echo.leave(&#039;orders&#039;);
```

###### **定制命名空间**

```php
window.Echo = new Echo({
    broadcaster: &#039;socket.io&#039;,
    host: window.location.hostname + &#039;:6001&#039;
    namespace: &#039;App.Other.Namespace&#039;    // 默认 App\Events
});
```

###### **客户端事件**

```html
<!doctype html>
<html lang="en">
<head>
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <script src="/js/app.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue/dist/vue.js"></script>
    <script>
        var roomId = "{{ $roomId }}";
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

        Echo.private('chat')
            .listenForWhisper('typing', (e) => {
                console.log('typing...');
                console.log(e);
            });
    </script>

</head>
<body>
<div id="app">
    <input type="text" id="msg" v-model="msg">

    <button onclick="axios.get('/room/{{ $roomId }}/'+document.getElementById('msg').value)">发送</button>
</div>


</body>
<script>
    new Vue({
        el: '#app',
        data() {
            return {
                msg: ''
            }
        },
        watch: {
            msg(value) {
                Echo.private('chat')
                    .whisper('typing', {
                        roomId: roomId
                    });
            }
        }
    })
</script>
</html>
```

```php
Broadcast::channel('chat', function () {
    return true;
});
```
