> 配套视频地址：[https://www.bilibili.com/video/av77035719](https://www.bilibili.com/video/av77035719 "https://www.bilibili.com/video/av77035719")

---

#### **配置与准备**

```doc
配置文件：config/cache.php, .env
可配置内容：
    1. 使用哪个驱动
    2. 驱动的配置信息
    3. 所有缓存的统一 key 前缀

注意点：
    1. 使用 redis 驱动，需要安装 phpredis 拓展，并且注释 config/app.php 中的 alias 中 Redis 一行防止 Redis 命名空间冲突。
    2. 缓存的时间单位由早期的分钟变成了秒。
    3. Cache::flush(); 会删除 redis 中的所有缓存，不只是涉及你的项目！
    4. 原子锁只支持 memcached, dynamodb, redis 驱动，并且只与同一台中心 cache 服务器通信。
    5. file 和 database 驱动不支持 tag 功能。
        
提示：
    1. 尽量不要用永久存储。
    2. 如果 tag 功能涉及到永久存储，可以使用自动清除老旧记录的驱动，类似 memcached。
```

#### **配置不同的驱动**

```php
数据库
    php artisan cache:table
    php artisan migrate
```

```php
memcached
    回环地址链接
        &#039;memcached&#039; =&gt; [
            [
                &#039;host&#039; =&gt; &#039;127.0.0.1&#039;,
                &#039;port&#039; =&gt; 11211,
                &#039;weight&#039; =&gt; 100
            ],
        ],
    unix socket 连接	
        &#039;memcached&#039; =&gt; [
            [
                &#039;host&#039; =&gt; &#039;/var/run/memcached/memcached.sock&#039;,
                &#039;port&#039; =&gt; 0,
                &#039;weight&#039; =&gt; 100
            ],
        ],
```

###### **增**

```php
Cache::put(&#039;key&#039;, &#039;value&#039;, 10);
Cache::put(&#039;key&#039;, &#039;value&#039;);
Cache::put(&#039;key&#039;, &#039;value&#039;, now()-&gt;addMinutes(10));  // 第三个参数可以是 DateTime 对象

Cache::putMany([&#039;helen&#039; =&gt; new Student(&#039;helen&#039;), &#039;jack&#039; =&gt; &#039;10&#039;], 10);  // 这个是 ok 的
// 不生效 cache([&#039;helen&#039; =&gt; new Student(&#039;helen&#039;), &#039;jack&#039; =&gt; &#039;10&#039;], 10); 

$bool = Cache::add(&#039;key&#039;, &#039;value&#039;, 10);   // add: 如果缓存存在了，就返回 false；如果不存在，则添加缓存，然后返回 true

Cache::forever(&#039;key&#039;, &#039;value&#039;); // === Cache::put(&#039;key&#039;, &#039;value&#039;);

Cache::store(&#039;redis&#039;)-&gt;put(&#039;bar&#039;, &#039;baz&#039;, 600); // 10 Minutes
```

###### **删**

```php
Cache::forget(&#039;key&#039;);
Cache::put(&#039;key&#039;, &#039;value&#039;, 0);
Cache::put(&#039;key&#039;, &#039;value&#039;, -5);
```

> 注意：
>
> `Cache::flush();   // 不区别前缀，删除所有`

###### **查**

```php
$value = Cache::get(&#039;key&#039;);
$value = Cache::get(&#039;key&#039;, &#039;default&#039;);
if (Cache::has(&#039;key&#039;)) {   // 只要 key 的值是 null，返回 false。
    //
}

$value = Cache::store(&#039;file&#039;)-&gt;get(&#039;foo&#039;);
```
```php
$aStudent = Cache::many([&#039;helen&#039;, &#039;jack&#039;]);

$aStudent = Cache::many([&#039;helen&#039; =&gt; new Student(), &#039;jack&#039;]);
```

###### **改**

```php
Cache::put(&#039;key&#039;, &#039;value&#039;, 10);
Cache::put(&#039;key&#039;, &#039;value&#039;);
Cache::put(&#039;key&#039;, &#039;value&#039;, now()-&gt;addMinutes(10));  // 第三个参数可以是 DateTime 对象

Cache::putMany([&#039;helen&#039; =&gt; new Student(&#039;helen&#039;), &#039;jack&#039; =&gt; &#039;10&#039;], 10);  // 这个是 ok 的
// 不生效 cache([&#039;helen&#039; =&gt; new Student(&#039;helen&#039;), &#039;jack&#039; =&gt; &#039;10&#039;], 10); 

Cache::forever(&#039;key&#039;, &#039;value&#039;); // === Cache::put(&#039;key&#039;, &#039;value&#039;);

Cache::increment(&#039;key&#039;);
$amount = Cache::increment(&#039;key&#039;, 3)		// 如果 key 不存在，返回 3，并且设置缓存 key 为 3
Cache::decrement(&#039;key&#039;);
Cache::decrement(&#039;key&#039;, $amount);
```

#### **复合操作**

```php
$value = Cache::pull(&#039;key&#039;);  // 如果不存在 key，返回 null
$value = Cache::pull(&#039;key&#039;, &#039;value&#039;);
```

```php
$value = Cache::get(&#039;key&#039;, function () {
    return DB::table(...)-&gt;get();
});
```

```php
$users = Cache::get(&#039;users&#039;);
if (is_null($users)) {
    $users = DB::table(&#039;users&#039;)-&gt;get();
    Cache::set(&#039;users&#039;, $users, 10);
}

// 等同于

$users = Cache::remember(&#039;users&#039;, 10, function () {
    return DB::table(&#039;users&#039;)-&gt;get();
});
```

```php
$users = Cache::rememberForever(&#039;users&#039;, function () {  // 同名方法 sear
    return DB::table(&#039;users&#039;)-&gt;get();
});
```
#### **辅助全局函数**

```php
cache(&#039;key&#039;);

cache([&#039;key&#039; =&gt; &#039;value&#039;], $seconds);

cache([&#039;key&#039; =&gt; &#039;value&#039;], now()-&gt;addSeconds(10));

cache()-&gt;sear(&#039;users&#039;, function () {
    return DB::table(&#039;users&#039;)-&gt;get();
});
```

#### **tag: 设置标签**
```php
# 取的时候标签内容必须一致，不能多，不能少。
# 删的时候标签内容中只要命中，则删除
Cache::tags([&#039;people&#039;, &#039;artists&#039;])-&gt;put(&#039;John&#039;, 190, $seconds);
Cache::tags([&#039;people&#039;, &#039;authors&#039;])-&gt;put(&#039;Anne&#039;, 166, $seconds);

Cache::tags([&#039;people&#039;, &#039;authors&#039;])-&gt;get(&#039;Anne&#039;); 

Cache::tags(&#039;authors&#039;)-&gt;flush();  
Cache::tags([&#039;people&#039;, &#039;authors&#039;])-&gt;flush(); // 删除 people, authors, people and authors
```

#### **原子锁**

```php
$lock = Cache::lock(&#039;foo&#039;, 10);

if ($lock-&gt;get()) {
    // 获取锁 10s，超时释放锁

    $lock-&gt;release();
}
```

```php
Cache::lock(&#039;foo&#039;)-&gt;get(function () {
    // 闭包执行完毕，自动释放锁
});
```

```php
$lock = Cache::lock(&#039;foo&#039;, 10);

try {
    $lock-&gt;block(5); // 没有获取到锁的话，五秒内不断尝试再获取，超时就异常
    // ...
} catch (LockTimeoutException $e) {
    // Unable to acquire lock...
} finally {
    optional($lock)-&gt;release();
}

Cache::lock(&#039;foo&#039;, 10)-&gt;block(5, function () {
    // 没有获取到锁的话，五秒内不断尝试再获取，超时就异常。如果得到锁就执行闭包。闭包执行完毕，自动释放锁。
});
```