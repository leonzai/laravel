> 配套视频地址：[https://www.bilibili.com/video/av73028135?p=5](https://www.bilibili.com/video/av73028135?p=5 "https://www.bilibili.com/video/av73028135?p=5")

---

**一个帖子属于一个作者，该作者就读一所学校。帖子可通过作者访问作者所在的学校。**

简单的说：Thread 模型可以通过 Author 模型访问唯一的 School 模型。

---

#### **表**

```php
threads: id, title
authors: id, name, thread_id
schools: id, name, author_id
// 一对一指的是：作者对学校
```

#### **模型**

```php
# App\Thread
public function authorSchool()
{
    return $this-&gt;hasOneThrough(&#039;App\School&#039;, &#039;App\Author&#039;);
}
```

```php
# App\Thread
public function authorSchool()
{
    return $this-&gt;hasOneThrough(
        &#039;App\School&#039;,
        &#039;App\Author&#039;,
        &#039;thread_id&#039;, // 作者表外键
        &#039;author_id&#039;, // 学校表外键
        &#039;id&#039;,        // 帖子本地键
        &#039;id&#039;,        // 作者本地键
    );
}
```

#### **使用**

```php
$school = \App\Thread::find(2)-&gt;authorSchool;
$threadsWithSchool = \App\Thread::with(&#039;authorSchool&#039;)-&gt;get();
```

