> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=5](https://www.bilibili.com/video/av70545323?p=5 "https://www.bilibili.com/video/av70545323?p=5")

----

> 定义控制器可以不去继承 `Controllers`，这样你将不能使用  `middleware`、` validate`、`dispatch` 等方法。

###### **如果你的控制器只有一个方法，可以这么玩儿：**

```php
// Route::get('user/{id}', 'ShowProfile');
// php artisan make:controller ShowProfile --invokable
public function __invoke($id)
{
     return view('user.profile', ['user' => User::findOrFail($id)]);
}
```

###### **中间件**

路由中间件

`Route::get('profile', 'UserController@show')->middleware('auth');`

控制器的中间件

```php
$this->middleware('auth');
$this->middleware('auth')->only('index');
$this->middleware('auth')->except('store');
$this->middleware(function ($request, $next) {
       return $next($request);
});
```

###### **Resource 控制器**

```php
php artisan make:controller PhotoController --resource
php artisan make:controller PhotoController --resource --model=Photo

Route::resource('photos', 'PhotoController');
Route::apiResource('photos', 'PhotoController');    // 没有 create 和 edit

Route::resources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);

Route::apiResources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);

Route::resource('photos', 'PhotoController')->names([
    'create' => 'photos.build'
]);  // 重命名路由名称


Route::resource('photo', 'PhotoController', ['only' => [
    'index', 'show'
]]);
Route::resource('photo', 'PhotoController', ['except' => [
    'create', 'store', 'update', 'destroy'
]]);
```

| Verb      | URI                    | Action  | Route Name     |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |


###### **命名 resource 路由的参数**

默认情况下，`Route::resource` 会根据资源名称的「单数」形式创建资源路由的路由参数。你可以在选项数组中传入 `parameters` 参数来轻松地覆盖每个资源。`parameters` 数组应该是资源名称和参数名称的关联数组：

```php
Route::resource('user', 'PhotoController', ['parameters' => [
    'photo' => 'photo_in_phone'
]]);
# /user/{photo_in_phone}
```

重命名所有的动词名 create、edit

```php
// AppServiceProvider
Route::resourceVerbs([
    'create' => 'crear',
    'edit' => 'editar',
]);
```

> 如果想加入其他控制器方法，尽量写在 `resource` 控制器路由之前，否则可能会被 `resource` 控制器的路由覆盖。

写的控制器要专一，如果你需要典型的 `resource` 操作之外的方法，可以考虑将你的控制器分成两个更小的控制器。  

###### **参数必须在依赖注入之后传入**

```php
# Route::put('user/{id}', 'UserController@update');
public function update(Request $request, $id)
{
    // 上面的 Request $request 相当于： $request = new Resquest();
    $url = $request->url();
    //
}
```
