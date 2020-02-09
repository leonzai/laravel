> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=6](https://www.bilibili.com/video/av70545323?p=6 "https://www.bilibili.com/video/av70545323?p=6")

----

#### **创建模型**

```php
php artisan make:model User       // 默认对应的表是 users
php artisan make:model AbCd       // 默认对应的表是 ab_cds
/**
规则：
    1. 除第一个大写字母，其他大写字母前都加上下划线
    2. 所有的大写字母改成小写
    3. 末尾加 s
*/
```

#### **模型之读**

```php
\App\User::all();                          // 返回包含所有对象的集合
\App\User::where('name', 'John')->first(); // 返回对象
\App\User::where('id', 1)->value('name');   // 返回值，例如 'leon'
\App\User::find(3);                        // 返回主键等于 3 的对象
\App\User::find([1, 2]);                   // 返回主键等于 1 和 2 的对象的集合
\App\User::pluck('age');                   // 返回包含字段值的集合
\App\User::pluck('age', 'id');             // 返回关联集合 id => age，pluck 最多 2 个参数
\App\User::count();                        // 返回记录总数
\App\User::max('id');                      // 返回数字，库没有任何记录返回 null
\App\User::min('id');                      // 返回数字，库没有任何记录返回 null
\App\User::avg('age');                     // 返回数字，库没有任何记录返回 null，同名 averge
\App\User::sum('salary');                  // 返回数字，库没有任何记录返回 0
```

###### **where**

```php
where('votes', '=', 100)
where('votes', 100)
where('votes', '>=', 100)
where('votes', '<>', 100)
where('name', 'like', 'T%')
where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])
where('votes', '>', 100)->orWhere('name', 'John')
whereBetween('votes', [1, 100])           // 包含了 1 和 100
whereNotBetween('votes', [1, 100])
whereIn('id', [1, 2, 3])
whereNotIn('id', [1, 2, 3])
whereNull('last_name')
whereNotNull('updated_at')
whereDate('created_at', '2016-12-31')   // where date(created_at) = '2016-12-31')
whereMonth('created_at', '12')
whereDay('created_at', '31')
whereYear('created_at', '2016')
whereTime('created_at', '=', '11:20:45')
whereColumn('first_name', 'last_name')        // 判断两个字段 相等
whereColumn('updated_at', '>', 'created_at')

whereColumn([
    ['first_name', '=', 'last_name'],
    ['updated_at', '>', 'created_at']
])

where('finalized', 1)->exists();      // 返回 true 或者 false
// select exists(select * from `xxx` where `finalized` = 1) as `exists`
where('finalized', 1)->doesntExist();
// 运行的 SQL 和上面的一样，Laravel 把运行结果取反就达成目的了
    
where('name', '=', 'John')
->orWhere(function ($query) {  // 传入闭包进 orWhere，避免全局 scope 产生不良影响
    $query->where('votes', '>', 100)
          ->where('title', '<>', 'Admin');
})
// where `name` = John or (`votes` > 100 and `title` <> Admin)

whereExists(function ($query) {
    $query->selectRaw(1)
          ->from('orders')
          ->whereRaw('orders.user_id = users.id');
})   
// where exists ( select 1 from orders where orders.user_id = users.id )
  
->whereRaw('price > IF(state = "TX", ?, 100)', [200])->get();
// IF 用法：IF(expr1,expr2,expr3)   
// 如果 (expr1 <> 0 and expr1 <> NULL)，那么 expr1 就是 true。
```

###### **json**

```php
$users = \App\User::where('options->language', 'en')
                ->get();
// select * from `users` where json_unquote(json_extract(`options`, '$."language"')) = 'en'

$users = \App\User::where('preferences->dining->meal', 'salad')
                ->get();
// select * from `users` where json_unquote(json_extract(`preferences`, '$."dining"."meal"')) = 'salad'

$users = \App\User::whereJsonLength('options->languages', 0)
                ->get();
// select * from `users` where json_length(`options`, '$."languages"') = 0

$users = \App\User::whereJsonLength('options->languages', '>', 1)
                ->get();
// select * from `users` where json_length(`options`, '$."languages"') > 1
```

```php
\App\User::orderBy('date')->where('active', false)->chunk(100, function ($users) {  // 取每 100 个一组
    foreach ($users as $user) {
        // ...
        // return false;                                           // 随时可以退出
    }
});      
// select * from `users` where `active` = 0 order by `date` asc limit 3 offset 0
// select * from `users` where `active` = 0 order by `date` asc limit 3 offset 3
// 注意，laravel 把 false 转化成了 0


// 不会出现 chunk 结束后，还有很多记录没被处理的情况
\App\User::where('active', false)->orderBy('date')
    ->chunkById(100, function ($users) {
        foreach ($users as $user) {
            \App\User::where('id', $user->id)
                ->update(['active' => true]);
        }
    });
// select * from `users` where `active` = 0 order by `date` asc, `id` asc limit 3
// select * from `users` where `active` = 0 and `id` > 3 order by `date` asc, `id` asc limit 3
```

###### **select**

```php
$users = \App\User::select('name', 'email as user_email')->get();

$users = \App\User::distinct()->get();
// select distinct * from `users1`

$query = \App\User::select('name');
$users = $query->addSelect('age')->get();
// select `name`, `age` from `users`
```

```php
->selectRaw('department, SUM(price) as total_sales')
->groupBy('department')
->havingRaw('SUM(price) > ?', [2500])
->orderByRaw('updated_at - created_at DESC')
->get();
// select department, SUM(price) as total_sales from `destinations` group by `department` having SUM(price) > 2500 order by updated_at - created_at DESC
```

###### **join 和 union**

```php
# inner join
->join('contacts', 'users.id', '=', 'contacts.user_id')
->join('orders', 'users.id', '=', 'orders.user_id')
->select('users.*', 'contacts.phone', 'orders.price')
->get();

# left join

->leftJoin('posts', 'users.id', '=', 'posts.user_id')
->get();

# right join

->rightJoin('posts', 'users.id', '=', 'posts.user_id')
->get();

# cross join

->crossJoin('colours')
->get();

# 高级 join

->join('contacts', function ($join) {
	$join->on('users.id', '=', 'contacts.user_id')->orOn('users.pid', '=', 'contacts.pid');
})
->get();
// select * from `users` 
//     inner join 
// `contacts` 
//     on `users`.`id` = `contacts`.`user_id` or `users`.`pid` = `contacts`.`pid`


->join('contacts', function ($join) {
	$join->on('users.id', '=', 'contacts.user_id')
	 	->where('contacts.user_id', '>', 5);
})
->get();
// select * from `users` inner join `contacts` 
//     on
// `users`.`id` = `contacts`.`user_id` and `contacts`.`user_id` > 5


# union
$first = App\User::whereNull('first_name');

$users = App\Student::whereNull('last_name')
            ->union($first)
            ->get();
// (select * from `students` where `last_name` is null) union (select * from `users` where `first_name` is null)

# unionAll 和 union 参数一样
// unionAll 不会去除重复值
```

###### **排序、分组和分页**

```php
orderBy('name', 'desc')

latest()                // === orderBy('created_at', 'desc')

inRandomOrder()
// order by RAND()

->groupBy('account_id')->having('account_id', '>', 100)

->groupBy('site', 'qianjinyike.com')->having('account_id', '>', 100)
    
skip(10)->take(5)        // 等价于 offset(10)->limit(5)
// limit 5 offset 10
```

###### **分支执行 sql**

```php
// $role 有值才会执行闭包
$role = $request->input('role');

$users = \App\User::when($role, function ($query) use ($role) {
                    return $query->where('role_id', $role);
                })
                ->get();

// $role 有值执行第一个闭包，否则执行第二个闭包
$sortBy = null;
$users = \App\User::when($sortBy, function ($query, $sortBy) {
                    return $query->orderBy($sortBy);
                }, function ($query) {
                    return $query->orderBy('name');
                })
                ->get();
```

###### **刷新模型**

```php
$flight = App\Flight::where('number', 'FR 900')->first();
$freshFlight = $flight->fresh();  // 去数据库里面重新取了一次

$flight = App\Flight::where('number', 'FR 900')->first();
$flight->number = 'FR 456';
$flight->refresh();
$flight->number; // "FR 900"
```

###### **游标**

`cursor` 允许你使用游标来遍历数据库数据，一次只执行单个查询。在处理大数据量请求时 `cursor` 方法可以大幅度减少内存的使用：

```php
foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
    //
}

$users = App\User::cursor()->filter(function ($user) {
    return $user->id > 500;
});
```

###### **高级子查询**

```php
Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderBy('arrived_at', 'desc')
    ->latest()
    ->limit(1)
])->get();
// select `destinations`.*, (select `name` from `flights` where `destination_id` = `destinations`.`id` order by `arrived_at` desc, `created_at` desc limit 1) as `last_flight` from `destinations`
```

###### **未找到异常**

`findOrFail` 以及 `firstOrFail` 方法会取回查询的第一个结果。如果没有找到相应结果，则会抛出一个 `Illuminate\Database\Eloquent\ModelNotFoundException`： 

```php
$model = App\Flight::findOrFail(1);

$model = App\Flight::where('legs', '>', 100)->firstOrFail();
```

###### **全局作用域**

```php
// 可以自由在  app 文件夹下创建 Scopes 文件夹来存放
<?php
namespace App\Scopes;
use Illuminate\Database\Eloquent\Scope;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class AgeScope implements Scope {
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('age', '>', 200);
    }   // 使用 addSelect 而不是 select,可以避免覆盖
}

// 需要重写给定模型的 boot 方法并使用 addGlobalScope 方法
<?php
namespace App;
use App\Scopes\AgeScope;
use Illuminate\Database\Eloquent\Model;
class User extends Model {
    protected static function boot() {
        parent::boot();

        static::addGlobalScope(new AgeScope);
    }
}
```

```sql
# 添加作用域后，如果使用 User::all() 查询则会生成如下SQL语句：
select * from `users` where `age` > 200
```

###### **匿名全局作用域（专门用来处理单个模型）**

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;
class User extends Model {
    protected static function boot() {
        parent::boot();

        static::addGlobalScope('age', function (Builder $builder) {
            $builder->where('age', '>', 200);
        });
    }
}
```

###### **移除全局作用域**

```php
# 我们还可以通过以下方式，利用 age 标识符来移除全局作用：
User::withoutGlobalScope('age')->get();

# 移除指定全局作用域
User::withoutGlobalScope(AgeScope::class)->get();

# 移除几个或全部全局作用域
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();

# 移除所有全局作用域
User::withoutGlobalScopes()->get();
```

###### **本地作用域**

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
class User extends Model {

    public function scopePopular($query) {
        return $query->where('votes', '>', 100);
    }

    public function scopeActive($query) {
        return $query->where('active', 1);
    }
}
```

```php
# 在进行方法调用时不需要加上 scope 前缀
$users = App\User::popular()->active()->orderBy('created_at')->get();

$users = App\User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();

$users = App\User::popular()->orWhere->active()->get();
```

###### **动态范围**

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
class User extends Model {
    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }
}

```

现在，你可以在范围调用时传递参数：

```php
$users = App\User::ofType('admin')->get();
```

