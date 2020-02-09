> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=3](https://www.bilibili.com/video/av70545323?p=3 "https://www.bilibili.com/video/av70545323?p=3")

----

###### **目录结构**

1. `app`：应用程序核心目录，几乎项目所有的类都在这里。
2. `bootstrap`：包含框架启动文件 `app.php`，和启动时为了优化性能而生成的文件。
3. `config`：包含所有配置文件。最好是读一遍这些文件，了解你可以轻松配置哪些内容。
4. `database`：包含数据库填充、迁移、模型工厂文件。可以用作 `SQLite` 数据库存放目录。
5. `public`：静态资源目录，并包含了首页文件 `index.php`。
6. `resource`：包含了未编译的源文件（模板、语言、资源）。
7. `routes`：包含了所有的路由定义。
8. `storage`：包含了编译好的模板文件，session 文件，缓存文件，日志等文件。
9. `tests`：包含了自动测试文件。运行测试命令 `php vendor/bin/phpunit`。
10. `vendor`：包含了所有 `composer` 依赖。

###### **初始化配置**

1. 必须设置 web 服务器可读写 `storage` 和 `bootstrap/cache` 目录及其子目录。
	storage：用来存放用户上传的文件、应用程序动态生成的日志等。
	cache：用来存放启动时为了优化性能而生成的缓存文件。

2. 即去除链接中的 `index.php`，比如 `xx.com/index.php/path` 变成 `xx.com/path`。
	apache：打开 mod_rewrite 模块
	nginx：
	
```php
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

3. 配置数据库（配置文件 `.env`）

