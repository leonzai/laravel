> 配套视频地址：[https://www.bilibili.com/video/av73028135?p=4](https://www.bilibili.com/video/av73028135?p=4 "https://www.bilibili.com/video/av73028135?p=4")

---

**多对多：一个人可以扮演多个角色，一个角色可以被多个人扮演。**

---

#### **数据结构**

```php
# users: id, name
# roles: id, title
# role_user: id, role_id, user_id
```

#### **模型配置**
```php
// App\Role
public function users(){
    return $this-&gt;belongsToMany(&#039;App\User&#039;);
}
```

```php
// App\User
public function roles(){
    return $this-&gt;belongsToMany(&#039;App\Role&#039;);
}
```

#### **增删改查**

###### **增**

```php
$user = App\User::find(1);
$user-&gt;roles()-&gt;attach($roleId); 
// $roleId = 1;
// $roleId = [1, 2];
// $roleId = Role::find(1);

$user-&gt;roles()-&gt;attach($roleId, [&#039;expires&#039; =&gt; $expires]);

$user-&gt;roles()-&gt;attach([
    1 =&gt; [&#039;expires&#039; =&gt; $expires],
    2 =&gt; [&#039;expires&#039; =&gt; $expires],
]);
```

###### **删**

```php
$user-&gt;roles()-&gt;detach($roleId);

$user-&gt;roles()-&gt;detach();
```

###### **改**

```php
$user-&gt;roles()-&gt;sync([1, 2, 3]);
$user-&gt;roles()-&gt;sync([1 =&gt; [&#039;expires&#039; =&gt; true], 2, 3]);

$user-&gt;roles()-&gt;syncWithoutDetaching([1, 2, 3]);

$user-&gt;roles()-&gt;toggle([1, 2, 3]);
```

```php
// 更新中间表记录
$user = App\User::find(1);

$user-&gt;roles()-&gt;updateExistingPivot($roleId, $attributes);
```

###### **查**

```php
$role = User::find(1);
$role-&gt;users;

Role::with(&#039;users&#039;)-&gt;find([1, 2]);
```

#### **定制模型**

###### **指定模型关系中的中间表名称，例如指定中间表名称为 rid_uid**
```php
// App\User
public function roles(){
    return $this-&gt;belongsToMany(&#039;App\Role&#039;, &#039;rid_uid&#039;);
}

// App\Role
public function users(){
    return $this-&gt;belongsToMany(&#039;App\User&#039;, &#039;rid_uid&#039;);
}
```

###### **指定中间表中外键字段名称**

```php
// App\User
public function roles(){
    return $this-&gt;belongsToMany(&#039;App\Role&#039;, &#039;role_user&#039;, &#039;uid&#039;, &#039;rid&#039;);
}

// App\Role
public function users(){
    return $this-&gt;belongsToMany(&#039;App\User&#039;, &#039;role_user&#039;, &#039;rid&#039;, &#039;uid&#039;);
}
```

###### **取出中间表字段**

```php
$user = App\User::find(1);

foreach ($user-&gt;roles as $role) {
    echo $role-&gt;pivot-&gt;role_id;
}
```

###### **重命名 pivot**

```php
return $this-&gt;belongsToMany(&#039;App\Role&#039;)-&gt;as(&#039;subscription&#039;);
```

在 pivot 中添加 created_at, updated_at

```php
return $this-&gt;belongsToMany(&#039;App\Role&#039;)-&gt;withTimestamps();
```

###### **在 pivot 中添加其他字段**

```php
return $this-&gt;belongsToMany(&#039;App\Role&#039;)-&gt;withPivot(&#039;column1&#039;, &#039;column2&#039;);
```

###### **模型关系中设置筛选条件**

```php
return $this-&gt;belongsToMany(&#039;App\Role&#039;)-&gt;wherePivot(&#039;approved&#039;, 1);

return $this-&gt;belongsToMany(&#039;App\Role&#039;)-&gt;wherePivotIn(&#039;priority&#039;, [1, 2]);
```

