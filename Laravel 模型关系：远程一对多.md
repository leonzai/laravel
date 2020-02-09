> 配套视频地址：[https://www.bilibili.com/video/av73028135?p=5](https://www.bilibili.com/video/av73028135?p=5 "https://www.bilibili.com/video/av73028135?p=5")

---

简单的说：**Thread 模型可以通过 Author 模型访问多个的 Book 模型**。

#### **表**

```php
threads: id, title
authors: id, name, thread_id
books: id, name, author_id
```

#### **模型**

```php
# App\Thread
public function authorBook()
{
    return $this-&gt;hasManyThrough(&#039;App\Book&#039;, &#039;App\Author&#039;);
}
```

```php
# App\Thread
public function authorBooks()
{
    return $this-&gt;hasOneThrough(
        &#039;App\Book&#039;,
        &#039;App\Author&#039;,
        &#039;thread_id&#039;, // 作者表外键
        &#039;author_id&#039;, // 书表外键
        &#039;id&#039;,        // 帖子本地键
        &#039;id&#039;,        // 作者本地键
    );
}
```

#### **使用**

```php
$books = \App\Thread::find(2)-&gt;authorBooks;
$threadsWithBooks = \App\Thread::with(&#039;authorBooks&#039;)-&gt;get();
```

