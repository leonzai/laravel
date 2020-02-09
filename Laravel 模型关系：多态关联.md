> 配套视频地址：[https://www.bilibili.com/video/av73028135?p=6](https://www.bilibili.com/video/av73028135?p=6 "https://www.bilibili.com/video/av73028135?p=6")

---

## **多态一对一**

**项目：运营人员就一个图片，发布一篇博客或者一篇文章。**

#### **表**

```AsciiDoc
articles
    id - integer
    title - string

blogs
    id - integer
    title - string

images
    id - integer
    url - string
    imageable_id - integer
    imageable_type - string
```

#### **模型**

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

#### **使用**

```php
$article = App\Article::find(1);
$image= $article-&gt;image;
```
```php
$articleWithImages = App\Article::with(&#039;image&#039;)-&gt;find(1);
$articlesWithImages = App\Article::with(&#039;image&#039;)-&gt;find([1, 2]);
```

```php
$image = App\Image::find(1);
$imageable = $image-&gt;imageable;
```

---

## **多态一对多**

**文章可以接受多条评论，视频也可以接受多条评论。**

#### **表**

```php
articles
    id - integer
    title - string
    body - text

videos
    id - integer
    title - string
    url - string

comments
    id - integer
    body - text
    commentable_id - integer
    commentable_type - string
```

#### **模型**

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

#### **使用**

```php
$article = App\Article::find(1);

foreach ($article-&gt;comments as $comment) {
    //
}
```

```php
$articleWithComments = App\Article::with(&#039;comments&#039;)-&gt;find(1);

$articlesWithComments = App\Article::with(&#039;comments&#039;)-&gt;find([1, 2]);
```

```php
$comment = App\Comment::find(1);
$commentable = $comment-&gt;commentable;
```

----

## **多态多对多**

**可以给一篇文章打上多个标签。可以给一个视频打上多个标签。**

**一个标签可以贴在多个文章上面。一个标签也可以贴在多个视频上。**

#### **表**

```php
articles
    id - integer
    name - string

videos
    id - integer
    name - string

tags
    id - integer
    name - string

taggables
    tag_id - integer
    taggable_id - integer
    taggable_type - string
```

#### **模型**

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
```

```php
# App\Article
public function tags()
{
    return $this-&gt;morphToMany(&#039;App\Tag&#039;, &#039;taggable&#039;);
}
```

#### **使用**

```php
$video = App\Video::find(1);

foreach ($video-&gt;tags as $tag) {
    //
}
```

```php
$tag = App\Tag::find(1);

foreach ($tag-&gt;videos as $video) {
    //
}
```

```php
$videoWithTags = App\Video::with(&#039;tags&#039;)-&gt;find(1);
$videosWithTags = App\Video::with(&#039;tags&#039;)-&gt;find([1, 2]);
```

```php
$tagWithVideos = App\Tag::with(&#039;videos&#039;)-&gt;find(1);
$tagsWithVideos = App\Tag::with(&#039;videos&#039;)-&gt;find([1, 2]);
```

#### **定制**

```php
# AppServiceProvider
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::morphMap([
    &#039;posts&#039; =&gt; &#039;App\Post&#039;,  // posts 是 *_type 字段所存储的值
    &#039;videos&#039; =&gt; &#039;App\Video&#039;,
]);
```

