> 配套视频地址：[https://www.bilibili.com/video/av74879198?p=5](https://www.bilibili.com/video/av74879198?p=5 "https://www.bilibili.com/video/av74879198?p=5")

---

#### **准备工作**

```php
composer create-project --prefer-dist laravel/laravel laravel6
.env 数据库配置
修改数据库默认字符串长度
入口文件中替换原生 Request 为 BaseRequest // 使得 request 和 response 都是 json 格式
php artisan make:controller PassportController
composer require laravel/passport
php artisan migrate // 创建表来存储客户端和 access_token 等
php artisan passport:install // 生成加密 access_token 的 key、密码授权客户端、个人访问客户端
Laravel\Passport\HasApiTokens Trait 添加到 App\User 模型中 // 提供一些辅助函数检查已认证用户的令牌和使用范围
```

```php
composer require guzzlehttp/guzzle
```

```php
// config/auth.php

&#039;defaults&#039; =&gt; [
    &#039;guard&#039; =&gt; &#039;api&#039;,
    &#039;passwords&#039; =&gt; &#039;users&#039;,
],

&#039;guards&#039; =&gt; [
    &#039;web&#039; =&gt; [
        &#039;driver&#039; =&gt; &#039;session&#039;,
        &#039;provider&#039; =&gt; &#039;users&#039;,
    ],

    &#039;api&#039; =&gt; [
        &#039;driver&#039; =&gt; &#039;passport&#039;,
        &#039;provider&#039; =&gt; &#039;users&#039;,
        &#039;hash&#039; =&gt; false,
    ],
],
```

```php
// AuthServiceProvider 中，设置 token 过期时间
Passport::tokensExpireIn(now()-&gt;addDays(15)); // access_token 过期时间
Passport::refreshTokensExpireIn(now()-&gt;addDays(60)); // refresh_token 过期时间
```

```php
Route::post(&#039;/oauth/token&#039;, &#039;\Laravel\Passport\Http\Controllers\AccessTokenController@issueToken&#039;);


Route::post(&#039;/register&#039;, &#039;PassportController@register&#039;);
Route::post(&#039;/login&#039;, &#039;PassportController@login&#039;);
Route::post(&#039;/refresh&#039;, &#039;PassportController@refresh&#039;);
Route::post(&#039;/logout&#039;, &#039;PassportController@logout&#039;);


Route::get(&#039;test&#039;, function () {
    return &#039;ok&#039;;
})-&gt;middleware(&#039;auth&#039;);
```

#### **登录、注册、刷新令牌、登出**

```php
&lt;?php
namespace App\Http\Controllers;

use App\User;
use GuzzleHttp\Client;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Validator;

class PassportController extends Controller
{
    protected $clientId;
    protected $clientSecret;

    public function __construct()
    {
        $this-&gt;middleware(&#039;auth&#039;)-&gt;except(&#039;login&#039;, &#039;register&#039;, &#039;refresh&#039;);
        $client = \DB::table(&#039;oauth_clients&#039;)-&gt;where(&#039;id&#039;, 2)-&gt;first();
        $this-&gt;clientId = $client-&gt;id;
        $this-&gt;clientSecret = $client-&gt;secret;
    }

    protected function username()
    {
        return &#039;email&#039;;
    }

    public function register()
    {
        $this-&gt;validator(request()-&gt;all())-&gt;validate();

        $this-&gt;create(request()-&gt;all());

        return $this-&gt;getToken();
    }

    protected function validator(array $data)
    {
        return Validator::make($data, [
            &#039;name&#039; =&gt; [&#039;required&#039;, &#039;string&#039;, &#039;max:255&#039;, &#039;unique:users&#039;,],
            &#039;email&#039; =&gt; [&#039;required&#039;, &#039;string&#039;, &#039;email&#039;, &#039;max:255&#039;,],
            &#039;password&#039; =&gt; [&#039;required&#039;, &#039;string&#039;, &#039;min:8&#039;, &#039;confirmed&#039;],
        ]);
    }

    protected function create(array $data)
    {
        return User::forceCreate([
            &#039;name&#039; =&gt; $data[&#039;name&#039;],
            &#039;email&#039; =&gt; $data[&#039;email&#039;],
            &#039;password&#039; =&gt; password_hash($data[&#039;password&#039;], PASSWORD_DEFAULT),
        ]);
    }

//    public function logout(Request $request)
//    {
//        auth()-&gt;user()-&gt;token()-&gt;revoke(); // 只会使 access_token 失效
//
//        return [&#039;message&#039; =&gt; &#039;退出登录成功&#039;];
//    }


    public function logout()
    {
        $tokenModel = auth()-&gt;user()-&gt;token();
        $tokenModel-&gt;update([
            &#039;revoked&#039; =&gt; 1,
        ]);

        \DB::table(&#039;oauth_refresh_tokens&#039;)
            -&gt;where([&#039;access_token_id&#039; =&gt; $tokenModel-&gt;id])-&gt;update([
                &#039;revoked&#039; =&gt; 1,
            ]);

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

        return $this-&gt;getToken();
    }

    public function refresh()
    {
        $response = (new Client())-&gt;post(&#039;http://lishen.com/api/oauth/token&#039;, [
            &#039;form_params&#039; =&gt; [
                &#039;grant_type&#039; =&gt; &#039;refresh_token&#039;,
                &#039;refresh_token&#039; =&gt; request(&#039;refresh_token&#039;),
                &#039;client_id&#039; =&gt; $this-&gt;clientId,
                &#039;client_secret&#039; =&gt; $this-&gt;clientSecret,
                &#039;scope&#039; =&gt; &#039;*&#039;,
            ],
        ]);

        return $response;
    }

    /**
     * @param Request $request
     * @param Client  $guzzle
     *
     * @return \Psr\Http\Message\ResponseInterface
     */
    private function getToken()
    {
        $response = (new Client())-&gt;post(&#039;http://lishen.com/api/oauth/token&#039;, [
            &#039;form_params&#039; =&gt; [
                &#039;grant_type&#039; =&gt; &#039;password&#039;,
                &#039;username&#039; =&gt; request(&#039;email&#039;),
                &#039;password&#039; =&gt; request(&#039;password&#039;),
                &#039;client_id&#039; =&gt; $this-&gt;clientId,
                &#039;client_secret&#039; =&gt; $this-&gt;clientSecret,
                &#039;scope&#039; =&gt; &#039;*&#039;,
            ],
        ]);

        return $response-&gt;getBody();
    }
}
```

#### **使用 scope**

```php
// AppServiceProvider 注册 scope
Passport::tokensCan([
    &#039;test1&#039; =&gt; &#039;for test1&#039;,
    &#039;test2&#039; =&gt; &#039;for test2&#039;,
]);

// 注册中间件
&#039;scopes&#039; =&gt; \Laravel\Passport\Http\Middleware\CheckScopes::class,  // and
&#039;scope&#039; =&gt; \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,  // or

// 使用
-&gt;middleware(&#039;scopes:test1,test2&#039;);
-&gt;middleware(&#039;scope:test1,test2&#039;);
```

```php
if (auth()-&gt;user()-&gt;tokenCan(&#039;place-orders&#039;)) {
    //
}
```

#### **其他操作**

```php
Laravel\Passport\Passport::scopeIds(); // [&quot;test1&quot;,&quot;test2&quot;]

Laravel\Passport\Passport::scopes(); // [{&quot;id&quot;:&quot;test1&quot;,&quot;description&quot;:&quot;for test1&quot;},{&quot;id&quot;:&quot;test2&quot;,&quot;description&quot;:&quot;for test2&quot;}]

Laravel\Passport\Passport::scopesFor([&#039;test1&#039;, &#039;check-status&#039;]); // [{&quot;id&quot;:&quot;test1&quot;,&quot;description&quot;:&quot;for test1&quot;}]

Laravel\Passport\Passport::hasScope(&#039;place-orders&#039;); // false
```

