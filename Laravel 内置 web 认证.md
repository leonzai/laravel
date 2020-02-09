> 配套视频地址：[https://www.bilibili.com/video/av74879198/](https://www.bilibili.com/video/av74879198/ "https://www.bilibili.com/video/av74879198/")

---

#### **原理**

1. 注册：用户注册成功后。在服务器端生成 session 文件。给用户传递 session （文件名）。
2. 登陆：用户使用账号密码登陆成功。在服务器端生成 session 文件。给用户传递 session （文件名）。
3. 认证：将用户传来的 session 作为文件名去查找文件，找到了就认证成功，否则失败。

#### **准备**

```php
composer create-project --prefer-dist laravel/laravel laravel6
下载 node   https://nodejs.org/en/
composer require laravel/ui
php artisan ui vue --auth
npm install cnpm -g --registry=https://registry.npm.taobao.org
cnpm install
cnpm run prod
php artisan migrate
访问 http://your-app.dev/register

如果不需要注册，可以路由中指定下，Auth::routes([&#039;register&#039; =&gt; false]);。
```
#### **使用**

###### **修改跳转地址**

```php
// LoginController,  RegisterController, ResetPasswordController, ConfirmPasswordController and  VerificationController
protected $redirectTo = &#039;/&#039;;
```
```php
# 方法的优先级高于属性定义
protected function redirectTo()
{
	// 可以写一些逻辑
    return &#039;/path&#039;;
    // return route(&#039;login&#039;);
}
```

###### **认证字段修改**

```php
public function username(){
    return &#039;name&#039;;  // 默认 email
}

// $request-&gt;validate([
//     $this-&gt;username() =&gt; &#039;required|string&#039;,
//     &#039;password&#039; =&gt; &#039;required|string&#039;,
// ]);
```

###### **获取登陆后的信息**

```php
$user = Auth::user();
$id = Auth::id();
if (Auth::check()) // 最好使用中间件！

$request-&gt;user()   // use \Illuminate\Http\Request;
```

###### **添加认证条件**

```php
Route::get(&#039;profile&#039;, function () {
    // Only authenticated users may enter...
})-&gt;middleware(&#039;auth&#039;);

public function __construct()
{
    $this-&gt;middleware(&#039;auth&#039;);
}

```

```php
Route::get(&#039;/settings/security&#039;, function () {
    // Users must confirm their password before continuing...
})-&gt;middleware(&#039;password.confirm&#039;);
```

> 如果登录失败次数过多，会禁止登录一段时间。默认五次。禁止登陆一分钟。
> 判断的标准是 username 方法返回值和 ip 。

###### **登出**

`Auth::logout();`

###### **过期时间**

```php
// 默认过期时间是 env(&#039;SESSION_LIFETIME&#039;, 120);    120 分钟从最后一次访问服务器开始算。
// &#039;expire_on_close&#039; =&gt; false                     如果是 true，关闭浏览器就过期
```

###### **手动认证用户**

```php
# 当你不喜欢自带的控制器去认证用户，你可以移除这些控制器，
# 引入 Auth facade，利用 attempt 手动认证
class LoginController extends Controller
{
    public function authenticate(Request $request)
    {
        $credentials = $request-&gt;only(&#039;email&#039;, &#039;password&#039;);

        if (Auth::attempt($credentials)) {
            // Authentication passed...
            return redirect(&#039;/some/url&#039;);
        }
    }
}

// Route::post(&#039;/authenticate&#039;, &#039;Auth\LoginController@authenticate&#039;)-&gt;name(&#039;authenticate&#039;);

if (Auth::attempt([&#039;email&#039; =&gt; $email, &#039;password&#039; =&gt; $password, &#039;active&#039; =&gt; 1])) {
    // 字段 active 必须是 1
}
```

###### **记住用户 （无限期）**

```php
# $remember 是个 bool 值
if (Auth::attempt([&#039;email&#039; =&gt; $email, &#039;password&#039; =&gt; $password], $remember)) {
    // The user is being remembered... 内置的 LoginController 已经实现 remember
}
```

```php
Auth::login($user);
Auth::login($user, true);  // 记住用户 （无限期）
Auth::loginUsingId(1);
Auth::loginUsingId(1, true);
```

```php
Auth::once($credentials); // 临时认证，无状态的。
```

###### **无登录页面, 利用弹窗请求认证用户**

```php
Route::get(&#039;profile&#039;, function(){
    // ...
})-&gt;middleware(&#039;auth.basic&#039;);
```

###### **单设备登录**

```php
// 取消登陆在别的设备上的认证
// 取消注释：\Illuminate\Session\Middleware\AuthenticateSession::class,
Auth::logoutOtherDevices($password);
```