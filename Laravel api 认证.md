> 配套视频地址：[https://www.bilibili.com/video/av74879198?p=3](https://www.bilibili.com/video/av74879198?p=3 "https://www.bilibili.com/video/av74879198?p=3")

---

#### **原理**

1. 注册：用户注册成功后，随机生成长字符串作为 token，原生 token 返回给用户。哈希后的 token 存到数据库里。
2. 登陆：用户使用账号密码登陆成功，随机生成长字符串作为 token，原生 token 返回给用户。哈希后的 token 存到数据库里。
3. 认证：将用户传来的 token 进行哈希，然后去数据库中查找哈希后的 token ，找到了就认证成功，否则失败。

#### **创建项目与配置**

```php
composer create-project --prefer-dist laravel/laravel laravel6
php artisan migrate
添加 api_token 字段，可空，唯一，默认 null。（可直接修改，也可以创建下面的代码片段然后迁移）
```

```php
Schema::table(&#039;users&#039;, function ($table) {
    $table-&gt;string(&#039;api_token&#039;, 80)-&gt;after(&#039;password&#039;)
                        -&gt;unique()
                        -&gt;nullable()
                        -&gt;default(null);
});

php artisan migrate
```

我们的例子还需要设置 email 可为空，因为我们以用户名作为认证的依据

###### **设置模型可以操作 api_token 字段**

```php
# App\User.php
protected $fillable = [
    &#039;name&#039;, &#039;email&#039;, &#039;password&#039;, &#039;api_token&#039;,
];
```

###### **修改 api_token 这个名称**

如果修改字段名称 api_token，请记得修改配置文件 config/auth.php 中的 storage_key

```php
&#039;api&#039; =&gt; [
    &#039;driver&#039; =&gt; &#039;token&#039;,
    &#039;provider&#039; =&gt; &#039;users&#039;,
    &#039;hash&#039; =&gt; false,
    &#039;storage_key&#039; =&gt; &#039;api_token&#039;,
],
```

###### **guard 设置为 api，hash 设置为 true**

```php
// config/auth.php

&#039;defaults&#039; =&gt; [
    &#039;guard&#039; =&gt; &#039;api&#039;,     // 默认 api 认证
    &#039;passwords&#039; =&gt; &#039;users&#039;,
],

&#039;api&#039; =&gt; [
    &#039;driver&#039; =&gt; &#039;token&#039;,
    &#039;provider&#039; =&gt; &#039;users&#039;,
    &#039;hash&#039; =&gt; true,       // 利用 SHA-256 算法哈希你的令牌
],
```

###### **设置所有请求和响应都是 json 格式**

> 请注意不要使用 `php artisan make:request BaseRequest`，这会使得你的 BaseRequest 继承 FormRequest，而我们要覆盖的是 `Illuminate\Http\Request`，所以我们应当继承的是 Request。

新建文件 `app\Http\Requests\BaseRequest.php`：
```php
&lt;?php

namespace App\Http\Requests;

use Illuminate\Http\Request;

class BaseRequest extends Request
{

    public function expectsJson()
    {
        return true;
    }
    public function wantsJson()
    {
        return true;
    }
}
```

```php
// index.php
$response = $kernel-&gt;handle(
    $request = \App\Http\Requests\BaseRequest::capture()
);
```

#### **编写 api 认证代码**

```php
Route::post(&#039;/register&#039;, &#039;Auth\ApiController@register&#039;);
Route::post(&#039;/login&#039;, &#039;Auth\ApiController@login&#039;);
Route::post(&#039;/refresh&#039;, &#039;Auth\ApiController@refresh&#039;);
Route::post(&#039;/logout&#039;, &#039;Auth\ApiController@logout&#039;);
```

`php artisan make:controller Auth\ApiController`

```php
&lt;?php

// email 设置可为空
// request 和 response 都是 json 格式
// api_token 设置可插入数据库    
    
namespace App\Http\Controllers\Auth;

use App\Http\Controllers\Controller;
use App\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;
use Illuminate\Support\Str;

class ApiController extends Controller
{
    public function __construct()
    {
        $this-&gt;middleware(&#039;auth&#039;)-&gt;except(&#039;login&#039;, &#039;register&#039;);
    }

    protected function username()
    {
        return &#039;name&#039;;
    }

    public function register(Request $request)
    {
        $this-&gt;validator($request-&gt;all())-&gt;validate();

        $api_token = Str::random(80);
        $data = array_merge($request-&gt;all(), compact(&#039;api_token&#039;));
        $this-&gt;create($data);

        return compact(&#039;api_token&#039;);
    }

    protected function validator(array $data)
    {
        return Validator::make($data, [
            &#039;name&#039; =&gt; [&#039;required&#039;, &#039;string&#039;, &#039;max:255&#039;, &#039;unique:users&#039;,],
//            &#039;email&#039; =&gt; [&#039;required&#039;, &#039;string&#039;, &#039;email&#039;, &#039;max:255&#039;,],
            &#039;password&#039; =&gt; [&#039;required&#039;, &#039;string&#039;, &#039;min:8&#039;, &#039;confirmed&#039;],
        ]);
    }

    protected function create(array $data)
    {
        return User::forceCreate([
            &#039;name&#039; =&gt; $data[&#039;name&#039;],
//            &#039;email&#039; =&gt; $data[&#039;email&#039;],
            &#039;password&#039; =&gt; password_hash($data[&#039;password&#039;], PASSWORD_DEFAULT),
            &#039;api_token&#039; =&gt; hash(&#039;sha256&#039;, $data[&#039;api_token&#039;]),
        ]);
    }

    public function logout()
    {
        auth()-&gt;user()-&gt;update([&#039;api_token&#039; =&gt; null]);

        return [&#039;message&#039; =&gt; &#039;退出登录成功&#039;];
    }

    public function login()
    {
        $user = User::where($this-&gt;username(), request($this-&gt;username()))
            -&gt;firstOrFail();

        if (!password_verify(request(&#039;password&#039;), $user-&gt;password)) {
            return response()-&gt;json([&#039;error&#039; =&gt; &#039;抱歉，账号名或者密码错误！&#039;],
                403);
        }

        $api_token = Str::random(80);
        $user-&gt;update([&#039;api_token&#039; =&gt; hash(&#039;sha256&#039;, $api_token)]);

        return compact(&#039;api_token&#039;);
    }

    public function refresh()
    {
        $api_token = Str::random(80);
        auth()-&gt;user()-&gt;update([&#039;api_token&#039; =&gt; hash(&#039;sha256&#039;, $api_token)]);

        return compact(&#039;api_token&#039;);
    }
}
```

#### **保护路由**

`middleware('auth:api')`

#### **给 Request 传 token**

```php
$response = $client-&gt;request(&#039;GET&#039;, &#039;/api/user?api_token=&#039;.$token);

$response = $client-&gt;request(&#039;POST&#039;, &#039;/api/user&#039;, [
    &#039;headers&#039; =&gt; [
        &#039;Accept&#039; =&gt; &#039;application/json&#039;,
    ],
    &#039;form_params&#039; =&gt; [
        &#039;api_token&#039; =&gt; $token,
    ],
]);

$response = $client-&gt;request(&#039;POST&#039;, &#039;/api/user&#039;, [
    &#039;headers&#039; =&gt; [
        &#039;Authorization&#039; =&gt; &#039;Bearer &#039;.$token,
        &#039;Accept&#039; =&gt; &#039;application/json&#039;,
    ],
]);
```