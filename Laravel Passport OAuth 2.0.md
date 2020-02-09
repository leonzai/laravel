> 配套视频地址：[https://www.bilibili.com/video/av74879198](https://www.bilibili.com/video/av74879198 "https://www.bilibili.com/video/av74879198")

---

```php
composer require laravel/passport
php artisan migrate // 创建表来存储客户端和 access_token
php artisan passport:install // 生成加密 access_token 的 key、密码授权客户端、个人访问客户端
Laravel\Passport\HasApiTokens Trait 添加到 App\User 模型中 // 提供一些辅助函数检查已认证用户的令牌和使用范围
在 AuthServiceProvider 的 boot 方法中调用 Passport::routes 函数 // 访问令牌并撤销访问令牌路由，客户端和个人访问令牌相关路由
config/auth.php 中 api 的 driver 选项改为 passport
```

#### **自定义 passport migration**

` php artisan vendor:publish --tag=passport-migrations `

#### **生成加密 access_token 的 key**

`php artisan passport:keys`

#### **AuthServiceProvider 中指定 passport key 加载路径**

`Passport::loadKeysFrom('/secret-keys/oauth');`

```php
PASSPORT_PRIVATE_KEY=&quot;-----BEGIN RSA PRIVATE KEY-----
&lt;private key here&gt;
-----END RSA PRIVATE KEY-----&quot;

PASSPORT_PUBLIC_KEY=&quot;-----BEGIN PUBLIC KEY-----
&lt;public key here&gt;
-----END PUBLIC KEY-----&quot;
```

## **配置**

#### **过期时间**

```php
Passport::tokensExpireIn(now()-&gt;addDays(15)); // access_token
Passport::refreshTokensExpireIn(now()-&gt;addDays(30));// refresh_token
Passport::personalAccessTokensExpireIn(now()-&gt;addMonths(6)); // personal access_token
```

#### **重写模型**

```php
use App\Models\Passport\AuthCode;
use App\Models\Passport\Client;
use App\Models\Passport\PersonalAccessClient;
use App\Models\Passport\Token;

/**
 * Register any authentication / authorization services.
 *
 * @return void
 */
public function boot()
{
    $this-&gt;registerPolicies();

    Passport::routes();

    Passport::useTokenModel(Token::class);
    Passport::useClientModel(Client::class);
    Passport::useAuthCodeModel(AuthCode::class);
    Passport::usePersonalAccessClientModel(PersonalAccessClient::class);
}
```

#### **管理客户端**

###### **命令行创建客户端**

```php
php artisan passport:client
// 设置回调地址白名单的格式：http://example.com/callback,http://examplefoo.com/callback （逗号隔开）
```

###### **api 管理客户端**

```php
axios.get(&#039;/oauth/clients&#039;)
    .then(response =&gt; {
        console.log(response.data);
    });
```

```php
const data = {
    name: &#039;Client Name&#039;,
    redirect: &#039;http://example.com/callback&#039;
};

axios.post(&#039;/oauth/clients&#039;, data)
    .then(response =&gt; {
        console.log(response.data);
    })
    .catch (response =&gt; {
        // List errors on response...
    });
```

```php
const data = {
    name: &#039;New Client Name&#039;,
    redirect: &#039;http://example.com/callback&#039;
};

axios.put(&#039;/oauth/clients/&#039; + clientId, data)
    .then(response =&gt; {
        console.log(response.data);
    })
    .catch (response =&gt; {
        // List errors on response...
    });
```

```php
axios.delete(&#039;/oauth/clients/&#039; + clientId)
    .then(response =&gt; {
        //
    });
```

#### **授权码模式**

请求 token

```php
Route::get(&#039;/redirect&#039;, function (Request $request) {
    $request-&gt;session()-&gt;put(&#039;state&#039;, $state = Str::random(40));

    $query = http_build_query([
        &#039;client_id&#039; =&gt; &#039;client-id&#039;,
        &#039;redirect_uri&#039; =&gt; &#039;http://example.com/callback&#039;,
        &#039;response_type&#039; =&gt; &#039;code&#039;,
        &#039;scope&#039; =&gt; &#039;&#039;,
        &#039;state&#039; =&gt; $state,
    ]);

    return redirect(&#039;http://your-app.com/oauth/authorize?&#039;.$query);
});
```

###### **自定义用户授权页面**

```php
php artisan vendor:publish --tag=passport-views
```

###### **跳过用户授权页面**

```php
&lt;?php

namespace App\Models\Passport;

use Laravel\Passport\Client as BaseClient;

class Client extends BaseClient
{
    public function skipsAuthorization()
    {
        return $this-&gt;firstParty();
    }
}
```

###### **获取 access_token**

```php
Route::get(&#039;/callback&#039;, function (Request $request) {
    $state = $request-&gt;session()-&gt;pull(&#039;state&#039;);

    throw_unless(
        strlen($state) &gt; 0 &amp;&amp; $state === $request-&gt;state,
        InvalidArgumentException::class
    );

    $http = new GuzzleHttp\Client;

    $response = $http-&gt;post(&#039;http://your-app.com/oauth/token&#039;, [
        &#039;form_params&#039; =&gt; [
            &#039;grant_type&#039; =&gt; &#039;authorization_code&#039;,
            &#039;client_id&#039; =&gt; &#039;client-id&#039;,
            &#039;client_secret&#039; =&gt; &#039;client-secret&#039;,
            &#039;redirect_uri&#039; =&gt; &#039;http://example.com/callback&#039;,
            &#039;code&#039; =&gt; $request-&gt;code,
        ],
    ]);

    return json_decode((string) $response-&gt;getBody(), true);
});
```

###### **刷新令牌**

```php
$http = new GuzzleHttp\Client;

$response = $http-&gt;post(&#039;http://your-app.com/oauth/token&#039;, [
    &#039;form_params&#039; =&gt; [
        &#039;grant_type&#039; =&gt; &#039;refresh_token&#039;,
        &#039;refresh_token&#039; =&gt; &#039;the-refresh-token&#039;,
        &#039;client_id&#039; =&gt; &#039;client-id&#039;,
        &#039;client_secret&#039; =&gt; &#039;client-secret&#039;,
        &#039;scope&#039; =&gt; &#039;&#039;,
    ],
]);

return json_decode((string) $response-&gt;getBody(), true);
```

#### **密码模式**

```php
php artisan passport:client --password
```

```php
$http = new GuzzleHttp\Client;

$response = $http-&gt;post(&#039;http://your-app.com/oauth/token&#039;, [
    &#039;form_params&#039; =&gt; [
        &#039;grant_type&#039; =&gt; &#039;password&#039;,
        &#039;client_id&#039; =&gt; &#039;client-id&#039;,
        &#039;client_secret&#039; =&gt; &#039;client-secret&#039;,
        &#039;username&#039; =&gt; &#039;taylor@laravel.com&#039;,
        &#039;password&#039; =&gt; &#039;my-password&#039;,
        &#039;scope&#039; =&gt; &#039;&#039;, // &#039;*&#039;是所有范围，应该只在密码模式和客户端模式时候使用
    ],
]);

return json_decode((string) $response-&gt;getBody(), true);
```

###### **自定义密码验证和 username 字段**

```php
public function validateForPassportPasswordGrant($password)
{
    return Hash::check($password, $this-&gt;password);
}

public function findForPassport($username)
{
    return $this-&gt;where(&#039;username&#039;, $username)-&gt;first();
}
```

#### **隐式模式**

```php
Passport::enableImplicitGrant();
```

```php
Route::get(&#039;/redirect&#039;, function (Request $request) {
    $request-&gt;session()-&gt;put(&#039;state&#039;, $state = Str::random(40));

    $query = http_build_query([
        &#039;client_id&#039; =&gt; &#039;client-id&#039;,
        &#039;redirect_uri&#039; =&gt; &#039;http://example.com/callback&#039;,
        &#039;response_type&#039; =&gt; &#039;token&#039;,
        &#039;scope&#039; =&gt; &#039;&#039;,
        &#039;state&#039; =&gt; $state,
    ]);

    return redirect(&#039;http://your-app.com/oauth/authorize?&#039;.$query);
});
```

#### **客户端模式**

```php
php artisan passport:client --client
```

```php
use Laravel\Passport\Http\Middleware\CheckClientCredentials;

protected $routeMiddleware = [
    &#039;client&#039; =&gt; CheckClientCredentials::class,
];
```

```php
Route::get(&#039;/orders&#039;, function (Request $request) {
    ...
})-&gt;middleware(&#039;client&#039;);
```

```php
Route::get(&#039;/orders&#039;, function (Request $request) {
    ...
})-&gt;middleware(&#039;client:check-status,your-scope&#039;);
```

```php
$guzzle = new GuzzleHttp\Client;

$response = $guzzle-&gt;post(&#039;http://your-app.com/oauth/token&#039;, [
    &#039;form_params&#039; =&gt; [
        &#039;grant_type&#039; =&gt; &#039;client_credentials&#039;,
        &#039;client_id&#039; =&gt; &#039;client-id&#039;,
        &#039;client_secret&#039; =&gt; &#039;client-secret&#039;,
        &#039;scope&#039; =&gt; &#039;your-scope&#039;,
    ],
]);

return json_decode((string) $response-&gt;getBody(), true)[&#039;access_token&#039;];
```

#### **使用 access_token**

```php
$response = $client-&gt;request(&#039;GET&#039;, &#039;/api/user&#039;, [
    &#039;headers&#039; =&gt; [
        &#039;Accept&#039; =&gt; &#039;application/json&#039;,
        &#039;Authorization&#039; =&gt; &#039;Bearer &#039;.$accessToken,
    ],
]);
```

#### **玩转 scope**

```php
# AuthServiceProvider
use Laravel\Passport\Passport;

Passport::tokensCan([
    &#039;place-orders&#039; =&gt; &#039;Place orders&#039;,
    &#039;check-status&#039; =&gt; &#039;Check order status&#039;,
]);

Passport::setDefaultScope([
    &#039;check-status&#039;,
    &#039;place-orders&#039;,
]);
```

```php
Route::get(&#039;/redirect&#039;, function () {
    $query = http_build_query([
        &#039;client_id&#039; =&gt; &#039;client-id&#039;,
        &#039;redirect_uri&#039; =&gt; &#039;http://example.com/callback&#039;,
        &#039;response_type&#039; =&gt; &#039;code&#039;,
        &#039;scope&#039; =&gt; &#039;place-orders check-status&#039;, // 传递 scope 格式
    ]);

    return redirect(&#039;http://your-app.com/oauth/authorize?&#039;.$query);
});
```

###### **检验 scope**

```php
# app/Http/Kernel.php 中 $routeMiddleware
&#039;scopes&#039; =&gt; \Laravel\Passport\Http\Middleware\CheckScopes::class,
&#039;scope&#039; =&gt; \Laravel\Passport\Http\Middleware\CheckForAnyScope::class,
```

```php
Route::get(&#039;/orders&#039;, function () {
    // Access token has both &quot;check-status&quot; and &quot;place-orders&quot; scopes...
})-&gt;middleware(&#039;scopes:check-status,place-orders&#039;);
```

```php
Route::get(&#039;/orders&#039;, function () {
    // Access token has either &quot;check-status&quot; or &quot;place-orders&quot; scope...
})-&gt;middleware(&#039;scope:check-status,place-orders&#039;);
```

就算含有访问令牌验证的请求已经通过应用程序的验证，你仍然可以使用当前授权 `User` 实例上的 `tokenCan` 方法来验证令牌是否拥有指定的作用域 

```php
use Illuminate\Http\Request;

Route::get(&#039;/orders&#039;, function (Request $request) {
    if ($request-&gt;user()-&gt;tokenCan(&#039;place-orders&#039;)) {
        //
    }
});
```

`scopeIds` 方法将返回所有已定义 ID / 名称的数组：

```php
Laravel\Passport\Passport::scopeIds();
```

`scopes` 方法将返回一个包含所有已定义作用域数组的 `Laravel\Passport\Scope` 实例：

```php
Laravel\Passport\Passport::scopes();
```

`scopesFor` 方法将返回与给定 ID / 名称匹配的 `Laravel\Passport\Scope` 实例数组：

```php
Laravel\Passport\Passport::scopesFor([&#039;place-orders&#039;, &#039;check-status&#039;]);
```

你可以使用 `hasScope` 方法确定是否已定义给定作用域：

```php
Laravel\Passport\Passport::hasScope(&#039;place-orders&#039;);
```

#### **事件**

```php
protected $listen = [
    &#039;Laravel\Passport\Events\AccessTokenCreated&#039; =&gt; [
        &#039;App\Listeners\RevokeOldTokens&#039;,
    ],

    &#039;Laravel\Passport\Events\RefreshTokenCreated&#039; =&gt; [
        &#039;App\Listeners\PruneOldTokens&#039;,
    ],
];
```

#### **javascript 中使用 api**

```php
&#039;web&#039; =&gt; [
    // Other middleware...
    \Laravel\Passport\Http\Middleware\CreateFreshApiToken::class,
],
// 注意：你应该确保在您的中间件堆栈中 CreateFreshApiToken 中间件之前列出了 EncryptCookies 中间件。
```

```php
axios.get(&#039;/api/user&#039;)
    .then(response =&gt; {
        console.log(response.data);
    });
```

###### **自定义 Cookie 名称**

```php
public function boot()
{
    $this-&gt;registerPolicies();

    Passport::routes();

    Passport::cookie(&#039;custom_name&#039;);
}
```



#### **测试**

###### **actingAs 方法可以指定当前已认证用户及其作用域 。**

```php
use App\User;
use Laravel\Passport\Passport;

public function testServerCreation()
{
    Passport::actingAs(
        factory(User::class)-&gt;create(),
        [&#039;create-servers&#039;]
    );

    $response = $this-&gt;post(&#039;/api/create-server&#039;);

    $response-&gt;assertStatus(201);
}
```

###### **actingAsClient 方法可以指定当前已认证客户端及其作用域 。**


```php
use Laravel\Passport\Client;
use Laravel\Passport\Passport;

public function testGetOrders()
{
    Passport::actingAsClient(
        factory(Client::class)-&gt;create(),
        [&#039;check-status&#039;]
    );

    $response = $this-&gt;get(&#039;/api/orders&#039;);

    $response-&gt;assertStatus(200);
}
```

