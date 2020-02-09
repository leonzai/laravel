> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=4](https://www.bilibili.com/video/av70545323?p=4 "https://www.bilibili.com/video/av70545323?p=4")

----

> 路由：从源地址传输到目的地址的活动。

#### **基础路由**

```php
// 两种目的地址：闭包和控制器
Route::get(&#039;foo&#039;, function () {
    return &#039;Hello World&#039;;
});

Route::get(&#039;/user&#039;, &#039;UserController@index&#039;);



// 带参数的源地址和目的地址
Route::get(&#039;posts/{post}&#039;, function ($postId) {
    //
});
```

#### **参数**

```php
Route::get(&#039;posts/{post}/comments/{comment}&#039;, function ($pid, $cid) {
    //
});

Route::get(&#039;user/{name?}&#039;, function ($name = &#039;John&#039;) {   // 一定要给可选参数设置默认值
    return $name;
});

# 对参数局部约束
Route::get(&#039;user/{id}&#039;, function ($id) {
    //
})-&gt;where(&#039;id&#039;, &#039;[0-9]+&#039;);

Route::get(&#039;user/{id}/{name}&#039;, function ($id, $name) {
    //
})-&gt;where([&#039;id&#039; =&gt; &#039;[0-9]+&#039;, &#039;name&#039; =&gt; &#039;[a-z]+&#039;]);



# 全局约束
// RouteServiceProvider
public function boot()
{
    Route::pattern(&#039;id&#039;, &#039;[0-9]+&#039;);

    parent::boot();
}

Route::get(&#039;user/{id}&#039;, function ($id) {
    // id 各位都是整数才能执行这儿
});
```

#### **参数绑定**

###### **隐式绑定**

```php
Route::get('api/users/{user}', function (App\User $user) {
    // 源地址中的 {} 中的变量名（即：user）和传参名必须完全一致
    return $user->email;
});



# 自定义键名，在模型中修改（默认指的是数据库中的主键 id）：
# App/User.php
public function getRouteKeyName()
{
    return 'slug';
    // api/users/2
    // select * from `users` where `slug` = 2 limit 1
}
```

###### **显式绑定**
```php
# RouteServiceProvider
public function boot()
{
    parent::boot();

    Route::model('user', App\User::class);
}

Route::get('profile/{user}', function ($user) {
    //
});
```

###### **自定义解析逻辑**

```php
// 在 app/Providers/RouteServiceProvider.php 中
public function boot()
{
    parent::boot();

    Route::bind('user', function ($value) {
        return App\User::where('name', $value)->first() ?? abort(404);
    });
}
```

```php
// 在 app/User.php 中
public function resolveRouteBinding($value)
{
    return $this->where('name', $value)->first() ?? abort(404);
}
```

#### **http 请求方法**

###### **单个方法**

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);        // 全体更新
Route::patch($uri, $callback);      // 局部更新
Route::delete($uri, $callback);
Route::options($uri, $callback);    // 允许客户端检查性能

Route::resource('photos', 'PhotoController');
```

方法|uri|路由名称|控制器@方法
--|--|--|--
GET  | photos |photos.index|PhotoController@index               
POST      | photos                     | photos.store   | PhotoController@store                 
GET| photos/create              | photos.create  | PhotoController@create      
GET| photos/{photo}             | photos.show    | PhotoController@show      
PUT/PATCH | photos/{photo}             | photos.update  | PhotoController@update     
DELETE    | photos/{photo}             | photos.destroy | PhotoController@destroy         
GET  | photos/{photo}/edit        | photos.edit    | PhotoController@edit  

```php
* GET /photos
index()  //  展示照片列表

* POST /photos
store()  // 添加照片

* GET /photos/create
create()  // 展示用来创建照片的表单

* GET /photos/{id}
show($id)  // 展示一张照片

* PUT /photos/{id}
update($id)  // 更新一张照片

* DELETE /photos/{id} 
destroy($id)  // 移除一张照片

* GET /photos/{id}/edit
edit($id)  // 展示编辑照片表单
```

###### **组合**

```php
Route::any($uri, $callback);        // 任意 method

Route::match(['get', 'post'], '/', function () {
    //
});
```

**注意点
在 web.php 路由里的 POST, PUT, DELETE 方法，在提交表单时候必须加上CSRF参数。**

```html
<form method="POST" action="/profile">
	@csrf
	...
</form>
```

###### **表单伪造**

```php
<input type="hidden" name="_method" value="PUT">
// 或者 @method('PUT')
```

#### **命名路由**

```php
Route::get('user/profile', function () {
    //
})->name('profile');

// 使用
$url = route('profile');
return redirect()->route('profile');



// 带参数的情况
Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1]);
```

#### **根据需求丰富路由**

###### **命名空间**

```php
Route::namespace('Admin')->group(function () {
    // 在 "App\Http\Controllers\Admin" 命名空间下的控制器
    Route::get('/user', 'UserController@index');  
    // Admin/UserController@index
    Route::get('/user2', 'UserController@user2');  
    // Admin/UserController@user2
});

Route::namespace('Admin')->get('/user', 'UserController@index');  
// Admin/UserController@index

Route::namespace('Admin')->get('/user2', 'UserController@user2');  
// Admin/UserController@user2
```

###### **子域名路由**

```php
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});

# 在注册根域路由之前注册子域路由。 这将防止根域路由覆盖具有相同 URI 路径的子域路由。
```

###### **路由前缀**

```php
Route::prefix('admin')->group(function () {
    Route::get('users', function () {

    });
});
// 相当于
Route::get('admin/users', function () {

});
```

###### **路由命名前缀**

```php
Route::name('admin.')->group(function () {
    Route::get('users', function () {
       
    })->name('users');  // name('admin.users')
});
```

###### **添加中间件**

[跳转到：Laravel 中间件](http://qianjinyike.com/laravel-中间件/ &quot;跳转到：Laravel 中间件&quot;)

```php
Route::middleware('throttle:60,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});

Route::get('/user', function () {
        //
})->middleware('auth:api');

Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // 使用 `first` 和 `second` 中间件
    });

    Route::get('user/profile', function () {
        // 使用 `first` 和 `second` 中间件
    });
});
```

#### **一些简化的基本路由**

```php
# 重定向路由
Route::redirect('/here', '/there');
Route::permanentRedirect('/here', '/there');  // 301
Route::redirect('/here', '/there', 301);  // 第三个参数不写则默认为 302

# 只需要返回一个视图
Route::view('/welcome', 'welcome');
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

#### **默认路由**

```php
Route::fallback(function () {
    // 处理 404
});  // 一定要放在所有路由最后面
```

#### **获取当前路由信息**

```php
// 假设有路由： Route::get('/', 'TestController@test')->name("mytest");

$route = Route::current(); // 返回  object(Illuminate\Routing\Route)
$name = Route::currentRouteName(); // 返回 mytest 

$action = Route::currentRouteAction(); // 控制器中返回：App\Http\Controllers\TestController@test  闭包中返回：null
```