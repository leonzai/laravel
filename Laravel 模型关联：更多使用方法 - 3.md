> 配套视频地址：[https://www.bilibili.com/video/av73028135?p=11](https://www.bilibili.com/video/av73028135?p=11 "https://www.bilibili.com/video/av73028135?p=11")

---

## **查询多态模型**

#### **morphTo**

```php
use Illuminate\Database\Eloquent\Builder;

// 查询与帖子或视频相关并且标题以 foo 开头的评论...
$comments = App\Comment::whereHasMorph(
    'commentable',
    ['App\Post', 'App\Video'],
    function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    }
)->get();

// 查询与帖子相关的评论，标题不是以 foo 开头的...
$comments = App\Comment::whereDoesntHaveMorph(
    'commentable',
    'App\Article',
    function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    }
)->get();
/**
SELECT *
FROM `comments`
WHERE `commentable_type` = 'App\Article'
	AND NOT EXISTS (
		SELECT *
		FROM `articles`
		WHERE `comments`.`commentable_id` = `articles`.`id`
			AND `body` LIKE 'foo%'
	)
*/

// $type 参数根据相关模型添加不同的约束：
$comments = App\Comment::whereHasMorph(
    'commentable',
    ['App\Post', 'App\Video'],
    function (Builder $query, $type) {
        $query->where('title', 'like', 'foo%');

        if ($type === 'App\Post') {
            $query->orWhere('content', 'like', 'foo%');
        }
    }
)->get();

// 所有多态类型
$comments = App\Comment::whereHasMorph('commentable', '*', function (Builder $query) {
    $query->where('title', 'like', 'foo%');
})->get();
```

#### **模型计数**

```php
$posts = App\Post::withCount('comments')->get();
// select `posts`.*, (select count(*) from `comments` where `posts`.`id` = `comments`.`post_id`) as `comments_count` from `posts`

foreach ($posts as $post) {
    echo $post->comments_count;
}
```

```php
$posts = App\Post::withCount(['votes', 'comments' => function (Builder $query) {
    $query->where('content', 'like', 'foo%');
}])->get();

echo $posts[0]->votes_count;
echo $posts[0]->comments_count;
```

```php
$posts = App\Post::withCount([
    'comments',
    'comments as pending_comments_count' => function (Builder $query) {
        $query->where('approved', false);
    },
])->get();

echo $posts[0]->comments_count;

echo $posts[0]->pending_comments_count;
```

```php
App\Post::select(['title', 'body'])->withCount('comments')->get();
```

```php
$post = App\Post::first();
$post->loadCount('comments');
// select `id`, (select count(*) from `comments` where `posts`.`id` = `comments`.`post_id`) as `comments_count` from `posts` where `posts`.`id` in (1)

$post->loadCount(['comments' => function ($query) {
    $query->where('rating', 5);
}])
```
