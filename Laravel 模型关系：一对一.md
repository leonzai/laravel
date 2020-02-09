> 配套视频地址：[https://www.bilibili.com/video/av73028135](https://www.bilibili.com/video/av73028135 "https://www.bilibili.com/video/av73028135")

---

>  五个部分
>
> 1. 创建数据表：migration
> 2. 填充数据表：seeder
> 3. 模型配置：定义模型关系
> 4. 基本使用：增删改查
> 5. 可能遇到的问题

---

**一对一：一个人只有一个身份证，一个身份证只属于一个人。**

#### **表结构**

```php
users: id, name
identity_cards: id, city, user_id
// 身份证表的 user_id 是 users 表的外键。
```

#### **创建数据表**

你可以自己手动创建各种数据表。但 Laravel 内置提供了更方便的数据库版本控制器。

```php
php artisan make:migration create_identity_cards_table  
// 生成迁移文件，该文件是用来创建数据表的

php artisan migrate               // 运行迁移文件，即创建数据表
php artisan migrate:rollback      // 回滚迁移操作
```

#### **填充数据表**

```php
php artisan make:factory IdentityCardFactory --model=IdentityCard
```

#### **模型配置**

```php
// App\User
public function identity_card(){
    return $this-&gt;hasOne(&#039;App\IdentityCard&#039;);
}
```

```php
// App\IdentityCard
public function user(){
    return $this-&gt;belongsTo(&#039;App\User&#039;);
}
```

#### **基本使用**

###### **增**

```php
$card = new App\IdentityCard([&#039;city&#039; =&gt; &#039;上海&#039;]);

$user = App\User::create([
    &#039;name&#039; =&gt; request(&#039;name&#039;),
    &#039;email&#039; =&gt; request(&#039;email&#039;),
    &#039;password&#039; =&gt; request(&#039;password&#039;),
]);

$user-&gt;identity_card()-&gt;save($card);
```

```php
$user = App\User::create([
    &#039;name&#039; =&gt; request(&#039;name&#039;),
    &#039;email&#039; =&gt; request(&#039;email&#039;),
    &#039;password&#039; =&gt; request(&#039;password&#039;),
]);

$user-&gt;identity_card()-&gt;create([&#039;city&#039; =&gt; &#039;上海&#039;]);
```

###### **删**

```php
// 删除某个用户的身份证
$user = \App\User::find(1);
$user-&gt;identity_card()-&gt;delete();

// 删除某张身份证的主人
\App\IdentityCard::find(3)-&gt;user()-&gt;delete();
```

###### **改**

```php
$card = \App\IdentityCard::find(6);
$card->user()->dissociate();
$card->save();
// update `identity_cards` set `user_id` = ?, `identity_cards`.`updated_at` = '2019-09-19 13:26:59' where `id` = 6, null
```

```php
$user = App\User::find(2);
$card = App\IdentityCard::find(6);

$card->user()->associate($user);
$card->save();
```

```php
$user = \App\User::find(1);
$user->identity_card()->update(['city' => '广州']);
```

###### **查**

```php
// 查看某个人的身份证
$identityCard = App\User::find(1)->identity_card;
// 根据身份证信息找出主人
$user = App\IdentityCard::find(1)->user;
```

```php
// 有几个身份证，找出它们的主人
// 1. 普通方式（建议使用下面一种）
$cards = App\IdentityCard::find([1, 2]);
foreach ($cards as $card) {
    echo $card->user->name;
}

// 2. 延迟加载
$cards = App\IdentityCard::with('user')->find([1, 2]);
foreach ($cards as $card) {
    echo $card->user->name;
}
```

#### **可能遇到的问题可能遇到的问题**

###### **php artisan migrate 报错 Specified key was too long**

```php
一些 mysql 版本限制键的长度是 767
utf8 255*3 765
utf8mb4 255*4 1020

Schema::defaultStringLength(191);
```

###### **指定 identity_cards 表中的外键字段名称**

```php
# users: id, name
# identity_cards: id, city, web_user_id

// App\User
public function identity_card(){
    return $this->hasOne('App\IdentityCard', 'web_user_id');
}
```

###### **指定 identity_cards 表中的外键对应 users 表中的字段名称**

```php
# users: id, name
# identity_cards: id, city, user_name

// App\User
public function identity_card(){
    return $this->hasOne('App\IdentityCard', 'user_name', 'name');
}
```

