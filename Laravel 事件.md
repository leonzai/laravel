> 配套视频地址：[https://www.bilibili.com/video/av77534496](https://www.bilibili.com/video/av77534496 "https://www.bilibili.com/video/av77534496")

---

## **目的：解耦。**

**简介：监听器监听到事件的发生，会执行 handler 方法。**

```php
// 原始代码
public function register(){
    // 注册用户
    // 成功后发送欢迎的信息通知
}


// 事件解耦后
public function register(){
    // 注册用户
    event(&#039;App\Events\RegisterOk&#039;);
}
```

#### **注册事件和监听器**

```php
# EventServiceProvider 
protected $listen = [
    &#039;App\Events\OrderShipped&#039; =&gt; [
        &#039;App\Listeners\SendShipmentNotification&#039;,
    ],
];



# 在 boot 方法里，以闭包方式注册
// event(&#039;event.name&#039;, $user);
Event::listen(&#039;event.name&#039;, function ($user) {
      
});
# 通配符
Event::listen(&#039;event.*&#039;, function ($eventName, array $data) {
    //
});



// 配置自动发现后可以不注册了，Laravel 会自动扫描目录。
# EventServiceProvider
public function shouldDiscoverEvents()
{
    return true;
}
// 配置自动扫描的目录
protected function discoverEventsWithin()
{
    return [
        $this-&gt;app-&gt;path(&#039;Listeners&#039;),
    ];
}
```

```doc
php artisan event:generate
// 避免手动建立事件类、监听器类，根据你在 `EventServiceProvider` 里 `$listen` 自动生成
```

```doc
// 生产环境避免每次请求扫描目录
php artisan event:cache
php artisan event:clear
```

#### **定义事件**

```php
<?php

namespace App\Events;

use App\Order;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use SerializesModels;

    public $order;

    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}
```

#### **定义监听器**

```php
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    public function __construct()
    {
        //
    }

    public function handle(OrderShipped $event)
    {
        // Access the order using $event->order...
        // 想禁止冒泡，请 return false
    }
}
```

#### **分发事件**

```php
$order = Order::findOrFail($orderId);
event(new OrderShipped($order));
```

#### **事件订阅器**

实现了在单个类中包含多个监听器。

```php
<?php

namespace App\Listeners;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin($event) {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout($event) {}

    /**
     * Register the listeners for the subscriber.
     *
     * @param  \Illuminate\Events\Dispatcher  $events
     */
    public function subscribe($events)
    {
        $events->listen(
            'Illuminate\Auth\Events\Login',
            'App\Listeners\UserEventSubscriber@handleUserLogin'
        );

        $events->listen(
            'Illuminate\Auth\Events\Logout',
            'App\Listeners\UserEventSubscriber@handleUserLogout'
        );
    }
}
```

###### **注册事件订阅器**

```php
# EventServiceProvider  
protected $subscribe = [
    'App\Listeners\UserEventSubscriber',
];
```