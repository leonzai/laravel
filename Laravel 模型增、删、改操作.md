> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=7](https://www.bilibili.com/video/av70545323?p=7 "https://www.bilibili.com/video/av70545323?p=7")

----

#### **写**
```php
\App\User::insert(
    [&#039;email&#039; =&gt; &#039;john@example.com&#039;, &#039;votes&#039; =&gt; 0]
);

\App\User::insert([
    [&#039;email&#039; =&gt; &#039;taylor@example.com&#039;, &#039;votes&#039; =&gt; 0],
    [&#039;email&#039; =&gt; &#039;dayle@example.com&#039;, &#039;votes&#039; =&gt; 0]
]);

\App\User::insertOrIgnore([
    [&#039;id&#039; =&gt; 1, &#039;email&#039; =&gt; &#039;taylor@example.com&#039;],
    [&#039;id&#039; =&gt; 2, &#039;email&#039; =&gt; &#039;dayle@example.com&#039;]
]);

$id = \App\User::insertGetId(
    [&#039;email&#039; =&gt; &#039;john@example.com&#039;, &#039;votes&#039; =&gt; 0]
);
# PostgreSQL 的 insertGetId 默认自增字段是 id，如果是其他的，需要传入字段名到 insertGetId 第二个参数。

$flight = new Flight;
$flight-&gt;name = $request-&gt;name;
$flight-&gt;save();
```

#### **改**

```php

$numbersOfRowsAffected = \App\User::where(&#039;id&#039;, 1)-&gt;update([&#039;votes&#039; =&gt; 1]);
// 当通过模型批量更新时，saving, saved, updating, and updated 模型事件将不会被更新后的模型触发。这是因为批量更新时，模型从来没有被取回。

$flight = App\Flight::find(1);
$flight-&gt;name = &#039;New Flight Name&#039;;
$flight-&gt;save();

# json
\App\User::where(&#039;id&#039;, 1)-&gt;update([&#039;options-&gt;enabled&#039; =&gt; true]);
```

```php
\App\User::increment(&#039;votes&#039;);
\App\User::increment(&#039;votes&#039;, 5);
\App\User::increment(&#039;votes&#039;, 1, [&#039;name&#039; =&gt; &#039;John&#039;]);
\App\User::decrement(&#039;votes&#039;);
\App\User::decrement(&#039;votes&#039;, 5);
```

#### **删**

```php
$numbersOfRowsAffected = \App\User::delete();
$numbersOfRowsAffected = \App\User::where('votes', '>', 100)->delete();
\App\User::truncate();


$flight = App\Flight::find(1);	// 取回模型再删除
$flight->delete();

// 或者
App\Flight::destroy(1);		// 直接删除
App\Flight::destroy([1, 2, 3]);
App\Flight::destroy(1, 2, 3);
App\Flight::destroy(collect([1, 2, 3]));

当使用 Eloquent 批量删除语句时，`deleting` 和 `deleted` 模型事件不会在被删除模型实例上触发。因为删除语句执行时，不会检索模型实例。
```
#### **软删除**

```php
use SoftDeletes;
protected $dates = ['deleted_at'];
```

&gt; 启用软删除的模型时，被软删除的模型将会自动从所有查询结果中排除。

**要确认指定的模型实例是否已经被软删除**

```php
if ($flight->trashed()) {
	//
}
```

**查询包含被软删除的模型**

```php
$flights = App\Flight::withTrashed()
                ->where('account_id', 1)
                ->get();
```

**只取出软删除数据**

```php
$flights = App\Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
```

**恢复软删除的模型**

```php
$flight->restore();

App\Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();
```

**永久删除模型**

```php
// 强制删除单个模型实例...
$flight->forceDelete();
```
