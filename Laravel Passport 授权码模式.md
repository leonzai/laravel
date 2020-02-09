> 配套视频地址：[https://www.bilibili.com/video/av74879198?p=7](https://www.bilibili.com/video/av74879198?p=7 "https://www.bilibili.com/video/av74879198?p=7")

---

1. 哔哩哔哩提供一个“微信登陆”的链接，用户点击跳转到微信授权服务器。
2. 用户根据微信授权服务器提示登陆微信并确认授权给哔哩哔哩。
3. 微信授权服务器返回用户代理（浏览器）一个授权码。
4. 用户代理（浏览器）把这个授权码传给哔哩哔哩。
5. 哔哩哔哩凭借授权码向微信授权服务器请求令牌。
6. 微信授权服务器发送令牌给哔哩哔哩。

#### **服务器端（微信）**

###### **配置**

```php
composer create-project --prefer-dist laravel/laravel laravel6
.env 数据库配置
修改数据库默认字符串长度
composer require laravel/passport
Laravel\Passport\HasApiTokens Trait 添加到 App\User 模型中 // 提供一些辅助函数检查已认证用户的令牌和使用范围
```

安装前端必备的东西（脚手架）

```php
下载 node   https://nodejs.org/en/
composer require laravel/ui
php artisan ui vue --auth
npm install cnpm -g --registry=https://registry.npm.taobao.org
cnpm install
cnpm run prod
```

```php
composer require guzzlehttp/guzzle // 伪造 http 请求
```

```php
// config/auth.php

&#039;api&#039; =&gt; [
    &#039;driver&#039; =&gt; &#039;passport&#039;,
    &#039;provider&#039; =&gt; &#039;users&#039;,
    &#039;hash&#039; =&gt; false,
],
```

```php
php artisan migrate // 创建表来存储客户端和 access_token 等
php artisan passport:keys // 加密生成的 access_token
```

```php
// 注册路由 AuthServiceProvider
Passport::routes();
```

```php
Passport::tokensExpireIn(now()-&gt;addDays(15)); // access_token 过期时间
Passport::refreshTokensExpireIn(now()-&gt;addDays(60)); // refresh_token 过期时间
```

###### **创建客户端**

```php
php artisan passport:client
```

#### **第三方应用程序（bilibili）**

###### **准备**

```php
composer create-project --prefer-dist laravel/laravel laravel6
composer require guzzlehttp/guzzle // 伪造 http 请求
```

###### **web.php**

```php
&lt;?php

$clientId = 1;
$clientSecret = &#039;8sGiTDgHb69Y6nTiFImTJO32jm3jB7x2BzMxrhDF&#039;;

// bili 登录页面
Route::view(&#039;/login&#039;, &#039;login&#039;);


// 第三方登陆，重定向
Route::get(&#039;/lishen/login&#039;,
    function (\Illuminate\Http\Request $request) use ($clientId) {
        $request-&gt;session()-&gt;put(&#039;state&#039;, $state = Str::random(40));

        $query = http_build_query([
            &#039;client_id&#039; =&gt; $clientId,
            &#039;redirect_uri&#039; =&gt; &#039;http://bili.com/auth/callback&#039;,
            &#039;response_type&#039; =&gt; &#039;code&#039;,
            &#039;scope&#039; =&gt; &#039;*&#039;,
            &#039;state&#039; =&gt; $state,
        ]);

        return redirect(&#039;http://lishen.com/oauth/authorize?&#039;.$query);
    });



// 回调地址，获取 code，并随后发出获取 token 请求
Route::view(&#039;/auth/callback&#039;, &#039;auth_callback&#039;);

Route::post(&#039;/get/token&#039;, function (\Illuminate\Http\Request $request) use (
    $clientId,
    $clientSecret
) {
    // csrf 攻击处理
    $state = $request-&gt;session()-&gt;pull(&#039;state&#039;);
    throw_unless(
        strlen($state) &gt; 0 &amp;&amp; $state === $request-&gt;params[&#039;state&#039;],
        InvalidArgumentException::class
    );


    $response
        = (new \GuzzleHttp\Client())-&gt;post(&#039;http://lishen.com/oauth/token&#039;, [
        &#039;form_params&#039; =&gt; [
            &#039;grant_type&#039; =&gt; &#039;authorization_code&#039;,
            &#039;client_id&#039; =&gt; $clientId,
            &#039;client_secret&#039; =&gt; $clientSecret,
            &#039;redirect_uri&#039; =&gt; &#039;http://bili.com/auth/callback&#039;,
            &#039;code&#039; =&gt; $request-&gt;params[&#039;code&#039;],
        ],
    ]);

    return json_decode((string)$response-&gt;getBody(), true);
});


// 刷新 token
Route::view(&#039;/refresh/page&#039;, &#039;refresh_page&#039;);

Route::post(&#039;/refresh&#039;, function (\Illuminate\Http\Request $request) use (
    $clientId,
    $clientSecret
) {
    $http = new GuzzleHttp\Client;
    $response = $http-&gt;post(&#039;http://lishen.com/oauth/token&#039;, [
        &#039;form_params&#039; =&gt; [
            &#039;grant_type&#039; =&gt; &#039;refresh_token&#039;,
            &#039;refresh_token&#039; =&gt; $request-&gt;params[&#039;refresh_token&#039;],
            &#039;client_id&#039; =&gt; $clientId,
            &#039;client_secret&#039; =&gt; $clientSecret,
        ],
    ]);

    return json_decode((string)$response-&gt;getBody(), true);
});
```

###### **refresh_page**

```php
&lt;script src=&quot;https://unpkg.com/axios/dist/axios.min.js&quot;&gt;&lt;/script&gt;
&lt;script&gt;
    axios.post(&#039;/refresh&#039;, {
        params: {
            refresh_token: &quot;def502009e634dd59ac4dcd4843be50c3a7a6c76fe0c26a6a948d45b99e393cdf99d1a212a8752d0ce02f4cbc25008972b524336f23b60dfc4198e5413b7e43250126b0d1780afb85443edc1579870e823eedea4313448ffcbe8ca73dc2441e1b1f54d3c0ffc31888e0afeb3b1d4516f6986e540b6a56490dfbfabfe7a88e9fb8539a18cb08f8a2ce10962a3c79e7eed137f137f605cb1ab26254e642750f7f07ebdf17a9ce07a370fabc85e769326cb4fbc9aad402bb69615357766f56e9e26feafac306a7338781317e8baa88e9df9dc0096c92522c8d3cdc1b77cf5273bb0866608575eec5688815d294de22cf8bdf1689cb7e11d6caeb2f3bd80cc57d911b712f79609a45e6e1def42709776c75ca16b56ce6449c25c1660635dfc4a590560db5d2bb52ffcb9be601b8a1ea51c221246815a4f08ed262290cf4fdf0c9c9d357c189f5fa4b9d32c7b9c98a8832666e1ee2eba38b9dc642b02fcc05c38bbdecc&quot;
        }
    })
        .then(function (response) {
            console.log(response.data);
        });
&lt;/script&gt;

```

###### **auth_callback.blade.php**

```php
&lt;script src=&quot;https://unpkg.com/axios/dist/axios.min.js&quot;&gt;&lt;/script&gt;
&lt;script&gt;
    function GetRequest() {
        var url = location.search; //获取url中&quot;?&quot;符后的字串
        var theRequest = {};
        if (url.indexOf(&quot;?&quot;) !== -1) {
            var str = url.substr(1);
            strs = str.split(&quot;&amp;&quot;);
            for (var i = 0; i &lt; strs.length; i++) {
                theRequest[strs[i].split(&quot;=&quot;)[0]] = decodeURI(strs[i].split(&quot;=&quot;)[1]);
            }
        }
        return theRequest;
    }

    //调用
    var Request = GetRequest();

    if (Request[&#039;error&#039;]) {
        // 用户未授权处理
        alert(Request[&#039;error&#039;]);
    }else
    {
        var code = Request[&#039;code&#039;];
        var state = Request[&#039;state&#039;];

        axios.post(&#039;/get/token&#039;, {
            params: {
                code,
                state
            }
        })
            .then(function (response) {
                console.log(response.data);
            });
    }
&lt;/script&gt;
```

