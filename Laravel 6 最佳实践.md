> 翻译来源：https://github.com/alexeymezenin/laravel-best-practices

#### **单一职责原则**

**不要这样做：**

```php
public function getFullNameAttribute()
{
    if (auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified()) {
        return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
    } else {
        return $this->first_name[0] . '. ' . $this->last_name;
    }
}
```

**这样做比较好：**

```php
public function getFullNameAttribute()
{
    return $this->isVerifiedClient() ? $this->getFullNameLong() : $this->getFullNameShort();
}

public function isVerifiedClient()
{
    return auth()->user() && auth()->user()->hasRole('client') && auth()->user()->isVerified();
}

public function getFullNameLong()
{
    return 'Mr. ' . $this->first_name . ' ' . $this->middle_name . ' ' . $this->last_name;
}

public function getFullNameShort()
{
    return $this->first_name[0] . '. ' . $this->last_name;
}
```



#### **数据库相关的逻辑代码写到模型或者 Repository 里面**

**不要这样做：**

```php
public function index()
{
    $clients = Client::verified()
        ->with(['orders' => function ($q) {
            $q->where('created_at', '>', Carbon::today()->subWeek());
        }])
        ->get();

    return view('index', ['clients' => $clients]);
}
```

**这样做比较好：**

```php
public function index()
{
    return view('index', ['clients' => $this->client->getWithNewOrders()]);
}

class Client extends Model
{
    public function getWithNewOrders()
    {
        return $this->verified()
            ->with(['orders' => function ($q) {
                $q->where('created_at', '>', Carbon::today()->subWeek());
            }])
            ->get();
    }
}
```



#### **将验证逻辑从控制器转移到 Request 类里面**

**不要这样做：**

```php
public function store(Request $request)
{
    $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
        'publish_at' => 'nullable|date',
    ]);

    // ....
}
```

**这样做比较好：**

```php
public function store(PostRequest $request)
{    
    // ....
}

class PostRequest extends Request
{
    public function rules()
    {
        return [
            'title' => 'required|unique:posts|max:255',
            'body' => 'required',
            'publish_at' => 'nullable|date',
        ];
    }
}
```



#### **业务逻辑应当写在 service 类里面**

**不要这样做：**

```php
public function store(Request $request)
{
    if ($request->hasFile('image')) {
        $request->file('image')->move(public_path('images') . 'temp');
    }
    
    // ....
}
```

**这样做比较好：**

```php
public function store(Request $request)
{
    $this->articleService->handleUploadedImage($request->file('image'));

    // ....
}

class ArticleService
{
    public function handleUploadedImage($image)
    {
        if (!is_null($image)) {
            $image->move(public_path('images') . 'temp');
        }
    }
}
```



#### **不要写重复代码**

**不要这样做：**

```php
public function getActive()
{
    return $this->where('verified', 1)->whereNotNull('deleted_at')->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->where('verified', 1)->whereNotNull('deleted_at');
        })->get();
}
```

**这样做比较好：**

```php
public function scopeActive($q)
{
    return $q->where('verified', 1)->whereNotNull('deleted_at');
}

public function getActive()
{
    return $this->active()->get();
}

public function getArticles()
{
    return $this->whereHas('user', function ($q) {
            $q->active();
        })->get();
}
```



#### **尽量选择用模型而不是 Query Builder 和原生 SQL 查询，尽量用集合而不是数组**

**不要这样做：**

```sql
SELECT *
FROM `articles`
WHERE EXISTS (SELECT *
              FROM `users`
              WHERE `articles`.`user_id` = `users`.`id`
              AND EXISTS (SELECT *
                          FROM `profiles`
                          WHERE `profiles`.`user_id` = `users`.`id`) 
              AND `users`.`deleted_at` IS NULL)
AND `verified` = '1'
AND `active` = '1'
ORDER BY `created_at` DESC
```

**这样做比较好：**

```php
Article::has('user.profile')->verified()->latest()->get();
```



#### **批量赋值**

**不要这样做：**

```php
$article = new Article;
$article->title = $request->title;
$article->content = $request->content;
$article->verified = $request->verified;

$article->category_id = $category->id;
$article->save();
```

**这样做比较好：**

```php
$category->article()->create($request->validated());
```



#### **避免 N+1 问题，使用延迟加载**

**不要这样做：**

```php
@foreach (User::all() as $user)
    {{ $user->profile->name }}
@endforeach
```

**这样做比较好：**

```php
$users = User::with('profile')->get();

...

@foreach ($users as $user)
    {{ $user->profile->name }}
@endforeach
```



#### **变量和方法命名要见名知意**

**不要这样做：**

```php
if (count((array) $builder->getQuery()->joins) > 0)
```

**这样做比较好：**

```php
// 判断是否有人加入
if (count((array) $builder->getQuery()->joins) > 0)
```

**这样做更好：**

```php
if ($this->hasJoins())
```



#### **模板里不要写 js，css 代码，前端任何代码不要写在 php 类文件里面**

**不要这样做：**

```php
let article = `{{ json_encode($article) }}`;
```

**这样做比较好：**

```php
<input id="article" type="hidden" value='@json($article)'>

或者

<button class="js-fav-article" data-article='@json($article)'>{{ $article->name }}<button>
```

js 文件：

```javascript
let article = $('#article').val();
```

**最好的办法是用特定的 php 到 js 包去传输数据。**


#### **使用配置、语言文件、常量代替代码中的文本**

**不要这样做：**

```php
public function isNormal()
{
    return $article->type === 'normal';
}

return back()->with('message', 'Your article has been added!');
```

**这样做比较好：**

```php
public function isNormal()
{
    return $article->type === Article::TYPE_NORMAL;
}

return back()->with('message', __('app.article_added'));
```



#### **使用标准 Laravel 工具，尽量不要使用第三方的**

任务 | 标准工具 | 第三方工具
------------ | ------------- | -------------
权限 | Policies | Entrust, Sentinel 或者其他扩展包
资源编译工具| Laravel Mix | Grunt, Gulp, 或者其他第三方包
开发环境| Homestead | Docker
部署 | Laravel Forge | Deployer 或者其他解决方案
自动化测试 | PHPUnit, Mockery | Phpspec
页面预览测试 | Laravel Dusk | Codeception
数据库操作 | Eloquent | SQL, Doctrine
模板 | Blade | Twig
数据操作 | Laravel 集合 | 数组
表单验证| Request | 他第三方包,甚至在控制器中做验证
认证 | 内置的 | 第三方包或者你自己解决
API 认证 | Laravel Passport | 第三方的 JWT 或者 OAuth 扩展包
创建 API | 内置的 | Dingo API 或者类似的扩展包
创建数据库结构 | Migrations | 直接用 DB 语句创建
本地化 | 内置的 |第三方包
实时消息队列 | Laravel Echo, Pusher | 使用第三方包或者直接使用 WebSockets
创建测试数据| Seeder，模型工厂，Faker | 手动创建测试数据
任务调度| Laravel Task Scheduler | 脚本和第三方包
数据库 | MySQL, PostgreSQL, SQLite, SQL Server | MongoDB



#### **遵循 Laravel 命名约定**

遵循 [PSR 2](http://www.php-fig.org/psr/psr-2/).
 
-- | 怎么写 | 好的 | 坏的
------------ | ------------- | ------------- | -------------
控制器 | 单数 | ArticleController | ~~ArticlesController~~
路由 | 复数 | articles/1 | ~~article/1~~
命名路由 | 带点符号的蛇形命名 | users.show_active | ~~users.show-active, show-active-users~~
模型|	单数 | User | ~~Users~~
`hasOne` 或 `belongsTo` | 单数 | articleComment | ~~articleComments, article_comment~~
其他模型关系 | 复数 | articleComments | ~~articleComment, article_comments~~
表名 | 复数 | article_comments | ~~article_comment, articleComments~~
中间表 | 按字母顺序排列模型、单数、单词之间加下划线 | article_user | ~~user_article, articles_users~~
字段 | 使用蛇形并且不要带模型名 | meta_title | ~~MetaTitle; article_meta_title~~
模型属性 | 蛇形命名 | $model->created_at | ~~$model->createdAt~~
外键 | 单数小写模型名称加上 ` _id` | article_id | ~~ArticleId, id_article, articles_id~~
主键 | - | id | ~~custom_id~~
迁移表名 | - | 2017_01_01_000000_create_articles_table | ~~2017_01_01_000000_articles~~
方法 | 驼峰命名 | getAll | ~~get_all~~
资源控制器方法名 | [table](https://laravel.com/docs/master/controllers#resource-controllers) | store | ~~saveArticle~~
测试类方法名 | 驼峰命名 | testGuestCannotSeeArticle | ~~test_guest_cannot_see_article~~
变量 | 驼峰命名 | $articlesWithAuthor | ~~$articles_with_author~~
集合 | 描述性的, 复数的 | $activeUsers = User::active()->get() | ~~$active, $data~~
对象 | 描述性的, 单数的 | $activeUser = User::active()->first() | ~~$users, $obj~~
配置和语言文件索引 | 蛇形命名 | articles_enabled | ~~ArticlesEnabled; articles-enabled~~
视图 | 短横杠命名 | show-filtered.blade.php | ~~showFiltered.blade.php, show_filtered.blade.php~~
配置 | 蛇形命名 | google_calendar.php | ~~googleCalendar.php, google-calendar.php~~
Contract (接口) | 形容词或名词 | Authenticatable | ~~AuthenticationInterface, IAuthentication~~
Trait | 形容词 | Notifiable | ~~NotificationTrait~~



#### **写更简短、更易读的代码**

**不要这样做：**

```php
$request->session()->get('cart');
$request->input('name');
```

**这样做比较好：**

```php
session('cart');
$request->name;
```

更多例子

普通写法 | 更简短、更易读的代码
------------ | -------------
`Session::get('cart')` | `session('cart')`
`$request->session()->get('cart')` | `session('cart')`
`Session::put('cart', $data)` | `session(['cart' => $data])`
`$request->input('name'), Request::get('name')` | `$request->name, request('name')`
`return Redirect::back()` | `return back()`
`is_null($object->relation) ? null : $object->relation->id` | `optional($object->relation)->id`
`return view('index')->with('title', $title)->with('client', $client)` | `return view('index', compact('title', 'client'))`
`$request->has('value') ? $request->value : 'default';` | `$request->get('value', 'default')`
`Carbon::now(), Carbon::today()` | `now(), today()`
`App::make('Class')` | `app('Class')`
`->where('column', '=', 1)` | `->where('column', 1)`
`->orderBy('created_at', 'desc')` | `->latest()`
`->orderBy('age', 'desc')` | `->latest('age')`
`->orderBy('created_at', 'asc')` | `->oldest()`
`->select('id', 'name')->get()` | `->get(['id', 'name'])`
`->first()->name` | `->value('name')`



#### **使用 IoC 容器或者 Facade 替代 new 创建新类**

**不要这样做：**

```php
$user = new User;
$user->create($request->validated());
```

**这样做比较好：**

```php
public function __construct(User $user)
{
    $this->user = $user;
}

// ....

$this->user->create($request->validated());
```



#### **用 config() 取配置信息，而不是用 env()**

**不要这样做：**

```php
$apiKey = env('API_KEY');
```

**这样做比较好：**

```php
// config/api.php
'key' => env('API_KEY'),

// 使用如下：
$apiKey = config('api.key');
```



#### **使用 accessors 和 mutators 修改日期格式**

**不要这样做：**

```php
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->toDateString() }}
{{ Carbon::createFromFormat('Y-d-m H-i', $object->ordered_at)->format('m-d') }}
```

**这样做比较好：**

```php
// 模型
protected $dates = ['ordered_at', 'created_at', 'updated_at'];
public function getSomeDateAttribute($date)
{
    return $date->format('m-d');
}

// 视图
{{ $object->ordered_at->toDateString() }}
{{ $object->ordered_at->some_date }}
```



#### **其他**

永远不要在路由文件里面写逻辑。

最好不要在模板文件里面写原始 php 代码。
