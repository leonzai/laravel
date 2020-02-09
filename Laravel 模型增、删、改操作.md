> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=7](https://www.bilibili.com/video/av70545323?p=7 "https://www.bilibili.com/video/av70545323?p=7")

----

#### **写**
```php
\App\User::insert(
    ['email' => 'john@example.com', 'votes' => 0]
);

\App\User::insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);

\App\User::insertOrIgnore([
    ['id' => 1, 'email' => 'taylor@example.com'],
    ['id' => 2, 'email' => 'dayle@example.com']
]);

$id = \App\User::insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
# PostgreSQL 的 insertGetId 默认自增字段是 id，如果是其他的，需要传入字段名到 insertGetId 第二个参数。

$flight = new Flight;
$flight->name = $request->name;
$flight->save();
```

#### **改**

```php

$numbersOfRowsAffected = \App\User::where('id', 1)->update(['votes' => 1]);
// 当通过模型批量更新时，saving, saved, updating, and updated 模型事件将不会被更新后的模型触发。这是因为批量更新时，模型从来没有被取回。

$flight = App\Flight::find(1);
$flight->name = 'New Flight Name';
$flight->save();

# json
\App\User::where('id', 1)->update(['options->enabled' => true]);
```

```php
\App\User::increment('votes');
\App\User::increment('votes', 5);
\App\User::increment('votes', 1, ['name' => 'John']);
\App\User::decrement('votes');
\App\User::decrement('votes', 5);
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

> 启用软删除的模型时，被软删除的模型将会自动从所有查询结果中排除。

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
