> 配套视频地址：[https://www.bilibili.com/video/av73028135?p=10](https://www.bilibili.com/video/av73028135?p=10 "https://www.bilibili.com/video/av73028135?p=10")

---

## **has、orHas**

```php
// 获取至少存在一条评论的所有文章...
$posts = App\Post::has('comments')->get();

// 获取评论超过三条的文章...
$posts = App\Post::has('comments', '>=', 3)->get();
// select * from `posts` where (select count(*) from `comments` where `posts`.`id` = `comments`.`post_id`) > 3

// 获取拥有至少一条带有投票评论的文章...
$articles = App\Article::has('comments.votes')->get();
/**
SELECT *
FROM `articles`
WHERE EXISTS (
	SELECT *
	FROM `comments`
	WHERE `articles`.`id` = `comments`.`article_id`
		AND EXISTS (
			SELECT *
			FROM `votes`
			WHERE `comments`.`id` = `votes`.`comment_id`
		)
)
*/
```

## **whereHas、orWhereHas**

```php
// 获取至少带有一条内容以 foo 开头的评论的文章...
$posts = App\Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
})->get();

// 获取至少带有十条评论内容包含 foo% 关键词的文章...
$posts = App\Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
}, '>=', 10)->get();
```

## **doesntHave、orDoesntHave**

```php
// 获取不存在任何评论的文章
$posts = App\Post::doesntHave('comments')->get();


// 获取评论内容不包含foo开头的文章
$posts = App\Post::whereDoesntHave('comments', function (Builder $query) {
    $query->where('content', 'like', 'foo%');
})->get();

// 获取没被禁用的作者评论的文章
$posts = App\Post::whereDoesntHave('comments.author', function (Builder $query) {
    $query->where('banned', 1);
})->get();
/**
SELECT *
FROM `posts`
WHERE NOT EXISTS (
	SELECT *
	FROM `comments`
	WHERE (`posts`.`id` = `comments`.`commentable_id`
		AND `comments`.`commentable_type` = 'App\Post'
		AND EXISTS (
			SELECT *
			FROM `authors`
			WHERE `comments`.`id` = `authors`.`comment_id`
				AND `banned` = 1
		))
)
*/
```

## **延迟加载**

```php
$books = App\Book::with(['author', 'publisher'])->get();
```

```php
App\Book::with('author:id,name')->get();
```

```php
protected $with = ['author'];

App\Book::without('author')->get();
```

```php
$users = App\User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%first%');
}])->get();
```

```php
$users = App\User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();
```

## **懒延迟加载**

```php
$books = App\Book::all();

if ($someCondition) {
    $books->load('author', 'publisher');
}
```

```php
$books->load(['author' => function ($query) {
    $query->orderBy('published_date', 'asc');
}]);
```

## **嵌套延迟加载多态、嵌套懒延迟加载多态**

```php
// Event, Photo 和 Post 模型可以创建 ActivityFeed 模型。 另外，我们假设 Event 模型属于 Calendar 模型，Photo 模型与 Tag 模型相关联，Post 模型属于 Author 模型。

$activities = ActivityFeed::with('parentable')
    ->get()
    ->loadMorph('parentable', [
        Event::class => ['calendar'],
        Photo::class => ['tags'],
        Post::class => ['author'],
    ]);

$activities = ActivityFeed::with(['parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWith([
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);
    }])->get();
```

