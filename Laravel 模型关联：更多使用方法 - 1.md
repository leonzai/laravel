> 配套视频地址：[https://www.bilibili.com/video/av73028135?p=9](https://www.bilibili.com/video/av73028135?p=9 "https://www.bilibili.com/video/av73028135?p=9")

---

## **复习模型关联**

###### **一对一: 一个人只有一个身份证，一个身份证只属于一个人。**

```php
// App\User
public function identity_card(){
    return $this-&gt;hasOne(&#039;App\IdentityCard&#039;);
}

// App\IdentityCard
public function user(){
    return $this-&gt;belongsTo(&#039;App\User&#039;);
}
```

###### **一对多：一篇文章可以有多个评论，一个评论只属于一篇文章。**

```php
// App\Article
public function comments(){
    return $this-&gt;hasMany(&#039;App\Comment&#039;);
}

// App\Comment
public function article(){
    return $this-&gt;belongsTo(&#039;App\Article&#039;);
}
```

###### **多对多：一个人可以扮演多个角色，一个角色可以被多个人扮演。**

```php
// App\Role
public function users(){
    return $this-&gt;belongsToMany(&#039;App\User&#039;);
}

// App\User
public function roles(){
    return $this-&gt;belongsToMany(&#039;App\Role&#039;);
}
```

###### **远程一对一：一个帖子属于一个作者，该作者就读一所学校。帖子可通过作者访问作者所在的学校。**

```php
# App\Thread
public function authorSchool()
{
    return $this-&gt;hasOneThrough(&#039;App\School&#039;, &#039;App\Author&#039;);
}
```

###### **远程一对多：一个帖子属于一个作者，该作者写过很多书。帖子可通过作者访问作者写过的所有书籍。**

```php
# App\Thread
public function authorBook()
{
    return $this-&gt;hasManyThrough(&#039;App\Book&#039;, &#039;App\Author&#039;);
}
```

###### **多态一对一：运营人员就一个图片，发布一篇博客或者一篇文章。**

```php
# App\Image
public function imageable()
{
    return $this-&gt;morphTo();
}

# App\Blog
public function image()
{
    return $this-&gt;morphOne(&#039;App\Image&#039;, &#039;imageable&#039;);
}

# App\Article
public function image()
{
    return $this-&gt;morphOne(&#039;App\Image&#039;, &#039;imageable&#039;);
}
```

###### **多态一对多：文章可以接受多条评论，视频也可以接受多条评论。**

```php
# App\Comment
public function commentable()
{
    return $this-&gt;morphTo();
}

# App\Video
public function comments()
{
    return $this-&gt;morphMany(&#039;App\Comment&#039;, &#039;commentable&#039;);
}

# App\Article
public function comments()
{
    return $this-&gt;morphMany(&#039;App\Comment&#039;, &#039;commentable&#039;);
}
```

###### **多态多对多：可以给一篇文章打上多个标签，也可以给一个视频打上多个标签。并且一个标签可以贴在多个文章上面。一个标签也可以贴在多个视频上。**

```php
# App\Tag
public function articles()
{
    return $this-&gt;morphedByMany(&#039;App\Article&#039;, &#039;taggable&#039;);
}

public function videos()
{
    return $this-&gt;morphedByMany(&#039;App\Video&#039;, &#039;taggable&#039;);
}

# App\Article
public function tags()
{
    return $this-&gt;morphToMany(&#039;App\Tag&#039;, &#039;taggable&#039;);
}

# App\Video
public function tags()
{
    return $this-&gt;morphToMany(&#039;App\Tag&#039;, &#039;taggable&#039;);
}
```

| 关系       | 正向           | 反向          |
| ---------- | -------------- | ------------- |
| 一对一     | hasOne         | belongsTo     |
| 一对多     | hasMany        | belongsTo     |
| 多对多     | belongsToMany  | belongsToMany |
| 远程一对一 | hasOneThrough  | --            |
| 远程一对多 | hasManyThrough | --            |
| 多态一对一 | morphTo        | morphOne      |
| 多态一对多 | morphTo        | morphMany     |
| 多态多对多 | morphedByMany  | morphToMany   |

---

## **一对一、一对多**

#### **增**

```php
$comment = new App\Comment(['message' => 'A new comment.']);
$post = App\Post::find(1);
$post->comments()->save($comment);
// insert into `comments` (`message`, `post_id`, `updated_at`, `created_at`) values ('A new comment.', 1, '2019-09-19 03:59:48', '2019-09-19 03:59:48')



$post->comments()->saveMany([
    new App\Comment(['message' => 'A new comment.']),
    new App\Comment(['message' => 'Another comment.']),
]);



$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);
// insert into `comments` (`message`, `post_id`, `updated_at`, `created_at`) values (A new comment., 1, 2019-09-19 04:07:43, 2019-09-19 04:07:43)
// 还可以使用 firstOrNew, firstOrCreate 和 updateOrCreate



$post->comments()->createMany([
    [
        'message' => 'A new comment.',
    ],
    [
        'message' => 'Another new comment.',
    ],
]);
```

#### **改**

```php
$post = App\Post::find(1);

$post->comments[0]->message = 'Message';
$post->comments[0]->name = 'Author Name';
$post->push();
// update `comments` set `message` = Message, `name` = Author Name, `comments`.`updated_at` = 2019-09-19 04:06:09 where `id` = 1
```



#### **belongsTo**

建立模型关系

```php
$user = App\User::find(10);
$comment =  App\Comment::find(1);
$comment->user()->associate($user);
$comment->save();
```

删除模型关系

```php
$comment->account()->dissociate();
$comment->save();
```

#### **默认模型**

```php
# belongsTo, hasOne, hasOneThrough, 和 morphOne
public function user()
{
    return $this->belongsTo('App\User')->withDefault();
}


return $this->belongsTo('App\User')->withDefault([
    'name' => 'Guest Author',
]);


return $this->belongsTo('App\User')->withDefault(function ($user, $post) {
    $user->name = 'Guest Author';
});
```



```php
belongsTo 或者 belongsToMany
当 Comment 模型被更新时，您要自动「触发」父级 Article 模型的 updated_at 时间戳的更新
    
protected $touches = ['article'];
```