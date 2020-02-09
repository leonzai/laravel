> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=5](https://www.bilibili.com/video/av70545323?p=5 "https://www.bilibili.com/video/av70545323?p=5")

----

> 定义控制器可以不去继承 `Controllers`，这样你将不能使用  `middleware`、` validate`、`dispatch` 等方法。

###### **如果你的控制器只有一个方法，可以这么玩儿：**

```php
// Route::get(&#039;user/{id}&#039;, &#039;ShowProfile&#039;);
// php artisan make:controller ShowProfile --invokable
public function __invoke($id)
{
     return view(&#039;user.profile&#039;, [&#039;user&#039; =&gt; User::findOrFail($id)]);
}
```

###### **中间件**

路由中间件

`Route::get('profile', 'UserController@show')->middleware('auth');`

控制器的中间件

```php
$this-&gt;middleware(&#039;auth&#039;);
$this-&gt;middleware(&#039;auth&#039;)-&gt;only(&#039;index&#039;);
$this-&gt;middleware(&#039;auth&#039;)-&gt;except(&#039;store&#039;);
$this-&gt;middleware(function ($request, $next) {
       return $next($request);
});
```

###### **Resource 控制器**

```php
php artisan make:controller PhotoController --resource
php artisan make:controller PhotoController --resource --model=Photo

Route::resource(&#039;photos&#039;, &#039;PhotoController&#039;);
Route::apiResource(&#039;photos&#039;, &#039;PhotoController&#039;);    // 没有 create 和 edit

Route::resources([
    &#039;photos&#039; =&gt; &#039;PhotoController&#039;,
    &#039;posts&#039; =&gt; &#039;PostController&#039;
]);

Route::apiResources([
    &#039;photos&#039; =&gt; &#039;PhotoController&#039;,
    &#039;posts&#039; =&gt; &#039;PostController&#039;
]);

Route::resource(&#039;photos&#039;, &#039;PhotoController&#039;)-&gt;names([
    &#039;create&#039; =&gt; &#039;photos.build&#039;
]);  // 重命名路由名称


Route::resource(&#039;photo&#039;, &#039;PhotoController&#039;, [&#039;only&#039; =&gt; [
    &#039;index&#039;, &#039;show&#039;
]]);
Route::resource(&#039;photo&#039;, &#039;PhotoController&#039;, [&#039;except&#039; =&gt; [
    &#039;create&#039;, &#039;store&#039;, &#039;update&#039;, &#039;destroy&#039;
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
Route::resource(&#039;user&#039;, &#039;PhotoController&#039;, [&#039;parameters&#039; =&gt; [
    &#039;photo&#039; =&gt; &#039;photo_in_phone&#039;
]]);
# /user/{photo_in_phone}
```

重命名所有的动词名 create、edit

```php
// AppServiceProvider
Route::resourceVerbs([
    &#039;create&#039; =&gt; &#039;crear&#039;,
    &#039;edit&#039; =&gt; &#039;editar&#039;,
]);
```

> 如果想加入其他控制器方法，尽量写在 `resource` 控制器路由之前，否则可能会被 `resource` 控制器的路由覆盖。

写的控制器要专一，如果你需要典型的 `resource` 操作之外的方法，可以考虑将你的控制器分成两个更小的控制器。  

###### **参数必须在依赖注入之后传入**

```php
# Route::put(&#039;user/{id}&#039;, &#039;UserController@update&#039;);
public function update(Request $request, $id)
{
    // 上面的 Request $request 相当于： $request = new Resquest();
    $url = $request-&gt;url();
    //
}
```