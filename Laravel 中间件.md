> 配套视频教程：[https://www.bilibili.com/video/av83019817](https://www.bilibili.com/video/av83019817)

### 作用

过滤 http 请求。

### 生成中间件的命令

```php
php artisan make:middleware ShowAge
```

### 前置与后置中间件

```php
<?php

namespace App\Http\Middleware;

use Closure;

class CheckAge
{
    public function handle($request, Closure $next)
    {
        echo "20";

        return $next($request);
    }
}
```

```php
<?php

namespace App\Http\Middleware;

use Closure;

class CheckAge
{
    public function handle($request, Closure $next)
    {
		$response = $next($request);
        echo "20";

        return $response;
    }
}
```

### 使用中间件

```php
// 在路由中
Route::get('/pay', 'OrderController@pay')->middleware('auth');

use App\Http\Middleware\CheckAge;
Route::get('/pay', 'OrderController@pay')->middleware(CheckAge::class, 'auth');
```

```php
// 在控制器中
public function __construct()
{
    $this->middleware('auth');
}

$this->middleware('auth');
$this->middleware('auth')->only('index');
$this->middleware('auth')->except('store');
$this->middleware(function ($request, $next) {
       return $next($request);
});

$this->middleware('auth')->except('login', 'register', 'refresh');
```

### 中间件分组

```php
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        'throttle:60,1',
        'auth:api',
    ],
];
```

```php
Route::get('/', function () {
    //
})->middleware('web');

Route::group(['middleware' => ['web']], function () {
    //
});

Route::middleware(['web', 'subscribed'])->group(function () {
    //
});
```

### 给中间件传参数

```php
<?php

namespace App\Http\Middleware;

use Closure;

class CheckRole
{
	// 注意这里 handle 多接收了一个参数
    public function handle($request, Closure $next, $role)
    {
        if (! $request->user()->hasRole($role)) {
            // Redirect...
        }

        return $next($request);
    }

}
```

```php
Route::put('post/{id}', function ($id) {
    //
})->middleware('role:editor');
// 使用冒号分隔中间件名称和需要传递的参数
```

