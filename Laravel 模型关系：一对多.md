> 配套视频地址：[https://www.bilibili.com/video/av73028135?p=3](https://www.bilibili.com/video/av73028135?p=3 "https://www.bilibili.com/video/av73028135?p=3")

---

**一对多：一篇文章可以有多个评论，一个评论只属于一篇文章。**

---

#### **表结构**

```php
# articles: id, title, content
# comments: id, content, article_id
```
#### **模型配置**

```php
// App\Article
public function comments(){
    return $this-&gt;hasMany(&#039;App\Comment&#039;);
}
```
```php
// App\Comment
public function article(){
    return $this-&gt;belongsTo(&#039;App\Article&#039;);
}
```

#### **使用**

###### **增**

```php
$article = App\Article::find(1);

$article-&gt;comments()-&gt;create([&#039;content&#039; =&gt; &#039;上海在哪儿&#039;]);
```

###### **删**

```php
$article = App\Article::find(1);

$article-&gt;comments()-&gt;delete();
```

###### **改**

```php
$comment = App\Comment::find(11);
$article = App\Article::find(1);

$comment-&gt;article()-&gt;associate($article);
$comment-&gt;save();
```

```php
$comment = App\Comment::find(1);

$comment-&gt;article()-&gt;update([&#039;title&#039; =&gt; &#039;广州在哪儿&#039;]);
```

###### **查**

```php
$comments = App\Article::find(1)-&gt;comments;
$comments = App\Article::find(1)-&gt;comments()-&gt;where(&#039;content&#039;, &#039;like&#039;, &#039;%教学%&#039;)-&gt;get();
$article = App\Comment::find(1)-&gt;article;

// 延迟加载
App\Ariticle::with(&#039;comments&#039;)-&gt;find([1,2])
```

#### **定制模型**

###### **指定 comments 表中的外键字段名称**

```php
# articles: id, title, content
# comments: id, content, arti_id

// App\Article
public function comments(){
    return $this-&gt;hasMany(&#039;App\Comment&#039;, &#039;arti_id&#039;);
}
```

###### **指定 comments 表中的外键对应 articles 表中的字段名称**

```php
# articles: id, title, content
# comments: id, content, article_title

// App\Article
public function comments(){
    return $this-&gt;hasMany(&#039;App\Comment&#039;, &#039;article_title&#039;, &#039;title&#039;);
}
```