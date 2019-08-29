> Laravel 学习交流 QQ 群：375462817


> # 本记录文档前言
> Laravel 文档写的很好，只是新手看起来会有点吃力，需要结合经验和网上的文章，多读、细读才能更好的理解。多读、细读官方文档！！！本文类似于一个大纲，欲知其中详情，且去细读官方文档：[Laravel 6.0 docs](https://laravel.com/docs/6.0)。

```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
# 第一章：前言（一、发布注意事项。二、升级指导。三、贡献指南。四、api 文档。）
```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```

## 一、发布注意事项

Laravel 6.0 是 LTS 版本，生产日期，2019 年 09 月。提供两年的 bug 修复和三年的安全修复。非 LTS 版本，只提供六个月的 bug 修复和一年的安全修复。

## 二、升级指导

[https://laravel.com/docs/6.0/upgrade](https://laravel.com/docs/6.0/upgrade)

## 三、贡献指南

[https://laravel.com/docs/6.0/contributions](https://laravel.com/docs/6.0/contributions)

## 四、api 文档

[官方 api 文档地址：https://laravel.com/api/6.0/](https://laravel.com/api/6.0/)

-------

```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
# 第二章：开始（一、安装。二、配置。三、目录结构。四、homestead。五、Valet。六、部署。）
```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
## 一、安装

#### 服务器需求
```
PHP >= 7.2.0
BCMath PHP Extension
Ctype PHP Extension
JSON PHP Extension
Mbstring PHP Extension
OpenSSL PHP Extension
PDO PHP Extension
Tokenizer PHP Extension
XML PHP Extension
```

#### 两种安装方式

1. `composer global require "laravel/installer"`
    `laravel new blog`                  // 无法指定版本
2. `composer create-project --prefer-dist laravel/laravel blog "6.0.*"`  

#### 安装须知

1. 根目录是`public`
2. 必须设置 web 服务器可读写`storage`和`bootstrap/cache`目录及其子目录
3. 在 web 服务器配置中设置优雅链接 （即去除`index.php`）
4. `php artisan key:generate`
5. `mv .env.example .env`

#### web 服务器上的配置
```c
# apache
# 打开 mod_rewrite 模块
# public/.htaccess
Options +FollowSymLinks -Indexes
RewriteEngine On

RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```
```c
# nginx
location / {
    try_files $uri $uri/ /index.php?$query_string;
}
```

## 二、配置

配置目录 `config`

#### 环境配置

1. `.env` 文件内的变量会被系统级别或服务器级别的变量覆盖。
2. 有空格的值请用双引号包含起来 `APP_NAME="My Application"`
3. `.env` 文件内的变量通过`env()`函数获取，`config`目录下的变量通过`config()`函数获取。
4. 在运行`PHPUnit`测试时或执行以`--env=testing`为选项`Artisan`命令时，`.env.testing`会覆盖 `.env` 文件中的值。

```php
$environment = App::environment();

if (App::environment('local')) {
    // 环境是 local
}

if (App::environment(['local', 'staging'])) {
    // 环境是 local 或 staging...
}
```

#### 隐藏 debug 页面中的环境变量
```php
# config/app.php
return [

    // ...

    'debug_blacklist' => [
        '_ENV' => [
            'APP_KEY',
            'DB_PASSWORD',
        ],

        '_SERVER' => [
            'APP_KEY',
            'DB_PASSWORD',
        ],

        '_POST' => [
            'password',
        ],
    ],
];
```

#### 操作配置、缓存配置

```php
$value = config('app.timezone');                           // 获取
config(['app.timezone' => 'America/Chicago']);             // 设置
php artisan config:cache                 // 缓存（执行该命令后 env 函数将会失效）
php artisan config:clear                 // 清除 
```
#### 维护模式
维护模式 === 503 === Service Unavailable
```php
php artisan down                                           
php artisan down --message="Upgrading Database" --retry=60 
// 优先加载：resources/views/errors/503.blade.php

# 5.5 版本的时候，尝试第二条命令的时候，你会发现并不能生效，一开始我也以为是个 bug，不知道更改没有
# 为此我还在 github 上 pull request
# 官方解释说你要自建模板 resources/views/errors/503.blade.php 去实现任何你想实现的功能。
# 我觉得这边做的不是很好，既然有这个命令选项，就应该实现对应的功能，
# 否则新手根据文档学习到的时候，会困惑为什么该指令的参数不能生效。
```
`php artisan down --allow=127.0.0.1 --allow=192.168.0.0/16`
`php artisan up`


> 当应用程序处于维护模式时，不会处理队列任务。退出维护模式后会继续处理。

## 三、目录结构

#### 根目录下的各个目录

1. `app`：应用程序核心目录，几乎项目所有的类都在这里。
2. `bootstrap`：包含框架启动文件 `app.php`，和启动时为了优化性能而生成的文件。
3. `config`：包含所有配置文件。最好是读一遍这些文件，了解你可以轻松配置哪些内容。
4. `database`：包含数据库填充、迁移、模型工厂文件。可以用作 `SQLite` 数据库存放目录。
5. `public`：静态资源目录，并包含了首页文件 `index.php`。
6. `resource`：包含了未编译的源文件（模板、语言、资源）。
7. `routes`：包含了所有的路由定义。
8. `storage`：包含了编译好的模板文件，session 文件，缓存文件，日志等文件。
9. `tests`：包含了自动测试文件。运行测试命令 `php vendor/bin/phpunit`。
10. `vendor`：`composer` 依赖目录。

#### `app`目录下的各个目录
app 目录下的很多目录是命令生成的。由 Composer 使用 PSR-4 自动加载标准自动加载。
查看生成命令：`php artisan make:list`。

1. `Broadcasting`：包含所有 broadcast channel 类。
2. `Console`：包含自定义的命令和用来注册命令、定义计划任务的内核文件。
3. `Events`：事件目录。
4. `Exceptions`：异常和异常处理目录。
5. `Http`：包含了控制器、中间件和表单请求。几乎所有请求的处理逻辑都被放在这里。
6. `Jobs`：储存队列任务。
7. `Listeners`：存储事件的监听。
8. `Mail`：存储邮件类目录。
9. `Notifications`：存放通知类。laravel 内置了很多驱动： email, Slack, SMS, database。
10. `Policies`：授权策略类目录。
11. `Providers`：包含了所有服务提供者。服务提供者通过在服务容器上绑定服务、注册事件或者执行其他任务来启动你的应用。
12. `Rules`：储存自定义验证规则对象。 

#### `routes`目录下的各个目录

1. `web.php`内的路由将应用 `web` 中间件组（功能：session 状态，CSRF 保护，cookie 加密等）。
2. `api.php`内的路由将应用 `api` 中间件组（功能：访问速率控制等）。所有的请求将通过 token 认证，无 session 状态。
3. `consoles.php`定义了所有基于控制台命令的闭包。
4. `channels.php`注册了所有的事件广播频道。

## 四、Homestead（TODO）

本节介绍了 `homestead` 是什么，怎么用。

#### 介绍

Homestead 是 Laravel 官方提供的 vagrant box。
vagrant 一个用来给虚拟机提供物品（box）的容器，放在容器（vagrant）里的东西被称作 box 。box 一般就是一个操作系统的镜像。

Homestead 提供了 ubantu、git、php、nginx、apache、mysql、mariadb、sqlite3、postgreSQL、composer、node(With Yarn, Bower, Grunt, and Gulp)、redis、memcached、beanstalkd、mailhog、elasticsearch、ngrok。

> windows 需要在 BIOS 中开启硬件虚拟功能 (VT-x)。如果在 UEFI 上使用 Hyper-V，为了使用 VT-x 需要禁用 Hyper-V。


#### 安装

1. 安装虚拟机和 vagrant。
（建议使用 virtualbox，vmware 收费并需要购买插件，parallels 也需要插件，Hyper-V 有局限性）
[vagrant 下载页](https://www.vagrantup.com/downloads.html)
[virtualbox 下载页](https://www.virtualbox.org/wiki/Downloads)
2. 往 vagrant 里面放 `laravel/homestead` box。
`vagrant box add laravel/homestead`
> 国内墙的厉害，需要下载下来再进行本地安装： 
[https://vagrantcloud.com/laravel/boxes/homestead/versions/x.x.x/providers/virtualbox.box](https://link.jianshu.com/?t=https%3A%2F%2Fvagrantcloud.com%2Flaravel%2Fboxes%2Fhomestead%2Fversions)
在上面链接中找到想要下载的版本，然后替换掉 x.x.x ，复制到迅雷中进行下载。
重命名为 larabox
`vagrant box add laravel/homestead ./larabox`

3. 克隆仓库 laravel/homestead。（用来初始化 vagrant，配置 homestead box 的程序。）
a. `git clone https://github.com/laravel/homestead.git ~/Homestead` 
b. `cd ~/Homestead` 
c. `git checkout v7.1.2`（切换到[稳定版本](https://github.com/laravel/homestead/releases)） 
d. `bash init.sh`或者`init.bat`（生成 `Homestead.yaml` 文件）

#### 配置（配置文件 Homestead.yaml）

###### 指定虚拟机
可以是 virtualbox, vmware_fusion, vmware_workstation, parallels 或者 hyperv
`provider: virtualbox`

###### 共享文件夹
`folders`属性用于配置本地和`homestead`环境的同步文件夹。
由于多站点或项目有太多文件而导致性能变差时候，可以设置不同的文件夹来同步不同的项目。
`NFS`也可以用来优化性能，windows 不支持。使用`NFS`时，安装`vagrant-bindfs`插件可维护正确的文件和文件夹权限。
利用 `options` 键可以设置其他选项。

###### nginx 站点
`homestead`使用`sites`属性让你轻松配置`nginx`域名。使用`vagrant reload --provision`更新虚拟机中的`nginx`配置。
配置好域名，记得在本地 hosts 文件添加解析记录。例如：`192.168.10.10  homestead.test`。

###### 启动和删除 box 
```
# 修改 homestead.rb 
# settings[“version”] ||= “>= 0”  
# 找到 composer self-update 删除它 
vagrant up
```
```
vagrant destroy --force
```

###### 为每个项目安装 homestead
这样你就可以在其他项目目录内，通过 vagrant up 来进行该项目的工作。
```
composer require laravel/homestead --dev
// make 命令自动生成 Vagrantfile 和 Homestead.yaml
php vendor/bin/homestead make    // linux & mac
vendor\\bin\\homestead make         // windows
```

###### 安装 mariadb
```
box: laravel/homestead
ip: "192.168.10.10"
memory: 2048
cpus: 4
provider: virtualbox
mariadb: true
```
###### 安装 elasticsearch
需要指定版本，默认安装将创建一个名为 'homestead' 的集群，确保你的虚拟机内存是`elasticsearch`的两倍。
```
box: laravel/homestead
ip: "192.168.10.10"
memory: 4096
cpus: 4
provider: virtualbox
elasticsearch: 6
```
###### `bash`命令别名
```
alias ..='cd ..'      // 随后记得 vagrant reload --provision
```

#### 日常用法

###### 全局访问 homestead 
mac 或 linux
```
function homestead() {
    ( cd ~/Homestead && vagrant $* )
}
```
windows
```
@echo off

set cwd=%cd%
set homesteadVagrant=C:\Homestead

cd /d %homesteadVagrant% && vagrant %*
cd /d %cwd%

set cwd=
set homesteadVagrant=
```
替换`C:\Homestead`，再将脚本加入环境变量即可。

设置完成后可以在任何地方使用 `homestead up`，`homestead ssh`

###### 连接数据库
`mysql 127.0.0.1 33060`
`postgreSQL 127.0.0.1 54320`
在虚拟机里面连接数据库的端口还是默认值 `3306` 和 `5432`。
账号密码：`homestead / secret`

###### 添加多个网站
```
sites:
    - map: homestead.test
      to: /home/vagrant/code/Laravel/public
    - map: another.test
      to: /home/vagrant/code/another/public

192.168.10.10  homestead.test
192.168.10.10  another.test
```

###### 添加 nginx 其他配置
```
sites:
    - map: homestead.test
      to: /home/vagrant/code/Laravel/public
      params:
          - key: FOO
            value: BAR
```

###### 配置时间任务
如果需要开启 `artisan` 命令`schedule:run`一样的效果，配置如下
（`schedule:run`命令的原理是每分钟都会去检查 App\Console\Kernel 类中是否有任务需要执行，有则执行。）
```
sites:
    - map: homestead.test
      to: /home/vagrant/code/Laravel/public
      schedule: true
```

###### 配置 Mailhog（.env）
Mailhog 用来捕获你的邮件，并检查它。你的邮件并不会真的发出去。
```
MAIL_DRIVER=smtp
MAIL_HOST=localhost
MAIL_PORT=1025
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
```
###### 端口配置
默认：
```
SSH: 2222 → Forwards To 22
ngrok UI: 4040 → Forwards To 4040
HTTP: 8000 → Forwards To 80
HTTPS: 44300 → Forwards To 443
MySQL: 33060 → Forwards To 3306
PostgreSQL: 54320 → Forwards To 5432
Mailhog: 8025 → Forwards To 8025
```
自定义：
```
ports:
    - send: 50000
      to: 5000
    - send: 7777
      to: 777
      protocol: udp
```

###### 分享你的环境
`vagrant share` 但是如果`Homestead.yaml`配置了多个站点，此命令不再支持。
解决方案，使用 homestead 内置命令：
```
vagrant ssh
share homestead.test
```
> vagrant 天生就不安全，当你使用 share 命令，就会在互联网中暴露的你虚拟机。

###### 切换 php 版本
> 只兼容 nginx

###### 切换 web 服务器
apache 和 nginx 可以同时安装，但不可同时运行，原理应该是抢占 80 端口。
运行命令 `flip` 可以关闭其中一个，开启另一个。

###### 配置网络接口
可以配置任意数量的接口：
```
networks:
    - type: "private_network"
      ip: "192.168.10.20"
```

想启用 [桥接](https://www.vagrantup.com/docs/networking/public_network.html) 接口，请配置 `bridge` 设置，并将网络类型更改为 `public_network` ：

```
networks:
    - type: "public_network"
      ip: "192.168.10.20"
      bridge: "en1: Wi-Fi (AirPort)"
```

要启用 [DHCP](https://www.vagrantup.com/docs/networking/public_network.html)，只需从配置中删除 `ip` 选项：

```
networks:
    - type: "public_network"
      bridge: "en1: Wi-Fi (AirPort)"
```

###### 更新 homestead
```
vagrant box update
git pull origin master
```
或者
```
vagrant box update
composer update      # 确保 composer.json 包含 "laravel/homestead": "^7"
```
> 如果 windows 下符号链接不生效，添加以下代码块到 `Vagrantfile`
```
config.vm.provider "virtualbox" do |v|
    v.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
end
```

## 五、Valet
[https://laravel.com/docs/6.0/valet](https://laravel.com/docs/6.0/valet)

## 六、部署
本节介绍了几个部署要点。

**nginx 部署**
```bash
server {
    listen 80;
    server_name example.com;
    root /example.com/public;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    index index.html index.htm index.php;

    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    error_page 404 /index.php;

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```
**部署优化**
```
composer install --optimize-autoloader --no-dev
php artisan config:cache
php artisan route:cache
# 开启 OpCache
```



```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
# 第三章：架构（一、生命周期。二、服务容器。三、服务提供者。四、facades。五、contracts。）
```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```

## 一、请求生命周期
本节主要概括了框架运行的生命周期。

1. 所有请求必定首先通过 `public/index.php`。
2. 在上述这个文件中首先加载 `composer` 自动加载文件，然后从 `bootstrap/app.php` 实例化一个服务容器（服务容器，顾名思义是一个容器，可以比作是一个药箱，药箱当然要放药了，药就是所谓的服务提供者。在启动时候当然不能把框架里的所有的药都加载进来，只能加载基础的药。所以这一步还加载了一些基础的服务提供者。）。
3. 接下来，框架会根据请求类型传送请求至 `app/Http/Kernel.php` 或者 `app/Console/Kernel.php`。
4. `app/Http/Kernel.php`扩展了`Illuminate\Foundation\Http\Kernel`类，父类强制在处理请求前应该做哪些操作，操作内容都放到了 `bootstrappers`数组里面（配置错误处理、配置记录日志、检测应用环境、注册和启动服务提供者等）。子类在数组`middleware`中规定了请求在被处理前必须经过的一些处理（读写`session`、判断是否处于维护模式、验证 csrf 令牌等）。
5. 实例化`Http Kernel`，处理请求，返回响应内容。请求将作为参数传入`handle`方法，返回值就是响应内容。

应用程序的所有服务提供程序都在 `config/app.php` 配置文件的 `providers` 数组中配置。首先，将在所有提供程序上调用register方法，然后，一旦所有提供程序都已注册，将调用 `boot` 方法。一旦应用程序被引导，`Request` 将被传递给 `router` 以进行分派。 `router` 会将请求分派给路由或控制器，以及运行任何路由特定的中间件。

## 二、服务容器
本节主要讲了服务容器中的绑定，解析，解析事件（类似于在药瓶中放药，拿药，拿药时会发生什么）。

如果不依赖任何接口，不需要指示容器如何构建这些对象，因为它可以使用反射自动解析这些对象。

#### 绑定
基础绑定
```php
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});  
```
绑定单例
```php
$this->app->singleton('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});   
```
绑定实例
```php
$api = new HelpSpot\API(new HttpClient);

$this->app->instance('HelpSpot\API', $api);   
```
绑定实例时给定初始化数据
```php
$this->app->when('App\Http\Controllers\UserController')
          ->needs('$variableName')
          ->give($value);     // 利用上下文给绑定设置初始数据
```
绑定接口到实例
```php
$this->app->bind(
    'App\Contracts\EventPusher',
    'App\Services\RedisEventPusher'
);     
```
根据上下文提供不同的绑定
```php
$this->app->when(PhotoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('local');
          });

$this->app->when(VideoController::class)
          ->needs(Filesystem::class)
          ->give(function () {
              return Storage::disk('s3');
          });
```
给绑定设置标签
```php
$this->app->bind('SpeedReport', function () {
    //
});

$this->app->bind('MemoryReport', function () {
    //
});

$this->app->tag(['SpeedReport', 'MemoryReport'], 'reports');

$this->app->bind('ReportAggregator', function ($app) {
    return new ReportAggregator($app->tagged('reports'));
});

```
当 `Service` 已经被解析，`extend` 方法可以用来修改解析出来的实例 `$service`：
```php
$this->app->extend(Service::class, function($service) {
    return new ModifyClass($service);
});
```

#### 解析
基础解析

```php
$api = $this->app->make('HelpSpot\API');
```
无法访问 `$app` 时，这样解析
```
$api = resolve('HelpSpot\API');
```
```
$api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);      // 解析时候通过关联数组注入依赖
```
类型提示解析（最常用）
```
public function __construct(UserRepository $users)

# laravel 实现了 PSR-11 接口，所以就可以用该接口的类型提示解析
use Psr\Container\ContainerInterface;
Route::get('/', function (ContainerInterface $container) {
    $service = $container->get('Service');
    //
});
```


#### 容器事件
容器解析任何对象时调用
```php
$this->app->resolving(function ($object, $app) {
    
});
```
容器解析`HelpSpot\API`时调用
```
$this->app->resolving(HelpSpot\API::class, function ($api, $app) {
    
});
```
## 三、服务提供者

加载服务提供者是框架启动的关键步骤之一，他们负责启动不同的组件（数据库、队列、验证、路由等），服务提供者被配置在 `config/app.php`中的`providers`数组。请注意，其中许多是“延迟”提供程序，这意味着它们不会在每个请求中加载，而是仅在实际需要它们提供的服务时加载。首先，他们的`register`方法会被调用，全部调用结束，才会依次调用他们的`boot`方法。当所有服务提供者都注册好了，`Request`将会被 `router` 分发到路由或控制器，同时运行路由指定的中间件。

#### 制作一个服务提供者

1. `php artisan make:provider RiakServiceProvider`
2. 服务提供者主要由两个方法：`register` 和  `boot` 。`register` 只负责绑定一些东西到容器。`boot` 可以使用类型提示解析等来完成任意你想做的事情，这些都归功于容器调用所有服务提供者的`register`方法之后才去调用`boot`方法。
3. 在`config/app.php`的`providers`数组中注册服务提供者。

boot 方法在所有服务提供者 `register` 调用完毕后执行。

#### 制作一个延迟服务提供者
如果只是绑定服务到服务容器，可以选择将该服务提供者设置为延迟（实现 `DeferrableProvider` 接口）。
```
class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection($app['config']['riak']);
        });
    }

    public function provides()
    {
        return [Connection::class];
    }
}
```
----
## 四、Facades

#### 原理
其实`facades`就是各种别名。当你使用某个`facade`的静态方法时，会触发它的父类的`__callStatic`方法，该方法会找到注册在容器中的`facade`原类名，最终调用原类名中的对应方法。

> 不要在一个类中，用太多的`facades`。过于臃肿的情况下应该将大类分解成几个小类。

#### 优点
方便测试（辅助函数和 facades 没什么区别，测试方法也是一样的）。
```
Route::get('/cache', function () {
    return Cache::get('key');     // === return cache('key');
});
```
```
public function testBasicExample()
{
    Cache::shouldReceive('get')
         ->with('key')
         ->andReturn('value');

    $this->visit('/cache')
         ->see('value');
}
```
#### 实时的 facades
原生用法 vs 实时用法 

```php
# 原生用法
use App\Contracts\Publisher;
public function publish(Publisher $publisher)
{
    $this->update(['publishing' => now()]);
    $publisher->publish($this);
}
```
```php
# 实时用法
use Facades\App\Contracts\Publisher;

Publisher::publish($this);
```
测试实时的 facades
```php
use Facades\App\Contracts\Publisher;
Publisher::shouldReceive('publish')->once()->with($podcast);
```
#### facades 列表

Facade  |  Class  |  Service Container Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](https://laravel.com/api/{{version}}/Illuminate/Foundation/Application.html)  |  `app`
Artisan  |  [Illuminate\Contracts\Console\Kernel](https://laravel.com/api/{{version}}/Illuminate/Contracts/Console/Kernel.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](https://laravel.com/api/{{version}}/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Contracts\Auth\Guard](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Guard.html)  |  `auth.driver`
Blade  |  [Illuminate\View\Compilers\BladeCompiler](https://laravel.com/api/{{version}}/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Broadcast  |  [Illuminate\Contracts\Broadcasting\Factory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Factory.html)  |  &nbsp;
Broadcast (Instance)  |  [Illuminate\Contracts\Broadcasting\Broadcaster](https://laravel.com/api/{{version}}/Illuminate/Contracts/Broadcasting/Broadcaster.html)  |  &nbsp;
Bus  |  [Illuminate\Contracts\Bus\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Bus/Dispatcher.html)  |  &nbsp;
Cache  |  [Illuminate\Cache\CacheManager](https://laravel.com/api/{{version}}/Illuminate/Cache/CacheManager.html)  |  `cache`
Cache (Instance)  |  [Illuminate\Cache\Repository](https://laravel.com/api/{{version}}/Illuminate/Cache/Repository.html)  |  `cache.store`
Config  |  [Illuminate\Config\Repository](https://laravel.com/api/{{version}}/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](https://laravel.com/api/{{version}}/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](https://laravel.com/api/{{version}}/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](https://laravel.com/api/{{version}}/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](https://laravel.com/api/{{version}}/Illuminate/Database/Connection.html)  |  `db.connection`
Event  |  [Illuminate\Events\Dispatcher](https://laravel.com/api/{{version}}/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Filesystem/Filesystem.html)  |  `files`
Gate  |  [Illuminate\Contracts\Auth\Access\Gate](https://laravel.com/api/{{version}}/Illuminate/Contracts/Auth/Access/Gate.html)  |  &nbsp;
Hash  |  [Illuminate\Contracts\Hashing\Hasher](https://laravel.com/api/{{version}}/Illuminate/Contracts/Hashing/Hasher.html)  |  `hash`
Lang  |  [Illuminate\Translation\Translator](https://laravel.com/api/{{version}}/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\LogManager](https://laravel.com/api/{{version}}/Illuminate/Log/LogManager.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](https://laravel.com/api/{{version}}/Illuminate/Mail/Mailer.html)  |  `mailer`
Notification  |  [Illuminate\Notifications\ChannelManager](https://laravel.com/api/{{version}}/Illuminate/Notifications/ChannelManager.html)  |  &nbsp;
Password  |  [Illuminate\Auth\Passwords\PasswordBrokerManager](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBrokerManager.html)  |  `auth.password`
Password (Instance)  |  [Illuminate\Auth\Passwords\PasswordBroker](https://laravel.com/api/{{version}}/Illuminate/Auth/Passwords/PasswordBroker.html)  |  `auth.password.broker`
Queue  |  [Illuminate\Queue\QueueManager](https://laravel.com/api/{{version}}/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance)  |  [Illuminate\Contracts\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Contracts/Queue/Queue.html)  |  `queue.connection`
Queue (Base Class)  |  [Illuminate\Queue\Queue](https://laravel.com/api/{{version}}/Illuminate/Queue/Queue.html)  |  &nbsp;
Redirect  |  [Illuminate\Routing\Redirector](https://laravel.com/api/{{version}}/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\RedisManager](https://laravel.com/api/{{version}}/Illuminate/Redis/RedisManager.html)  |  `redis`
Redis (Instance)  |  [Illuminate\Redis\Connections\Connection](https://laravel.com/api/{{version}}/Illuminate/Redis/Connections/Connection.html)  |  `redis.connection`
Request  |  [Illuminate\Http\Request](https://laravel.com/api/{{version}}/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Contracts\Routing\ResponseFactory](https://laravel.com/api/{{version}}/Illuminate/Contracts/Routing/ResponseFactory.html)  |  &nbsp;
Response (Instance)  |  [Illuminate\Http\Response](https://laravel.com/api/{{version}}/Illuminate/Http/Response.html)  |  &nbsp;
Route  |  [Illuminate\Routing\Router](https://laravel.com/api/{{version}}/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Builder](https://laravel.com/api/{{version}}/Illuminate/Database/Schema/Builder.html)  |  &nbsp;
Session  |  [Illuminate\Session\SessionManager](https://laravel.com/api/{{version}}/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](https://laravel.com/api/{{version}}/Illuminate/Session/Store.html)  |  `session.store`
Storage  |  [Illuminate\Filesystem\FilesystemManager](https://laravel.com/api/{{version}}/Illuminate/Filesystem/FilesystemManager.html)  |  `filesystem`
Storage (Instance)  |  [Illuminate\Contracts\Filesystem\Filesystem](https://laravel.com/api/{{version}}/Illuminate/Contracts/Filesystem/Filesystem.html)  |  `filesystem.disk`
URL  |  [Illuminate\Routing\UrlGenerator](https://laravel.com/api/{{version}}/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](https://laravel.com/api/{{version}}/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](https://laravel.com/api/{{version}}/Illuminate/Validation/Validator.html)  |  &nbsp;
View  |  [Illuminate\View\Factory](https://laravel.com/api/{{version}}/Illuminate/View/Factory.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](https://laravel.com/api/{{version}}/Illuminate/View/View.html)  |  


## 五、Contracts
`Facades` 和 `Contracts` 没有什么值得注意的区别，但是当你开发第三方包的时候，最好使用 `Contracts`，这样有利于你编写测试，否则如果使用 `Facades`，因为是第三方包，将不能访问 `facade` 测试函数。

#### 使用方法
在构造函数中类型提示注入就行了。
#### Contracts 列表
Contract  |  References Facade
------------- | -------------
[Illuminate\Contracts\Auth\Access\Authorizable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Authorizable.php) | &nbsp;
[Illuminate\Contracts\Auth\Access\Gate](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Access/Gate.php) | `Gate`
[Illuminate\Contracts\Auth\Authenticatable](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Authenticatable.php) | &nbsp;
[Illuminate\Contracts\Auth\CanResetPassword](https://github.com/illuminate/contracts/blob/{{version}}/Auth/CanResetPassword.php) | &nbsp;
[Illuminate\Contracts\Auth\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Factory.php) | `Auth`
[Illuminate\Contracts\Auth\Guard](https://github.com/illuminate/contracts/blob/{{version}}/Auth/Guard.php) | `Auth::guard()`
[Illuminate\Contracts\Auth\PasswordBroker](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBroker.php) | `Password::broker()`
[Illuminate\Contracts\Auth\PasswordBrokerFactory](https://github.com/illuminate/contracts/blob/{{version}}/Auth/PasswordBrokerFactory.php) | `Password`
[Illuminate\Contracts\Auth\StatefulGuard](https://github.com/illuminate/contracts/blob/{{version}}/Auth/StatefulGuard.php) | &nbsp;
[Illuminate\Contracts\Auth\SupportsBasicAuth](https://github.com/illuminate/contracts/blob/{{version}}/Auth/SupportsBasicAuth.php) | &nbsp;
[Illuminate\Contracts\Auth\UserProvider](https://github.com/illuminate/contracts/blob/{{version}}/Auth/UserProvider.php) | &nbsp;
[Illuminate\Contracts\Bus\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/Dispatcher.php) | `Bus`
[Illuminate\Contracts\Bus\QueueingDispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Bus/QueueingDispatcher.php) | `Bus::dispatchToQueue()`
[Illuminate\Contracts\Broadcasting\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Factory.php) | `Broadcast`
[Illuminate\Contracts\Broadcasting\Broadcaster](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/Broadcaster.php)  | `Broadcast::connection()`
[Illuminate\Contracts\Broadcasting\ShouldBroadcast](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcast.php) | &nbsp;
[Illuminate\Contracts\Broadcasting\ShouldBroadcastNow](https://github.com/illuminate/contracts/blob/{{version}}/Broadcasting/ShouldBroadcastNow.php) | &nbsp;
[Illuminate\Contracts\Cache\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Factory.php) | `Cache`
[Illuminate\Contracts\Cache\Lock](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Lock.php) | &nbsp;
[Illuminate\Contracts\Cache\LockProvider](https://github.com/illuminate/contracts/blob/{{version}}/Cache/LockProvider.php) | &nbsp;
[Illuminate\Contracts\Cache\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Repository.php) | `Cache::driver()`
[Illuminate\Contracts\Cache\Store](https://github.com/illuminate/contracts/blob/{{version}}/Cache/Store.php) | &nbsp;
[Illuminate\Contracts\Config\Repository](https://github.com/illuminate/contracts/blob/{{version}}/Config/Repository.php) | `Config`
[Illuminate\Contracts\Console\Application](https://github.com/illuminate/contracts/blob/{{version}}/Console/Application.php) | &nbsp;
[Illuminate\Contracts\Console\Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Console/Kernel.php) | `Artisan`
[Illuminate\Contracts\Container\Container](https://github.com/illuminate/contracts/blob/{{version}}/Container/Container.php) | `App`
[Illuminate\Contracts\Cookie\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/Factory.php) | `Cookie`
[Illuminate\Contracts\Cookie\QueueingFactory](https://github.com/illuminate/contracts/blob/{{version}}/Cookie/QueueingFactory.php) | `Cookie::queue()`
[Illuminate\Contracts\Database\ModelIdentifier](https://github.com/illuminate/contracts/blob/{{version}}/Database/ModelIdentifier.php) | &nbsp;
[Illuminate\Contracts\Debug\ExceptionHandler](https://github.com/illuminate/contracts/blob/{{version}}/Debug/ExceptionHandler.php) | &nbsp;
[Illuminate\Contracts\Encryption\Encrypter](https://github.com/illuminate/contracts/blob/{{version}}/Encryption/Encrypter.php) | `Crypt`
[Illuminate\Contracts\Events\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Events/Dispatcher.php) | `Event`
[Illuminate\Contracts\Filesystem\Cloud](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Cloud.php) | `Storage::cloud()`
[Illuminate\Contracts\Filesystem\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Factory.php) | `Storage`
[Illuminate\Contracts\Filesystem\Filesystem](https://github.com/illuminate/contracts/blob/{{version}}/Filesystem/Filesystem.php) | `Storage::disk()`
[Illuminate\Contracts\Foundation\Application](https://github.com/illuminate/contracts/blob/{{version}}/Foundation/Application.php) | `App`
[Illuminate\Contracts\Hashing\Hasher](https://github.com/illuminate/contracts/blob/{{version}}/Hashing/Hasher.php) | `Hash`
[Illuminate\Contracts\Http\Kernel](https://github.com/illuminate/contracts/blob/{{version}}/Http/Kernel.php) | &nbsp;
[Illuminate\Contracts\Mail\MailQueue](https://github.com/illuminate/contracts/blob/{{version}}/Mail/MailQueue.php) | `Mail::queue()`
[Illuminate\Contracts\Mail\Mailable](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailable.php) | &nbsp;
[Illuminate\Contracts\Mail\Mailer](https://github.com/illuminate/contracts/blob/{{version}}/Mail/Mailer.php) | `Mail`
[Illuminate\Contracts\Notifications\Dispatcher](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Dispatcher.php) | `Notification`
[Illuminate\Contracts\Notifications\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Notifications/Factory.php) | `Notification`
[Illuminate\Contracts\Pagination\LengthAwarePaginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/LengthAwarePaginator.php) | &nbsp;
[Illuminate\Contracts\Pagination\Paginator](https://github.com/illuminate/contracts/blob/{{version}}/Pagination/Paginator.php) | &nbsp;
[Illuminate\Contracts\Pipeline\Hub](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Hub.php) | &nbsp;
[Illuminate\Contracts\Pipeline\Pipeline](https://github.com/illuminate/contracts/blob/{{version}}/Pipeline/Pipeline.php) | &nbsp;
[Illuminate\Contracts\Queue\EntityResolver](https://github.com/illuminate/contracts/blob/{{version}}/Queue/EntityResolver.php) | &nbsp;
[Illuminate\Contracts\Queue\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Factory.php) | `Queue`
[Illuminate\Contracts\Queue\Job](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Job.php) | &nbsp;
[Illuminate\Contracts\Queue\Monitor](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Monitor.php) | `Queue`
[Illuminate\Contracts\Queue\Queue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/Queue.php) | `Queue::connection()`
[Illuminate\Contracts\Queue\QueueableCollection](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableCollection.php) | &nbsp;
[Illuminate\Contracts\Queue\QueueableEntity](https://github.com/illuminate/contracts/blob/{{version}}/Queue/QueueableEntity.php) | &nbsp;
[Illuminate\Contracts\Queue\ShouldQueue](https://github.com/illuminate/contracts/blob/{{version}}/Queue/ShouldQueue.php) | &nbsp;
[Illuminate\Contracts\Redis\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Redis/Factory.php) | `Redis`
[Illuminate\Contracts\Routing\BindingRegistrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/BindingRegistrar.php) | `Route`
[Illuminate\Contracts\Routing\Registrar](https://github.com/illuminate/contracts/blob/{{version}}/Routing/Registrar.php) | `Route`
[Illuminate\Contracts\Routing\ResponseFactory](https://github.com/illuminate/contracts/blob/{{version}}/Routing/ResponseFactory.php) | `Response`
[Illuminate\Contracts\Routing\UrlGenerator](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlGenerator.php) | `URL`
[Illuminate\Contracts\Routing\UrlRoutable](https://github.com/illuminate/contracts/blob/{{version}}/Routing/UrlRoutable.php) | &nbsp;
[Illuminate\Contracts\Session\Session](https://github.com/illuminate/contracts/blob/{{version}}/Session/Session.php) | `Session::driver()`
[Illuminate\Contracts\Support\Arrayable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Arrayable.php) | &nbsp;
[Illuminate\Contracts\Support\Htmlable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Htmlable.php) | &nbsp;
[Illuminate\Contracts\Support\Jsonable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Jsonable.php) | &nbsp;
[Illuminate\Contracts\Support\MessageBag](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageBag.php) | &nbsp;
[Illuminate\Contracts\Support\MessageProvider](https://github.com/illuminate/contracts/blob/{{version}}/Support/MessageProvider.php) | &nbsp;
[Illuminate\Contracts\Support\Renderable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Renderable.php) | &nbsp;
[Illuminate\Contracts\Support\Responsable](https://github.com/illuminate/contracts/blob/{{version}}/Support/Responsable.php) | &nbsp;
[Illuminate\Contracts\Translation\Loader](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Loader.php) | &nbsp;
[Illuminate\Contracts\Translation\Translator](https://github.com/illuminate/contracts/blob/{{version}}/Translation/Translator.php) | `Lang`
[Illuminate\Contracts\Validation\Factory](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Factory.php) | `Validator`
[Illuminate\Contracts\Validation\ImplicitRule](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ImplicitRule.php) | &nbsp;
[Illuminate\Contracts\Validation\Rule](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Rule.php) | &nbsp;
[Illuminate\Contracts\Validation\ValidatesWhenResolved](https://github.com/illuminate/contracts/blob/{{version}}/Validation/ValidatesWhenResolved.php) | &nbsp;
[Illuminate\Contracts\Validation\Validator](https://github.com/illuminate/contracts/blob/{{version}}/Validation/Validator.php) | `Validator::make()`
[Illuminate\Contracts\View\Engine](https://github.com/illuminate/contracts/blob/{{version}}/View/Engine.php) | &nbsp;
[Illuminate\Contracts\View\Factory](https://github.com/illuminate/contracts/blob/{{version}}/View/Factory.php) | `View`
[Illuminate\Contracts\View\View](https://github.com/illuminate/contracts/blob/{{version}}/View/View.php) | `View::make()`

```
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
# 第四章：基础（一、路由。二、中间件。三、CSRF 保护。四、控制器。五、请求。六、响应。七、视图。八、Url 生成。九、Session 。十、验证。十一、错误与日志）
```
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
## 一、路由
`web/api.php` 中定义的路由会自动添加`api`前缀，如需修改该前缀，可以在`RouteServiceProvider` 修改。
在 `web.php` 路由里的 `POST, PUT, DELETE` 方法，在提交表单时候必须加上`CSRF`参数。
```
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

#### 两个 api [Router](https://laravel.com/api/5.5/Illuminate/Routing/Router.html) 和 [Route](https://laravel.com/api/5.5/Illuminate/Routing/Route.html)

#### resource
```php
* GET /test
index()  //  展示列表

* GET /test/create
create()  // 展示用来创建的表单

* POST /test
store()  // 增加资源

* GET /test/{id}
show($id)  // 展示一个资源

* GET /test/{id}/edit
edit($id)  // 展示编辑表单

* PUT /test/{id}
update($id)  // 更新特定资源

* DELETE /test/{id} 
destroy($id)  // 移除资源
```


#### 基本路由

```php
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);        // 全体更新
Route::patch($uri, $callback);      // 局部更新
Route::delete($uri, $callback);
Route::options($uri, $callback);    // 允许客户端检查性能

Route::any($uri, $callback);        // 任意 method

Route::match(['get', 'post'], '/', function () {
    //
});
```

```php
# 重定向路由
Route::permanentRedirect('/here', '/there');  // 301
Route::redirect('/here', '/there', 301);  // 第三个参数不写则默认为 302

# 只需要返回一个视图
Route::view('/welcome', 'welcome');
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

#### 路由参数

```php
Route::get('posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});

Route::get('user/{name?}', function ($name = 'John') {   // 一定要给可选参数设置默认值
    return $name;
});

```

#### 正则表达式约束

```php

Route::get('user/{id}', function ($id) {
    //
})->where('id', '[0-9]+');

Route::get('user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

#### 全局约束

```php
// RouteServiceProvider
public function boot()
{
    Route::pattern('id', '[0-9]+');

    parent::boot();
}
```

#### 命名路由

```php
Route::get('user/profile', function () {
    //
})->name('profile');

// 使用
$url = route('profile');
// 生成重定向...
return redirect()->route('profile');


Route::get('user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1]);
```


```php
// 在中间件中检查当前路由
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }
    return $next($request);
}
```
#### 添加中间件
```php
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // 使用 `first` 和 `second` 中间件
    });

    Route::get('user/profile', function () {
        // 使用 `first` 和 `second` 中间件
    });
});
```
#### 命名空间
```php
Route::namespace('Admin')->group(function () {
    // 在 "App\Http\Controllers\Admin" 命名空间下的控制器
});
```
#### 子域名路由
```php
Route::domain('{account}.myapp.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```
#### 路由前缀
```php
Route::prefix('admin')->group(function () {
    Route::get('users', function () {
        // 匹配包含 "/admin/users" 的 URL
    });
});
```
#### 路由命名前缀
```php
Route::name('admin.')->group(function () {
    Route::get('users', function () {
        // Route assigned name "admin.users"...
    })->name('users');
});
```
#### 表单伪造
```php
<input type="hidden" name="_method" value="PUT">
// 或者 @method('PUT')
```
#### 获取当前路由信息
```php
// Route::get('/', 'TestController@test')->name("mytest");
$route = Route::current(); //  object(Illuminate\Routing\Route)
$name = Route::currentRouteName(); // mytest 
$action = Route::currentRouteAction(); // 控制器中：App\Http\Controllers\TestController@test  路由中：null
```
#### 隐式绑定
```php
Route::get('api/users/{user}', function (App\User $user) {
    return $user->email;	// 传人的 id
});

# 自定义键名 在模型中修改
# App/User.php
public function getRouteKeyName()
{
    return 'slug';
}
```
#### 显式绑定
```php
# RouteServiceProvider
public function boot()
{
    parent::boot();

    Route::model('user', App\User::class);
}

Route::get('profile/{user}', function ($user) {
    //
});
```
#### 自定义解析逻辑
```php
public function boot()
{
    parent::boot();

    Route::bind('user', function ($value) {
        return App\User::where('name', $value)->first() ?? abort(404);
    });
}
```
```
public function resolveRouteBinding($value)
{
    return $this->where('name', $value)->first() ?? abort(404);
}
```
#### 默认路由，一般用来 404，一定要放在所有路由最后面
```
Route::fallback(function () {
    //
});
```
#### 速率控制
```
Route::middleware('auth:api', 'throttle:60,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});

Route::middleware('auth:api', 'throttle:rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});

Route::middleware('throttle:10|60,1')->group(function () {
    // 60 次给认证过的用户，10 次给普通用户
});

Route::middleware('auth:api', 'throttle:10|rate_limit,1')->group(function () {
    Route::get('/user', function () {
        //
    });
});
```
## 二、中间件
`php artisan make:middleware TestMiddleware`
注册中间件到 `app/Http/Kernel.php`  `$routeMiddleware` 中。

```
所有中间件都通过服务容器解析，因此您可以在中间件的构造函数中键入提示所需的任何依赖项。

# 中间件操作发生请求被处理之前
public function handle($request, Closure $next)
{
    if ($request->age <= 200) {
       return redirect('home');
    }    // Perform action

    return $next($request);
}

# 中间件操作发生请求被处理之后
public function handle($request, Closure $next)
{
    $response = $next($request);
    // Perform action
    return $response;
}
```
注册全局中间件，就将完整类名写在`app/Http/Kernel.php`文件中的`$middleware`数组中。如果是非全局，部分的，可以放在文件的其他数组中。也可以不注册，在需要使用时，引入直接使用。

使用中间件的两种方式
```
# 常用方式
Route::get('admin/profile', function () {
    
})->middleware('auth');

# 不用注册的方式
use App\Http\Middleware\CheckAge;
Route::get('admin/profile', function () {

})->middleware(CheckAge::class);
```
如果想应用中间件组，请注册一个关联数组到`app/Http/Kernel.php`的`$middlewareGroups`中。这样就可以使用时填写该数组的键值就行了。默认情况下，`RouteServiceProvider`已将中间件组`web`应用在你的`web.php`的路由中。
```
Route::get('/', function () {
    //
})->middleware('web');

Route::group(['middleware' => ['web']], function () {
    //
});
```
给中间件排序
```
# app/Http/Kernel.php
protected $middlewarePriority = [
    \Illuminate\Session\Middleware\StartSession::class,
    \Illuminate\View\Middleware\ShareErrorsFromSession::class,
    \App\Http\Middleware\Authenticate::class,
    \Illuminate\Session\Middleware\AuthenticateSession::class,
    \Illuminate\Routing\Middleware\SubstituteBindings::class,
    \Illuminate\Auth\Middleware\Authorize::class,
];
```

给 middleware 传参
```
// role 中间件
public function handle($request, Closure $next, $role)
{
    if (! $request->user()->hasRole($role)) {
        // Redirect...
    }
   return $next($request);
}

Route::put('post/{id}', function ($id) {
    //
})->middleware('role:editor');
```
`terminate` 方法调用于将响应发送到浏览器之后。
```
# terminate 会从服务容器解析一个新的中间件实例。
# 如果你希望 handle 和 terminate 是同一实例，请将这个中间件单例绑定 singleton 到服务容器中。
public function handle($request, Closure $next)
{
    return $next($request);
}

public function terminate($request, $response)
{
    // Store the session data...
}
```
一旦定义了可终止的中间件，就应该将其添加到 `app/Http/Kernel.php` 文件中的路由或全局中间件列表中。
## 三、CSRF 保护
`VerifyCsrfToken`中间件存在于`web`中间件组中，它实现了 CSRF 保护。而该中间件组被默认应用在`web.php`文件中的所有路由。

默认情况下，`resources/assets/js/bootstrap.js`文件会使用`Axios HTTP`库注册`csrf-token` meta 标签的值。如果不使用这个库，需要自己去设置。

指定 uri 不去应用 csrf 保护
1.  不将路由写入`route/web.php`。
2.  在`VerifyCsrfToken`中间件的排除数组 $except 中添加你的 uri
```
protected $except = [
    'stripe/*',
    'http://example.com/foo/bar',
    'http://example.com/foo/*',
];
```
除了可以在验证 post 时作为表单参数传递 csrf ，还可以设置 meta 标签来进行 csrf 保护（如 ajax ）。
设置：`<meta name="csrf-token" content="{{ csrf_token() }}">`
ajax 获取：`'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')`

cookie 中，`XSRF-TOKEN`的值可以直接设为`X-CSRF-TOKEN`请求头的值。`Angular`、`Axios`等会自动执行上述操作。

默认情况下，`resources/js/bootstrap.js` 文件包含 `Axios HTTP` 库，它会自动为您发送此文件。

## 四、控制器
定义控制器可以不去继承`Controllers`，这样你将不能使用`middleware`、` validate`、`dispatch`等方法。

如果你的控制器只有一个方法，可以这么玩儿：
```
# Route::get('user/{id}', 'ShowProfile');
# php artisan make:controller ShowProfile --invokable
public function __invoke($id)
{
     return view('user.profile', ['user' => User::findOrFail($id)]);
}
```
`Route::get('profile', 'UserController@show')->middleware('auth');`
控制器的构造函数中的中间件的玩法
```
$this->middleware('auth');
$this->middleware('log')->only('index');
$this->middleware('subscribed')->except('store');
$this->middleware(function ($request, $next) {
       return $next($request);
});
```
Resource 控制器
```
php artisan make:controller PhotoController --resource
php artisan make:controller PhotoController --resource --model=Photo
php artisan make:controller API/PController --api

Route::resource('photos', 'PhotoController');
Route::apiResource('photo', 'PhotoController');    // 没有 create 和 edit

Route::resource('photos', 'PhotoController')->names([
    'create' => 'photos.build'
]);  // 重命名路由名称

Route::resources([
    'photos' => 'PhotoController',
    'posts' => 'PostController'
]);
Route::resource('photo', 'PhotoController', ['only' => [
    'index', 'show'
]]);
Route::resource('photo', 'PhotoController', ['except' => [
    'create', 'store', 'update', 'destroy'
]]);

Route::resource('photo', 'PhotoController', ['names' => [
    'create' => 'photo.build'
]]);

# 模拟 http 请求方法
@method('PUT')
```


| Verb      | URI                    | Action  | Route Name     |
| --------- | ---------------------- | ------- | -------------- |
| GET       | `/photos`              | index   | photos.index   |
| GET       | `/photos/create`       | create  | photos.create  |
| POST      | `/photos`              | store   | photos.store   |
| GET       | `/photos/{photo}`      | show    | photos.show    |
| GET       | `/photos/{photo}/edit` | edit    | photos.edit    |
| PUT/PATCH | `/photos/{photo}`      | update  | photos.update  |
| DELETE    | `/photos/{photo}`      | destroy | photos.destroy |


命名 resource 路由的参数

默认情况下，`Route::resource` 会根据资源名称的「单数」形式创建资源路由的路由参数。你可以在选项数组中传入 `parameters` 参数来轻松地覆盖每个资源。`parameters` 数组应该是资源名称和参数名称的关联数组：

```
Route::resource('user', 'PhotoController', ['parameters' => [
    'photo' => 'photo_in_phone'
]]);
# /user/{photo_in_phone}
```
重命名所有的动词名 create、edit
```
# AppServiceProvider
Route::resourceVerbs([
    'create' => 'crear',
    'edit' => 'editar',
]);
```
> 1. 如果想加入其他控制器方法，尽量写在`resource`控制器路由之前，否则可能会被`resource`控制器的路由覆盖。
> 2. 写的控制器要专一，如果你需要典型的 `resource` 操作之外的方法，可以考虑将你的控制器分成两个更小的控制器。  

参数必须在依赖注入之后传入
```
# Route::put('user/{id}', 'UserController@update');
public function update(Request $request, $id)
{
    //
}
```
## 五、请求
请求将经过`TrimStrings`、`ConvertEmptyStringsToNull `等中间件，这样可以不用担心标准化的问题。如果要禁用此行为，从 `App\Http\Kernel` 类的 `$ middleware` 属性中删除它们。
```
$request->path()
$request->is('admin/*')
$request->url()
$request->fullUrl()
$request->isMethod('post')
$request->all()
$request->input('name')
$request->input('name', 'Sally')     // 第二个参数为默认值
$request->input('products.0.name')
$request->input('products.*.name')
$request->input()
$request->query('name')
$request->query('name', 'Helen')
$request->query()    // 返回所有 query string 的关联数组
$request->name      // laravel 首先查找请求数据，在查找路由参数
$request->input('user.name')     // 获取 json，请求 contentType: "application/json"  
$request->only(['username', 'password'])    // 返回请求关联数组，但不会返回不存在请求数据
$input = $request->only('username', 'password');   // 同上
$input = $request->except(['credit_card']);
$input = $request->except('credit_card');
$request->has('name')
$request->has(['name', 'email'])    // 同时存在
$request->filled('name')    // 存在并且为非空
$request->flash()     // 将当前输入存进 session 中，以便下次请求可以使用它们
$request->flashOnly(['username', 'email'])
$request->flashExcept('password')
return redirect('form')->withInput()    // 等同于：$request->flash(); 
return redirect('form')->withInput(
    $request->except('password')
);
$request->old('username')   // 取出 flash() 内容，并从 session 中清除
<input type="text" name="username" value="{{ old('username') }}">   // 不存在返回 null
$request->cookie('name')   // === \Cookie::get('name')
response('Hello World')->cookie(
    'name', 'value', $minutes
);
response('Hello World')->cookie(
    'name', 'value', $minutes, $path, $domain, $secure, $httpOnly
);
Cookie::queue(Cookie::make('name', 'value', $minutes));
Cookie::queue('name', 'value', $minutes);

$cookie = cookie('name', 'value', $minutes);
return response('Hello World')->cookie($cookie);

$request->file('photo')
$request->photo
$request->hasFile('photo')
$request->file('photo')->isValid()
$request->photo->path()
$request->photo->extension()
$path = $request->photo->store('images');
$path = $request->photo->store('images', 's3');
$path = $request->photo->storeAs('images', 'filename.jpg');
$path = $request->photo->storeAs('images', 'filename.jpg', 's3');
```
配置可信任的代理
```
<?php

namespace App\Http\Middleware;

use Illuminate\Http\Request;
use Fideloper\Proxy\TrustProxies as Middleware;

class TrustProxies extends Middleware
{
    protected $proxies = [
        '192.168.1.1',
        '192.168.1.2',
    ];

    protected $headers = Request::HEADER_X_FORWARDED_ALL;
}
```
## 六、响应
laravel 会自动将你的 return 数据封装成响应。如果你返回数组，会自动封装成 json。如果返回 `Eloquent 集合`，也会自动封装成 json。

自定义
```
return response('Hello World', 200)->header('X-Header-One', 'Header Value')->header('Content-Type', 'text/plain');
// 或者
return response($content)
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
                'X-Header-Two' => 'Header Value',
            ])

# Laravel 内置了一个 cache.headers 中间件，可以用来快速地为路由组设置 Cache-Control 头信息。如果在指令集中声明了 etag，Laravel 会自动将 ETag 标识符设置为响应内容的 MD5 哈希值：

Route::middleware('cache.headers:public;max_age=2628000;etag')->group(function() {
    Route::get('privacy', function () {
        // ...
    });

    Route::get('terms', function () {
        // ...
    });
});

return response($content)
                ->header('Content-Type', $type)
                ->cookie('name', 'value', $minutes);
# ->cookie($name, $value, $minutes, $path, $domain, $secure, $httpOnly)
# Cookie::queue(Cookie::make('name', 'value', $minutes));
# Cookie::queue('name', 'value', $minutes);
# 如果不想某个 cookie 在客户端中加密，
# 请在 App\Http\Middleware\EncryptCookies 的 except 数组中进行配置。
```
重定向
```
return redirect('home/dashboard');
return back()->withInput();     // 请确保该路由在`web.php`中
return redirect()->route('login');
return redirect()->route('profile', ['id' => 1]);

return redirect()->route('profile', [$user]);    
// 第二个参数也可以是 $user，路由：profile/{id}。该写法会自动提取 $user 中的 id
# 如果想定制自动提取的字段
public function getRouteKey()
{
    return $this->slug;
} 

return redirect()->action('HomeController@index');

return redirect()->action(
    'UserController@profile', ['id' => 1]
);

return redirect()->away('https://www.google.com');

return redirect('dashboard')->with('status', 'Profile updated!');  
// 设置 cookie，获取 {{ session('status') }}
```
生成其他类型的响应
```
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);

return response()->json([
    'name' => 'Abigail',
    'state' => 'CA'
]);     // json 会自动设置响应头 Content-Type => application/json
```
实现 json 响应
```
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA']);
            
return response()
            ->json(['name' => 'Abigail', 'state' => 'CA'])
            ->withCallback($request->input('callback'));
```
实现下载
```
return response()->download($pathToFile);
return response()->download($pathToFile, $name, $headers);
return response()->download($pathToFile)->deleteFileAfterSend();
```
> 管理文件下载的扩展包 Symfony HttpFoundation，要求下载文件名必须是 ASCII 编码。

流下载
```
return response()->streamDownload(function () {
    echo GitHub::api('repo')
                ->contents()
                ->readme('laravel', 'laravel')['contents'];
}, 'laravel-readme.md');
```

展示图片、pdf 等
```
return response()->file($pathToFile);
return response()->file($pathToFile, $headers);
```
定制通用响应
```
// 定制 class ResponseMacroServiceProvider extends ServiceProvider 中的 boot 方法
Response::macro('caps', function ($value) {
    return Response::make(strtoupper($value));
});

// 使用 
return response()->caps('foo');
```
## 七、视图
```
return view('greeting', ['name' => 'James']);
return view('admin.profile', $data);
if (View::exists('emails.customer')) 
return view()->first(['custom.admin', 'admin'], $data);  
return View::first(['custom.admin', 'admin'], $data);   // 同上
return view('greetings', ['name' => 'Victoria']);
return view('greeting')->with('name', 'Victoria');

public function boot()
{
    View::share('key', 'value');  // class AppServiceProvider
    // 在所有视图中都可以使用 {{ $key }}
}
```
视图 composers
```
// service provider
public function boot()
{
    View::composer(
        'profile', 'App\Http\ViewComposers\ProfileComposer'
    );            // ProfileComposer@compose 在 profile 视图生成前调用

    View::composer('dashboard', function ($view) {
        //
    });
}

class ProfileComposer
{
    protected $users;

    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }

    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```
```
# 多个视图
View::composer(
    ['profile', 'dashboard'],
    'App\Http\ViewComposers\MyViewComposer'
);
```
```
# 所有
View::composer('*', function ($view) {
    //
});
```
视图 creator
```
# 视图 creators 和视图合成器非常相似。唯一不同之处在于：视图构造器在视图实例化之后立即执行，而视图合成器在视图即将渲染时执行。
View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
```
视图 creator 和视图合成器非常相似。唯一不同之处在于：视图构造器在视图实例化之后立即执行，而视图合成器在视图即将渲染时执行。creator 在 composer 之前。
## 八、url 生成
```
echo url("/posts/{$post->id}");
// http://example.com/posts/1

echo url()->current();     // === URL::current();
echo url()->full();        // === URL::full();
echo url()->previous();    // === URL::previous();

echo route('post.show', ['post' => 1]);
// http://example.com/post/1

echo route('post.show', ['post' => $post]);      
// $post 是一个Eloquent model，该写法自动提取出主键

$url = action('HomeController@index');
action([HomeController::class, 'index']);
$url = action('UserController@profile', ['id' => 1]);

URL::defaults(['locale' => $request->user()->locale]);  
// 设置默认值 Route::get('/{locale}/posts', function () { ... })->name('post.index');
```
签名 URL
```
# Laravel 允许你轻松地为命名路径创建 「签名」 URL。这些 URL 在查询字符串后附加了 「签名」哈希，允许 Laravel 验证 URL 自创建以来未被修改过。签名 URL 对于可公开访问但需要一层防止 URL 操作的路由特别有用。
return URL::signedRoute('unsubscribe', ['user' => 1]);

return URL::temporarySignedRoute(
    'unsubscribe', now()->addMinutes(30), ['user' => 1]
);

Route::get('/unsubscribe/{user}', function (Request $request) {
    if (! $request->hasValidSignature()) {
        abort(401);
    }
    // ...
})->name('unsubscribe');

# 添加到中间件
protected $routeMiddleware = [
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
];
Route::post('/unsubscribe/{user}', function (Request $request) {
    // ...
})->name('unsubscribe')->middleware('signed');
```
## 九、session
驱动设置为 array，是用在测试的时候保证不会持久存储 session。

使用 database 作为驱动
```
Schema::create('sessions', function ($table) {
    $table->string('id')->unique();
    $table->unsignedInteger('user_id')->nullable();
    $table->string('ip_address', 45)->nullable();
    $table->text('user_agent')->nullable();
    $table->text('payload');
    $table->integer('last_activity');
});

php artisan session:table
php artisan migrate
```
session 操作
```
$request->session()->get('key');
$request->session()->get('key', 'default');
$request->session()->get('key', function () {
    return 'default';
});

session('key');
session('key', 'default');
session(['key' => 'value']);

// 测试 assertSessionHas

$request->session()->all();
if ($request->session()->has('users'))     // 存在且不为 null
if ($request->session()->exists('users'))     // 可以是 null

$request->session()->put('key', 'value');
session(['key' => 'value']);

$request->session()->push('user.teams', 'developers');
$value = $request->session()->pull('key', 'default');     // 取出并删除

$request->session()->flash('status', 'Task was successful!');    // 短暂存储 session 一次
$request->session()->reflash();     // 保持 session 再存储一次
$request->session()->keep(['username', 'email']);     // 保持特定 session 存储一次

$request->session()->forget('key');
$request->session()->forget(['key1', 'key2']);
$request->session()->flush();

$request->session()->regenerate();    
// 手动重新生成 session id，LoginController 内置自动重新生成 session id
```
自定义 session 驱动
```
# SessionHandlerInterface 必须实现此接口
<?php
namespace App\Extensions;    // 随意放置于哪个目录

class MongoSessionHandler implements \SessionHandlerInterface
{
    public function open($savePath, $sessionName) {}
    public function close() {}
    public function read($sessionId) {}
    public function write($sessionId, $data) {}
    public function destroy($sessionId) {}
    public function gc($lifetime) {}
}


// service provider
public function boot()
{
    Session::extend('mongo', function ($app) {
        // Return implementation of SessionHandlerInterface...
        return new MongoSessionStore;
    });
}

// 在 config/session.php 中配置 mongo
```
## 十、验证
###### 基础验证
如果请求验证失败，会返回跳转响应，如果是 ajax ，会返回带有状态码 422 的 json 格式的响应。
```
$validatedData = $request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
]);

$validatedData = $request->validate([
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```
> 如果希望验证了某个字段失败后，不再验证后面的字段。请在前一个字段验证规则中添加`bail`。

###### 嵌套验证
```
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```
###### 验证失败
$errors 变量被 Web 中间件组提供的 Illuminate\View\Middleware\ShareErrorsFromSession 中间件绑定到视图中。 当这个中间件被应用后，在你的视图中就可以获取到 $error 变量 , 可以使一直假定 $errors 变量存在并且可以安全地使用。
```
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<input id="title" type="text" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

> 因为全局都会经过`TrimStrings `和`ConvertEmptyStringsToNull`中间件，所以如果要设置某个字段验证时可以为`null`，请在验证规则中添加`nullable`。
###### 表单请求验证
命令生成
`php artisan make:request StoreBlogPost`

验证规则
```
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```
```
public function store(StoreBlogPost $request)
{
    // The incoming request is valid...

    // Retrieve the validated input data...
    $validated = $request->validated();
}
```
添加 after 钩子
```
public function withValidator($validator)      // withValidator 在验证之前做一些操作
{
    $validator->after(function ($validator) {
        if ($this->somethingElseIsInvalid()) {
            $validator->errors()->add('field', 'Something is wrong with this field!');
        }
    });
}
```
授权验证
```
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```
定制错误消息
```
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'body.required'  => 'A message is required',
    ];
}

```
定制验证属性
```
# :attribute
public function attributes()
{
    return [
        'email' => 'email address',
    ];
}
```
###### 手动创建验证
```
$validator = Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
]);

if ($validator->fails()) {
    return redirect('post/create')
        ->withErrors($validator)       // withErrors 接受 validator 或者 MessageBag 或者 array 作为参数
        ->withInput();
}
```
自动跳转
```
Validator::make($request->all(), [
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
])->validate();
```
命名错误包
```
return redirect('register')
            ->withErrors($validator, 'login');

// {{ $errors->login->first('email') }}
```
after 钩子
```
$validator = Validator::make(...);

$validator->after(function ($validator) {
    if ($this->somethingElseIsInvalid()) {
        $validator->errors()->add('field', 'Something is wrong with this field!');
    }
});

if ($validator->fails()) {
    //
}
```
###### 错误信息
```
$errors = $validator->errors();
echo $errors->first('email');

foreach ($errors->get('email') as $message) {
    //
}

foreach ($errors->get('attachments.*') as $message) {
    //
}

foreach ($errors->all() as $message) {
    //
}

if ($errors->has('email')) {
    //
}
```
自定义错误消息
```
$messages = [
    'required' => 'The :attribute field is required.',
];
$validator = Validator::make($input, $rules, $messages);
```
```
$messages = [
    'email.required' => 'We need to know your e-mail address!',
];
```
在语言文件中指定自定义消息
```
在大多数情况下，您可能会在语言文件中指定自定义消息，而不是直接将它们传递给 Validator。为此，需要把你的消息放置于 resources/lang/xx/validation.php 语言文件内的 custom 数组中。

'custom' => [
    'email' => [
        'required' => 'We need to know your e-mail address!',
    ],
],
```
在语言文件中指定自定义属性
```
如果你希望将验证消息的 :attribute 占位符替换为自定义属性名称，你可以在 resources/lang/xx/validation.php 语言文件的 attributes 数组中指定自定义名称：

'attributes' => [
    'email' => 'email address',
],
```
在语言文件中指定自定义值
```
有时可能需要将验证消息的 :value 占位符替换为值的自定义文字。例如，如果 payment_type 的值为 cc，使用以下验证规则，该规则指定需要信用卡号：

$request->validate([
    'credit_card_number' => 'required_if:payment_type,cc'
]);
```
如果此验证规则失败，则会产生以下错误消息：
```
    当payment type为cc时，credit card number 不能为空。
```
您可以通过定义 values 数组，在 validation 语言文件中指定自定义值表示，而不是显示 cc 作为支付类型值：
```
'values' => [
    'payment_type' => [
        'cc' => '信用卡'
    ],
],
```
现在，如果验证规则失败，它将产生以下消息：
```
    当payment type 为信用卡时，credit card number不能为空。
```
#### 可用的验证规则
[Accepted](https://laravel.com/docs/6.0/validation#rule-accepted)
[Active URL](https://laravel.com/docs/6.0/validation#rule-active-url)
[After (Date)](https://laravel.com/docs/6.0/validation#rule-after)
[After Or Equal (Date)](https://laravel.com/docs/6.0/validation#rule-after-or-equal)
[Alpha](https://laravel.com/docs/6.0/validation#rule-alpha)
[Alpha Dash](https://laravel.com/docs/6.0/validation#rule-alpha-dash)
[Alpha Numeric](https://laravel.com/docs/6.0/validation#rule-alpha-num)
[Array](https://laravel.com/docs/6.0/validation#rule-array)
[Bail](https://laravel.com/docs/6.0/validation#rule-bail)
[Before (Date)](https://laravel.com/docs/6.0/validation#rule-before)
[Before Or Equal (Date)](https://laravel.com/docs/6.0/validation#rule-before-or-equal)
[Between](https://laravel.com/docs/6.0/validation#rule-between)
[Boolean](https://laravel.com/docs/6.0/validation#rule-boolean)
[Confirmed](https://laravel.com/docs/6.0/validation#rule-confirmed)
[Date](https://laravel.com/docs/6.0/validation#rule-date)
[Date Equals](https://laravel.com/docs/6.0/validation#rule-date-equals)
[Date Format](https://laravel.com/docs/6.0/validation#rule-date-format)
[Different](https://laravel.com/docs/6.0/validation#rule-different)
[Digits](https://laravel.com/docs/6.0/validation#rule-digits)
[Digits Between](https://laravel.com/docs/6.0/validation#rule-digits-between)
[Dimensions (Image Files)](https://laravel.com/docs/6.0/validation#rule-dimensions)
[Distinct](https://laravel.com/docs/6.0/validation#rule-distinct)
[E-Mail](https://laravel.com/docs/6.0/validation#rule-email)
[Ends With](https://laravel.com/docs/6.0/validation#rule-ends-with)
[Exists (Database)](https://laravel.com/docs/6.0/validation#rule-exists)
[File](https://laravel.com/docs/6.0/validation#rule-file)
[Filled](https://laravel.com/docs/6.0/validation#rule-filled)
[Greater Than](https://laravel.com/docs/6.0/validation#rule-gt)
[Greater Than Or Equal](https://laravel.com/docs/6.0/validation#rule-gte)
[Image (File)](https://laravel.com/docs/6.0/validation#rule-image)
[In](https://laravel.com/docs/6.0/validation#rule-in)
[In Array](https://laravel.com/docs/6.0/validation#rule-in-array)
[Integer](https://laravel.com/docs/6.0/validation#rule-integer)
[IP Address](https://laravel.com/docs/6.0/validation#rule-ip)
[JSON](https://laravel.com/docs/6.0/validation#rule-json)
[Less Than](https://laravel.com/docs/6.0/validation#rule-lt)
[Less Than Or Equal](https://laravel.com/docs/6.0/validation#rule-lte)
[Max](https://laravel.com/docs/6.0/validation#rule-max)
[MIME Types](https://laravel.com/docs/6.0/validation#rule-mimetypes)
[MIME Type By File Extension](https://laravel.com/docs/6.0/validation#rule-mimes)
[Min](https://laravel.com/docs/6.0/validation#rule-min)
[Not In](https://laravel.com/docs/6.0/validation#rule-not-in)
[Not Regex](https://laravel.com/docs/6.0/validation#rule-not-regex)
[Nullable](https://laravel.com/docs/6.0/validation#rule-nullable)
[Numeric](https://laravel.com/docs/6.0/validation#rule-numeric)
[Present](https://laravel.com/docs/6.0/validation#rule-present)
[Regular Expression](https://laravel.com/docs/6.0/validation#rule-regex)
[Required](https://laravel.com/docs/6.0/validation#rule-required)
[Required If](https://laravel.com/docs/6.0/validation#rule-required-if)
[Required Unless](https://laravel.com/docs/6.0/validation#rule-required-unless)
[Required With](https://laravel.com/docs/6.0/validation#rule-required-with)
[Required With All](https://laravel.com/docs/6.0/validation#rule-required-with-all)
[Required Without](https://laravel.com/docs/6.0/validation#rule-required-without)
[Required Without All](https://laravel.com/docs/6.0/validation#rule-required-without-all)
[Same](https://laravel.com/docs/6.0/validation#rule-same)
[Size](https://laravel.com/docs/6.0/validation#rule-size)
[Sometimes](https://laravel.com/docs/6.0/validation#conditionally-adding-rules)
[Starts With](https://laravel.com/docs/6.0/validation#rule-starts-with)
[String](https://laravel.com/docs/6.0/validation#rule-string)
[Timezone](https://laravel.com/docs/6.0/validation#rule-timezone)
[Unique (Database)](https://laravel.com/docs/6.0/validation#rule-unique)
[URL](https://laravel.com/docs/6.0/validation#rule-url)
[UUID](https://laravel.com/docs/6.0/validation#rule-uuid)


官方详细内容：[验证规则](https://laravel.com/docs/6.0/validation#available-validation-rules)。
###### 按条件添加验证规则
```
Validator::make($data, [
    'email' => 'sometimes|required|email',               // sometimes $data 内含有 email 才验证
]);
```
复杂的验证规则
```
$v = Validator::make($data, [
    'email' => 'required|email',
    'games' => 'required|numeric',
]);

$v->sometimes('reason', 'required|max:500', function ($input) {
    return $input->games >= 100;
});

$v->sometimes(['reason', 'cost'], 'required', function ($input) {
    return $input->games >= 100;
});
# 传入 闭包 的 $input 参数是 Illuminate\Support\Fluent 的一个实例，可用来访问你的输入或文件对象。
```
验证数组
```
$validator = Validator::make($request->all(), [
    'photos.profile' => 'required|image',
]);

$validator = Validator::make($request->all(), [
    'person.*.email' => 'email|unique:users',
    'person.*.first_name' => 'required_with:person.*.last_name',
]);

'custom' => [
    'person.*.email' => [
        'unique' => 'Each person must have a unique e-mail address',
    ]
],
```
###### 自定义验证规则
```
php artisan make:rule Uppercase

class Uppercase implements Rule`
{
    public function passes($attribute, $value)
    {
        return strtoupper($value) === $value;
    }
    
    public function message()
    {
        return 'The :attribute must be uppercase.';
    }
}
```
```
public function message()
{
    return trans('validation.uppercase');
}
```
直接使用
```
use App\Rules\Uppercase;

$request->validate([
    'name' => ['required', new Uppercase],
]);
```
闭包
```
$validator = Validator::make($request->all(), [
    'title' => [
        'required',
        'max:255',
        function ($attribute, $value, $fail) {
            if ($value === 'foo') {
                $fail($attribute.' is invalid.');
            }
        },
    ],
]);
```

注册再使用
```
# AppServiceProvider
public function boot()
    {
        Validator::extend('foo', function ($attribute, $value, $parameters, $validator) {
            return $value == 'foo';
        });
    }

Validator::extend('foo', 'FooValidator@validate');
```

自定义替换占位符
为错误消息定义自定义替换占位符
```
# "foo" => "Your input was invalid!",
# "accepted" => "The :attribute must be accepted.",
public function boot()
{
    Validator::extend(...);

    Validator::replacer('foo', function ($message, $attribute, $rule, $parameters) {
        return str_replace(...);
    });
}
```
隐藏验证

默认情况下，如果字段值为 null 或是空字符串，并且未设置规则含有 required，那么验证规则都将不再使用，也就是结果会被通过。
所以如果是必须的字段，那么一定要设置 required。通过下面，使得字段一定是 required 。
```
$rules = ['name' => 'unique:users,name'];
$input = ['name' => ''];
Validator::make($input, $rules)->passes(); // true
```
```
Validator::extendImplicit('foo', function ($attribute, $value, $parameters, $validator) {  
    return $value == 'foo';
});
```
> “隐式”扩展只意味着该属性是必需的。 它是否实际上使缺失或空属性失效取决于您。（TODO：不是太明白，也用不太到，暂时不管）
如果您希望在属性为空时运行规则对象，则应实现 `Illuminate\Contracts\Validation\ImplicitRule` 接口。该接口用作验证器的“标记接口”; 因此，它不包含您需要实现的任何方法。
## 十一、错误与日志

当您启动一个新的Laravel项目时，已经为您配置了错误和异常处理。`App\Exceptions\Handler` 类是记录应用程序触发的所有异常然后呈现给用户的位置。

```
public function report(Exception $exception)
{
    if ($exception instanceof CustomException) {
        //
    }

    parent::report($exception);
}

protected function context()
{
    return array_merge(parent::context(), [
        'foo' => 'bar',
    ]);
}

public function isValid($value)
{
    try {
        // Validate the value...
    } catch (Exception $e) {
        report($e);

        return false;
    }
}


# dontReport 不报告错误
protected $dontReport = [
    \Illuminate\Auth\AuthenticationException::class,
    \Illuminate\Auth\Access\AuthorizationException::class,
    \Symfony\Component\HttpKernel\Exception\HttpException::class,
    \Illuminate\Database\Eloquent\ModelNotFoundException::class,
    \Illuminate\Validation\ValidationException::class,
];


public function render($request, Exception $exception)
{
    if ($exception instanceof CustomException) {
        return response()->view('errors.custom', [], 500);
    }

    return parent::render($request, $exception);
}
```
```
<?php

namespace App\Exceptions;

use Exception;

class RenderException extends Exception
{
    public function report()
    {
        //
    }

    public function render($request)
    {
        return response(...);
    }
}
```
resources/views/errors/404.blade.php

<h2>{{ $exception->getMessage() }}</h2>
php artisan vendor:publish --tag=laravel-errors


`Handler.php`中的`render`方法
```
public function render($request, Exception $exception)
{
    if ($exception instanceof CustomException) {
        return response()->view('errors.custom', [], 500);
    }

    return parent::render($request, $exception);
}
```
自定义异常
```
php artisan make:exception TalkException
```
简单粗暴的 abort
```
abort(404);

abort(403, 'Unauthorized action.');     // <h2>{{ $exception->getMessage() }}</h2>
```


Laravel使用Monolog库，为各种强大的日志处理程序提供支持。

配置文件 config/logging.php

默认情况下，Monolog使用与当前环境匹配的“通道名称”进行实例化，例如生产或本地环境。 要更改此值，请为频道的配置添加名称选项：
```
'stack' => [
    'driver' => 'stack',
    'name' => 'channel-name',
    'channels' => ['single', 'slack'],
],
```

名称	|描述
----|----
stack|	一个便于创建『多通道』通道的包装器
single|	单个文件或者基于日志通道的路径 (StreamHandler)
daily	|一个每天轮换的基于 Monolog 驱动的 RotatingFileHandler
slack|	一个基于 Monolog 驱动的 SlackWebhookHandler
papertrail|	一个基于 Monolog 驱动的 SyslogUdpHandler 
syslog|	一个基于 Monolog 驱动的 SyslogHandler
errorlog|	一个基于 Monolog 驱动的 ErrorLogHandler
monolog	|一个可以使用任何支持 Monolog 处理程序的 Monolog 工厂驱动程序
custom	|一个调用指定工厂创建通道的驱动程序

配置 Single 和 Daily 通道
single 和 daily 通道包含三个可选配置项：bubble 、permission 和 locking.

名称	|描述|	默认值
---|---|---
bubble|	消息处理后，指示消息是否推送到其他通道|	true
permission|	日志文件权限|	644
locking|	写入之前尝试锁定日志文件|	false

stack 驱动允许你在单一日志通道中整合多个通道。让我们通过一个产品级应用的配置实例来看看如果使用日志堆栈：
```
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['syslog', 'slack'],
    ],

    'syslog' => [
        'driver' => 'syslog',
        'level' => 'debug',
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => 'Laravel Log',
        'emoji' => ':boom:',
        'level' => 'critical',
    ],
],
```
Log::info('User failed to login.', ['id' => $user->id]);

写入指定通道
Log::channel('slack')->info('Something happened!');

Log::stack(['single', 'slack'])->info('Something happened!');

```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
# 第五章：前端开发（一、Blade。二、本地化。三、前端脚手架。四、Laravel mix 。）
​```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```

## 一、Blade
```
@yield('title')

@section('sidebar')
	This is the master sidebar.
@show
```
```
// 继承  @endsection 指令仅定义了一个片段， @show 则在定义的同时 立即 yield 这个片段。
@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```
```
@yield('content', View::make('view.name'))
```
```
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    {{ $slot }}
</div>

@component('alert')
    <strong>Whoops!</strong> Something went wrong!
@endcomponent

@componentFirst(['custom.alert', 'alert'])
    <strong>Whoops!</strong> Something went wrong!
@endcomponent
```
```
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    <div class="alert-title">{{ $title }}</div>

    {{ $slot }}
</div>

@component('alert')
    @slot('title')
        Forbidden
    @endslot

    You are not allowed to access this resource!
@endcomponent

@component('alert', ['foo' => 'bar'])
    ...
@endcomponent
```

```
# AppServiceProvider boot
Blade::component('components.alert', 'alert');

@alert(['type' => 'danger'])
    You are not allowed to access this resource!
@endalert

@alert
    You are not allowed to access this resource!
@endalert
```
```
{!! $name !!}
```
```
<script>
    var app = @json($array);

    var app = @json($array, JSON_PRETTY_PRINT);
</script>
<example-component :some-prop='@json($array)'></example-component>
# 在元素属性中使用@json要求它用单引号括起来。
```
默认情况下，Blade（和Laravel e帮助程序）将对HTML实体进行双重编码。 
如果要禁用双重编码，请从AppServiceProvider的引导方法中调用Blade :: withoutDoubleEncoding方法：
```
    public function boot()
    {
        Blade::withoutDoubleEncoding();
    }
```
```
@{{ name }}

@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```
```
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif

@unless (Auth::check())
    You are not signed in.
@endunless

@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty

@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest

@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest

@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break

    @default
        Default case...
@endswitch

@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile

@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>

    @if ($user->number == 5)
        @break
    @endif
@endforeach

@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach

@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif

    <p>This is user {{ $user->id }}</p>
@endforeach

@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

#### section 有内容则生效
```
@hasSection('navigation')
    <div class="pull-right">
        @yield('navigation')
    </div>

    <div class="clearfix"></div>
@endif
```

属性	|描述
--|--
$loop->index	|当前迭代的索引（从 0 开始计数）。
$loop->iteration|	当前循环迭代 (从 1 开始计算）。
$loop->remaining|	循环中剩余迭代的数量。
$loop->count|	被迭代的数组元素的总数。
$loop->first|	是否为循环的第一次迭代。
$loop->last	|是否为循环的最后一次迭代。
$loop->even	|是否为循环的偶数次迭代。
$loop->odd	|是否为循环的奇数次迭代。
$loop->depth|	当前迭代的嵌套深度级数。
$loop->parent|	嵌套循环中，父循环的循环变量
```
{{-- This comment will not be present in the rendered HTML --}}

@php
    //
@endphp

@include('shared.errors')

@includeWhen($boolean, 'view.name', ['some' => 'data'])

@includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
```
```
# resources/views/includes/input.blade.php
<input type="{{ $type ?? 'text' }}">
Blade::include('includes.input', 'input');
```
```
@each('view.name', $jobs, 'job')
@each('view.name', $jobs, 'job', 'view.empty')
```
```
@push('scripts')
    <script src="/example.js"></script>
@endpush

@stack('scripts')
```
```
@push('scripts')
    This will be second...
@endpush

// Later...

@prepend('scripts')
    This will be first...
@endprepend
```
#### 拓展 Blade

```php
# class AppServiceProvider 中 boot 方法
Blade::directive('datetime', function ($expression) {
	return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
});
```

> 更新`Blade`指令的逻辑之后，您需要删除所有缓存的`Blade`视图(`php artisan view:clear`)。 
```html
<!-- some.blade.php -->
@datetime($var)
// <?php echo ($var)->format('m/d/Y H:i'); ?>
```

#### 自定义模版语法

```php
use Illuminate\Support\Facades\Blade;

public function boot()
{
    Blade::if('env', function ($environment) {
        return app()->environment($environment);
    });
}
```

```php
@env('local')
    // The application is in the local environment...
@else
    // The application is not in the local environment...
@endenv
```
#### 堆栈

```php
<!-- extend.blade.php -->
@push('scripts')
    <script src="/example.js"></script>
@endpush

/**  输出 start
<script src="/example.js"></script>
<script src="/example.js"></script>
     输出 end  **/
```
```
<!-- common.blade.php -->
@stack('scripts')
@stack('scripts')
```

#### 服务注入

```php
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
</div>
```
#### 继承

```html
<!-- layouts/app.blade.php -->
@yield('title')

@section('sidebar')
这是 master 的侧边栏。
@show
```

```HTML
<!-- child.blade.php -->
@extends('layouts.app')
  
@section('title',' Here is a div ')
// 等同于
@section('title') Here is a div @endsection
  
@section('sidebar')
    @parent
    <p>This is appended to the master sidebar.</p>
@endsection
```

#### slot

```html
<!-- alert.blade.php -->
<div class="alert alert-danger">
    {{ $slot }}
</div>
  
<!-- other.blade.php -->
@component('alert')
    <strong>哇！</strong> 出现了一些问题！
@endcomponent
```

```php
<!-- alert.blade.php -->
<div class="alert alert-danger">
    <div class="alert-title">{{ $title }}</div>
    {{ $slot }}
</div>
  
<!-- other.blade.php -->  
@component('alert')
    @slot('title')
        拒绝
    @endslot
    你没有权限访问这个资源！
@endcomponent

/**  输出 start
<div class="alert alert-danger">
    <div class="alert-title">拒绝</div>
    你没有权限访问这个资源！
</div>
     输出 end  **/
```

```php
@component('alert', ['foo' => 'bar'])
    ...
@endcomponent
# 所有的数据都将以变量的形式传递给组件模版 alert.blade.php
```

#### Blade & JavaScript 框架

```php
@{{ name }}
# 最终输出 {{ name }} ，供一些 js 框架使用。 
```

如果是大量的需要这么做，可以使用

```html
@verbatim
    <div class="container">
        Hello, {{ name }}.
    </div>
@endverbatim
```




#### 为集合数组渲染视图

```php+HTML
// 子视图使用 key 变量作为当前迭代的键名。
// 注意：通过 @each 不会从父视图继承变量。 如果子需要这些变量，应该使用 @foreach 和 @include。
@each('view.name', $jobs, 'job')
@each('view.name', $jobs, 'job', 'view.empty')
  
// 自己做的例子
<!-- alert.blade.php -->
<div class="alert alert-danger">
    {{ $aa }}
</div>

<!-- other.blade.php -->
@each('alert', ['aaa','bbb'], 'aa')

/********** 输出 *********
<div class="alert alert-danger">
    aaa
</div>
<div class="alert alert-danger">
    bbb
</div>
************************/
```

## 二、本地化

1. 实时设置本地化语言：`App::setLocale($locale);`
2. `config/app.php`中的`fallback_locale`选项是设置备用语言，当第一语言不可用，则使用这个。
3. `if (App::isLocale('en'))`确认当前语言。
4. 在`resource/lang`文件夹下可以自定义其他国家的语言，与已存在的文件相对应即可。
```
/resources
    /lang
        /en
            messages.php
        /es
            messages.php
			
return [
    'welcome' => 'Welcome to our application'
];
```
#### 长语句翻译

如果有大量东西需要翻译，你会发现如果用短字符串或者单词，容易看蒙了，这时候可以使用长的语句去翻译。例如建立文件`resources/lang/es.json`：
```
{
    "I love programming.": "Me encanta programar."
}
```
#### 取出翻译
```
echo __('messages.welcome');       // 如果不存在，返回 messages.welcome
echo __('I love programming.');

{{ __('messages.welcome') }}
@lang('messages.welcome')

echo __('messages.welcome', ['name' => 'dayle']);    // 'welcome' => 'Welcome, :name',

'welcome' => 'Welcome, :NAME', // Welcome, DAYLE
'goodbye' => 'Goodbye, :Name', // Goodbye, Dayle

'apples' => 'There is one apple|There are many apples',

'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',

echo trans_choice('messages.apples', 10);          

'minutes_ago' => '{1} :value minute ago|[2,*] :value minutes ago',
echo trans_choice('time.minutes_ago', 5, ['value' => 5]);

'apples' => '{0} There are none|{1} There is one|[2,*] There are :count',
```
#### 包语言覆盖

建立对应文件：`resources/lang/vendor/{package}/{locale}`。
举个栗子， `skyrim/hearthfire`覆盖`messages.php`翻译，你应该建立文件`resources/lang/vendor/hearthfire/en/messages.php`。

## 三、前端脚手架
如果不需要 vue 和 bootstrap：`php artisan preset none`。

写 css

在编译 css 之前，请安装前端依赖
```
npm install
```
`npm run dev`命令会处理`webpack.mix.js`文件中的指令。编译的 css 文件将放在目录`public/css`中。
```
npm run dev
```
写 js

安装依赖

```
npm install
```
编译
```
npm run dev
```

编写 vue 组件

```
Vue.component(
    'example-component',
    require('./components/ExampleComponent.vue')
);
```
使用组件
```
@extends('layouts.app')

@section('content')
    <example-component></example-component>
@endsection
```
使用 react
```
php artisan preset react
```
## 四、Laravel mix

#### 安装与配置
安装 node 和 npm。
```
node -v
npm -v
```
**Laravel Mix**

安装 mix
```
npm install
```

#### 运行 mix

```
// Run all Mix tasks...
npm run dev

// Run all Mix tasks and minify output...
npm run production
```

**监听运行 mix**

```
npm run watch
```
如果无效请尝试以下命令：
```
npm run watch-poll
```

#### 与样式一起编译

**Less**

less 方法编译 less 到 css。

```
mix.less('resources/assets/less/app.less', 'public/css');
```
```
mix.less('resources/assets/less/app.less', 'public/css')
   .less('resources/assets/less/admin.less', 'public/css');
```
指定文件名
```
mix.less('resources/assets/less/app.less', 'public/stylesheets/styles.css');
```
设置 less 选项
```
mix.less('resources/assets/less/app.less', 'public/css', {
    strictMath: true
});
```

**Sass**

```
mix.sass('resources/assets/sass/app.scss', 'public/css');
```
```
mix.sass('resources/assets/sass/app.sass', 'public/css')
   .sass('resources/assets/sass/admin.sass', 'public/css/admin');
```
```
mix.sass('resources/assets/sass/app.sass', 'public/css', {
    precision: 5
});
```
**Stylus**

```
mix.stylus('resources/assets/stylus/app.styl', 'public/css');
```
安装插件
```
# npm install rupture

mix.stylus('resources/assets/stylus/app.styl', 'public/css', {
    use: [
        require('rupture')()
    ]
});
```

**PostCSS**

```
mix.sass('resources/assets/sass/app.scss', 'public/css')
   .options({
        postCss: [
            require('postcss-css-variables')()
        ]
   });
```

**原生 CSS**
```
mix.styles([
    'public/css/vendor/normalize.css',
    'public/css/vendor/videojs.css'
], 'public/css/all.css');
```

**URL 处理**
```
.example {
    background: url('../images/example.png');
}
```


默认情况下， Laravel Mix 和 Webpack 会找到 `example.png`, 拷贝到 `public/images`，然后改写`url()`，变成这样子:
```
.example {
  background: url(/images/example.png?d41d8cd98f00b204e9800998ecf8427e);
}
```
> `url('/images/thing.png')` 或者 `url('http://example.com/images/thing.png')` 不会变.

如果你的目录结构不是默认那样，可以禁用重写`url()`
```
mix.sass('resources/assets/app/app.scss', 'public/css')
   .options({
      processCssUrls: false
   });
```
禁用后，原来是啥样，现在还是啥样
```
.example {
    background: url("../images/thing.png");
}
```

**Source Maps**

启用 source maps
```
mix.js('resources/assets/js/app.js', 'public/js')
   .sourceMaps();
```

#### 与 js 一起编译
```
mix.js('resources/assets/js/app.js', 'public/js');
```
上面一句代码就支持了以下功能
1. ES 2015 语法
2. 模块
3. 编译 .vue 文件
4. 生产环境压缩代码

**依赖提取**
如果将依赖绑定你的 js。这样你更新一点点代码就要使得客户端重新下载你的依赖。这时候可以使用依赖提取方法`extract`。
```
mix.js('resources/assets/js/app.js', 'public/js')
   .extract(['vue'])
```
运行后生成以下文件
*   `public/js/manifest.js`: *The Webpack 运行时的 manifest 文件*
*   `public/js/vendor.js`: *你的依赖*
*   `public/js/app.js`: *你的 js*
> 引用顺序不能出错。

**React**

Mix 可以自动安装 Babel 插件来支持 React。为了使用 React，你只需将 mix.js() 的调用替换成 mix.react() 即可：
```
mix.react('resources/assets/js/app.jsx', 'public/js');
```

**原生 JS**

类似使用 mix.styles() 来合并多个样式表一样，你也可以使用 scripts() 方法来合并并压缩多个 JavaScript 文件：
```
mix.scripts([
    'public/js/admin.js',
    'public/js/dashboard.js'
], 'public/js/all.js');
```

> `mix.babel()`和`mix.scripts()`有点不同，那就是`mix.babel()`会将所有 ES2015 的代码转换为所有浏览器都能识别的原生 JavaScript。

#### 自定义 webpack 配置

两种方式

**合并定制配置**
```
mix.webpackConfig({
    resolve: {
        modules: [
            path.resolve(__dirname, 'vendor/laravel/spark/resources/assets/js')
        ]
    }
});
```
**定制配置文件**

复制`node_modules/laravel-mix/setup/webpack.config.js`到你的根目录。然后修改`package.json`文件中的`--config`参数。采用这种方法进行自定义，如果后续有更新时，需要手动合并到你的自定义文件中。

#### 拷贝文件和目录
```
mix.copy('node_modules/foo/bar.css', 'public/css/bar.css');
```
拷贝目录使用 copyDirectory 防止扁平化目录结构（目录分别都是从属关系）
```
mix.copyDirectory('assets/img', 'public/img');
```

#### 版本控制以清除缓存
```
mix.js('resources/assets/js/app.js', 'public/js')
   .version();
```
获取文件名
```
<link rel="stylesheet" href="{{ mix('/css/app.css') }}">
```

指定版本控制只适用于生产环境

 `npm run production`:

```
mix.js('resources/assets/js/app.js', 'public/js');

if (mix.inProduction()) {
    mix.version();
}
```

#### Browsersync 重新加载
[BrowserSync](https://browsersync.io/) 可以自动监控你的文件变化，并将更改注入浏览器，而无需手动刷新。你可以通过调用 `mix.browserSync()` 方法来启用这个功能的支持：
```
mix.browserSync('my-domain.test');

// Or...

// https://browsersync.io/docs/options
mix.browserSync({
    proxy: 'my-domain.test'
});
```
你可以将字符串 (代理) 或者对象 (BrowserSync 设置) 传给这个方法。再使用 npm run watch 命令来开启 Webpack 的开发服务器。现在，当你修改脚本或者 PHP 文件时，浏览器会即时刷新页面以响应你的更改。

#### 环境变量
在`.env`文件中设置
```
MIX_SENTRY_DSN_PUBLIC=http://example.com
```
> 如果 npm run watch 下更改了值，需要重新 npm run watch
```
process.env.MIX_SENTRY_DSN_PUBLIC
```

#### 通知
默认会进行系统通知编译情况，如果不希望在生产服务器上通知：
```
mix.disableNotifications();
```

```
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
# 第六章：安全（一、认证。二、api 认证。三、授权。四、加密。五、哈希。六、重置密码。七、邮箱验证。）
```
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
## 一、认证
#### 简介
认证系统主要由两部分组成：
```
guards
每个请求中，怎么认证用户
例如：session guard 利用 session 和 cookie 来认证

providers
怎样从持久化存储中获取用户
例如：通过 Eloquent, DB
```
一般来说，不用修改默认的认证配置
> 使用数据库认证时候，并且使用内置的认证机制时：
> 1. 必须设置密码字段大于等于 60 个字符长。
> 2. remember_token 必须100个字符长以上，且可为空。
#### 认证快速开始
```
1. composer require laravel/ui
2. php artisan ui vue --auth
3. php artisan migrate
4. 访问 http://your-app.dev/register

如果不需要注册，可以路由中指定下，`Auth::routes(['register' => false]);`。
```
修改跳转地址
```php
// LoginController,  RegisterController, ResetPasswordController, and  VerificationController
protected $redirectTo = '/';
```
```php
# 方法的优先级高于属性定义
protected function redirectTo()
{
	// 可以写一些逻辑
    return '/path';
}
```
认证字段修改
```
public function username(){
    return 'username';
}
```
使用其他的 guard
```
# LoginController, RegisterController, ResetPasswordController
use Illuminate\Support\Facades\Auth;
# 返回的应该是 a guard instance
protected function guard()
{
    return Auth::guard('guard-name');
}
```
取出认证信息
```
$user = Auth::user();
$id = Auth::id();
if (Auth::check()) // 最好使用中间件！

$request->user()
```
路由认证
```
Route::get('profile', function () {
    // Only authenticated users may enter...
})->middleware('auth');

public function __construct()
{
    $this->middleware('auth');
}

# 指定 guard
->middleware('auth:api');
```
如果登录失败次数过多，会禁止登录一段时间。
判断的标准是 username 方法返回值和 ip 。
#### 手动认证用户
```php
# 当你不喜欢自带的控制器去认证用户，你可以移除这些控制器，
# 引入 Auth facade，利用 attempt 手动认证
class LoginController extends Controller
{
    public function authenticate(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials)) {
            // Authentication passed...
            return redirect()->intended('dashboard');
        }
    }
}

if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // 字段 active 必须是 1
}

if (Auth::guard('admin')->attempt($credentials)) {
    // 指定 guard
}
```
登出
```
Auth::logout();
```
记住用户 （无限期）
```
# $remember 是个 bool 值
if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
    // The user is being remembered... 内置的 LoginController 已经实现 remember
}

# 判断是否选择了记住用户
if (Auth::viaRemember())
```
```
Auth::login($user);
Auth::login($user, true);  // 记住
Auth::guard('admin')->login($user);
Auth::loginUsingId(1);
Auth::loginUsingId(1, true);
Auth::once($credentials); // 临时认证，无状态的。

```



#### 无登录页面, 利用弹窗请求认证用户

```
Route::get('profile', function(){
    // ...
})->middleware('auth.basic');

# 如果使用 php fastcgi，可能会失效，可以在 .htaccess 里面加入
RewriteCond %{HTTP:Authorization} ^(.+)$
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]
```
利用中间件
```
namespace App\Http\Middleware;

use Illuminate\Support\Facades\Auth;

class AuthenticateOnceWithBasicAuth
{
    public function handle($request, $next)
    {
        return Auth::onceBasic() ?: $next($request);     // Auth::onceBasic() 无返回值就代表认证成功
    }

}
# 注册中间件后 ->middleware('auth.basic.once');
```
#### 登出
`Auth::logout();`

```php
// 取消登陆在别的设备上的认证
// 取消注释：\Illuminate\Session\Middleware\AuthenticateSession::class,
Auth::logoutOtherDevices($password);
```
#### 社交登陆

#### 增加自定义 guard
```
# extend 方法
class AuthServiceProvider extends ServiceProvider
{
    public function boot()
    {
        $this->registerPolicies();

        Auth::extend('jwt', function ($app, $name, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\Guard...

            return new JwtGuard(Auth::createUserProvider($config['provider']));
        });
    }
}
```
```
'guards' => [
    'api' => [
        'driver' => 'jwt',
        'provider' => 'users',
    ],
],
```
闭包请求 guard
```
public function boot()
{
    $this->registerPolicies();

    Auth::viaRequest('custom-token', function ($request) {
        return User::where('token', $request->token)->first();
    });
}

'guards' => [
    'api' => [
        'driver' => 'custom-token',
    ],
],
```
#### 增加自定义 provider
```
# extend 方法
class AuthServiceProvider extends ServiceProvider
{
    public function boot()
    {
        $this->registerPolicies();

        Auth::provider('riak', function ($app, array $config) {
            // Return an instance of Illuminate\Contracts\Auth\UserProvider...

            return new RiakUserProvider($app->make('riak.connection'));
        });
    }
}
```
```
'providers' => [
    'users' => [
        'driver' => 'riak',
    ],
],

'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],
],
```
```
namespace Illuminate\Contracts\Auth;

interface UserProvider {

    public function retrieveById($identifier);
    public function retrieveByToken($identifier, $token);
    public function updateRememberToken(Authenticatable $user, $token);
    public function retrieveByCredentials(array $credentials);
    public function validateCredentials(Authenticatable $user, array $credentials);

}



namespace Illuminate\Contracts\Auth;

interface Authenticatable {

    public function getAuthIdentifierName();
    public function getAuthIdentifier();
    public function getAuthPassword();
    public function getRememberToken();
    public function setRememberToken($value);
    public function getRememberTokenName();

}
```
#### 认证事件
```
protected $listen = [
    'Illuminate\Auth\Events\Registered' => [
        'App\Listeners\LogRegisteredUser',
    ],

    'Illuminate\Auth\Events\Attempting' => [
        'App\Listeners\LogAuthenticationAttempt',
    ],

    'Illuminate\Auth\Events\Authenticated' => [
        'App\Listeners\LogAuthenticated',
    ],

    'Illuminate\Auth\Events\Login' => [
        'App\Listeners\LogSuccessfulLogin',
    ],

    'Illuminate\Auth\Events\Failed' => [
        'App\Listeners\LogFailedLogin',
    ],

    'Illuminate\Auth\Events\Logout' => [
        'App\Listeners\LogSuccessfulLogout',
    ],

    'Illuminate\Auth\Events\Lockout' => [
        'App\Listeners\LogLockout',
    ],

    'Illuminate\Auth\Events\PasswordReset' => [
        'App\Listeners\LogPasswordReset',
    ],
];
```


## 二、api 认证
#### 简介
Laravel 提供了简单的 api 认证，更推荐你使用 Laravel Passport。
#### 配置
```php
Schema::table('users', function ($table) {
    $table->string('api_token', 80)->after('password')
                        ->unique()
                        ->nullable()
                        ->default(null);
});
# php artisan migrate
# 如果修改字段名称 api_token，请记得改配置文件 config/auth.php 中的 stroage_key
```
#### 生成 token
```php
protected function create(array $data)
{
    return User::create([
        'name' => $data['name'],
        'email' => $data['email'],
        'password' => Hash::make($data['password']),
        'api_token' => Str::random(60),
    ]);
}
```
hash token
```php

'api' => [
    'driver' => 'token',
    'provider' => 'users',
    'hash' => true,   // hash your API tokens using SHA-256 hashing
],
```
hash token 不应该在注册时候生成。您需要在应用程序中实现自己的 API token 管理页面。 此页面应允许用户初始化和刷新其 API 令牌。 当用户发出初始化或刷新其令牌的请求时，您应该在数据库中存储令牌的哈希副本，并将令牌的纯文本副本返回到视图/前端客户端以进行一次性显示。
```php
class ApiTokenController extends Controller
{
    public function update(Request $request)
    {
        $token = Str::random(60);

        $request->user()->forceFill([
            'api_token' => hash('sha256', $token),
        ])->save();

        return ['token' => $token];
    }
}
// 由于上面示例中的API令牌具有足够的熵，因此创建“彩虹表”以查找散列令牌的原始值是不切实际的。 因此，不需要诸如bcrypt的慢速散列方法。
```
#### 保护路由
`middleware('auth:api')`
#### 给 Request 传 token
```
$response = $client->request('GET', '/api/user?api_token='.$token);

$response = $client->request('POST', '/api/user', [
    'headers' => [
        'Accept' => 'application/json',
    ],
    'form_params' => [
        'api_token' => $token,
    ],
]);

$response = $client->request('POST', '/api/user', [
    'headers' => [
        'Authorization' => 'Bearer '.$token,
        'Accept' => 'application/json',
    ],
]);
```
## 三、授权
#### Gates 和 Policies 异同点
一般情况下，可以替换使用这两者进行认证。相比较而言，Gates 一般是个闭包，简单的逻辑。而 Policies 可能会涉及到模型或者资源。

#### Gates 定义
```php
# App\Providers\AuthServiceProvider
public function boot()
{
    $this->registerPolicies();

    Gate::define('edit-settings', function ($user) {
        return $user->isAdmin;
    });

    Gate::define('update-post', function ($user, $post) {
        return $user->id == $post->user_id;
    });
}

public function boot()
{
    $this->registerPolicies();

    Gate::define('update-post', 'App\Policies\PostPolicy@update');
}
```
#### Gates 使用
```php
if (Gate::allows('edit-setting')) 

if (Gate::allows('update-post', $post)) 

if (Gate::denies('update-post', $post))

if (Gate::forUser($user)->allows('update-post', $post))

if (Gate::forUser($user)->denies('update-post', $post))

// allows, denies, check, any,  none, authorize, can, cannot
// Blade: @can,  @cannot, @canany
if (Gate::any(['update-post', 'delete-post'], $post)) 
   
if (Gate::none(['update-post', 'delete-post'], $post))

Gate::authorize('update-post', $post);  // Illuminate\Auth\Access\AuthorizationException
```

```php
Gate::define('create-post', function ($user, $category, $extraFlag) {
    return $category->group > 3 && $extraFlag === true;
});

if (Gate::check('create-post', [$category, $extraFlag]))
```
#### Gate Chencks
```php
Gate::before(function ($user, $ability) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});

Gate::after(function ($user, $ability, $result, $arguments) {
    if ($user->isSuperAdmin()) {
        return true;
    }
});
```
#### Gate Responses
```php
Gate::define('edit-settings', function ($user) {
    return $user->isAdmin
                ? Response::allow()
                : Response::deny('You must be a super administrator.');
                

$response = Gate::inspect('edit-settings', $post);

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}
```
#### Policies 定义
```php
php artisan make:policy PostPolicy
php artisan make:policy PostPolicy --model=Post

# AuthServiceProvider 中注册
use App\Policies\PostPolicy;
use App\Post;
class AuthServiceProvider extends ServiceProvider
{
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

    public function boot()
    {
        $this->registerPolicies();
    }
}

# Policy 自动发现，在 app/Policies 下，请 User，在 app 下，请 UserPolicy。如果想修改发现 policy 逻辑，在 AuthServiceProvider 中的 boot:
Gate::guessPolicyNamesUsing(function ($modelClass) {
    // return policy class name...
}); #在 AuthServiceProvider 中显式映射的任何策略都将优先于任何潜在的自动发现策略。

# PostPolicy
public function update(User $user, Post $post)
{
    return $user->id === $post->user_id;
}

public function create(User $user)
{
    //
}

public function update(User $user, Post $post)
{
    return $user->id === $post->user_id
                ? Response::allow()
                : Response::deny('You do not own this post.');
                // Illuminate\Auth\Access\Response
}

// 通过声明“可选”类型提示或为用户参数定义提供null默认值来允许这些授权检查传递到您的门和策略
public function update(?User $user, Post $post)
{
	return $user->id === $post->user_id;
}


$response = Gate::inspect('update', $post);

if ($response->allowed()) {
    // The action is authorized...
} else {
    echo $response->message();
}


Gate::authorize('update', $post);
```
```php
# 在最前端认证，一般是管理员在这里认证
# 如果您要拒绝用户的所有授权，则应从before方法返回false。 如果返回null，则授权将落入策略方法。
# 如果未能找到和被检测的能力的名称对应的 policy 方法名， before 方法不被调用
public function before($user, $ability): bool
{
    if ($user->isSuperAdmin()) {
        return true;
    }
}

```
#### Policies 使用

```php
if ($user->can('update', $post)) {
    //
}
# 如果给定的模型 即 $post 已经注册了 policy ，会调用合适的 policy，
# 如果没有注册，会尝试查找和操作名称相匹配的 Gate 。

# 如果是只有一个参数，如 create 操作
use App\Post;
if ($user->can('create', Post::class)) {
    // Executes the "create" method on the relevant policy...
}



# 通过中间件
use App\Post;
Route::put('/post/{post}', function (Post $post) {
    // The current user may update the post...
})->middleware('can:update,post');   // 第二个参数是 post 路由参数
// 认证失败返回 403

# 如果是只有一个参数，如 create 操作
Route::post('/post', function () {
    // The current user may create posts...
})->middleware('can:create,App\Post');



# 在控制器中
public function update(Request $request, Post $post)
{
    $this->authorize('update', $post);
    // The current user can update the blog post...
    // 授权失败返回 403
}
# 如果是只有一个参数，如 create 操作
public function create(Request $request)
{
    $this->authorize('create', Post::class);
}

$this->authorizeResource(Post::class, 'post');
```

Controller Method|	Policy Method
--|--
index|	viewAny
show|	view
create|	create
store	|create
edit	|update
update|	update
destroy	|delete
```php

# 在 blade 中使用
```
```html
@can('update', $post)
    <!-- The Current User Can Update The Post -->
@elsecan('create', App\Post::class)
    <!-- The Current User Can Create New Post -->
@endcan

@cannot('update', $post)
    <!-- The Current User Can't Update The Post -->
@elsecannot('create', App\Post::class)
    <!-- The Current User Can't Create New Post -->
@endcannot
# 类似于
@if (Auth::user()->can('update', $post))
    <!-- The Current User Can Update The Post -->
@endif

@unless (Auth::user()->can('update', $post))
    <!-- The Current User Can't Update The Post -->
@endunless

@canany(['update', 'view', 'delete'], $post)
    // The current user can update, view, or delete the post
@elsecanany(['create'], \App\Post::class)
    // The current user can create a post
@endcanany

@can('create', App\Post::class)
    <!-- The Current User Can Create Posts -->
@endcan
# 如果是只有一个参数，如 create 操作
@cannot('create', App\Post::class)
    <!-- The Current User Can't Create Posts -->
@endcannot
```

#### 额外的上下文
```php
# PostPolicy
public function update(User $user, Post $post, int $category)
{
    return $user->id === $post->user_id && 
           $category->group > 3;
}

# controller 
public function update(Request $request, Post $post)
{
    $this->authorize('update', [$post, $request->input('category')]);

    // The current user can update the blog post...
}
```

## 四、加密
```php
php artisan key:generate


encrypt($yourString)   // 这是内部序列化后再加密
decrypt($yourString) 
try {
    $decrypted = decrypt($encryptedValue);   
    // 如果加密值被修改，或非法值传入，抛出异常 DecryptException
} catch (DecryptException $e) {
    //
}


# 如果不需要序列化加解密
$encrypted = Crypt::encryptString('Hello world.');
$decrypted = Crypt::decryptString($encrypted);
```
## 五、哈希
config/hashing.php 默认 Bcrypt 和 Argon2 算法
Argon2i 需要 PHP 7.2.0 或更高级的版本，Argon2id 需要 PHP 7.3.0 或更高级的版本。
利用了 Bcrypt，会随着硬件的加强而加强加密 hash。
```
$passwordIntoDB = Hash::make($request->newPassword);
// 等价于 bcrypt

// 修改 Bcrypt 因子
Hash::make('password', [
    'rounds' => 12
]);

// 修改 Argon2 因子
$hashed = Hash::make('password', [
    'memory' => 1024,
    'time' => 2,
    'threads' => 2,
]);

# 验证
if (Hash::check('plain-text', $hashedPassword)) 

# 验证是否需要重新加密
/**
The needsRehash function allows you to determine 
if the work factor used by the hasher has changed 
since the password was hashed.
这句话理解很久，不太明白它的意思。
since 这儿应该翻译成自从。
意思是自从密码哈希后，你可以利用这个函数确定哈希的哈希因子是否改变过，
以至于我们需要重新进行哈希。
*/

if (Hash::needsRehash($hashed)) {
    $hashed = Hash::make('plain-text');
}
```
## 六、重置密码
在使用 Laravel的密码重置功能之前，您的用户必须使用 Illuminate\Notifications\Notifiable trait。
重置密码 token 有效时长为一小时。可在`config/auth.php`中修改时长。

#### 开箱即用
```php
App\User 实现接口 Illuminate\Contracts\Auth\CanResetPassword 
php artisan migrate
composer require laravel/ui 
php artisan ui vue --auth
# 默认 token 一小时失效，可以修改失效时间，config/auth.php 中的 expire。
```
自定义密码 guard
```php
# ResetPasswordController
protected function guard()
{
    return Auth::guard('guard-name');
}
```

自定义密码 broker

在`auth.php`配置文件中，你可以配置多个密码 `brokers`，用于多个用户表上的密码重置。你可以通过重写`ForgotPasswordController`和 `ResetPasswordController`中的`brokers`方法来选择你想使用的自定义`brokers`：
```
# ForgotPasswordController 和 ResetPasswordController
protected function broker()
{
    return Password::broker('name');
}
```

自定义发送重置密码通知
```
# App\User
public function sendPasswordResetNotification($token)
{
    $this->notify(new ResetPasswordNotification($token));
}
```
## 七、邮箱验证
#### 入门
```php
php artisan migrate # 在数据库中添加字段 email_verified_at
# 实现接口
use Illuminate\Notifications\Notifiable;
use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable implements MustVerifyEmail
{
    use Notifiable;

    // ...
}
```
#### 路由
```php
Auth::routes(['verify' => true]);

->middleware('verified')
```
#### 验证邮箱之后
```php
# VerificationController
# 默认自动跳转到 /home
protected $redirectTo = '/dashboard';
```
#### 事件
```php
# Laravel 在电子邮件验证过程中发送事件。 您可以在 EventServiceProvider 中将侦听器附加到这些事件：
protected $listen = [
    'Illuminate\Auth\Events\Verified' => [
        'App\Listeners\LogVerifiedUser',
    ],
];
```
```
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```

# 第七章：深入挖掘（一、Artisan 命令行。二、广播。三、缓存。四、集合。五、事件。六、文件存储。七、辅助函数。八、通知。九、开发拓展包。十、队列。十一、任务计划。）
```
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
## 一、Artisan 命令行
#### 基础
```php
php artisan list
```
```php
php artisan help migrate
```
```php
php artisan tinker

# 您可以使用 vendor：publish 命令发布 Tinker 的配置文件：
php artisan vendor:publish --provider="Laravel\Tinker\TinkerServiceProvider"

# 您可以运行 clear-compiled，down，env，inspire，migrate，optimize 和 up 命令。 如果您想列出更多命令，可以将它们添加到 tinker.php 配置文件中的命令数组中：
'commands' => [
    // App\Console\Commands\ExampleCommand::class,
],

'dont_alias' => [
    App\User::class,
],
```
#### 编写一条命令
**第一种**
```
php artisan make:command SendEmails
```
```
<?php

namespace App\Console\Commands;

use App\User;
use App\DripEmailer;
use Illuminate\Console\Command;

class SendEmails extends Command
{
    protected $signature = 'email:send {user}';

    protected $description = 'Send drip e-mails to a user';

    public function __construct()
    {
        parent::__construct();
    }

    public function handle(DripEmailer $drip)
    {
        $drip->send(User::find($this->argument('user')));
    }
}
```
**第二种**
```
Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
});

Artisan::command('email:send {user}', function (DripEmailer $drip, $user) {
    $drip->send(User::find($user));
});

Artisan::command('build {project}', function ($project) {
    $this->info("Building {$project}!");
})->describe('Build the project');
```
#### 定义预期输入
```
email:send {user}
email:send {user?}
email:send {user=foo}
```
有 **两种** 选项，一种可以赋值，另一种不用赋值。
不用赋值的作为一个开关存在。
```
# protected $signature = 'email:send {user} {--queue}';
# php artisan email:send 1 --queue
```
必须赋值
```
# protected $signature = 'email:send {user} {--queue=}';       // 可以设置默认值 --queue=default
# php artisan email:send 1 --queue=value

email:send {user} {--queue=default}
```
设置选项的快捷方式
```
email:send {user} {--Q|queue}
```
如果想要参数接受数组
```
email:send {user*}
# php artisan email:send foo bar
```
如果需要选项接受数组
```
email:send {user} {--id=*}
php artisan email:send --id=1 --id=2
```
给选项或参数设置描述
```
protected $signature = 'email:send
                        {user : The ID of the user}
                        {--queue= : Whether the job should be queued}';
```
#### 命令输入与输出
获取输入，如果不存在，返回 null。
```
public function handle()
{
    $userId = $this->argument('user');
}

$arguments = $this->arguments();

$queueName = $this->option('queue');

$options = $this->options();
```
提示输入，confirm 默认为 false，输入 y 或 yes 为 true。
```
$name = $this->ask('What is your name?');

$password = $this->secret('What is the password?');

if ($this->confirm('Do you wish to continue?')) {
    //
}
```
```
$name = $this->anticipate('What is your name?', ['Taylor', 'Dayle']);
// 可以自己输入，不输入则随机其中一个

$name = $this->choice('What is your name?', ['Taylor', 'Dayle'], $defaultIndex);
// 只可以选择，第三个参数设置默认的，值为数组的键
```

#### 编写输出
颜色
```
# line, info, comment, question, error
$this->info('Display this on the screen');
```
表格
```
$headers = ['Name', 'Email'];

$users = App\User::all(['name', 'email'])->toArray();

$this->table($headers, $users);
```
进度条
```
$users = App\User::all();

$bar = $this->output->createProgressBar(count($users));

$bar->start();

foreach ($users as $user) {
    $this->performTask($user);

    $bar->advance();
}

$bar->finish();
```
#### 注册命令
由于在控制台内核文件中的 commands 方法调用了 load 方法，所以 app/Console/Commands 目录下的所有命令都将自动注册到 Artisan。 实际上，你可以自由地调用 load 方法来扫描 Artisan 命令的其他目录：
```
protected function commands()
{
    $this->load(__DIR__.'/Commands');
    $this->load(__DIR__.'/MoreCommands');

    // ...
}
```
手动注册命令
```
protected $commands = [
    Commands\SendEmails::class
];
```
#### 代码执行命令
```php
Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);
});

# 或者 Artisan::call('email:send 1 --queue=default');


Route::get('/foo', function () {
    Artisan::queue('email:send', [             // 后台执行，确保你已经配置了队列并运行了队列
        'user' => 1, '--queue' => 'default'
    ]);

    //
});

Artisan::queue('email:send', [
    'user' => 1, '--queue' => 'default'
])->onConnection('redis')->onQueue('commands');

Route::get('/foo', function () {
    $exitCode = Artisan::call('email:send', [
        'user' => 1, '--id' => [5, 13]
    ]);
});

$exitCode = Artisan::call('migrate:refresh', [
    '--force' => true,
]);
```
有时候你希望从现有的 Artisan 命令中调用其它命令。你可以使用 call 方法。

```
public function handle()
{
    $this->call('email:send', [
        'user' => 1, '--queue' => 'default'
    ]);

    //
}
```
如果要调用另一个控制台命令并阻止其所有输出，可以使用 callSilent 方法。 callSilent 和 call 方法用法一样：
```
$this->callSilent('email:send', [
    'user' => 1, '--queue' => 'default'
]);
```

## 二、广播
## 简介
#### 配置
所有关于事件广播的配置都保存在 config/broadcasting.php 配置文件中。 Laravel 自带了几个广播驱动： Pusher 、 Redis ， 和一个用于本地开发与调试的 log 驱动。另外，还有一个 null 驱动允许你完全关闭广播系统。

在对事件进行广播之前，你必须先注册 App\Providers\BroadcastServiceProvider 。对于一个新建的 Laravel 应用程序，你只需要在 config/app.php 配置文件的 providers 数组中取消对该提供者的注释即可。

Laravel Echo 需要访问当前会话的 CSRF 令牌。你应当验证你的应用程序的 head HTML 元素是否定义了包含 CSRF 令牌的 meta 标签：

`<meta name="csrf-token" content="{{ csrf_token() }}">`
#### 对驱动的要求
###### Pusher
`composer require pusher/pusher-php-server "~3.0"`
然后，你需要在 config/broadcasting.php 配置文件中配置你的 Pusher 证书。该文件中已经包含了一个 Pusher 示例配置，你可以快速地指定你的 Pusher key 、secret 和 application ID。 config/broadcasting.php 文件的 pusher 配置项同时也允许你指定 Pusher 支持的额外 options ，例如 cluster：
```php
'options' => [
    'cluster' => 'eu',
    'encrypted' => true
],
```
当 Pusher 和 Laravel Echo 一起使用时，你应该在 resources/assets/js/bootstrap.js 文件中实例化 Echo 对象时指定 pusher 作为所需要的 broadcaster ：
```php
import Echo from "laravel-echo"

window.Pusher = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key'
});
```

###### Redis
如果你使用 Redis 广播器，请安装 Predis 库：

`composer require predis/predis`
Redis 广播器会使用 Redis 的 发布 / 订阅 特性来广播消息；尽管如此，你仍需将它与能够从 Redis 接收消息的 WebSocket 服务器配对使用以便将消息广播到你的 WebSocket 频道上去。

当 Redis 广播器发布一个事件的时候，该事件会被发布到它指定的频道上去，传输的数据是一个采用 JSON 编码的字符串。该字符串包含了事件名、 data 数据和生成该事件 socket ID 的用户（如果可用的话）。

###### Socket.IO
`npm install --save socket.io-client`
```
import Echo from "laravel-echo"

window.io = require('socket.io-client');

window.Echo = new Echo({
    broadcaster: 'socket.io',
    host: window.location.hostname + ':6001'
});

#最后，你需要运行一个与 Laravel 兼容的 Socket.IO 服务器。 Laravel 官方并没有内置 Socket.IO 服务器实现；不过，可以选择一个由社区驱动维护的项目 tlaverdure/laravel-echo-server ，目前托管在 GitHub 。
```
在开始广播事件之前，你还需要配置和运行 队列监听器 。所有的事件广播都是通过队列任务来完成的，因此应用程序的响应时间不会受到明显影响。
## 概念综述
#### 使用示例程序
`event(new ShippingStatusUpdated($update));`
实现 ShouldBroadcast 接口
```php
namespace App\Events;

use Illuminate\Broadcasting\Channel;
use Illuminate\Queue\SerializesModels;
use Illuminate\Broadcasting\PrivateChannel;
use Illuminate\Broadcasting\PresenceChannel;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Contracts\Broadcasting\ShouldBroadcast;

class ShippingStatusUpdated implements ShouldBroadcast
{
    /**
     * 有关配送状态更新的信息。
     *
     * @var string
     */
    public $update;
}
```
ShouldBroadcast 接口要求事件定义一个 broadcastOn 方法。该方法负责指定事件被广播到哪些频道。我们希望只有订单的创建者能够看到状态的更新，所以我们要把该事件广播到与这个订单绑定的私有频道上去：
```php
public function broadcastOn()
{
    return new PrivateChannel('order.'.$this->update->order_id);
}
```
记住，用户只有在被授权之后才能监听私有频道。我们可以在 routes/channels.php 文件中定义频道的授权规则。
```php
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});
```
所有的授权回调接收当前被认证的用户作为第一个参数，任何额外的通配符参数作为后续参数。在本例中，我们使用 {orderId} 占位符来表示频道名称的 「ID」 部分是通配符。

接下来，就只剩下在 JavaScript 应用程序中监听事件了。我们可以通过 Laravel Echo 来实现。
```php
Echo.private(`order.${orderId}`)
    .listen('ShippingStatusUpdated', (e) => {
        console.log(e.update);
    });
```
## 定义广播事件
要告知 Laravel 一个给定的事件需要广播，只需要在事件类中实现 Illuminate\Contracts\Broadcasting\ShouldBroadcast 接口即可。该接口已被导入到所有由框架生成的事件类中，所以你可以很方便地将它添加到你自己的事件中。
ShouldBroadcast 接口要求你实现一个方法： broadcastOn 。 该方法返回一个频道或者一个频道数组，事件会被广播到这些频道。这些频道必须是 Channel 、PrivateChannel 或者 PresenceChannel 的实例。 Channel 代表任何用户都可以订阅的公开频道， 而 PrivateChannels 和 PresenceChannels 则代表需要 频道授权 的私有频道：
```php
class ServerCreated implements ShouldBroadcast
{
    use SerializesModels;

    public $user;

	public function __construct(User $user)
    {
        $this->user = $user;
    }

    public function broadcastOn()
    {
        return new PrivateChannel('user.'.$this->user->id);
    }
}
```
一旦事件被触发，一个 队列任务 会自动广播事件到你指定的广播驱动上。

#### 广播名称
```php
public function broadcastAs()
{
    return 'server.created';
}

# 如果你使用了 broadcastAs 方法来自定义广播名称，你应当确保在你注册监听器时加上一个 . 的前缀。这将指示 Echo 不要在事件之前添加应用程序的命名空间：

.listen('.server.created', function (e) {
    // ....
});
```
#### 广播数据
当一个事件被广播时，其所有的 public 属性都会自动序列化并作为事件有效载荷进行广播，这允许你在 JavaScript 应用程序中访问到事件所有的公有数据。
```php
# 返回一个你想要作为事件有效载荷进行广播的数据数组：
public function broadcastWith()
{
    return ['id' => $this->user->id];
}
```
#### 广播队列
`public $broadcastQueue = 'your-queue-name';`
```
# 如果你想使用 sync 队列而不是默认队列驱动来广播事件，你可以实现 ShouldBroadcastNow 接口而不是 ShouldBroadcast ：
use Illuminate\Contracts\Broadcasting\ShouldBroadcastNow;

class ShippingStatusUpdated implements ShouldBroadcastNow
{
    //
}
```
#### 广播条件
```php
public function broadcastWhen()
{
    return $this->value > 100;
}
```
## 授权频道

#### 定义授权路由
`Broadcast::routes();`
`Broadcast::routes($attributes);`
```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    authEndpoint: '/custom/endpoint/auth'
});
```
#### 定义授权回调
```php
Broadcast::channel('order.{orderId}', function ($user, $orderId) {
    return $user->id === Order::findOrNew($orderId)->user_id;
});

use App\Order;
Broadcast::channel('order.{order}', function ($user, Order $order) {
    return $user->id === $order->user_id;
});


Broadcast::channel('channel', function() {
    // ...
}, ['guards' => ['web', 'admin']])
```
#### 定义 channel 类
```php
php artisan make:channel OrderChannel

# routes/channels.php
use App\Broadcasting\OrderChannel;
Broadcast::channel('order.{order}', OrderChannel::class);

class OrderChannel
{
    public function __construct()
    {
        //
    }

    # join 方法将保存你通常放置在频道授权闭包中的相同逻辑。
    public function join(User $user, Order $order)
    {
        return $user->id === $order->user_id;
    }
}
```
## 广播事件
`event(new ShippingStatusUpdated($update));`
#### 只广播给他人
`broadcast(new ShippingStatusUpdated($update))->toOthers();`
为了更好地理解什么时候使用 toOthers 方法，让我们假设有一个任务列表的应用程序，用户可以通过输入任务名来新建任务。要新建任务，你的应用程序需要发起一个请求到一个 /task 路由，该路由会广播任务的创建，并返回新任务的 JSON 响应。当你的 JavaScript 应用程序从路由收到响应后，它会直接将新任务插入到任务列表中，就像这样：
```js
axios.post('/task', task)
    .then((response) => {
        this.tasks.push(response.data);
    });
```
然而，别忘了，我们还广播了任务的创建。如果你的 JavaScript 应用程序正在监听该事件以便添加任务至任务列表，任务列表中将出现重复的任务：一个来自路由响应，另一个来自广播。你可以通过使用 toOthers 方法告知广播器不要将事件广播到当前用户来解决这个问题。

toOthers 方法，必须使用 Illuminate\Broadcasting\InteractsWithSockets trait 。

配置
当你初始化 Laravel Echo 实例的时候，一个套接字 ID 会被分配到该连接。如果你使用了 Vue 和 Axios ，该套接字 ID 会自动地以 X-Socket-ID 头的方式添加到每一个传出请求中。那么，当你调用 toOthers 方法时，Laravel 会从请求头中取出套接字 ID ，并告知广播器不要广播任何消息到带有这个套接字 ID 的连接上。

手动配置 JavaScript 应用程序来发送 X-Socket-ID 请求头
`var socketId = Echo.socketId();`
## 接收广播
#### 安装 Laravel Echo
`npm install --save laravel-echo pusher-js`
```js
import Echo from "laravel-echo"

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-channels-key'
});

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    cluster: 'eu',
    encrypted: true
});
```
```js
const client = require('pusher-js');

window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    client: client
});
```

#### 对事件进行监听
```
Echo.channel('orders')
    .listen('OrderShipped', (e) => {
        console.log(e.order.name);
    });
```
```js
Echo.private('orders')
    .listen(...)
    .listen(...)
    .listen(...);
```
#### 退出频道
```js
Echo.leaveChannel('orders');

# 如果您想离开私有频道和在线频道，您可以调用 leave 方法：
Echo.leave('orders');
```
#### 命名空间
```js
window.Echo = new Echo({
    broadcaster: 'pusher',
    key: 'your-pusher-key',
    namespace: 'App.Other.Namespace'
});

你可以在使用 Echo 订阅事件的时候为事件类加上 . 前缀。这样就可以指定完全限定名称的类名了：
Echo.channel('orders')
    .listen('.Namespace.Event.Class', (e) => {
        //
    });
```
## Presence 频道
#### 授权 Presence 频道
所有的 presence 频道也是私有频道；因此，用户必须被 授权之后才能访问 。不过，在给 presence 频道定义授权回调函数时，如果一个用户已经加入了该频道，那么不应该返回 true ，而应该返回一个关于该用户信息的数组。

由授权回调函数返回的数据能够在你的 JavaScript 应用程序中被 presence 频道事件监听器所使用。如果用户没有被授权加入该 presence 频道，那么你应该返回 false 或者 null ：
```php
Broadcast::channel('chat.{roomId}', function ($user, $roomId) {
    if ($user->canJoinRoom($roomId)) {
        return ['id' => $user->id, 'name' => $user->name];
    }
});
```
#### 加入 Presence 频道
```
Echo.join(`chat.${roomId}`)
    .here((users) => {
        //
    })
    .joining((user) => {
        console.log(user.name);
    })
    .leaving((user) => {
        console.log(user.name);
    });
```
#### 广播到 Presence 频道
```
public function broadcastOn()
{
    return new PresenceChannel('room.'.$this->message->room_id);
}
```
```
broadcast(new NewMessage($message));

broadcast(new NewMessage($message))->toOthers();
```
```
Echo.join(`chat.${roomId}`)
    .here(...)
    .joining(...)
    .leaving(...)
    .listen('NewMessage', (e) => {
        //
    });
```
## 客户端事件
```
# 有时，你可能希望广播一个事件给其它已经连接的客户端，但不通知你的 Laravel 应用程序。这在处理「输入中」这类事情的通知时尤其有用，比如提醒你应用的用户，另一个用户正在给定屏幕上输入信息。
Echo.private('chat')
    .whisper('typing', {
        name: this.user.name
    });  // 广播
      
Echo.private('chat')
    .listenForWhisper('typing', (e) => {
        console.log(e.name);
    });  // 监听
```
## 消息通知
```
Echo.private(`App.User.${userId}`)
    .notification((notification) => {
        console.log(notification.type);
    });
// 一个针对 App.User.{id} 频道的授权回调函数已经包含在 Laravel 框架内置的 BroadcastServiceProvider 中了。
```
## 三、缓存
#### 配置
使用数据做驱动：
```php
Schema::create('cache', function ($table) {
    $table->string('key')->unique();
    $table->text('value');
    $table->integer('expiration');
});

# 或者 php artisan cache:table
```
使用 memcache 做驱动：
```
'memcached' => [
    [
        'host' => '127.0.0.1',
        'port' => 11211,
        'weight' => 100
    ],
],

或者
'memcached' => [
    [
        'host' => '/var/run/memcached/memcached.sock',
        'port' => 0,
        'weight' => 100
    ],
],
```
#### 缓存的使用
```php
Cache::has('key')		// 未设置 key 或者值是 null 时候，返回 null
Cache::get('key')       // 值是什么返回什么，未设置，返回 null
Cache::get('key', 'defult') 
Cache::increment('key') 
Cache::increment('key', 3)		// 如果 key 不存在，返回 3，并存储 key 为 3; 失败返回 false
Cache::decrement('key');
Cache::decrement('key', $amount);```
$value = Cache::store('file')->get('foo');  // 访问其他驱动
Cache::store('redis')->put('bar', 'baz', 600); // 10 Minutes
$value = Cache::get('key', function () {
    return DB::table(...)->get();   
});
$users = Cache::remember('users', $seconds, function () {
    return DB::table('users')->get();
});
$value = Cache::rememberForever('users', function () {
    return DB::table('users')->get();
});
$value = Cache::pull('key');  // 如果不存在 key，返回 null，删除 key
Cache::put('key', 'value', $seconds);
Cache::put('key', 'value');
Cache::put('key', 'value', now()->addMinutes(10));
Cache::add('key', 'value', 10); //  如果缓存存在了，就返回 false；如果不存在，则添加缓存，然后返回 true
Cache::forever('key', 'value');
Cache::forget('key');
Cache::put('key', 'value', 0);  // 删除
Cache::put('key', 'value', -5);  // 删除
Cache::flush(); # 清空缓存的方法并不会考虑缓存前缀，会将缓存中的所有内容删除。因此在清除与其它应用程序共享的缓存时，请慎重考虑。


```
###### 原子锁
```php
# 注意：要使用该特性，你的应用必须使用 memcached， dynamodb 或 redis 缓存驱动作为你应用的默认缓存驱动。此外，所有服务器必须与同一中央缓存服务器进行通信。
if ($lock->get()) {
    //获取锁定10秒...
    $lock->release();
}

Cache::lock('foo')->get(function () {
    // 获取无限期锁并自动释放...
});
```
```php
$lock = Cache::lock('foo', 10);
try {
    $lock->block(5);
    // 等待最多5秒后获取的锁...
} catch (LockTimeoutException $e) {
    // 无法获取锁...
} finally {
    optional($lock)->release();
}

Cache::lock('foo', 10)->block(5, function () {
    // 等待最多5秒后获取的锁...
});
```
有时，你希望在一个进程中获取锁并在另外一个进程中释放它。例如，你可以在 Web 请求期间获取锁，并希望在该请求触发的队列作业结束时释放锁。在这种情况下，你应该将锁的作用域「owner token」传递给队列作业，以便作业可以使用给定的 token 重新实例化锁：
```php
$podcast = Podcast::find($id);

$lock = Cache::lock('foo', 120);

if ($result = $lock->get()) {
    ProcessPodcast::dispatch($podcast, $lock->owner());
}

// Within ProcessPodcast Job...
Cache::restoreLock('foo', $this->owner)->release();

// If you would like to release a lock without respecting its current owner, you may use the forceRelease method:

Cache::lock('foo')->forceRelease();
```
###### Cache 辅助函数
```php
$value = cache('key');
cache(['key' => 'value'], $seconds);
cache(['key' => 'value'], now()->addMinutes(10));
cache()->remember('users', $seconds, function () {
    return DB::table('users')->get();
});
```
#### 缓存标记
```php
# file、database 驱动不支持
# 取的时候标签内容必须一致，不能多，不能少。
Cache::tags(['people', 'artists'])->put('John', 141, $seconds);
Cache::tags(['people', 'authors'])->put('Anne', 22, $seconds);
Cache::tags('authors')->flush(); // 只删除 authors 的缓存
Cache::tags(['people', 'authors'])->flush(); // 标签 people、authors、people 和 authors 都会被删除
dump(Cache::tags(['people', 'authors'])->get('Anne'));    // null
dump(Cache::tags(['people', 'artists'])->get('John'));    // '141'
```
#### 增加自定义缓存驱动
```php
<?php

namespace App\Extensions;

use Illuminate\Contracts\Cache\Store;

class MongoStore implements Store
{
    public function get($key) {}
    public function many(array $keys);
    public function put($key, $value, $seconds) {}
    public function putMany(array $values, $seconds);
    public function increment($key, $value = 1) {}
    public function decrement($key, $value = 1) {}
    public function forever($key, $value) {}
    public function forget($key) {}
    public function flush() {}
    public function getPrefix() {}
}

Cache::extend('mongo', function ($app) {
    return Cache::repository(new MongoStore);
});

// 注册
# CacheServiceProvider 
public function boot()
{
	Cache::extend('mongo', function ($app) {
		return Cache::repository(new MongoStore);
	});
}
```
#### 事件
```
protected $listen = [
    'Illuminate\Cache\Events\CacheHit' => [
        'App\Listeners\LogCacheHit',
    ],

    'Illuminate\Cache\Events\CacheMissed' => [
        'App\Listeners\LogCacheMissed',
    ],

    'Illuminate\Cache\Events\KeyForgotten' => [
        'App\Listeners\LogKeyForgotten',
    ],

    'Illuminate\Cache\Events\KeyWritten' => [
        'App\Listeners\LogKeyWritten',
    ],
];

```

## 四、集合

Eloquent 查询的返回值总是一个集合。
```php
$collection = collect(['taylor', 'abigail', null])->map(function ($name) {
    return strtoupper($name);
})
->reject(function ($name) {
    return empty($name);
});
```
#### 集合拓展
```
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function ($value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']
```

集合 api 详细用法：[集合 api 官方文档](https://laravel.com/docs/6.0/collections#available-methods)。
> toArray 会递归的把集合转化为数组，如果你想转化成原来的数组，请用 all 方法。
#### 高阶
// average,  avg, contains, each, every, filter, first, flatMap, groupBy, keyBy, map, max,  min
, partition, reject, some, sortBy, sortByDesc, sum, unique.
```php
$users = User::where('votes', '>', 500)->get();
$users->each->markAsVip();

$users = User::where('group', 'Development')->get();
return $users->sum->votes;
```
#### 懒集合
```
use App\LogEntry;
use Illuminate\Support\LazyCollection;

LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');

    while (($line = fgets($handle)) !== false) {
        yield $line;
    }
})->chunk(4)->map(function ($lines) {
    return LogEntry::fromLines($lines);
})->each(function (LogEntry $logEntry) {
    // Process the log entry...
});

```
```php
$users = App\User::all()->filter(function ($user) {
    return $user->id > 500;
});



$users = App\User::cursor()->filter(function ($user) {
    return $user->id > 500;
});

foreach ($users as $user) {
    echo $user->id;
}
```
```php
use Illuminate\Support\LazyCollection;

LazyCollection::make(function () {
    $handle = fopen('log.txt', 'r');

    while (($line = fgets($handle)) !== false) {
        yield $line;
    }
})l
```
## 五、事件
#### 注册事件和监听器
事件指的是某个操作，某个时间点，等等。监听器监听到事件的发生，会触发对应的方法。
```
# EventServiceProvider 
protected $listen = [
    'App\Events\OrderShipped' => [
        'App\Listeners\SendShipmentNotification',
    ],
];

# 也可以注册在 boot 方法里
Event::listen('event.name', function ($foo, $bar) {
      
});
# 通配符
Event::listen('event.*', function ($eventName, array $data) {
    //
});
```
根据你的注册，生成
```
php artisan event:generate
```
自动发现
```php
use App\Events\PodcastProcessed;

class SendPodcastProcessedNotification
{
    public function handle(PodcastProcessed $event)
    {
        //
    }
}
```
```php
# 需要配置了，才能自动发现
# EventServiceProvider
public function shouldDiscoverEvents()
{
    return true;
}

# 配置发现其他目录
protected function discoverEventsWithin()
{
    return [
        $this->app->path('Listeners'),
    ];
}
```
```
php artisan event:cache
php artisan event:clear
```


#### 定义事件
```
<?php

namespace App\Events;

use App\Order;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    use SerializesModels;

    public $order;

    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}
```
#### 定义监听器
```
<?php

namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    public function __construct()
    {
        //
    }

    public function handle(OrderShipped $event)
    {
        // Access the order using $event->order...
        // 想禁止冒泡，请 return false
    }
}
```
#### 队列化事件监听器
```
# 需要实现接口 ShouldQueue
# 用命令生成的自动加上 implement ShouldQueue
<?php

namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    //
}
```
在监听器中设置队列链接和名称
```
public $connection = 'sqs';
public $queue = 'listeners';
public $delay = 60;   // The time (seconds) before the job should be processed.
```
设置队列条件
```php
class RewardGiftCard implements ShouldQueue
{
    public function handle(OrderPlaced $event)
    {
        //
    }

	public function shouldQueue(OrderPlaced $event)
    {
        return $event->order->subtotal >= 5000;
    }
}
```
手动访问队列
```
# 如果想在监听器中 delete 和 release 任务，
# 需要使用 Illuminate\Queue\InteractsWithQueue trait。
public function handle(OrderShipped $event)
{
    if (true) {
        $this->release(30);
    }
}
```
处理失败的任务
```
# 超过尝试次数，依然异常，则调用监听器的 failed 方法
public function failed(OrderShipped $event, $exception)
{
    //
}
```
#### 分发事件
```
$order = Order::findOrFail($orderId);
event(new OrderShipped($order));
```
#### 事件订阅器
实现了在单个类中包含多个监听器。
```
<?php

namespace App\Listeners;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function handleUserLogin($event) {}

    /**
     * Handle user logout events.
     */
    public function handleUserLogout($event) {}

    /**
     * Register the listeners for the subscriber.
     *
     * @param  \Illuminate\Events\Dispatcher  $events
     */
    public function subscribe($events)
    {
        $events->listen(
            'Illuminate\Auth\Events\Login',
            'App\Listeners\UserEventSubscriber@handleUserLogin'
        );

        $events->listen(
            'Illuminate\Auth\Events\Logout',
            'App\Listeners\UserEventSubscriber@handleUserLogout'
        );
    }
}
```
注册事件订阅器
```
class EventServiceProvider extends ServiceProvider
{
    protected $listen = [
        //
    ];

    protected $subscribe = [
        'App\Listeners\UserEventSubscriber',
    ];
}
```


## 六、文件系统

#### 配置
```
php artisan storage:link
// echo asset('storage/file.txt');
```
配置文件 `config/filesystems.php`
```
Storage::disk('local')->put('file.txt', 'Contents');
// 存储于 storage/app/file.txt
```
修改权限
```
'local' => [
    'driver' => 'local',
    'root' => storage_path('app'),
    'permissions' => [
        'file' => [
            'public' => 0664,
            'private' => 0600,
        ],
        'dir' => [
            'public' => 0775,
            'private' => 0700,
        ],
    ],
],
```
SFTP 
```
composer require league/flysystem-sftp ~1.0
# Amazon S3: league/flysystem-aws-s3-v3 ~1.0

# 使用缓存适配器是提高性能的一个绝对必要条件。你需要一个额外的包：
league/flysystem-cached-adapter ~1.0

'ftp' => [
    'driver' => 'ftp',
    'host' => 'ftp.example.com',
    'username' => 'your-username',
    'password' => 'your-password',

    // Optional FTP Settings...
    // 'port' => 21,
    // 'root' => '',
    // 'passive' => true,
    // 'ssl' => true,
    // 'timeout' => 30,
],

'sftp' => [
    'driver' => 'sftp',
    'host' => 'example.com',
    'username' => 'your-username',
    'password' => 'your-password',

    // Settings for SSH key based authentication...
    // 'privateKey' => '/path/to/privateKey',
    // 'password' => 'encryption-password',

    // Optional SFTP Settings...
    // 'port' => 22,
    // 'root' => '',
    // 'timeout' => 30,
],
```
缓存
```php
's3' => [
    'driver' => 's3',

    // Other Disk Options...

    'cache' => [
        'store' => 'memcached',
        'expire' => 600,
        'prefix' => 'cache-prefix',
    ],
],
```
#### 获得文件
```
Storage::put('avatars/1', $fileContents);
Storage::disk('s3')->put('avatars/1', $fileContents);
$contents = Storage::get('file.jpg');
$exists = Storage::disk('s3')->exists('file.jpg');

return Storage::download('file.jpg');

return Storage::download('file.jpg', $name, $headers);

$url = Storage::url('file1.jpg');  // 获取相对链接

$url = Storage::temporaryUrl(
    'file.jpg', now()->addMinutes(5)
);

$url = Storage::temporaryUrl(
    'file.jpg',
    now()->addMinutes(5),
    ['ResponseContentType' => 'application/octet-stream'],
);

'public' => [
    'driver' => 'local',
    'root' => storage_path('app/public'),
    'url' => env('APP_URL').'/storage',
    'visibility' => 'public',
],

$size = Storage::size('file.jpg');

$time = Storage::lastModified('file.jpg');
```

#### 存文件
```
Storage::put('file.jpg', $contents);

Storage::put('file.jpg', $resource);

Storage::putFile('photos', new File('/path/to/photo'));

// Manually specify a file name...
Storage::putFileAs('photos', new File('/path/to/photo'), 'photo.jpg');

Storage::putFile('photos', new File('/path/to/photo'), 'public');
// 设置为可见

Storage::prepend('file.log', 'Prepended Text');
Storage::append('file.log', 'Appended Text');
Storage::copy('old/file1.jpg', 'new/file1.jpg');
Storage::move('old/file1.jpg', 'new/file1.jpg');

$path = $request->file('avatar')->store('avatars');
$path = Storage::putFile('avatars', $request->file('avatar'));

$path = $request->file('avatar')->storeAs(
    'avatars', $request->user()->id
);
$path = Storage::putFileAs(
    'avatars', $request->file('avatar'), $request->user()->id
);

$path = $request->file('avatar')->store(
    'avatars/'.$request->user()->id, 's3'
);

Storage::put('file.jpg', $contents, 'public');

$visibility = Storage::getVisibility('file.jpg');
Storage::setVisibility('file.jpg', 'public')
```
#### 删文件
```
Storage::delete('file.jpg');
Storage::delete(['file1.jpg', 'file2.jpg']);

Storage::disk('s3')->delete('folder_path/file_name.jpg');
```
#### 文件夹
获取所有文件
```
$files = Storage::files($directory);

$files = Storage::allFiles($directory);
```
获取所有文件夹
```
$directories = Storage::directories($directory);

// Recursive...
$directories = Storage::allDirectories($directory);
```
创建文件夹（包含子文件夹）
```
Storage::makeDirectory($directory);
```
删除文件夹（包含子文件夹和文件）
```
Storage::deleteDirectory($directory);
```
#### 定制文件系统
```
composer require spatie/flysystem-dropbox

<?php
namespace App\Providers;

use Storage;
use League\Flysystem\Filesystem;
use Illuminate\Support\ServiceProvider;
use Spatie\Dropbox\Client as DropboxClient;
use Spatie\FlysystemDropbox\DropboxAdapter;

class DropboxServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Storage::extend('dropbox', function ($app, $config) {
            $client = new DropboxClient(
                $config['authorization_token']
            );

            return new Filesystem(new DropboxAdapter($client));
        });
    }

    public function register()
    {
        //
    }
}
```
```php
'providers' => [
    // ...
    App\Providers\DropboxServiceProvider::class,
];
```

## 七、辅助函数
[辅助函数](https://laravel.com/docs/6.0/helpers)
## 八、邮件
#### 生成邮件
```
php artisan make:mail OrderShipped
```
#### 编写邮件
在 build 方法中， from, subject, view,  attach。
**from**
1. 在 build 方法中
```
public function build()
{
    return $this->from('example@example.com')
                ->view('emails.orders.shipped');
}
```
2. 全局
```
# config/mail.php
'from' => ['address' => 'example@example.com', 'name' => 'App Name'],

'reply_to' => ['address' => 'example@example.com', 'name' => 'App Name'],
```
**view**
```
return $this->view('emails.orders.shipped');
return $this->text('emails.orders.shipped_plain');    // 纯文本
return $this->view('emails.orders.shipped')
                ->text('emails.orders.shipped_plain');    // 结合
```
给邮件模板传值
1. 全局属性自动传值
```
public function __construct(Order $order)
{
    $this->order = $order;
}

public function build()
{
    return $this->view('emails.orders.shipped');
}

// 使用
<div>
    Price: {{ $order->price }}
</div>
```
2. with 方法
```
# $order 只有属性是全局属性才能自动传值。否则只能通过 with 方法。
public function build()
{
    return $this->view('emails.orders.shipped')
        ->with([
            'orderName' => $this->order->name,
            'orderPrice' => $this->order->price,
        ]);
}

// 使用
<div>
    Price: {{ $orderPrice }}
</div>
```
**附件**
```
return $this->view('emails.orders.shipped')
                    ->attach('/path/to/file');

return $this->view('emails.orders.shipped')
                    ->attach('/path/to/file', [
                        'as' => 'name.pdf',     // 指定名称
                        'mime' => 'application/pdf',
                    ]); 
                    
return $this->view('email.orders.shipped')
               ->attachFromStorage('/path/to/file');
               
return $this->view('email.orders.shipped')
               ->attachFromStorage('/path/to/file', 'name.pdf', [
                   'mime' => 'application/pdf'
               ]);
               
return $this->view('email.orders.shipped')
               ->attachFromStorageDisk('s3', '/path/to/file');
```
附件是源数据，例如：内存中的 pdf
```
return $this->view('emails.orders.shipped')
                    ->attachData($this->pdf, 'name.pdf', [
                        'mime' => 'application/pdf',
                    ]);
```
内联附件
```
<body>
    Here is an image:
    <img src="{{ $message->embed($pathToFile) }}">
</body>
```
> $message 在 markdown 中不可用

内联嵌入源数据附件
`$message` 变量在纯文本邮件中不可用，因为纯文本邮件不使用内联附件。
```
<body>
    Here is an image from raw data:
    <img src="{{ $message->embedData($data, $name) }}">
</body>
```
**自定义 SwiftMailer 消息**
```
public function build()
{
    $this->view('emails.orders.shipped');

    $this->withSwiftMessage(function ($message) {
        $message->getHeaders()
                ->addTextHeader('Custom-Header', 'HeaderValue');
    });
}
```
#### markdown 邮件
```
php artisan make:mail OrderShipped --markdown=emails.orders.shipped
```
```
public function build()
{
    return $this->from('example@example.com')
                ->markdown('emails.orders.shipped');
}
```
```
@component('mail::message')
# Order Shipped

Your order has been shipped!

@component('mail::button', ['url' => $url])
View Order
@endcomponent

Thanks,<br>
{{ config('app.name') }}
@endcomponent

@component('mail::button', ['url' => $url, 'color' => 'success'])
View Order
@endcomponent

@component('mail::panel')
This is the panel content.
@endcomponent

@component('mail::table')
| Laravel       | Table         | Example  |
| ------------- |:-------------:| --------:|
| Col 2 is      | Centered      | $10      |
| Col 3 is      | Right-Aligned | $20      |
@endcomponent
```
自定义 markdown 的渲染
```
# 修改主题，可以在 theme 文件夹下新建 css，然后再配置文件中修改配置
php artisan vendor:publish --tag=laravel-mail
```
如果您想为 `Laravel` 的 `Markdown` 组件构建一个全新的主题，您可以在 `html/themes` 目录中放置一个 `CSS` 文件。 命名并保存 `CSS` 文件后，请更新邮件配置文件的主题选项以匹配新主题的名称。

要自定义单个可邮寄主题，可以将 `mailable` 类的 `$theme` 属性设置为发送可邮寄时应使用的主题名称。

#### 在浏览器中预览邮件
```
$invoice = App\Invoice::find(1);

return (new App\Mail\InvoicePaid($invoice))->render();


Route::get('mailable', function () {
    $invoice = App\Invoice::find(1);

    return new App\Mail\InvoicePaid($invoice);
});
```
#### 发送邮件
```
Mail::to($request->user())->send(new OrderShipped($order));

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->send(new OrderShipped($order));

# cc 抄送，bcc 暗抄送
```
队列化邮件
```
Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue(new OrderShipped($order));
```
延迟队列中的邮件
```
$when = now()->addMinutes(10);

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->later($when, new OrderShipped($order));
```
推送指定的队列
```
$message = (new OrderShipped($order))
                ->onConnection('sqs')
                ->onQueue('emails');

Mail::to($request->user())
    ->cc($moreUsers)
    ->bcc($evenMoreUsers)
    ->queue($message);
```
默认塞进队列（实现 ShouldQueue）
```
use Illuminate\Contracts\Queue\ShouldQueue;

class OrderShipped extends Mailable implements ShouldQueue
{
    //
}
```

#### 本地化邮件
```
Mail::to($request->user())->locale('es')->send(
    new OrderShipped($order)
);

use Illuminate\Contracts\Translation\HasLocalePreference;

class User extends Model implements HasLocalePreference
{
    public function preferredLocale()
    {
        return $this->locale;
    }
}

Mail::to($request->user())->send(new OrderShipped($order));
```
#### 邮件和本地开发
1. 在`config/mail.php`设置 to 选项，将开发时候的邮件都发给这个邮箱
```
'to' => [
    'address' => 'example@example.com',
    'name' => 'Example'
],
```
2. 设置邮件驱动为 log
#### 邮件事件
邮件在发送的前后生效，是发送邮件的时候，在队列中不算。
```
protected $listen = [
    'Illuminate\Mail\Events\MessageSending' => [
        'App\Listeners\LogSendingMessage',
    ],
    'Illuminate\Mail\Events\MessageSent' => [
        'App\Listeners\LogSentMessage',
    ],
];
```
## 八、通知
#### 介绍
Laravel 支持利用不同的传送渠道来发送通知，包括邮件、sms、slack 。

#### 创建通知
```php
php artisan make:notification InvoicePaid
```
#### 发送通知
**默认渠道 mail**

第一种方式
```php
use Notifiable;
# App\SomeModel
use App\Notifications\InvoicePaid;
$user->notify(new InvoicePaid($invoice));
```
第二种方式：优点是可以针对集合发送
```php
Notification::send($users, new InovicePaid($invoice));   
```

**指定渠道**

开箱即用的渠道：`mail, database, broadcast, nexmo, slack`。
```
public function via($notifiable)
{
    return $notifiable->prefers_sms ? ['nexmo'] : ['mail', 'database'];
}
```

**塞入队列**

引入 Queueable trait，实现 ShouldQueue 接口即可。
`$user->notify(new InvoicePaid($invoice));`

**延迟发送**
```
$when = now()->addMinutes(10);
$user->notify((new InvoicePaid($invoice))->delay($when));
```
**按需通知**

举个例子：用户未存储在应用中，指定临时通知路由信息。
```php
Notification::route('mail', 'taylor@laravel.com')
            ->route('nexmo', '5555555555')
            ->notify(new InvoicePaid($invoice));
```
#### 邮件通知
```php
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);
// $this->invoice： 你可以在构造器中传入你需要用到的数据。
    return (new MailMessage)
                ->greeting('Hello!')
                ->line('One of your invoices has been paid!')
                ->action('View Invoice', $url)
                ->line('Thank you for using our application!');
}
# 发送邮件通知时，确保在配置文件 config/app.php 中设置了 name 的值，该值将会用在邮件通知消息的头部和尾部。
```
```php
public function toMail($notifiable)
{
    return (new MailMessage)->view(
        'emails.name', ['invoice' => $this->invoice]
    );
}
```
```php
public function toMail($notifiable)
{
    return (new Mailable($this->invoice))->to($this->user->email);
}
```
```php
public function toMail($notifiable)
{
    return (new MailMessage)
                ->error()
                ->subject('Notification Subject')
                ->line('...');
}
```
**定制邮件发送者**
```php
public function toMail($notifiable)
{
    return (new MailMessage)
                ->from('test@example.com', 'Example')
                ->line('...');
}
```
**定制邮件接收者**
```
class User extends Authenticatable
{
    use Notifiable;

    public function routeNotificationForMail($notification)
    {
        return $this->email_address;
    }
}
```
**定制主题**

如果不定制主题，会根据通知类的名词默认主题名。如`InvoicePaid`转化成 Invoice Paid 。
```
public function toMail($notifiable)
{
    return (new MailMessage)
                ->subject('Notification Subject')
                ->line('...');
}
```

**自定义通知模板**
```
# 执行改命令后，resources/views/vendor/notifications 下面的文件可以根据自己需要进行修改
php artisan vendor:publish --tag=laravel-notifications 
```
预览邮件通知
```php
Route::get('mail', function () {
    $invoice = App\Invoice::find(1);

    return (new App\Notifications\InvoicePaid($invoice))
                ->toMail($invoice->user);
});
```
#### markdown 邮件通知
```php
php artisan make:notification InvoicePaid --markdown=mail.invoice.paid
```
```php
public function toMail($notifiable)
{
    $url = url('/invoice/'.$this->invoice->id);

    return (new MailMessage)
                ->subject('Invoice Paid')
                ->markdown('mail.invoice.paid', ['url' => $url]);
}
```
如何定制模板，怎么编写模板中的 markdown，请参考上一节 **邮件** 。

#### 数据库通知
```
php artisan notifications:table
php artisan migrate
```
**定义通知格式**

toArray 和 toDatabase 都可以实现。区别是，toArray 还可以作用于`broadcast`渠道。
```
public function toArray($notifiable)
{
    return [
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ];
}
```

**访问通知**
```
# use Illuminate\Notifications\Notifiable;
$user = App\User::find(1);

foreach ($user->notifications as $notification) {
    echo $notification->type;
}
```
```
$user = App\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    echo $notification->type;
}

# 若要从 JavaScript 客户端访问通知，需要为应用定义一个通知控制器，它返回可通知实体的通知，比如当前用户。可以从 JavaScript 客户端向该控制器 URI 发送 HTTP 请求。
```
**标记已读，删除通知**
```php
$user = App\User::find(1);

foreach ($user->unreadNotifications as $notification) {
    $notification->markAsRead();
}

$user->unreadNotifications->markAsRead();
```
```php
$user = App\User::find(1);

$user->unreadNotifications()->update(['read_at' => now()]);
```
```php
$user->notifications()->delete();
```

#### 广播通知
```
use Illuminate\Notifications\Messages\BroadcastMessage;

public function toBroadcast($notifiable)
{
    return new BroadcastMessage([
        'invoice_id' => $this->invoice->id,
        'amount' => $this->invoice->amount,
    ]);
}
```
```
return (new BroadcastMessage($data))
                ->onConnection('sqs')
                ->onQueue('broadcasts');
                
# 除了指定的数据，广播通知还包含 type 域，它包括通知类的类名。
```
**监听通知**
```
Echo.private('App.User.' + userId)
    .notification((notification) => {
        console.log(notification.type);
    });
```
**定制通知渠道**
```
# App\User
public function receivesBroadcastNotificationsOn()
{
    return 'users.'.$this->id;    // 就是替换上面的 'App.User.' + userId
}
```

#### sms 通知
[sms 通知](https://laravel.com/docs/6.0/notifications#sms-notifications)
#### slack 通知
[slack 通知](https://laravel.com/docs/6.0/notifications#slack-notifications)
#### 本地化通知
```php
$user->notify((new InvoicePaid($invoice))->locale('es'));
Notification::locale('es')->send($users, new InvoicePaid($invoice));
```
不同用户的不同本地化
```php
class User extends Model implements HasLocalePreference
{
    public function preferredLocale()
    {
        return $this->locale;
    }
}

$user->notify(new InvoicePaid($invoice));
```
#### 通知事件
```php
# EventServiceProvider
protected $listen = [
    'Illuminate\Notifications\Events\NotificationSent' => [
        'App\Listeners\LogNotification',
    ],
];
```
```
public function handle(NotificationSent $event)
{
    // $event->channel
    // $event->notifiable
    // $event->notification
    // $event->response
}
```
#### 定制渠道

```php
<?php

namespace App\Channels;

use Illuminate\Notifications\Notification;

class VoiceChannel
{

    public function send($notifiable, Notification $notification)
    {
        $message = $notification->toVoice($notifiable);

        // Send notification to the $notifiable instance...
    }
}
```
```php
<?php

namespace App\Notifications;

use Illuminate\Bus\Queueable;
use App\Channels\VoiceChannel;
use App\Channels\Messages\VoiceMessage;
use Illuminate\Notifications\Notification;
use Illuminate\Contracts\Queue\ShouldQueue;

class InvoicePaid extends Notification
{
    use Queueable;

    public function via($notifiable)
    {
        return [VoiceChannel::class];
    }

    public function toVoice($notifiable)
    {
        // ...
    }
}
```
## 九、开发拓展包
## 简介
有些扩展包只能在 Laravel 中使用。这些扩展包可能包含专门用来增强 Laravel 应用的路由、控制器、视图和配置的文件。
#### Facades 注解
在进行扩展包开发的时候，扩展包并不能访问 Laravel 提供的所有测试辅助函数。如果你想像在 Laravel 应用中一样编写扩展包的测试用例，你可以使用扩展包 Orchestral Testbench。
## 发现扩展包
你可以将服务提供者定义到扩展包的 composer.json 文件中的 extra 部分，而不是让用户手动将你的服务提供者添加到列表中。除了服务提供者，还可以列出所有你想注册的 facades：
```json
"extra": {
    "laravel": {
        "providers": [
            "Barryvdh\\Debugbar\\ServiceProvider"
        ],
        "aliases": {
            "Debugbar": "Barryvdh\\Debugbar\\Facade"
        }
    }
},
```
展包的用户，想要禁止一个扩展包被发现
```php
"extra": {
    "laravel": {
        "dont-discover": [
            "barryvdh/laravel-debugbar"
        ]
    }
},

// 所有
"extra": {
    "laravel": {
        "dont-discover": [
            "*"
        ]
    }
},
```
## 资源文件
#### 配置
```php
public function boot()
{
    $this->publishes([
        __DIR__.'/path/to/config/courier.php' => config_path('courier.php'),
    ]);
}
// $value = config('courier.option');
```
#### 数据库迁移
```php
# 你也可以将扩展包默认配置与应用的副本配置合并在一起。
public function register()
{
    $this->mergeConfigFrom(
        __DIR__.'/path/to/config/courier.php', 'courier'
    );
}
# 你也可以将扩展包默认配置与应用的副本配置合并在一起。
```
#### 路由
此方法将自动判断应用的路由是否已被缓存，如果路由已缓存，将不会加载你的路由文件：
```php
public function boot()
{
    $this->loadRoutesFrom(__DIR__.'/routes.php');
}
```
#### 迁移
当运行 php artisan migrate 命令时他们就会被自动执行。
```php
public function boot()
{
    $this->loadMigrationsFrom(__DIR__.'/path/to/migrations');
}
```
#### 语言包与翻译
```php
public function boot()
{
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');
}
```
```php
# 扩展包翻译约定使用 package::file.line 语法进行引用。
echo trans('courier::messages.welcome');
```
```php
public function boot()
{
    $this->loadTranslationsFrom(__DIR__.'/path/to/translations', 'courier');

    $this->publishes([
        __DIR__.'/path/to/translations' => resource_path('lang/vendor/courier'),
    ]);
}
```
#### 视图
```php
public function boot()
{
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');
}

Route::get('admin', function () {
    return view('courier::admin');
});
```
Laravel 首先会检查开发人员是否在 resources/views/vendor/courier 中提供了一个自定义版本的视图。然后，如果视图尚未被定义，Laravel 将会搜索在 loadViewsFrom 中定义的视图目录。
```
public function boot()
{
    $this->loadViewsFrom(__DIR__.'/path/to/views', 'courier');

    $this->publishes([
        __DIR__.'/path/to/views' => resource_path('views/vendor/courier'),
    ]);
}
```
## 命令
```php
public function boot()
{
    if ($this->app->runningInConsole()) {
        $this->commands([
            FooCommand::class,
            BarCommand::class,
        ]);
    }
}
```
## 公共资源文件
```php
public function boot()
{
    $this->publishes([
        __DIR__.'/path/to/assets' => public_path('vendor/courier'),
    ], 'public');
}

# 由于每次更新扩展包时通常都需要覆盖资源文件，因此需要使用 --force 标签：
php artisan vendor:publish --tag=public --force
```
## 发布群组文件
```php
public function boot()
{
    $this->publishes([
        __DIR__.'/../config/package.php' => config_path('package.php')
    ], 'config');

    $this->publishes([
        __DIR__.'/../database/migrations/' => database_path('migrations')
    ], 'migrations');
}

# 根据定义的标签来发布不同的群组文件：
php artisan vendor:publish --tag=config
```
## 十、队列
#### 介绍
支持：关系数据库，redis，Beanstalk，Amazon SQS，synchronous，null。
其中，sync 是用来本地开发用的，会立即执行。null 是用来禁止使用队列的。
```
// queue 的名字是配置文件中的 queue 的值
Job::dispatch();

// queue 的名字是 email
Job::dispatch()->onQueue('emails');
```
设置 queue 优先级
```
php artisan queue:work --queue=high,default    // 队列 hign 比队列 default 的优先级高
```
队列驱动 database
```
php artisan queue:table
php artisan migrate
```
队列驱动 redis
```
# config/database.php 配置 redis 数据库
# 如果是 redis 群组，请在队列名里面包含键哈希标志
'redis' => [
    'driver' => 'redis',
    'connection' => 'default',
    'queue' => '{default}',
    'retry_after' => 90,
    # block_for 配置项来具体说明驱动应该在将任务重新放入 Redis 数据库以及处理器轮询之前阻塞多久。
    'block_for' => 5,
],
```
使用其他驱动需要的包
```php
Amazon SQS: aws/aws-sdk-php ~3.0
Beanstalkd: pda/pheanstalk ~4.0
Redis: predis/predis ~1.0
```
#### 创建作业
```
php artisan make:job ProcessPodcast
# 生成的类实现了 Illuminate\Contracts\Queue\ShouldQueue 接口，这意味着这个任务将会被推送到队列中，而不是同步执行。

public function handle(AudioProcessor $processor)
{
// Process uploaded podcast...
}
```
handle 方法在作业被处理时候调用。
> 二进制数据应当 `base64_encode` 后再塞入队列。


```php
# 完全控制容器如何将依赖对象注入至 handle 方法，可以使用容器的 bindMethod 方法
use App\Jobs\ProcessPodcast;

$this->app->bindMethod(ProcessPodcast::class.'@handle', function ($job, $app) {
    return $job->handle($app->make(AudioProcessor::class));
});
```

```php
# 利用Laravel的Redis速率限制功能，每五秒只允许一个作业处理：

public function handle()
{
    Redis::throttle('key')->block(0)->allow(1)->every(5)->then(function () {
        info('Lock obtained...');

        // Handle job...
    }, function () {
        // Could not obtain lock...

        return $this->release(5);
    });
}
```
虽然此代码有效，但处理方法的结构变得有噪，因为它与Redis速率限制逻辑混杂在一起。 此外，对于我们要进行速率限制的任何其他作业，必须复制此速率限制逻辑。

我们可以定义一个处理速率限制的作业中间件，而不是处理方法中的速率限制。 Laravel没有作业中间件的默认位置，因此欢迎您将应用程序中间件放在应用程序的任何位置。 在此示例中，我们将中间件放在app / Jobs / Middleware目录中：
```php
namespace App\Jobs\Middleware;

class RateLimited
{
    public function handle($job, $next)
    {
        Redis::throttle('key')
                ->block(0)->allow(1)->every(5)
                ->then(function () use ($job, $next) {
                    // Lock obtained...

                    $next($job);
                }, function () use ($job) {
                    // Could not obtain lock...

                    $job->release(5);
                });
    }
}
```
```php
use App\Jobs\Middleware\RateLimited;

public function middleware()
{
    return [new RateLimited];
}
```
#### 分发作业

```
# PodcastController extends Controller
ProcessPodcast::dispatch($podcast);
```
延迟分发
```php
ProcessPodcast::dispatch($podcast)
                ->delay(now()->addMinutes(10));
# The Amazon SQS queue service has a maximum delay time of 15 minutes.
```
分发后立即运行
```
ProcessPodcast::dispatchNow($podcast);
```
作业链
withChain 方法中的作业依次执行，一个不成功，后续的不会进行。
```
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch();
# 使用 $this->delete() 方法删除作业不会阻止处理链接作业。 只有当链中的作业失败时，链才会停止执行。
```
```php
ProcessPodcast::withChain([
    new OptimizePodcast,
    new ReleasePodcast
])->dispatch()->allOnConnection('redis')->allOnQueue('podcasts');
```
分发作业到特定队列
```
ProcessPodcast::dispatch($podcast)->onQueue('processing');
```
分发作业到特定队列处理链接

```php
ProcessPodcast::dispatch($podcast)->onConnection('sqs');
// 或者 public $connection = 'sqs';


ProcessPodcast::dispatch($podcast)
              ->onConnection('sqs')
              ->onQueue('processing');
```
指定最大尝试次数为 3 次
`php artisan queue:work --tries=3`
在作业类里面指定的最大尝试次数优先级更高
```
# App\Jobs\ProcessPodcast
public $tries = 3;
```
基于时间的尝试：在多久范围内可以不断尝试
```
public function retryUntil(){
    return now()->addSeconds(5);
}

# 你也可以在队列事件监听器内去定义该方法。
```
timeout 
timeout 特性对于 PHP 7.1+ 和 pcntl PHP 扩展进行了优化.
```php
php artisan queue:work --timeout=30
# 任务执行最大秒数
```
```
public $timeout = 120;
```
速率控制
> 这个特性需要你的应用和 redis 服务器互动，多应用于 APIs 。
```php
Redis::throttle('key')->allow(10)->every(60)->then(function () {
    // Job logic...
}, function () {
    // Could not obtain lock...
    return $this->release(10);
});
# 限制一个给定类型的任务每 60 秒只执行 10 次。
# key 可以是唯一的字符串，用来指定速率控制的作业类型。
# release ，如果锁不能被获取，你应该 release 到队列中，以便 10s 后再次尝试。
# 注意：将受限制的作业释放回队列，仍然会增加工作的总数 attempts。
```
限制一个作业处理器一次性可以做多少个作业
```
Redis::funnel('key')->limit(1)->then(function () {
    // Job logic...
}, function () {
    // Could not obtain lock...

    return $this->release(10);
});
```
> 当使用速率控制时，很难决定作业被成功执行完所需要的次数。因此，使用速率控制和基于时间的尝试相结合将非常的有用。

错误处理

有异常时，会自动释放回队列，在规定次数内不断尝试，直到成功。控制参数就是命令行的参数 `--tries` 或者类中的 `tries`。

队列闭包
你也可以直接调用闭包，在当前请求周期之外执行。
```php
$podcast = App\Podcast::find(1);

dispatch(function () use ($podcast) {
    $podcast->publish();
});
// 闭包的代码内容将以加密方式签名，因此无法在传输过程中对其进行修改。
```
#### 运行作业处理器
作业处理器一旦运行，不会关闭，除非你手动关闭它。
```
php artisan queue:work
# 确保在后台永久的运行中，你可以使用进程监控 Supervisor。

php artisan queue:listen
# 修改代码后，自动重启队列，但是，没有上面那条命令高效
```
只处理一个单一的作业
```
php artisan queue:work once
```
指定使用哪个队列连接
```
php artisan queue:work redis
```
指定队列
```
php artisan queue:work redis --emails
```
处理所有队列最后退出
```
php artisan queue:work --stop-when-empty
```
> 守护进程队列在处理每个作业的时候不会重启框架，所以要注意释放资源。例如：在使用 GD 库的时候，记得使用 imagedestroy 释放内存。

队列优先级
```
dispatch((new Job)->onQueue('high'));

php artisan queue:work --queue=high-queue,low-queue
```
重启队列
```
php artisan queue:restart
# 重启信号存储与缓存。所以在使用该命令之前记得配置好 cache 。
```
retry_after

retry_after=90 在作业被执行 90 秒后，依然没有被删除，则会被释放回队列。注意，设置合理的值（你的作业可能被处理的最大合理秒数）。

主队列程序杀死子队列程序
```
php artisan queue:work --timeout=50
```
> --timeout 的数值应该总是小于 retry_after 。

当没有作业时候，设置休息时间
```
php artisan queue:work --sleep=2
```
#### Supervisor 配置
```
sudo apt-get install supervisor
```
> 如果不会玩这样，可以使用 Laravel Forge （无痛的 php 服务器）。
> 在`/etc/supervisor/conf.d`目录下创建任意个监控进程配置文件。例如：laravel-worder.conf 监控 queue:work 进程。如果失败，自动重启。
```
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /home/forge/app.com/artisan queue:work sqs --sleep=3 --tries=3
autostart=true
autorestart=true
user=forge
numprocs=8
redirect_stderr=true
stdout_logfile=/home/forge/app.com/worker.log
```
写好配置文件，你可以更新 Supervisor 配置并且重启进程
```
sudo supervisorctl reread

sudo supervisorctl update

sudo supervisorctl start laravel-worker:*
```
更多内容查看 [Supervisor 文档](http://supervisord.org/index.html)

#### 处理失败任务
多次尝试后依然失败的任务会存储在表 failed_jobs 中。
```
php artisan queue:failed-table
php artisan migrate
```
> 如果不设置 tries 参数，将只尝试一次。


`php artisan queue:work redis --tries=3 --delay=3`
或者
`public $retryAfter = 3;`

failed 方法
常用来作业最终失败后提醒用户或者恢复作业所做的一些操作。传入的参数是作业失败的异常类。
```
# App\Jobs\ProcessPodcast 
public function failed(Exception $exception)
{
 // Send user notification of failure, etc...
}
```
失败作业事件
```
# 如果你想在任务失败时注册一个可调用的事件，你可以使用 Queue::failing 方法。该事件是通过 email 或 Slack 通知你团队的绝佳时机。

# AppServiceProvider
public function boot()
{
    Queue::failing(function (JobFailed $event) {
        // $event->connectionName
        // $event->job
        // $event->exception
    });
}
```
#### 重试失败作业
查看失败的作业
```
php artisan queue:failed
```
重试 ID 是 5 的作业
```
php artisan queue:retry 5
```
重试所有
```
php artisan queue:retry all
```
删除作业
```
php artisan queue:forget 5
```
```
php artisan queue:flush # 删除所有
```
忽略确实模型的作业
When injecting an Eloquent model into a job, it is automatically serialized before being placed on the queue and restored when the job is processed. However, if the model has been deleted while the job was waiting to be processed by a worker, your job may fail with a ModelNotFoundException.

For convenience, you may choose to automatically delete jobs with missing models by setting your job's deleteWhenMissingModels property to true:
```php
public $deleteWhenMissingModels = true;
```
#### 队列事件
1. before
2. after
3. looping    在队列处理器从队列中获取作业之前调用。例如：回滚之前失败作业的事务
```
# AppServiceProvider 
public function boot()
{
    Queue::before(function (JobProcessing $event) {
        // $event->connectionName
        // $event->job
        // $event->job->payload()
    });

    Queue::after(function (JobProcessed $event) {
        // $event->connectionName
        // $event->job
        // $event->job->payload()
    });

    Queue::looping(function () {      // 回滚之前失败作业的事务
        while (DB::transactionLevel() > 0) {
            DB::rollBack();
        }
    });
}
```

#### 介绍
服务器自带的任务计划不足之处是你必须每次 ssh 上去配置。
```
# app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    // $schedule->command('inspire')
    //          ->hourly();
}
```
为了使 laravel 的生效，请在服务器中 crontab -e
```
* * * * * php /path-to-your-project/artisan schedule:run >> /dev/null 2>&1
```
#### 定义计划执行
```
# app/Console/Kernel.php     schedule 方法
$schedule->call(function () {
	DB::table('recent_users')->delete();
})->daily();

// 在每天的午夜清空 recent_users 表
```
计划执行 artisan 命令
```
$schedule->command('emails:send --force')->daily();

$schedule->command(EmailsCommand::class, ['--force'])->daily();
```
计划执行队列里的作业
```
$schedule->job(new Heartbeat)->everyFiveMinutes();
```
计划执行 shell 命令
```
$schedule->exec('node /home/forge/script.js')->daily();
```
| Method                            | Description                                      |
| --------------------------------- | ------------------------------------------------ |
| `->cron('* * * * *');`            | Run the task on a custom Cron schedule           |
| `->everyMinute();`                | Run the task every minute                        |
| `->everyFiveMinutes();`           | Run the task every five minutes                  |
| `->everyTenMinutes();`            | Run the task every ten minutes                   |
| `->everyFifteenMinutes();`        | Run the task every fifteen minutes               |
| `->everyThirtyMinutes();`         | Run the task every thirty minutes                |
| `->hourly();`                     | Run the task every hour                          |
| `->hourlyAt(17);`                 | Run the task every hour at 17 mins past the hour |
| `->daily();`                      | Run the task every day at midnight               |
| `->dailyAt('13:00');`             | Run the task every day at 13:00                  |
| `->twiceDaily(1, 13);`            | Run the task daily at 1:00 & 13:00               |
| `->weekly();`                     | Run the task every week                          |
| `->monthly();`                    | Run the task every month                         |
| `->monthlyOn(4, '15:00');`        | Run the task every month on the 4th at 15:00     |
| `->quarterly();`                  | Run the task every quarter                       |
| `->yearly();`                     | Run the task every year                          |
| `->timezone('America/New_York');` | Set the timezone                                 |

结合使用，制作更精确的时间

    // Run once per week on Monday at 1 PM...
    $schedule->call(function () {
        //
    })->weekly()->mondays()->at('13:00');
    
    // Run hourly from 8 AM to 5 PM on weekdays...
    $schedule->command('foo')
              ->weekdays()
              ->hourly()
              ->timezone('America/Chicago')
              ->between('8:00', '17:00');



| Method                     | Description                                       |
| -------------------------- | ------------------------------------------------- |
| `->weekdays();`            | Limit the task to weekdays                        |
| `->sundays();`             | Limit the task to Sunday                          |
| `->mondays();`             | Limit the task to Monday                          |
| `->tuesdays();`            | Limit the task to Tuesday                         |
| `->wednesdays();`          | Limit the task to Wednesday                       |
| `->thursdays();`           | Limit the task to Thursday                        |
| `->fridays();`             | Limit the task to Friday                          |
| `->saturdays();`           | Limit the task to Saturday                        |
| `->between($start, $end);` | Limit the task to run between start and end times |
| `->when(Closure);`         | Limit the task based on a truth test              |

```
$schedule->command('reminders:send')
                    ->hourly()
                    ->between('7:00', '22:00');
```
```
$schedule->command('reminders:send')
                    ->hourly()
                    ->unlessBetween('23:00', '4:00');
```
```
$schedule->command('emails:send')->daily()->when(function () {
    return true;
});
```
```
# 和 when 相反
$schedule->command('emails:send')->daily()->skip(function () {
    return true;
});
```
阻止任务重叠
TODO：没怎么搞懂这是个什么鬼
```
# 默认情况下，即使任务的前一个实例仍在运行，计划任务也会运行。
$schedule->command('emails:send')->withoutOverlapping();
```
默认 withoutOverlapping 的过期时间是 24 小时。
```
$schedule->command('emails:send')->withoutOverlapping(10);
```
在维护模式时候强制执行计划任务
```
$schedule->command('emails:send')->evenInMaintenanceMode();
```
#### 任务输出
```
$schedule->command('emails:send')
         ->daily()
         ->sendOutputTo($filePath);
```
```
$schedule->command('emails:send')
         ->daily()
         ->appendOutputTo($filePath)
```
将输出邮寄给别人
```
$schedule->command('foo')
         ->daily()
         ->sendOutputTo($filePath)
         ->emailOutputTo('foo@example.com');
```
> 输出只支持 command 方法，不支持 call 方法。
#### 任务钩子
```
$schedule->command('emails:send')
         ->daily()
         ->before(function () {
             // Task is about to start...
         })
         ->after(function () {
             // Task is complete...
         });
```
## 十一、任务计划
## 简介
`* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1`
## 定义调度
```php
# App\Console\Kernel
protected function schedule(Schedule $schedule)
{
    $schedule->call(function () {
    DB::table('recent_users')->delete();
    })->daily();
}

$schedule->call(new DeleteRecentUsers)->daily();
```
#### Artisan 命令调度
```php
$schedule->command('emails:send Taylor --force')->daily();
$schedule->command(EmailsCommand::class, ['Taylor', '--force'])->daily();
```
#### 队列任务调度
```php
$schedule->job(new Heartbeat)->everyFiveMinutes();

// Dispatch the job to the "heartbeats" queue...
$schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();
```
#### Shell 命令调度
`$schedule->exec('node /home/forge/script.js')->daily();`
#### 调度频率设置

Method|	Description
--|--
->cron('* * * * *');|	Run the task on a custom Cron schedule
->everyMinute();|	Run the task every minute
->everyFiveMinutes();	|Run the task every five minutes
->everyTenMinutes();|	Run the task every ten minutes
->everyFifteenMinutes();	|Run the task every fifteen minutes
->everyThirtyMinutes();|	Run the task every thirty minutes
->hourly();	Run the task every hour
->hourlyAt(17);|	Run the task every hour at 17 mins past the hour
->daily();|	Run the task every day at midnight
->dailyAt('13:00');|	Run the task every day at 13:00
->twiceDaily(1, 13);|	Run the task daily at 1:00 & 13:00
->weekly();	|Run the task every week
->weeklyOn(1, '8:00');|	Run the task every week on Monday at 8:00
->monthly();|	Run the task every month
->monthlyOn(4, '15:00');|	Run the task every month on the 4th at 15:00
->quarterly();	|Run the task every quarter
->yearly();	|Run the task every year
->timezone('America/New_York');	|Set the timezone

```php
// Run once per week on Monday at 1 PM...
$schedule->call(function () {
    //
})->weekly()->mondays()->at('13:00');

// Run hourly from 8 AM to 5 PM on weekdays...
$schedule->command('foo')
          ->weekdays()
          ->hourly()
          ->timezone('America/Chicago')
          ->between('8:00', '17:00');
```

Method|	Description
--|--
->weekdays();	|Limit the task to weekdays
->weekends();	|Limit the task to weekends
->sundays();	|Limit the task to Sunday
->mondays();	|Limit the task to Monday
->tuesdays();	|Limit the task to Tuesday
->wednesdays();	|Limit the task to Wednesday
->thursdays();	|Limit the task to Thursday
->fridays();	|Limit the task to Friday
->saturdays();	|Limit the task to Saturday
->between($start, $end);	|Limit the task to run between start and end times
->when(Closure);	|Limit the task based on a truth test
->environments($env);	|Limit the task to specific environments

```php
$schedule->command('reminders:send')
                    ->hourly()
                    ->between('7:00', '22:00');
                    
$schedule->command('reminders:send')
                    ->hourly()
                    ->unlessBetween('23:00', '4:00');
```

```php
# 如果给定的闭包返回结果为 true，只要没有其他约束条件阻止任务运行，任务就会一直执行下去，skip 相反
$schedule->command('emails:send')->daily()->when(function () {
    return true;
});

$schedule->command('emails:send')->daily()->skip(function () {
    return true;
});

# 环境约束
$schedule->command('emails:send')
            ->daily()
            ->environments(['staging', 'production']);
```
#### 时区
```php
$schedule->command('report:generate')
         ->timezone('America/New_York')
         ->at('02:00')
         
# app/Console/Kernel.php
protected function scheduleTimezone()
{
    return 'America/Chicago';
}
```
#### 避免任务重复
```php
$schedule->command('emails:send')->withoutOverlapping();
# 如果你的任务执行时间不确定，且你又不能准确预估出任务的执行时间，那么 withoutOverlapping 方法会显得特别有用。

$schedule->command('emails:send')->withoutOverlapping(10);
// 如果有需要，你可以指定「without overlapping」锁指定的时间范围。默认情况下，锁将在 24 小时后过期。
```
#### 任务只运行在一台服务器上
要使用这个特性，你的应用默认缓存驱动必须是 memcached 或者 redis。除此之外，所有的服务器必须使用同一个中央缓存服务器通信。

为了说明任务应该在单个服务器上运行，在定义调度任务时使用 onOneServer 方法。第一个获取到任务的服务器会生成一个原子锁，用来防止其他服务器在同一时刻执行相同任务。
```php
$schedule->command('report:generate')
                ->fridays()
                ->at('17:00')
                ->onOneServer();
```
#### 后台任务
```php
$schedule->command('analytics:report')
         ->daily()
         ->runInBackground();   // 在后台同时运行
         
# The runInBackground method may only be used when scheduling tasks via the command and exec methods.
```
#### 维护模式
```php
$schedule->command('emails:send')->evenInMaintenanceMode();
```
## 任务输出
```php
$schedule->command('emails:send')
         ->daily()
         ->sendOutputTo($filePath);
         
$schedule->command('emails:send')
         ->daily()
         ->appendOutputTo($filePath);
         
$schedule->command('foo')
         ->daily()
         ->sendOutputTo($filePath)
         ->emailOutputTo('foo@example.com');
         
$schedule->command('foo')
         ->daily()
         ->emailOutputOnFailure('foo@example.com');
         
// The emailOutputTo, emailOutputOnFailure, sendOutputTo, and appendOutputTo methods are exclusive to the command and exec methods.
```
## 任务钩子
```php

$schedule->command('emails:send')
         ->daily()
         ->before(function () {
             // Task is about to start...
         })
         ->after(function () {
             // Task is complete...
         });
         
$schedule->command('emails:send')
         ->daily()
         ->onSuccess(function () {
             // The task succeeded...
         })
         ->onFailure(function () {
             // The task failed...
         });
         
# 此方法对于通知外部服务（例如Laravel Envoyer）您的计划任务正在开始或已完成执行非常有用：
$schedule->command('emails:send')
         ->daily()
         ->pingBefore($url)
         ->thenPing($url);
         
$schedule->command('emails:send')
         ->daily()
         ->pingBeforeIf($condition, $url)
         ->thenPingIf($condition, $url);
  // 只有在给定条件为 true 时，才能使用 pingBeforeIf 和 thenPingIf 方法 ping 指定的 URL：
  
$schedule->command('emails:send')
         ->daily()
         ->pingOnSuccess($successUrl)
         ->pingOnFailure($failureUrl);
         
所有 ping 方法都需要 Guzzle HTTP 库。你可以使用 Composer 来添加 Guzzle 到你的项目中：
composer require guzzlehttp/guzzle
```
```
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```

# 第八章：数据库（一、开始。二、查询构造器。三、分页。四、迁移。五、填充。六、Redis。）
```
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
## 一、开始
## 简介
`* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1`
## 定义调度
```php
# App\Console\Kernel
protected function schedule(Schedule $schedule)
{
    $schedule->call(function () {
    DB::table('recent_users')->delete();
    })->daily();
}

$schedule->call(new DeleteRecentUsers)->daily();
```
#### Artisan 命令调度
```php
$schedule->command('emails:send Taylor --force')->daily();
$schedule->command(EmailsCommand::class, ['Taylor', '--force'])->daily();
```
#### 队列任务调度
```php
$schedule->job(new Heartbeat)->everyFiveMinutes();

// Dispatch the job to the "heartbeats" queue...
$schedule->job(new Heartbeat, 'heartbeats')->everyFiveMinutes();
```
#### Shell 命令调度
`$schedule->exec('node /home/forge/script.js')->daily();`
#### 调度频率设置

Method|	Description
--|--
->cron('* * * * *');|	Run the task on a custom Cron schedule
->everyMinute();|	Run the task every minute
->everyFiveMinutes();	|Run the task every five minutes
->everyTenMinutes();|	Run the task every ten minutes
->everyFifteenMinutes();	|Run the task every fifteen minutes
->everyThirtyMinutes();|	Run the task every thirty minutes
->hourly();	Run the task every hour
->hourlyAt(17);|	Run the task every hour at 17 mins past the hour
->daily();|	Run the task every day at midnight
->dailyAt('13:00');|	Run the task every day at 13:00
->twiceDaily(1, 13);|	Run the task daily at 1:00 & 13:00
->weekly();	|Run the task every week
->weeklyOn(1, '8:00');|	Run the task every week on Monday at 8:00
->monthly();|	Run the task every month
->monthlyOn(4, '15:00');|	Run the task every month on the 4th at 15:00
->quarterly();	|Run the task every quarter
->yearly();	|Run the task every year
->timezone('America/New_York');	|Set the timezone

```php
// Run once per week on Monday at 1 PM...
$schedule->call(function () {
    //
})->weekly()->mondays()->at('13:00');

// Run hourly from 8 AM to 5 PM on weekdays...
$schedule->command('foo')
          ->weekdays()
          ->hourly()
          ->timezone('America/Chicago')
          ->between('8:00', '17:00');
```

Method|	Description
--|--
->weekdays();	|Limit the task to weekdays
->weekends();	|Limit the task to weekends
->sundays();	|Limit the task to Sunday
->mondays();	|Limit the task to Monday
->tuesdays();	|Limit the task to Tuesday
->wednesdays();	|Limit the task to Wednesday
->thursdays();	|Limit the task to Thursday
->fridays();	|Limit the task to Friday
->saturdays();	|Limit the task to Saturday
->between($start, $end);	|Limit the task to run between start and end times
->when(Closure);	|Limit the task based on a truth test
->environments($env);	|Limit the task to specific environments

```php
$schedule->command('reminders:send')
                    ->hourly()
                    ->between('7:00', '22:00');
                    
$schedule->command('reminders:send')
                    ->hourly()
                    ->unlessBetween('23:00', '4:00');
```

```php
# 如果给定的闭包返回结果为 true，只要没有其他约束条件阻止任务运行，任务就会一直执行下去，skip 相反
$schedule->command('emails:send')->daily()->when(function () {
    return true;
});

$schedule->command('emails:send')->daily()->skip(function () {
    return true;
});

# 环境约束
$schedule->command('emails:send')
            ->daily()
            ->environments(['staging', 'production']);
```
#### 时区
```php
$schedule->command('report:generate')
         ->timezone('America/New_York')
         ->at('02:00')
         
# app/Console/Kernel.php
protected function scheduleTimezone()
{
    return 'America/Chicago';
}
```
#### 避免任务重复
```php
$schedule->command('emails:send')->withoutOverlapping();
# 如果你的任务执行时间不确定，且你又不能准确预估出任务的执行时间，那么 withoutOverlapping 方法会显得特别有用。

$schedule->command('emails:send')->withoutOverlapping(10);
// 如果有需要，你可以指定「without overlapping」锁指定的时间范围。默认情况下，锁将在 24 小时后过期。
```
#### 任务只运行在一台服务器上
要使用这个特性，你的应用默认缓存驱动必须是 memcached 或者 redis。除此之外，所有的服务器必须使用同一个中央缓存服务器通信。

为了说明任务应该在单个服务器上运行，在定义调度任务时使用 onOneServer 方法。第一个获取到任务的服务器会生成一个原子锁，用来防止其他服务器在同一时刻执行相同任务。
```php
$schedule->command('report:generate')
                ->fridays()
                ->at('17:00')
                ->onOneServer();
```
#### 后台任务
```php
$schedule->command('analytics:report')
         ->daily()
         ->runInBackground();   // 在后台同时运行
         
# The runInBackground method may only be used when scheduling tasks via the command and exec methods.
```
#### 维护模式
```php
$schedule->command('emails:send')->evenInMaintenanceMode();
```
## 任务输出
```php
$schedule->command('emails:send')
         ->daily()
         ->sendOutputTo($filePath);
         
$schedule->command('emails:send')
         ->daily()
         ->appendOutputTo($filePath);
         
$schedule->command('foo')
         ->daily()
         ->sendOutputTo($filePath)
         ->emailOutputTo('foo@example.com');
         
$schedule->command('foo')
         ->daily()
         ->emailOutputOnFailure('foo@example.com');
         
// The emailOutputTo, emailOutputOnFailure, sendOutputTo, and appendOutputTo methods are exclusive to the command and exec methods.
```
## 任务钩子
```php

$schedule->command('emails:send')
         ->daily()
         ->before(function () {
             // Task is about to start...
         })
         ->after(function () {
             // Task is complete...
         });
         
$schedule->command('emails:send')
         ->daily()
         ->onSuccess(function () {
             // The task succeeded...
         })
         ->onFailure(function () {
             // The task failed...
         });
         
# 此方法对于通知外部服务（例如Laravel Envoyer）您的计划任务正在开始或已完成执行非常有用：
$schedule->command('emails:send')
         ->daily()
         ->pingBefore($url)
         ->thenPing($url);
         
$schedule->command('emails:send')
         ->daily()
         ->pingBeforeIf($condition, $url)
         ->thenPingIf($condition, $url);
  // 只有在给定条件为 true 时，才能使用 pingBeforeIf 和 thenPingIf 方法 ping 指定的 URL：
  
$schedule->command('emails:send')
         ->daily()
         ->pingOnSuccess($successUrl)
         ->pingOnFailure($failureUrl);
         
所有 ping 方法都需要 Guzzle HTTP 库。你可以使用 Composer 来添加 Guzzle 到你的项目中：
composer require guzzlehttp/guzzle
```
## 二、查询构造器
pdo 没有字段绑定，不要允许用户提供字段名，如果必须，那么请设置字段白名单。
#### 读

```php
DB::select('select * from users where id = :id', ['id' => 1]);   // 原生语句

DB::table('users')->get();                             // 返回包含所有对象的集合
DB::table('users')->where('name', 'John')->first();	   // 返回对象
DB::table('users')->where('id', 17)->value('age');     // 返回 17
DB::table('users')->find(3);                           // 找出主键等于 3 的
DB::table('users')->pluck('age');                      // 返回包含字段值的集合
DB::table('users')->pluck('age', 'id');                // 返回关联集合 id => age，最多 2 个参数
DB::table('users')->count();                           // 返回数字   类似的统计函数支持 (count, max, min, avg, sum)

DB::table('users')->max('id');                         // 返回数字或 null
DB::table('users')->where('class_id', '1')->average('age'); // 返回四位小数的平均值或 null
```

```php
DB::table('users')->orderBy('id')->chunk(100, function ($users) {  // 取每 100 个一组
    foreach ($users as $user) {
        // ...
        // return false;                                           // 随时可以退出
    }
});      

// 不会出现 chunk 结束后，还有很多记录没被处理的情况
DB::table('users')->where('active', false)
    ->chunkById(100, function ($users) {
        foreach ($users as $user) {
            DB::table('users')
                ->where('id', $user->id)
                ->update(['active' => true]);
        }
    });
```

```php
DB::table('orders')->where('finalized', 1)->exists();  
DB::table('orders')->where('finalized', 1)->doesntExist();

$users = DB::table('users')->select('name', 'email as user_email')->get();

$users = DB::table('users')->distinct()->get();

$query = DB::table('users')->select('name');
$users = $query->addSelect('age')->get();
```

```php
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();

// selectRaw 可以传一个参数绑定   selectRaw 等价于 select(DB::raw(...))
DB::table('orders')
                ->selectRaw('price * ? as price_with_tax', [1.0825])
                ->get();
```

```php
$orders = DB::table('orders')
                ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                ->get();
```

```php
$orders = DB::table('orders')
                ->select('department', DB::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > ?', [2500])
                ->get();

$orders = DB::table('orders')
                ->orderByRaw('updated_at - created_at DESC')
                ->get();
```

```php
# inner join
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();

# left join
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

# right join
$users = DB::table('users')
            ->rightJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();

# cross join
$users = DB::table('sizes')
            ->crossJoin('colours')
            ->get();

# 高级 join
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();

DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();

# 子查询 join
$latestPosts = DB::table('posts')
                   ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                   ->where('is_published', true)
                   ->groupBy('user_id');

$users = DB::table('users')
        ->joinSub($latestPosts, 'latest_posts', function ($join) {
            $join->on('users.id', '=', 'latest_posts.user_id');
        })->get();

# union
$first = DB::table('users')
            ->whereNull('first_name');

$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();

# unionAll 和 union 参数一样
```

```php
where('votes', '=', 100)
where('votes', 100)
where('votes', '>=', 100)
where('votes', '<>', 100)
where('name', 'like', 'T%')
where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])
where('votes', '>', 100)->orWhere('name', 'John')
whereBetween('votes', [1, 100])
whereNotBetween('votes', [1, 100])
whereIn('id', [1, 2, 3])
whereNotIn('id', [1, 2, 3])
whereNull('last_name')
whereNotNull('updated_at')
whereDate('created_at', '2016-12-31')
whereMonth('created_at', '12')
whereDay('created_at', '31')
whereYear('created_at', '2016')
whereTime('created_at', '=', '11:20:45')
whereColumn('first_name', 'last_name')        // 判断两个字段 相等
whereColumn('updated_at', '>', 'created_at')

whereColumn([
    ['first_name', '=', 'last_name'],
    ['updated_at', '>', 'created_at']
])

where('name', '=', 'John')
->orWhere(function ($query) {  // 你应该总是传入闭包进 orWhere，避免全局 scope 产生不良影响
    $query->where('votes', '>', 100)
          ->where('title', '<>', 'Admin');
})

whereExists(function ($query) {
    $query->select(DB::raw(1))
          ->from('orders')
          ->whereRaw('orders.user_id = users.id');
})   // where exists ( select 1 from orders where orders.user_id = users.id )
  
# json
$users = DB::table('users')
                ->where('options->language', 'en')
                ->get();

$users = DB::table('users')
                ->where('preferences->dining->meal', 'salad')
                ->get();

$users = DB::table('users')
                ->whereJsonContains('options->languages', 'en')
                ->get();    // 不支持 SQLite

$users = DB::table('users')
                ->whereJsonContains('options->languages', ['en', 'de'])
                ->get();    // 只支持 MySQL and PostgreSQL

$users = DB::table('users')
                ->whereJsonLength('options->languages', 0)
                ->get();

$users = DB::table('users')
                ->whereJsonLength('options->languages', '>', 1)
                ->get();
```

```php
orderBy('name', 'desc')

latest()                // === orderBy('created_at', 'desc')

inRandomOrder()

groupBy('account_id')->having('account_id', '>', 100)
->groupBy('first_name', 'status')->having('account_id', '>', 100)
    
skip(10)->take(5)        // === offset(10)->limit(5)
```

```php
// $role 有值才会执行闭包
$role = $request->input('role');

$users = DB::table('users')
                ->when($role, function ($query) use ($role) {
                    return $query->where('role_id', $role);
                })
                ->get();

// $role 有值执行第一个闭包，否则执行第二个闭包
$sortBy = null;
$users = DB::table('users')
                ->when($sortBy, function ($query, $sortBy) {
                    return $query->orderBy($sortBy);
                }, function ($query) {
                    return $query->orderBy('name');
                })
                ->get();
```

#### 写
```php
DB::table('users')->insert(
    ['email' => 'john@example.com', 'votes' => 0]
);

DB::table('users')->insert([
    ['email' => 'taylor@example.com', 'votes' => 0],
    ['email' => 'dayle@example.com', 'votes' => 0]
]);

DB::table('users')->insertOrIgnore([
    ['id' => 1, 'email' => 'taylor@example.com'],
    ['id' => 2, 'email' => 'dayle@example.com']
]);

$id = DB::table('users')->insertGetId(
    ['email' => 'john@example.com', 'votes' => 0]
);
# PostgreSQL 的 insertGetId 默认自增字段是 id，如果是其他的，需要传入字段名到 insertGetId 第二个参数。
```
#### 改
```php

$numbersOfRowsAffected = DB::table('users')
            ->where('id', 1)
            ->update(['votes' => 1]);

# json
DB::table('users')
            ->where('id', 1)
            ->update(['options->enabled' => true]);
```

```php
DB::table('users')->increment('votes');
DB::table('users')->increment('votes', 5);
DB::table('users')->decrement('votes');
DB::table('users')->decrement('votes', 5);

DB::table('users')->increment('votes', 1, ['name' => 'John']);
```
#### 删
```php
$numbersOfRowsAffected = DB::table('users')->delete();
$numbersOfRowsAffected = DB::table('users')->where('votes', '>', 100)->delete();
DB::table('users')->truncate();
```
#### 悲观锁
共享锁可以在事务提交前禁止选定行被修改。
```
DB::table('users')->where('votes', '>', 100)->sharedLock()->get();
```
for update 锁可以阻止被修改，或者阻止被另一个共享锁使用。
```
DB::table('users')->where('votes', '>', 100)->lockForUpdate()->get();
```

#### 调试
```
DB::table('users')->where('votes', '>', 100)->dd();

DB::table('users')->where('votes', '>', 100)->dump();
```
## 三、分页

查询构造器和 Eloquent ORM 都可以用。   
`$users = DB::table('users')->paginate(15);`
> Laravel 分页暂不支持 groupBy 语句。 可以自行实现。

只显示上一页，下一页
```
$users = DB::table('users')->simplePaginate(15);
```
```
$users = App\User::paginate(15);

$users = User::where('votes', '>', 100)->paginate(15);

$users = User::where('votes', '>', 100)->simplePaginate(15);
```
```
<div class="container">
    @foreach ($users as $user)
        {{ $user->name }}
    @endforeach
</div>

{{ $users->links() }}
```
```
// http://example.com/custom/url?page=N
Route::get('users', function () {
    $users = App\User::paginate(15);

    $users->withPath('custom/url');
});
```
```
// &sort=votes
{{ $users->appends(['sort' => 'votes'])->links() }}
```
```
// #foo
{{ $users->fragment('foo')->links() }}

// 你能够控制在分页器的 “窗口” 的每一侧显示多少个附加链接。默认情况下，主分页链接的每侧显示三个链接。可以使用 onEachSide 方法改变这个数值：
{{ $users->onEachSide(5)->links() }}
```
转成 json
```
return App\User::paginate();
```
```
{
   "total": 50,
   "per_page": 15,
   "current_page": 1,
   "last_page": 4,
   "first_page_url": "http://laravel.app?page=1",
   "last_page_url": "http://laravel.app?page=4",
   "next_page_url": "http://laravel.app?page=2",
   "prev_page_url": null,
   "path": "http://laravel.app",
   "from": 1,
   "to": 15,
   "data":[
        {
            // Result Object
        },
        {
            // Result Object
        }
   ]
}
```
定制分页视图
```
{{ $paginator->links('view.name') }}

// Passing data to the view...
{{ $paginator->links('view.name', ['foo' => 'bar']) }}
```
```
php artisan vendor:publish --tag=laravel-pagination
```
修改默认模版
```php
public function boot()
{
    Paginator::defaultView('view-name');

    Paginator::defaultSimpleView('view-name');
}
```

更多方法
```
$results->count()
$results->currentPage()
$results->firstItem()
$results->getOptions()
$results->getUrlRange($start, $end)
$results->hasMorePages()
$results->items()
$results->lastItem()
$results->lastPage() (Not available when using simplePaginate)
$results->nextPageUrl()
$results->onFirstPage()
$results->perPage()
$results->previousPageUrl()
$results->total() (Not available when using simplePaginate)
$results->url($page)
```

## 四、迁移

迁移相当于数据库的一个php版版本控制工具。
#### 创建迁移
```
php artisan make:migration create_users_table

php artisan make:migration create_users_table --create=users     // 等同于上面的

php artisan make:migration add_votes_to_users_table --table=users  
```
迁移类中的 up 和 down 方法应当实现完全相反的操作。
```php
public function up()
{
    Schema::create('flights', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->string('airline');
    $table->timestamps();
});
}

public function down()
{
	Schema::drop('flights');
}
```
#### 运行 migration
```
php artisan migrate 
php artisan migrate --force    // 在生产中或有其他提示情况下强制执行
```
回滚操作
```
php artisan migrate:rollback   // 回滚到上一版本
php artisan migrate:rollback --step=5    // 回滚到前面五个迁移
php artisan migrate:reset    // 回滚所有
php artisan migrate:refresh   // 回滚并且 migrate
php artisan migrate:refresh --seed   // 回滚并且 migrate 再 seed
php artisan migrate:refresh --step=5    // 回滚五步，在 migrate 五步
```
直接删除再 migrate
```
php artisan migrate:fresh

php artisan migrate:fresh --seed
```
#### 表
```
Schema::create('users', function (Blueprint $table) {
    $table->bigIncrements('id');
});
```
```
if (Schema::hasTable('users')) {
    //
}

if (Schema::hasColumn('users', 'email')) {
    //
}
```
```
Schema::connection('foo')->create('users', function (Blueprint $table) {
    $table->bigIncrements('id');
});
```

| Command                                | Description                                            |
| -------------------------------------- | ------------------------------------------------------ |
| $table->engine = 'InnoDB';             | Specify the table storage engine (MySQL).              |
| $table->charset = 'utf8';              | Specify a default character set for the table (MySQL). |
| $table->collation = 'utf8_unicode_ci'; | Specify a default collation for the table (MySQL).     |
| $table->temporary();                   | Create a temporary table (except SQL Server).          |
```
Schema::rename($from, $to);

Schema::drop('users');

Schema::dropIfExists('users');
```
#### 字段
| Command                                    | Description                                                  |
| ------------------------------------------ | ------------------------------------------------------------ |
| `$table->bigIncrements('id');`             | Auto-incrementing UNSIGNED BIGINT (primary key) equivalent column. |
| `$table->bigInteger('votes');`             | BIGINT equivalent column.                                    |
| `$table->binary('data');`                  | BLOB equivalent column.                                      |
| `$table->boolean('confirmed');`            | BOOLEAN equivalent column.                                   |
| `$table->char('name', 100);`               | CHAR equivalent column with an optional length.              |
| `$table->date('created_at');`              | DATE equivalent column.                                      |
| `$table->dateTime('created_at');`          | DATETIME equivalent column.                                  |
| `$table->dateTimeTz('created_at');`        | DATETIME (with timezone) equivalent column.                  |
| `$table->decimal('amount', 8, 2);`         | DECIMAL equivalent column with a precision (total digits) and scale (decimal digits). |
| `$table->double('amount', 8, 2);`          | DOUBLE equivalent column with a precision (total digits) and scale (decimal digits). |
| `$table->enum('level', ['easy', 'hard']);` | ENUM equivalent column.                                      |
| `$table->float('amount', 8, 2);`           | FLOAT equivalent column with a precision (total digits) and scale (decimal digits). |
| `$table->geometry('positions');`           | GEOMETRY equivalent column.                                  |
| `$table->geometryCollection('positions');` | GEOMETRYCOLLECTION equivalent column.                        |
| `$table->increments('id');`                | Auto-incrementing UNSIGNED INTEGER (primary key) equivalent column. |
| `$table->integer('votes');`                | INTEGER equivalent column.                                   |
| `$table->ipAddress('visitor');`            | IP address equivalent column.                                |
| `$table->json('options');`                 | JSON equivalent column.                                      |
| `$table->jsonb('options');`                | JSONB equivalent column.                                     |
| `$table->lineString('positions');`         | LINESTRING equivalent column.                                |
| `$table->longText('description');`         | LONGTEXT equivalent column.                                  |
| `$table->macAddress('device');`            | MAC address equivalent column.                               |
| `$table->mediumIncrements('id');`          | Auto-incrementing UNSIGNED MEDIUMINT (primary key) equivalent column. |
| `$table->mediumInteger('votes');`          | MEDIUMINT equivalent column.                                 |
| `$table->mediumText('description');`       | MEDIUMTEXT equivalent column.                                |
| `$table->morphs('taggable');`              | Adds `taggable_id` UNSIGNED BIGINT and `taggable_type` VARCHAR equivalent columns. |
| `$table->multiLineString('positions');`    | MULTILINESTRING equivalent column.                           |
| `$table->multiPoint('positions');`         | MULTIPOINT equivalent column.                                |
| `$table->multiPolygon('positions');`       | MULTIPOLYGON equivalent column.                              |
| `$table->nullableMorphs('taggable');`      | Adds nullable versions of `morphs()` columns.                |
| `$table->nullableUuidMorphs('taggable');`      | Adds nullable versions of  uuidMorphs() columns.                |
| `$table->nullableTimestamps();`            | Alias of `timestamps()` method.                              |
| `$table->point('position');`               | POINT equivalent column.                                     |
| `$table->polygon('positions');`            | POLYGON equivalent column.                                   |
| `$table->rememberToken();`                 | Adds a nullable `remember_token` VARCHAR(100) equivalent column. |
| `$table->set('flavors', ['strawberry', 'vanilla']);	`           |SET equivalent column. |
| `$table->smallIncrements('id');`           | Auto-incrementing UNSIGNED SMALLINT (primary key) equivalent column. |
| `$table->smallInteger('votes');`           | SMALLINT equivalent column.                                  |
| `$table->softDeletes();`                   | Adds a nullable `deleted_at` TIMESTAMP equivalent column for soft deletes. |
| `$table->softDeletesTz();`                 | Adds a nullable `deleted_at` TIMESTAMP (with timezone) equivalent column for soft deletes. |
| `$table->string('name', 100);`             | VARCHAR equivalent column with a optional length.            |
| `$table->text('description');`             | TEXT equivalent column.                                      |
| `$table->time('sunrise');`                 | TIME equivalent column.                                      |
| `$table->timeTz('sunrise');`               | TIME (with timezone) equivalent column.                      |
| `$table->timestamp('added_on');`           | TIMESTAMP equivalent column.                                 |
| `$table->timestampTz('added_on');`         | TIMESTAMP (with timezone) equivalent column.                 |
| `$table->timestamps();`                    | Adds nullable `created_at` and `updated_at` TIMESTAMP equivalent columns. |
| `$table->timestampsTz();`                  | Adds nullable `created_at` and `updated_at` TIMESTAMP (with timezone) equivalent columns. |
| `$table->tinyIncrements('id');`            | Auto-incrementing UNSIGNED TINYINT (primary key) equivalent column. |
| `$table->tinyInteger('votes');`            | TINYINT equivalent column.                                   |
| `$table->unsignedBigInteger('votes');`     | UNSIGNED BIGINT equivalent column.                           |
| `$table->unsignedDecimal('amount', 8, 2);` | UNSIGNED DECIMAL equivalent column with a precision (total digits) and scale (decimal digits). |
| `$table->unsignedInteger('votes');`        | UNSIGNED INTEGER equivalent column.                          |
| `$table->unsignedMediumInteger('votes');`  | UNSIGNED MEDIUMINT equivalent column.                        |
| `$table->unsignedSmallInteger('votes');`   | UNSIGNED SMALLINT equivalent column.                         |
| `$table->unsignedTinyInteger('votes');`    | UNSIGNED TINYINT equivalent column.                          |
| `$table->uuid('id');`                      | UUID equivalent column.                                      |
| `$table->year('birth_year');`              | YEAR equivalent column.                                      |

修改字段
```php
Schema::table('users', function (Blueprint $table) {
    $table->string('email')->nullable();
});
```
| Modifier                         | Description                                                  |
| -------------------------------- | ------------------------------------------------------------ |
| `->after('column')`              | Place the column "after" another column (MySQL)              |
| `->autoIncrement()`              | Set INTEGER columns as auto-increment (primary key)          |
| `->charset('utf8')`              | Specify a character set for the column (MySQL)               |
| `->collation('utf8_unicode_ci')` | Specify a collation for the column (MySQL/SQL Server)        |
| `->comment('my comment')`        | Add a comment to a column (MySQL)                            |
| `->default($value)`              | Specify a "default" value for the column                     |
| `->first()`                      | Place the column "first" in the table (MySQL)                |
| `->nullable($value = true)`      | Allows (by default) NULL values to be inserted into the column |
| `->storedAs($expression)`        | Create a stored generated column (MySQL)                     |
| `->unsigned()`                   | Set INTEGER columns as UNSIGNED (MySQL)                      |
| `->useCurrent()`                 | Set TIMESTAMP columns to use CURRENT_TIMESTAMP as default value |
| `->virtualAs($expression)`       | Create a virtual generated column (MySQL)                    |
| `->generatedAs($expression)`       | Create an identity column with specified sequence options (PostgreSQL)                 |
| `->always()`       | Defines the precedence of sequence values over input for an identity column (PostgreSQL)                 |

修改字段需要安装拓展包
```
composer require doctrine/dbal
```
```
Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->change();
});

Schema::table('users', function (Blueprint $table) {
    $table->string('name', 50)->nullable()->change();
});

Schema::table('users', function (Blueprint $table) {
    $table->renameColumn('from', 'to');
});    // enum 字段类型不被支持
```
> 可以被修改的书写：bigInteger, binary, boolean, date, dateTime, dateTimeTz, decimal, integer, json, longText, mediumText, smallInteger, string, text, time, unsignedBigInteger, unsignedInteger and unsignedSmallInteger

```
Schema::table('users', function (Blueprint $table) {
    $table->dropColumn('votes');
});

Schema::table('users', function (Blueprint $table) {
    $table->dropColumn(['votes', 'avatar', 'location']);
});
```
> SQLite 不支持在一个迁移内修改或删除多个字段。

| Command                        | Description                                     |
| ------------------------------ | ----------------------------------------------- |
| `$table->dropMorphs('morphable');` | Drop the morphable_id and morphable_type columns.               |
| `$table->dropRememberToken();` | Drop the `remember_token` column.               |
| `$table->dropSoftDeletes();`   | Drop the `deleted_at` column.                   |
| `$table->dropSoftDeletesTz();` | Alias of `dropSoftDeletes()` method.            |
| `$table->dropTimestamps();`    | Drop the `created_at` and `updated_at` columns. |
| `$table->dropTimestampsTz();`  | Alias of `dropTimestamps()` method.             |

#### 索引
```
$table->string('email')->unique();    // 等同于下面的

$table->unique('email');

$table->index(['account_id', 'created_at']);    // 组合索引

$table->unique('email', 'unique_email');    // 自定义索引名称

```

| Command                                 | Description                           |
| --------------------------------------- | ------------------------------------- |
| `$table->primary('id');`                | Adds a primary key.                   |
| `$table->primary(['id', 'parent_id']);` | Adds composite keys.                  |
| `$table->unique('email');`              | Adds a unique index.                  |
| `$table->index('state');`               | Adds a plain index.                   |
| `$table->spatialIndex('location');`     | Adds a spatial index. (except SQLite) |

Command	| Description
-----| ----
$table->dropPrimary('users_id_primary');	| Drop a primary key from the "users" table.
$table->dropUnique('users_email_unique');	| Drop a unique index from the "users" table.
$table->dropIndex('geo_state_index');	| Drop a basic index from the "geo" table.
$table->dropSpatialIndex('geo_location_spatialindex');	| Drop a spatial index from the "geo" table (except SQLite).

删除索引，laravel 自动创建索引名称：`无前缀表名_字段名_索引类型`

`$table->renameIndex('from', 'to')`
```
Schema::table('geo', function (Blueprint $table) {
    $table->dropIndex(['state']); // Drops index 'geo_state_index'
});
```
外键限制
```
Schema::table('posts', function (Blueprint $table) {
    $table->integer('user_id')->unsigned();

    $table->foreign('user_id')->references('id')->on('users');
});
```
```
$table->foreign('user_id')
      ->references('id')->on('users')
      ->onDelete('cascade');
```
类似于索引，删除外键
```
$table->dropForeign('posts_user_id_foreign');

$table->dropForeign(['user_id']);
```
启用禁用外键限制
```
Schema::enableForeignKeyConstraints();

Schema::disableForeignKeyConstraints();
```


## 五、填充
```
php artisan make:seeder UserTableSeeder
```
> 填充时候，大量赋值保护自动失效。

```php
class DatabaseSeeder extends Seeder
{
    public function run()
    {
        DB::table('users')->insert([
            'name' => Str::random(10),
            'email' => Str::random(10).'@gmail.com',
            'password' => bcrypt('password'),
        ]);
    }
}
```

利用模型工厂
```
public function run()
{
    factory(App\User::class, 50)->create()->each(function ($user) {
        $user->posts()->save(factory(App\Post::class)->make());
    });
}
```
调用其他的 seeders
```
public function run()
{
    $this->call([
        UsersTableSeeder::class,
        PostsTableSeeder::class,
        CommentsTableSeeder::class,
    ]);
}
```
#### 运行 seeders
写完你的 seeder，你需要`composer dump-autoload`。
```
php artisan db:seed     // 默认运行 DatabaseSedder 类

php artisan db:seed --class=UsersTableSeeder     // 运行指定类
```
重建你的数据库（之前又提到过哦）
```
php artisan migrate:refresh --seed
```
`php artisan db:seed --force`
## 六、Redis

    composer require predis/predis
    
这个包作者不维护了，未来也会从 laravel 中消失

```
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'default' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_DB', 0),
    ],

    'cache' => [
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD', null),
        'port' => env('REDIS_PORT', 6379),
        'database' => env('REDIS_CACHE_DB', 1),
    ],

],
```
redis 集群
```
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'clusters' => [
        'default' => [
            [
                'host' => env('REDIS_HOST', 'localhost'),
                'password' => env('REDIS_PASSWORD', null),
                'port' => env('REDIS_PORT', 6379),
                'database' => 0,
            ],
        ],
    ],

],
```
如果要想用原生的 redis 集群，请指定 options
```
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    'options' => [
        'cluster' => env('REDIS_CLUSTER', 'redis'),
    ],

    'clusters' => [
        // ...
    ],

],
```
predis
```
'redis' => [

    'client' => env('REDIS_CLIENT', 'predis'),

    // Rest of Redis configuration...
],

'default' => [
    'host' => env('REDIS_HOST', 'localhost'),
    'password' => env('REDIS_PASSWORD', null),
    'port' => env('REDIS_PORT', 6379),
    'database' => 0,
    'read_write_timeout' => 60,
],
```
phpredis
```
'redis' => [

    'client' => env('REDIS_CLIENT', 'phpredis'),

    // Rest of Redis configuration...
],
```
```
'default' => [
    'host' => env('REDIS_HOST', 'localhost'),
    'password' => env('REDIS_PASSWORD', null),
    'port' => env('REDIS_PORT', 6379),
    'database' => 0,
    'read_timeout' => 60,
],
```
使用
```
Redis::get('user:profile:'.$id);
Redis::set('name', 'Taylor');
Redis::lrange('names', 5, 10);
Redis::command('lrange', ['name', 5, 10]);

Redis::connection();
Redis::connection('my-connection');

Redis::pipeline(function ($pipe) {
    for ($i = 0; $i < 1000; $i++) {
        $pipe->set("key:$i", $i);
    }
});
```
#### publish 和 subscribe 

```
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\Redis;

class RedisSubscribe extends Command
{

    protected $signature = 'redis:subscribe';


    protected $description = 'Subscribe to a Redis channel';

    public function handle()
    {
        Redis::subscribe(['test-channel'], function ($message) {
            echo $message;
        });
    }
}

```
```
Redis::publish('test-channel', json_encode(['foo' => 'bar']));
```
```
Redis::psubscribe(['*'], function ($message, $channel) {
    echo $message;
});

Redis::psubscribe(['users.*'], function ($message, $channel) {
    echo $message;
});
```

```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
# 第九章：Eloquent ORM（一、入门。二、模型关联。三、集合。四、Mutators。五、API 资源。六、序列化。）
```
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
## 一、入门
#### 定义模型
```php
php artisan make:model User

php artisan make:model User --migration # 创建一个模型，并且创建他的迁移文件
// 等同于
php artisan make:model User -m
```
```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;

class Flight extends Model {
    protected $table = 'my_flights'; // 默认 flights 
  	protected $primaryKey = 'student_id'; // 默认 id
    public $incrementing = false; // 当你的主键不是自增或不是int类型
    protected $keyType = 'string'; // 当你的主键不是整型
    public $timestamps = false; // 不自动维护created_at 和 updated_at 字段
    # protected $dateFormat = 'U'; // 自定义自己的时间戳格式
    protected $connection = 'connection-name'; // 为模型指定不同的连接

	const CREATED_AT = 'creation_date'; // 自定义用于存储时间戳的字段名
    const UPDATED_AT = 'last_update'; // 同上
}
```

#### 取模型
```php
$flights = App\Flight::all();
foreach ($flights as $flight) {
    echo $flight->name;
}
```

```php
# 每个 Eloquent 模型都可以当作一个 查询构造器
$flights = App\Flight::where('active', 1)
               ->orderBy('name', 'desc')
               ->take(10)
               ->get();
```
#### 刷新模型

```php
$flight = App\Flight::where('number', 'FR 900')->first();
$freshFlight = $flight->fresh();

$flight = App\Flight::where('number', 'FR 900')->first();
$flight->number = 'FR 456';
$flight->refresh();
$flight->number; // "FR 900"
```

#### 分块结果

```php
$flights = $flights->reject(function ($flight) {
    return $flight->cancelled;
});

Flight::chunk(200, function ($flights) {
    foreach ($flights as $flight) {
        //
    }
});
```

#### 使用游标

`cursor` 允许你使用游标来遍历数据库数据，一次只执行单个查询。在处理大数据量请求时 `cursor` 方法可以大幅度减少内存的使用：

```php
foreach (Flight::where('foo', 'bar')->cursor() as $flight) {
    //
}

$users = App\User::cursor()->filter(function ($user) {
    return $user->id > 500;
});
```

#### 高级子查询

```php
Destination::addSelect(['last_flight' => Flight::select('name')
    ->whereColumn('destination_id', 'destinations.id')
    ->orderBy('arrived_at', 'desc')
    ->latest()
    ->limit(1)
])->get();

Destination::orderByDesc(
    Flight::select('arrived_at')
        ->whereColumn('destination_id', 'destinations.id')
        ->orderBy('arrived_at', 'desc')
        ->latest()
        ->limit(1)
)->get();  // sort all destinations based on when the last flight arrived at that destination
```

#### 取回单个模型／集合

```php
$flight = App\Flight::find(1);

$flight = App\Flight::where('active', 1)->first();
```

```php
$flights = App\Flight::find([1, 2, 3]);
```

#### 未找到异常

`findOrFail` 以及 `firstOrFail` 方法会取回查询的第一个结果。如果没有找到相应结果，则会抛出一个 `Illuminate\Database\Eloquent\ModelNotFoundException`： 

```php
$model = App\Flight::findOrFail(1);

$model = App\Flight::where('legs', '>', 100)->firstOrFail();
```

如果该异常没有被捕获，则会自动返回 HTTP `404` 响应给用户，因此当使用这些方法时，你没有必要明确编写检查来返回 `404` 响应：

```php
Route::get('/api/flights/{id}', function ($id) {
    return App\Flight::findOrFail($id);
});

```
#### 聚合函数
```php
// 聚合函数返回标量值
$count = App\Flight::where('active', 1)->count();
$max = App\Flight::where('active', 1)->max('price');
```

#### 添加和更新模型

        $flight = new Flight;
        $flight->name = $request->name;
        $flight->save();
    
        $flight = App\Flight::find(1);
        $flight->name = 'New Flight Name';
        $flight->save();

```php
# 批量更新
App\Flight::where('active', 1)
          ->where('destination', 'San Diego')
          ->update(['delayed' => 1]);
```
> 当通过“Eloquent”批量更新时，`saving`, `saved`, `updating`, and updated 模型事件将不会被更新后的模型触发。这是因为批量更新时，模型从来没有被取回。

#### 批量赋值

要先在你的模型上定义一个 `fillable` 或 `guarded` 属性

```php
protected $fillable = ['name']; # name 可以赋值
$flight = App\Flight::create(['name' => 'Flight 10']);
# 或者已有实例
$flight->fill(['name' => 'Flight 22']);
```

`$guarded` 属性应该包含一个你不想要被批量赋值的属性数组

用的时候应该只选择 `$fillable` 或 `$guarded` 中的其中一个。

```php
protected $guarded = ['price'];
# 除了 price 所有的属性都可以被批量赋值
```

```php
# 如果你想让所有的属性都可以被批量赋值，你应该定义 $guarded为空数组。
protected $guarded = [];
```

### firstOrNew 、 firstOrCreate、updateOrCreate

`firstOrCreate` 等价于 `firstOrNew + save()`

```php
$flight = App\Flight::firstOrNew(['name' => 'Flight 10']);     // 如果没找出来就返回实例。但并不存于数据库。
$flight = App\Flight::firstOrCreate(['name' => 'Flight 10']);

// Retrieve by name, or instantiate with the name, delayed, and arrival_time attributes...
$flight = App\Flight::firstOrNew(
    ['name' => 'Flight 10'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

// Retrieve flight by name, or create it with the name and delayed attributes...
$flight = App\Flight::firstOrCreate(
    ['name' => 'Flight 10'],
    ['delayed' => 1, 'arrival_time' => '11:30']
);

$flight = App\Flight::updateOrCreate(
    ['departure' => 'Oakland', 'destination' => 'San Diego'],
    ['price' => 99]
);
// 如果有从 Oakland 到 San Diego 的航班，设置价格 99，如果没有，那就设置一个 99 元的从 Oakland 到 San Diego 的航班。
```

### 删除模型

```php
$flight = App\Flight::find(1);	// 取回模型再删除
$flight->delete();
// 或者
App\Flight::destroy(1);		// 直接删除
App\Flight::destroy([1, 2, 3]);
App\Flight::destroy(1, 2, 3);
App\Flight::destroy(collect([1, 2, 3]));

$deletedRows = App\Flight::where('active', 0)->delete();
```

当使用 Eloquent 批量删除语句时，`deleting` 和 `deleted` 模型事件不会在被删除模型实例上触发。因为删除语句执行时，不会检索模型实例。

### 软删除

    use SoftDeletes;
    protected $dates = ['deleted_at'];


创建 deleted_at 字段

    Schema::table('flights', function ($table) {
        $table->softDeletes();
    });


启用软删除的模型时，被软删除的模型将会自动从所有查询结果中排除。

要确认指定的模型实例是否已经被软删除

    if ($flight->trashed()) {
         //
    }


查询包含被软删除的模型

```php
$flights = App\Flight::withTrashed()
                ->where('account_id', 1)
                ->get();

# withTrashed 方法也可以被用在 关联 查询：
$flight->history()->withTrashed()->get();
```
只取出软删除数据
```php
$flights = App\Flight::onlyTrashed()
                ->where('airline_id', 1)
                ->get();
```
恢复软删除的模型
```php
$flight->restore();

App\Flight::withTrashed()
        ->where('airline_id', 1)
        ->restore();

$flight->history()->restore(); // 可以被用在 关联 查询
```
永久删除模型

```php
// 强制删除单个模型实例...
$flight->forceDelete();

// 强制删除所有相关模型...
$flight->history()->forceDelete();
```

#### 全局作用域

```php
# 可以自由在  app 文件夹下创建 Scopes 文件夹来存放
<?php
namespace App\Scopes;
use Illuminate\Database\Eloquent\Scope;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;

class AgeScope implements Scope {
    public function apply(Builder $builder, Model $model)
    {
        $builder->where('age', '>', 200);
    }   // 使用 addSelect 而不是 select,可以避免覆盖
}

# 需要重写给定模型的 boot 方法并使用 addGlobalScope 方法
<?php
namespace App;
use App\Scopes\AgeScope;
use Illuminate\Database\Eloquent\Model;
class User extends Model {
    protected static function boot() {
        parent::boot();

        static::addGlobalScope(new AgeScope);
    }
}
```

```sql
# 添加作用域后，如果使用 User::all() 查询则会生成如下SQL语句：
select * from `users` where `age` > 200
```

匿名全局作用域（专门用来处理单个模型）

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Builder;
class User extends Model {
    protected static function boot() {
        parent::boot();

        static::addGlobalScope('age', function (Builder $builder) {
            $builder->where('age', '>', 200);
        });
    }
}
```

移除全局作用域

```php
# 我们还可以通过以下方式，利用 age 标识符来移除全局作用：
User::withoutGlobalScope('age')->get();

# 移除指定全局作用域
User::withoutGlobalScope(AgeScope::class)->get();

# 移除几个或全部全局作用域
User::withoutGlobalScopes([
    FirstScope::class, SecondScope::class
])->get();

User::withoutGlobalScopes()->get();
```

本地作用域

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
class User extends Model {

    public function scopePopular($query) {
        return $query->where('votes', '>', 100);
    }

    public function scopeActive($query) {
        return $query->where('active', 1);
    }
}
```

```php
# 在进行方法调用时不需要加上 scope 前缀
$users = App\User::popular()->active()->orderBy('created_at')->get();

$users = App\User::popular()->orWhere(function (Builder $query) {
    $query->active();
})->get();

$users = App\User::popular()->orWhere->active()->get();
```
动态范围
```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
class User extends Model {
    public function scopeOfType($query, $type)
    {
        return $query->where('type', $type);
    }
}

```

现在，你可以在范围调用时传递参数：

```php
$users = App\User::ofType('admin')->get();

```

比较模型

```php
if ($post->is($anotherPost)) {
    //
}
```

#### 事件

```php
retrieved, creating, created, updating, updated, 
saving, saved, deleting, deleted,  restoring, restored.
```

模型被取出的时候触发 retrieved。

 当一个新模型被初次保存将会触发 `creating` 以及 `created` 事件。如果一个模型已经存在于数据库且调用了 `save` 方法，将会触发 `updating` 和 `updated` 事件。在这两种情况下都会触发 `saving` 和 `saved` 事件。

```php
<?php

namespace App;

use App\Events\UserSaved;
use App\Events\UserDeleted;
use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;

class User extends Authenticatable
{
    use Notifiable;

    protected $dispatchesEvents = [
        'saved' => UserSaved::class,
        'deleted' => UserDeleted::class,
    ];
}
```
还可以这么玩

```php
php artisan make:observer UserObserver --model=User
```

```php
<?php

namespace App\Observers;

use App\User;

class UserObserver
{
    public function created(User $user)
    {
        //
    }

    public function updated(User $user)
    {
        //
    }

    public function deleted(User $user)
    {
        //
    }
}
```

```php
<?php

namespace App\Providers;

use App\User;
use App\Observers\UserObserver;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{

    public function boot()
    {
        User::observe(UserObserver::class);
    }


    public function register()
    {
        //
    }
}
```

## 二、模型关联
#### **一对一**
**正向关联**
一个 `User` 模型关联一个 `Phone` 模型。
```php
# class User extends Model
public function phone() {
    return $this->hasOne('App\Phone');
}

public function phone() {
    # 它会自动假设 Phone 模型拥有 user_id 外键。如果你想要重写这个约定，则可以传入第二个参数到 hasOne 方法里。
    return $this->hasOne('App\Phone', 'foreign_key');
}

public function phone() {
    # Eloquent 假设外键会和上层模型的 id 字段（或者自定义的 $primaryKey）的值相匹配。否则，请
    return $this->hasOne('App\Phone', 'foreign_key', 'local_key');
}
# 允许定义默认模型
			
# 使用
$phone = User::find(1)->phone;  # 当找不到 id=1 的 phone 记录，会查看是否有默认模型，没有则返回 null。
// select * from `by_users` where `by_users`.`id` = '1' limit 1
// select * from `by_phones` where `by_phones`.`user_id` = '1' and `by_phones`.`user_id` is not null limit 1
$email = App\Phone::where('content', '13334233434')->first()->user->email;
// select * from `by_phones` where `content` = '13334233434' limit 1
// select * from `by_users` where `by_users`.`id` = '1' limit 1
```

**反向关联**

```php
# class Phone extends Model 
public function user() {
    return $this->belongsTo('App\User');
    // return $this->belongsTo('App\User', 'foreign_key');
    // return $this->belongsTo('App\User', 'foreign_key', 'other_key');
}

// 使用：同正向关联
```
**关联默认模型**
`belongsTo，haoOne` 允许定义默认模型
```php
public function user()
{
    return $this->belongsTo('App\User')->withDefault(); // 空模型
    # 或者
    return $this->belongsTo('App\User')->withDefault([
        'name' => '游客',
    ]);
    # 或者
    return $this->belongsTo('App\User')->withDefault(function ($user) {
        $user->name = '游客';
    });
}
```
#### **一对多**
**正向关联**
一篇博客文章可能会有无限多个评论。

```php
# class Post extends Model 
public function comments() {
    return $this->hasMany('App\Comment');
    # return $this->hasMany('App\Comment', 'foreign_key');
    # return $this->hasMany('App\Comment', 'foreign_key', 'local_key');
}

# Controller
$comments = App\Post::find(1)->comments;  // 返回满足条件的评论的集合
// select * from `by_posts` where `by_posts`.`id` = '1' limit 1
// select * from `by_comments` where `by_comments`.`post_id` = '1' and `by_comments`.`post_id` is not null
foreach ($comments as $comment) {
    //
}

$comments = App\Post::find(1)->comments()->where('title', 'foo')->first();

$comment = App\Comment::find(1);
echo $comment->post->title;
```
**反向关联**
```php
// 同一对一的反向关联 ：语义化为 ( 一条评论只属于一篇文章 )
return $this->belongsTo('App\Post');
```
#### **多对多**
**正向关联**
一个用户可能拥有多种身份，而一种身份能同时被多个用户拥有。

需要使用三个数据表：`users`、`roles` 和 `role_user`。`role_user` 表命名是以相关联的两个模型数据表来**依照字母顺序**命名，并包含了 `user_id` 和 `role_id` 字段。

```php
# class User extends Model
public function roles() {
    return $this->belongsToMany('App\Role');
}

// 使用：等同于 hasMany
$roles = App\User::find(1)->roles;
// select * from `by_users` where `by_users`.`id` = '1' limit 1
/* 
select 
`by_roles`.*, 
`by_role_user`.`user_id` as `pivot_user_id`, 
`by_role_user`.`role_id` as `pivot_role_id` 
from 
`by_roles` inner join `by_role_user` 
on 
`by_roles`.`id` = `by_role_user`.`role_id` 
where 
`by_role_user`.`user_id` = '1'
*/
```

如前文提到那样，Eloquent 会合并两个关联模型的名称并依照字母顺序命名。当然你也可以随意重写这个约定。可通过传递第二个参数至 `belongsToMany` 方法来实现：

```php
return $this->belongsToMany('App\Role', 'role_user');

return $this->belongsToMany('App\Role', 'role_user', 'user_id', 'role_id');
```

**反向关联**
```php
<?php
# class Role extends Model 
public function users() {
    return $this->belongsToMany('App\User'); 
}
```
**操作中间表**

获取中间表字段
```php
$user = App\User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}

# 我们取出的每个 Role 模型对象，都会被自动赋予 pivot 属性。
```

```php
# 默认情况下，pivot 对象只提供模型的键。
# 如果你的 pivot 数据表包含了其它的属性，则可以在定义关联方法时指定那些字段：

return $this->belongsToMany('App\Role')->withPivot('column1', 'column2');
```

```php
# 如果你想要中间表自动维护 created_at 和 updated_at 时间戳，
# 可在定义关联方法时加上  withTimestamps 方法：

return $this->belongsToMany('App\Role')->withTimestamps();
```
自定义 pivot 别名
```php
return $this->belongsToMany('App\Podcast')
                ->as('subscription')
                ->withTimestamps();

$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```
通过中间表过滤
```php
return $this->belongsToMany('App\Role')->wherePivot('approved', 1);

return $this->belongsToMany('App\Role')->wherePivotIn('priority', [1, 2]);
```
自定义中间表模型
```php
# 原中间表应该是 RoleUser
public function users()
{
    return $this>belongsToMany('App\User')->using('App\UserRole');
}
```
```php
namespace App;

use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    //
}
```
```php
class Role extends Model
{

If you have defined a many-to-many relationship that uses a custom pivot model, and that pivot model has an auto-incrementing primary key, you should ensure your custom pivot model class defines an incrementing property that is set to true.

/**
 * Indicates if the IDs are auto-incrementing.
 *
 * @var bool
 */
public $incrementing = true;public function users()
    {
        return $this->belongsToMany('App\User')
                        ->using('App\RoleUser')
                        ->withPivot([
                            'created_by',
                            'updated_by'
                        ]);
    }
}
```
If you have defined a many-to-many relationship that uses a custom pivot model, and that pivot model has an auto-incrementing primary key, you should ensure your custom pivot model class defines an incrementing property that is set to true.

`public $incrementing = true;`

#### **远层一对多一**
```php
# 如果每个供应商都有一个用户，并且每个用户与一个用户历史记录相关联
users
    id - integer
    supplier_id - integer

suppliers
    id - integer

history
    id - integer
    user_id - integer
    
    
class Supplier extends Model
{
    public function userHistory()
    {
        return $this->hasOneThrough('App\History', 'App\User');
    }
}

class Supplier extends Model
{
    public function userHistory()
    {
        return $this->hasOneThrough(
            'App\History',
            'App\User',
            'supplier_id', // Foreign key on users table...
            'user_id', // Foreign key on history table...
            'id', // Local key on suppliers table...
            'id' // Local key on users table...
        );
    }
}
```
#### **远层一对多**

一个 `Country` 模型可能通过中间的 `Users` 模型关联到多个 `Posts` 模型。

```php
countries
    id - integer
    name - string

users
    id - integer
    country_id - integer
    name - string

posts
    id - integer
    user_id - integer
    title - string
```


```php
class Country extends Model
{
    public function posts()
    {
        return $this->hasManyThrough(
            'App\Post',     # 最终要关联的模型
            'App\User',
            // 'country_id',   // Foreign key on users table...
            // 'user_id',      // Foreign key on posts table...
            // 'id',           // Local key on countries table...
            // 'id'            // Local key on users table...
        );
    }
}
```
```php
$posts = App\Country::find(1)->posts;
// select * from `by_countries` where `by_countries`.`id` = '1' limit 1
// select `by_posts`.*, `by_users`.`country_id` from `by_posts` inner join `by_users` on `by_users`.`id` = `by_posts`.`user_id` where `by_users`.`country_id` = '1'
```
#### 多态一对一
```php
posts
    id - integer
    name - string

users
    id - integer
    name - string

images
    id - integer
    url - string
    imageable_id - integer
    imageable_type - string
    
class Image extends Model
{
    public function imageable()
    {
        return $this->morphTo();
    }
}

class Post extends Model
{
    public function image()
    {
        return $this->morphOne('App\Image', 'imageable');
    }
}

class User extends Model
{
    public function image()
    {
        return $this->morphOne('App\Image', 'imageable');
    }
}

$post = App\Post::find(1);
$image = $post->image;

$image = App\Image::find(1);
$imageable = $image->imageable;
```
#### 多态关联

用户可以「评论」文章和视频。
**数据库设计**
```
posts
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
    commentable_type - string   // posts or videos
```
**定义关联**
```php
namespace App;
use Illuminate\Database\Eloquent\Model;
class Comment extends Model {
    /**
     * 获取所有拥有的 commentable 模型。
     */
    public function commentable() {
        return $this->morphTo();
    }
}

class Post extends Model {
    /**
     * 获取所有文章的评论。
     */
    public function comments() {
        return $this->morphMany('App\Comment', 'commentable');
    }
}

class Video extends Model {
    /**
     * 获取所有视频的评论。
     */
    public function comments(){
        return $this->morphMany('App\Comment', 'commentable');
    }
}
```
**使用**
```php
$post = App\Post::find(1);

foreach ($post->comments as $comment) {
    //
}
```

```php
$comment = App\Comment::find(1);

$commentable = $comment->commentable;
# 返回 Post 或 Video 实例
```

**自定义多态关联的类型字段**

```php
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::morphMap([
    'posts' => App\Post::class,
    'videos' => App\Video::class,
]);
# 你需要在你自己的 AppServiceProvider 中的 boot 函数注册这个 morphMap ，或者创建一个独立且满足你要求的服务提供者。
```

#### 多态多对多关联
**数据库设计**
博客的 `Post` 和 `Video` 模型可以共用多态关联至 `Tag` 模型。post 有多个标签，一个标签有多个 post 。

```
posts
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
**正向关联**
```php
namespace App;
use Illuminate\Database\Eloquent\Model;
class Post extends Model{
    /**
     * 获取该文章的所有标签。
     */
    public function tags(){
        return $this->morphToMany('App\Tag', 'taggable');
    }
}
```
**反向关联**
```php
namespace App;
use Illuminate\Database\Eloquent\Model;
class Tag extends Model{
    /**
     * 获取所有被赋予该标签的文章。
     */
    public function posts(){
        return $this->morphedByMany('App\Post', 'taggable');
    }

    /**
     * 获取所有被赋予该标签的视频。
     */
    public function videos(){
        return $this->morphedByMany('App\Video', 'taggable');
    }
}
```
**使用**
```php
$post = App\Post::find(1);
foreach ($post->tags as $tag) {
    //
}
```

```php
$tag = App\Tag::find(1);
foreach ($tag->videos as $video) {
    //
}
```
#### 自定义多态类型
```php
# AppServiceProvider 或者新建一个服务提供者
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::morphMap([
    'posts' => 'App\Post',
    'videos' => 'App\Video',
]);
```
`向现有应用程序添加“变形图”时，数据库中仍包含完全限定类的每个morphable *_type列值都需要转换为其“map”名称。`

#### 查找关联

**查找关联是否存在**

```php
// 获取那些至少拥有一条评论的文章...
$posts = App\Post::has('comments')->get();
// select * from `by_posts` where exists (select * from `by_comments` where `by_posts`.`id` = `by_comments`.`post_id`)
$posts = App\Post::doesntHave('comments')->get();
// select * from `by_posts` where not exists (select * from `by_comments` where `by_posts`.`id` = `by_comments`.`post_id`)
```

```php
// 获取所有至少有三条评论的文章...
$posts = Post::has('comments', '>=', 3)->get();
// select * from `by_posts` where (select count(*) from `by_comments` where `by_posts`.`id` = `by_comments`.`post_id`) > 4
```

```php
// 获取所有至少有一张卡的手机的用户...
$posts = User::has('phones.cards')->get(); 
// select * from `by_users` where exists (select * from `by_phones` where `by_users`.`id` = `by_phones`.`user_id` and exists (select * from `by_cards` where `by_phones`.`id` = `by_cards`.`phone_id`))
```

```php
# 如果你想要更高级的用法，则可以使用 whereHas 和 orWhereHas 方法，在 has 查找里设置「where」条件。此方法可以让你增加自定义条件至关联条件中，例如对评论内容进行检查：
//  获取那些至少有一条评论包含 foo 的文章
$posts = Post::whereHas('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();
// select * from `by_posts` where exists (select * from `by_comments` where `by_posts`.`id` = `by_comments`.`post_id` and `content` like 'foo%')

$posts = Post::whereDoesntHave('comments', function ($query) {
    $query->where('content', 'like', 'foo%');
})->get();

// 所有未被禁止的作者评论的帖子
$posts = App\Post::whereDoesntHave('comments.author', function (Builder $query) {
    $query->where('banned', 1);
})->get();
```
#### 查询多态关联
```php
// Retrieve comments associated to posts or videos with a title like foo%...
$comments = App\Comment::whereHasMorph(
    'commentable', 
    ['App\Post', 'App\Video'], 
    function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    }
)->get();

// Retrieve comments associated to posts with a title not like foo%...
$comments = App\Comment::whereDoesntHaveMorph(
    'commentable', 
    'App\Post', 
    function (Builder $query) {
        $query->where('title', 'like', 'foo%');
    }
)->get();    

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

$comments = App\Comment::whereHasMorph('commentable', '*', function (Builder $query) {
    $query->where('title', 'like', 'foo%');
})->get();
```


#### 关联数据计数

```php
# 如果你想对关联数据进行计数,请使用 withCount 方法，此方法会在你的结果集中增加一个 {relation}_count 字段
$posts = App\Post::withCount('comments')->get();

foreach ($posts as $post) {
    echo $post->comments_count;
}



```

```php
$posts = Post::withCount(['votes', 'comments' => function ($query) {
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
    }
])->get();

echo $posts[0]->comments_count;

echo $posts[0]->pending_comments_count;
```

```php
$posts = App\Post::select(['title', 'body'])->withCount('comments')->get();

echo $posts[0]->title;
echo $posts[0]->body;
echo $posts[0]->comments_count;
```

#### 预加载

```php
$books = App\Book::all();

foreach ($books as $book) {
    echo $book->author->name;
}
// 若存在着 25 本书，则循环就会执行 26 次查找：1 次是查找所有书籍，其它 25 次则是在查找每本书的作者。
```

```php
$books = App\Book::with('author')->get();

foreach ($books as $book) {
    echo $book->author->name;
}
// 对于该操作则只会执行两条 SQL 语句：
# select * from books
# select * from authors where id in (1, 2, 3, 4, 5, ...)
```


```php
$books = App\Book::with(['author', 'publisher'])->get();
```

```php
$books = App\Book::with('author.contacts')->get();
# 预加载所有书籍的作者，及所有作者的个人联系方式
```
```php 
$users = App\Book::with('author:id,name')->get();   
```
> with  这个方法，应该总是包含 id 字段。

morphTo 关系模型嵌套延迟加载
```php

class ActivityFeed extends Model
{
    public function parentable()
    {
        return $this->morphTo();
    }
}

$activities = ActivityFeed::query()
    ->with(['parentable' => function (MorphTo $morphTo) {
        $morphTo->morphWith([
            Event::class => ['calendar'],
            Photo::class => ['tags'],
            Post::class => ['author'],
        ]);
    }])->get();
```

默认预加载
```php
protected $with = ['author'];

public function author()
{
	return $this->belongsTo('App\Author');
}

$books = App\Book::without('author')->get();
```

#### 预加载条件限制

```php
$users = App\User::with(['posts' => function ($query) {
    $query->where('title', 'like', '%first%');
}])->get();
# 在这个例子里，Eloquent 只会预加载标题包含 first 的文章
```

```php
$users = App\User::with(['posts' => function ($query) {
    $query->orderBy('created_at', 'desc');
}])->get();


// 不能在限制中使用 limit 和 take 
```

#### 延迟预加载

有时你可能需要在上层模型被获取后才预加载关联。

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
```php
public function format(Book $book)
{
    $book->loadMissing('author');     // 当没有加载时候，加载

    return [
        'name' => $book->name,
        'author' => $book->author->name
    ];
}
```
morphTo

```php
class ActivityFeed extends Model
{
    public function parentable()
    {
        return $this->morphTo();
    }
}

$activities = ActivityFeed::with('parentable')
    ->get()
    ->loadMorph('parentable', [
        Event::class => ['calendar'],
        Photo::class => ['tags'],
        Post::class => ['author'],
    ]);
```
#### 插入 & 更新关联模型

**save 方法**

```php
$comment = new App\Comment(['message' => 'A new comment.']);

$post = App\Post::find(1);

$post->comments()->save($comment);
```

```php
$post = App\Post::find(1);

$post->comments()->saveMany([
    new App\Comment(['message' => 'A new comment.']),
    new App\Comment(['message' => 'Another comment.']),
]);


$post = App\Post::find(1);
$post->comments[0]->message = 'Message';
$post->comments[0]->author->name = 'Author Name';
$post->push();
```

**create 方法**

```php
$post = App\Post::find(1);

$comment = $post->comments()->create([
    'message' => 'A new comment.',
]);

// create 多个
$comment = $post->comments()->create(
[
    'message' => 'A new comment.',
],[
    'message' => 'Another new comment.',
]
);

```

`save` 允许传入一个完整的 Eloquent 模型实例，但 `create` 只允许传入原始的 PHP 数组。

你需要先在你的模型上定义一个 `fillable` 或 `guarded` 属性，因为所有的 Eloquent 模型都针对批量赋值（Mass-Assignment）做了保护。

**更新「从属」关联**

```php
$account = App\Account::find(10);

$user->account()->associate($account);

$user->save();
```



删除一个 `belongsTo` 关联时，使用 `dissociate` 方法会置该关联的外键为空 (null) 。

```php
$user->account()->dissociate();

$user->save();
```
**多对多关联**

一个用户可以拥有多个身份，且每个身份都可以被多个用户拥有。

附加一个规则至一个用户，并连接模型以及将记录写入至中间表，则可以使用 `attach` 方法：

```php
$user = App\User::find(1);

$user->roles()->attach($roleId);
```

也可以传递一个需被写入至中间表的额外数据数组：

```php
$user->roles()->attach($roleId, ['expires' => $expires]);
```

```php
// 移除用户身上某一身份...
$user->roles()->detach($roleId);

// 移除用户身上所有身份...
$user->roles()->detach();
```

```php
# 为了方便，attach 与 detach 都允许传入 ID 数组：
$user = App\User::find(1);

$user->roles()->detach([1, 2, 3]);

$user->roles()->attach([
    1 => ['expires' => $expires],
    2 => ['expires' => $expires]
]);
```

**同步关联**

```php
# sync 方法可以用数组形式的 IDs 插入中间的数据表。任何一个不存在于给定数组的 IDs 将会在中间表内被删除。操作完成之后，只有那些在给定数组内的 IDs 会被保留在中间表中。
$user->roles()->sync([1, 2, 3]);
```

```php
$user->roles()->sync([1 => ['expires' => true], 2, 3]);
```

如果你不想删除现有的 IDs ，你可以 `syncWithoutDetaching` 方法：

```php
$user->roles()->syncWithoutDetaching([1, 2, 3]);
```
切换关联
```php
// 存在就删除，不存在就创建
$user->roles()->toggle([1, 2, 3]);
```
在中间表上保存额外数据

```php
App\User::find(1)->roles()->save($role, ['expires' => $expires]);
```

更新中间表记录

```php
# 这个方法接收中间记录的外键和属性数组进行更新
$user = App\User::find(1);

$user->roles()->updateExistingPivot($roleId, $attributes);
```




**连动父级时间戳**

当一个模型 `belongsTo` 或 `belongsToMany` 另一个模型时，像是一个 `Comment` 属于一个 `Post`。这对于子级模型被更新时，要更新父级的时间戳相当有帮助。举例来说，当一个 `Comment` 模型被更新时，你可能想要「连动」更新 `Post` 所属的 `updated_at` 时间戳。

```php
<?php
namespace App;
use Illuminate\Database\Eloquent\Model;
class Comment extends Model{
    /**
     * 所有的关联将会被连动。
     *
     * @var array
     */
    protected $touches = ['post'];

    /**
     * 获取拥有此评论的文章。
     */
    public function post(){
        return $this->belongsTo('App\Post');
    }
}
```

```php
$comment = App\Comment::find(1);
$comment->text = 'Edit to this comment!';
$comment->save();
```
## 三、集合

大多数 eloquent 集合方法都返回一个 eloquent 集合。`pluck, keys, zip, collapse, flatten, flip`方法返回基础集合。`map`操作不包含 eloquent 模型数据时，返回基础集合。

集合方法列表
[https://laravel.com/docs/6.0/eloquent-collections#available-methods](https://laravel.com/docs/6.0/eloquent-collections#available-methods)
all
average
avg
chunk
collapse
combine
concat
contains
containsStrict
count
crossJoin
dd
diff
diffAssoc
diffKeys
dump
each
eachSpread
every
except
filter
first
firstWhere
flatMap
flatten
flip
forget
forPage
get
groupBy
has
implode
intersect
intersectByKeys
isEmpty
isNotEmpty
keyBy
keys
last
macro
make
map
mapInto
mapSpread
mapToGroups
mapWithKeys
max
median
merge
min
mode
nth
only
pad
partition
pipe
pluck
pop
prepend
pull
push
put
random
reduce
reject
reverse
search
shift
shuffle
slice
sort
sortBy
sortByDesc
sortKeys
sortKeysDesc
splice
split
sum
take
tap
times
toArray
toJson
transform
union
unique
uniqueStrict
unless
unwrap
values
when
where
whereStrict
whereIn
whereInStrict
whereInstanceOf
whereNotIn
whereNotInStrict
wrap
zip

自定义集合
```
<?php

namespace App;

use App\CustomCollection;
use Illuminate\Database\Eloquent\Model;

class User extends Model
{
    public function newCollection(array $models = [])
    {
        return new CustomCollection($models);
    }
}
```
## 四、Mutators
Accessor 和 Mutators 允许你在取得或者设置模型实例时格式化 eloquent 属性值。
```
class User extends Model
{
    public function getFirstNameAttribute($value)
    {
        return ucfirst($value);
    }
}

# $user = App\User::find(1);
# $firstName = $user->first_name;
```
```
public function getFullNameAttribute()
{
    return "{$this->first_name} {$this->last_name}";
}
```
```
class User extends Model
{
    public function setFirstNameAttribute($value)
    {
        $this->attributes['first_name'] = strtolower($value);
    }
}

# $user = App\User::find(1);
# $user->first_name = 'Sally';
```
**日期 Mutators**

    protected $dates = [
         'seen_at'
    ];

```
$user = App\User::find(1);
$user->deleted_at = now();
$user->save();

$user = App\User::find(1);
return $user->deleted_at->getTimestamp();
```
```php
    protected $dateFormat = 'U';

```
**属性类型强制转换**
支持的类型：`integer, real, float,  double,decimal:<digits>,  string, boolean, object, array, collection, date, datetime,  timestamp`。
```
class User extends Model
{
    protected $casts = [
        'is_admin' => 'boolean',
    ];
}
```
```
$user = App\User::find(1);

if ($user->is_admin) {     // bool
    //
}
```
```
class User extends Model
{
    protected $casts = [
        'options' => 'array',
    ];
}
```
```
$user = App\User::find(1);

$options = $user->options;    // 自动从 json 类型转化成数组

$options['key'] = 'value';

$user->options = $options;

$user->save();
```
```php
protected $casts = [
    'created_at' => 'datetime:Y-m-d',
];
```

## 五、API 资源
#### 生产资源
    php artisan make:resource UserResource
    
    php artisan make:resource Users  --collection
    php artisan make:resource UserCollection  // 同上

#### 核心概念
**resource**
```
<?php
namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\Resource;

class UserResource extends Resource
{
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'email' => $this->email,
            'created_at' => $this->created_at,
            'updated_at' => $this->updated_at,
        ];
    }
}
```
```
use App\User;
use App\Http\Resources\UserResource;

Route::get('/user', function () {
    return new UserResource(User::find(1));
});
```
**resourceCollection**
```
use App\User;
use App\Http\Resources\UserResource;

Route::get('/user', function () {
    return UserResource::collection(User::all());
});
```
如果需要定制返回集合的内容

    php artisan make:resource UserCollection
```
<?php
namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class UserCollection extends ResourceCollection
{
    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
}
```
```
use App\User;
use App\Http\Resources\UserCollection;

Route::get('/users', function () {
    return new UserCollection(User::all());
});
```
```php
// you may add a preserveKeys property to your resource class indicating if collection keys should be preserved
public $preserveKeys = true;

return UserResource::collection(User::all()->keyBy->id);
```
#### 编写资源
最简单的用法就是上面 resource 部分。
接下来看个关系资源
```
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => Post::collection($this->posts),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}


Route::get('/user', function () {
    return new UserResource(User::find(1));
});

Route::get('/user', function () {
    return UserResource::collection(User::all());
});

    public function toArray($request)
    {
        return [
            'data' => $this->collection,
            'links' => [
                'self' => 'link-value',
            ],
        ];
    }
```
**data 包裹**
默认资源响应转化成 json 会被 data 键包裹。
```
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ]
}
```
禁止包裹

    # AppServiceProvider
    public function boot()
    {
        Resource::withoutWrapping();
    }
> withoutWrapping 不会处理你手动添加到资源集合中的包裹。

**包裹嵌套资源**
```
class CommentsCollection extends ResourceCollection
{
    public function toArray($request)
    {
        return ['data' => $this->collection];
    }
}
```
**数据嵌套和分页**
当在资源响应返回分页集合的时候，laravel 会忽略 withoutWrapping 方法的效果，给你的资源数据加上 data 键。
```
{
    "data": [
        {
            "id": 1,
            "name": "Eladio Schroeder Sr.",
            "email": "therese28@example.com",
        },
        {
            "id": 2,
            "name": "Liliana Mayert",
            "email": "evandervort@example.com",
        }
    ],
    "links":{
        "first": "http://example.com/pagination?page=1",
        "last": "http://example.com/pagination?page=1",
        "prev": null,
        "next": null
    },
    "meta":{
        "current_page": 1,
        "from": 1,
        "last_page": 1,
        "path": "http://example.com/pagination",
        "per_page": 15,
        "to": 10,
        "total": 10
    }
}
```
**分页**
```
use App\User;
use App\Http\Resources\UserCollection;

Route::get('/users', function () {
    return new UserCollection(User::paginate());
});
```
**条件属性**
```
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'secret' => $this->when(Auth::user()->isAdmin(), 'secret-value'),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}

'secret' => $this->when(Auth::user()->isAdmin(), function () {
    return 'secret-value';
}),
```
多个属性取决于某个条件
```
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        $this->mergeWhen(Auth::user()->isAdmin(), [
            'first-secret' => 'value',
            'second-secret' => 'value',
        ]),
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```
> mergeWhen 里的数组不能又包含字符串作为索引，又包含数字作为索引。也不能存在乱序的数字索引。

**条件关联**
whenLoaded 的参数是关系的名称，而不是关系本身。避免了 n+1 的问题。
```
public function toArray($request)
{
    return [
        'id' => $this->id,
        'name' => $this->name,
        'email' => $this->email,
        'posts' => Post::collection($this->whenLoaded('posts')),
        # 如果没有加载 posts，响应将移除 posts 键
        'created_at' => $this->created_at,
        'updated_at' => $this->updated_at,
    ];
}
```
中间表数据

    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoaded('role_user', function () {
            return $this->pivot->expires_at;
        }),
    ];

**添加 meta 数据**

    return [
        'id' => $this->id,
        'name' => $this->name,
        'expires_at' => $this->whenPivotLoadedAs('subscription', 'role_user', function () {
            return $this->subscription->expires_at;
        }),
    ];

顶层 meta 数据

    # UserCollection 
    public function toArray($request)
    {
        return parent::toArray($request);
    }


    public function with($request)
    {
        return [
            'meta' => [
                'key' => 'value',
            ],
        ];
    }

添加顶层 meta 数据（控制器、路由中）

	return (new UserCollection(User::all()->load('roles')))
                ->additional(['meta' => [
                    'key' => 'value',
                ]]);
## 资源响应
如之前提到的，可以在路由和控制器中直接返回资源。
可以通过 reponse 自定义响应
```
Route::get('/user', function () {
    return new UserResource(User::find(1));
});

Route::get('/user', function () {
    return (new UserResource(User::find(1)))
                ->response()
                ->header('X-Value', 'True');
});
```
还可以在 UserResource 中统一自定义响应

    public function toArray($request)
    {
        return [
            'id' => $this->id,
        ];
    }
    
    public function withResponse($request, $response)
    {
        $response->header('X-Value', 'True');
    }

## 六、序列化
toArray 是递归的转化为数组的，即使里面又关联模型数据。
    
    $user = App\User::with('roles')->first();
    return $user->toArray();

```php
# 一个模型
$user = App\User::first();
return $user->attributesToArray();

$users = App\User::all();
return $users->toArray();
```
toJson 也是递归的

    $user = App\User::find(1);
    return $user->toJson();

隐式调用 toJson

    $user = App\User::find(1);
    return (string) $user;
    # 或者直接 return
    return App\User::all();

#### 在 json 中隐藏属性
```
class User extends Model
{
    protected $hidden = ['password'];     # 隐藏关联模型时，使用关联模型的方法名
}
```
```
class User extends Model
{
    protected $visible = ['first_name', 'last_name'];
}
```
临时修改属性的可见性

    return $user->makeVisible('attribute')->toArray();
    return $user->makeHidden('attribute')->toArray();


**在 json 中添加值**
```
class User extends Model
{
    public function getIsAdminAttribute()
    {
        return $this->attributes['admin'] == 'yes';
    }
}

# 还需要添加属性 $appends
class User extends Model
{
    protected $appends = ['is_admin'];
}
```
实时添加
```
return $user->append('is_admin')->toArray();

return $user->setAppends(['is_admin'])->toArray();    // 覆盖 appends 数组
```


#### 日期序列化
```php
protected $casts = [
    'birthday' => 'date:Y-m-d',
    'joined_at' => 'datetime:Y-m-d H:00',
];

# AppServiceProvider 
public function boot()
{
    Carbon::serializeUsing(function ($carbon) {
    	return $carbon->format('U');
    });
}
```

```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
# 第九章：测试（一、入门。二、HTTP 测试。三、浏览器测试。四、数据库。五、模拟。）
```php
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
#########################################################################################
```
## 一、入门
tests 目录包含两个目录：Feature 和 Unit 。Unit 主要是针对简单的、独立的代码，大多数时候是一个单独的方法。Feature 针对的是多一点的代码，包括几个对象交互甚至是整个 http 请求到 json 终端。
```
// Create a test in the Feature directory...
php artisan make:test UserTest

// Create a test in the Unit directory...
php artisan make:test UserTest --unit
```
> 如果你定义自己的 setUp / tearDown 方法，记得调用 parent::setUp() / parent::tearDown 。
## 二、HTTP 测试
#### 简单介绍
```php
public function testBasicTest()
{
	$response = $this->get('/');

	$response->assertStatus(200);
}

public function testBasicExample()
{
	$response = $this->withHeaders([
		'X-Header' => 'Value',
	])->json('POST', '/user', ['name' => 'Sally']);

	$response
		->assertStatus(201)
		->assertJson([
			'created' => true,
		]);
}

public function testBasicTest()
{
	$response = $this->get('/');

	$response->dumpHeaders();

	$response->dump();
}
```


#### Session 和 认证
```
$response = $this->withSession(['foo' => 'bar'])
                         ->get('/');
```
```
    public function testApplication()
    {
        $user = factory(User::class)->create();

        $response = $this->actingAs($user)
                         ->withSession(['foo' => 'bar'])
                         ->get('/');
    }
```

$this->actingAs($user, 'api')

```
$response = $this->json('POST', '/user', ['name' => 'Sally']);

$response
	->assertStatus(201)
	->assertJson([
		'created' => true,
	]);
```
> assertJson 方法会将响应转换为数组并且利用 PHPUnit::assertArraySubset 方法来验证传入的数组是否在应用返回的 JSON 中。也就是说，即使有其它的属性存在于该 JSON 响应中，但是只要指定的片段存在，此测试仍然会通过。

数组与返回的 JSON 完全匹配，使用 assertExactJson 方法。
```
$response = $this->json('POST', '/user', ['name' => 'Sally']);

$response
	->assertStatus(201)
	->assertExactJson([
		'created' => true,
	]);
```
#### 测试文件上传
```
public function testAvatarUpload()
{
	Storage::fake('avatars');

	$file = UploadedFile::fake()->image('avatar.jpg');

	$response = $this->json('POST', '/avatar', [
		'avatar' => $file,
	]);

	// Assert the file was stored...
	Storage::disk('avatars')->assertExists($file->hashName());

	// Assert a file does not exist...
	Storage::disk('avatars')->assertMissing('missing.jpg');
}
UploadedFile::fake()->image('avatar.jpg', $width, $height)->size(100);
UploadedFile::fake()->create('document.pdf', $sizeInKilobytes);


```
#### assert 相关方法
assert 方法官方文档
[https://laravel.com/docs/6.0/http-tests#response-assertions](https://laravel.com/docs/6.0/http-tests#response-assertions)
assertCookie
assertCookieExpired
assertCookieNotExpired
assertCookieMissing
assertDontSee
assertDontSeeText
assertExactJson
assertForbidden
assertHeader
assertHeaderMissing
assertJson
assertJsonCount
assertJsonFragment
assertJsonMissing
assertJsonMissingExact
assertJsonMissingValidationErrors
assertJsonStructure
assertJsonValidationErrors
assertLocation
assertNotFound
assertOk
assertPlainCookie
assertRedirect
assertSee
assertSeeInOrder
assertSeeText
assertSeeTextInOrder
assertSessionHas
assertSessionHasInput
assertSessionHasAll
assertSessionHasErrors
assertSessionHasErrorsIn
assertSessionHasNoErrors
assertSessionDoesntHaveErrors
assertSessionMissing
assertStatus
assertSuccessful
assertUnauthorized
assertViewHas
assertViewHasAll
assertViewIs
assertViewMissing

Method|	Description
--|--
$this->assertAuthenticated($guard = null);	|Assert that the user is authenticated.
$this->assertGuest($guard = null);|	Assert that the user is not authenticated.
$this->assertAuthenticatedAs($user, $guard = null);|	Assert that the given user is authenticated.
$this->assertCredentials(array $credentials, $guard = null);|	Assert that the given credentials are valid.
$this->assertInvalidCredentials(array $credentials, $guard = null);|	Assert that the given credentials are invalid.
## 三、浏览器测试
#### 介绍
Laravel Dusk 只需要使用一个单独的 [ChromeDriver](https://sites.google.com/a/chromium.org/chromedriver/home)，不再需要在你的机器中安装 JDK 或者 Selenium。不过，依然可以按照你自己的需要安装其他 Selenium 兼容的驱动引擎。

#### 安装
    composer require --dev laravel/dusk

> 不要在生产环境中注册 Dusk 。

    php artisan dusk:install

设置 .env 中的 APP_URL

    APP_URL=http://my-app.com

运行测试

    php artisan dusk    // 接收任何 phpunit 可以接受的参数

**使用其他的浏览器**
```
# tests/DuskTestCase.php
# 删除 startChromeDriver 方法
public static function prepare()
{
    // static::startChromeDriver();
}
```
修改 driver 方法来连接到你指定的 URL 和端口。同时，你要修改传递给 WebDriver 的「desired capabilities」：
```
protected function driver()
{
    return RemoteWebDriver::create(
        'http://localhost:4444/wd/hub', DesiredCapabilities::phantomjs()
    );
}
```

#### 入门
```
php artisan dusk:make LoginTest

php artisan dusk    // 运行测试
```
Dusk 会自动启动 ChromeDriver ，有些系统不能正常启动，你需要重新启动。你需要做如下操作：
```
# 注释这行
public static function prepare()
{
    // static::startChromeDriver();
}
```
如果你的端口不是 9515
```
protected function driver()
{
    return RemoteWebDriver::create(
        'http://localhost:9515', DesiredCapabilities::chrome()
    );
}
```
**环境处理**
在你项目的根目录创建 .env.dusk.{environment} 文件来强制 Dusk 使用自己的的环境文件来运行测试。简单来说，如果你想要以 local 环境来运行 dusk 命令，你需要创建一个 .env.dusk.local 文件。

运行测试的时候，Dusk 会备份你的 .env 文件，然后重命名你的 Dusk 环境文件为 .env。一旦测试结束之后，将会恢复你的 .env 文件。

**创建浏览器**
```
<?php

namespace Tests\Browser;

use App\User;
use Tests\DuskTestCase;
use Laravel\Dusk\Chrome;
use Illuminate\Foundation\Testing\DatabaseMigrations;

class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;

    public function testBasicExample()
    {
        $user = factory(User::class)->create([
            'email' => 'taylor@laravel.com',
        ]);

        $this->browse(function ($browser) use ($user) {
            $browser->visit('/login')
                    ->type('email', $user->email)
                    ->type('password', 'secret')
                    ->press('Login')
                    ->assertPathIs('/home');
        });
    }
}
```
**多浏览器**
```
$this->browse(function ($first, $second) {
    $first->loginAs(User::find(1))
          ->visit('/home')
          ->waitForText('Message');

    $second->loginAs(User::find(2))
           ->visit('/home')
           ->waitForText('Message')
           ->type('message', 'Hey Taylor')
           ->press('Send');

    $first->waitForText('Hey Taylor')
          ->assertSee('Jeffrey Way');
});
```
**调整浏览器窗口大小**
```
$browser->resize(1920, 1080);
$browser->maximize();
```
**认证**
```
$this->browse(function ($first, $second) {
    $first->loginAs(User::find(1))
          ->visit('/home');
});
```
> 使用 loginAs 方法后, 该用户的 session 将会持久化供改文件其他测试用例使用。

当你想用迁移，你应该用 DatabaseMigrations，而不是  RefreshDatabase 。RefreshDatabase 无法跨 http 请求。
```
class ExampleTest extends DuskTestCase
{
    use DatabaseMigrations;
}
```
#### 与元素互动
```
<button>Login</button>

// Test...
$browser->click('.login-page .container div > button');
```
上面这种测试非常不好，因为前段变化的概率较大，变化了，你的测试就没用了，取而代之的是下面这种测试方法。
```
// HTML...
<button dusk="login-button">Login</button>

// Test...
$browser->click('@login-button');
```
**点击链接**
```
$browser->clickLink($linkText);
```
这个方法用到了 jquery，如果你的文件中没用到 jquery，dusk 会自动注入 jquery。

**文本、值、属性**
```
// Retrieve the value...
$value = $browser->value('selector');

// Set the value...
$browser->value('selector', 'value');
```
```
$text = $browser->text('selector');
```
```
$attribute = $browser->attribute('selector', 'value');
```
**表单相关操作**
```
$browser->type('email', 'taylor@laravel.com');
```
> 注意：虽然 type 方法可以传递 CSS 选择器作为第一个参数，但这并不是强制要求。如果传入的不是 CSS 选择器，Dusk 会尝试匹配传入值与 name 属性相符的 input 框，如果没找到，最后 Dusk 会尝试查找匹配传入值与 name 属性相符的 textarea。

添加更多的值

    $browser->type('tags', 'foo')
        ->append('tags', ', bar, baz');

清除值

    $browser->clear('email');

selector 传入的应该是 option 的值

    $browser->select('size', 'Large');

随机传值，那就忽略第二个参数

     $browser->select('size');

check

    $browser->check('terms');
    
    $browser->uncheck('terms');

附件

    $browser->attach('photo', __DIR__.'/photos/me.png');

键盘

    $browser->keys('selector', ['{shift}', 'taylor'], 'otwell');
    // 数组代表一起按，上面是按住 shift 键，然后输入 taylor
    $browser->keys('.app', ['{command}', 'j']);

热键列表：[facebook/php-webdriver](https://github.com/facebook/php-webdriver/blob/community/lib/WebDriverKeys.php)

鼠标

    $browser->click('.selector');
    $browser->mouseover('.selector');
    $browser->drag('.from-selector', '.to-selector');
    $browser->dragLeft('.selector', 10);
    $browser->dragRight('.selector', 10);
    $browser->dragUp('.selector', 10);
    $browser->dragDown('.selector', 10);

范围指定

```
$browser->with('.table', function ($table) {
    $table->assertSee('Hello World')
          ->clickLink('Delete');
});
```

等待

    $browser->pause(1000);    // 等待整个 test 1000 秒
```
# 时间一到，抛出异常

// Wait a maximum of five seconds for the selector...
$browser->waitFor('.selector');

// Wait a maximum of one second for the selector...
$browser->waitFor('.selector', 1);

$browser->waitUntilMissing('.selector');

$browser->waitUntilMissing('.selector', 1);
```
```
# 等待到可用
$browser->whenAvailable('.modal', function ($modal) {
    $modal->assertSee('Hello World')
          ->press('OK');
});
```
```
// Wait a maximum of five seconds for the text...
$browser->waitForText('Hello World');

// Wait a maximum of one second for the text...
$browser->waitForText('Hello World', 1);
```
```
// Wait a maximum of five seconds for the link text...
$browser->waitForLink('Create');

// Wait a maximum of one second for the link text...
$browser->waitForLink('Create', 1);
```
```
$browser->waitForLocation('/secret');
```
```
$browser->click('.some-action')
        ->waitForReload()
        ->assertSee('something');
```

```
# 等待 js 表达式
// Wait a maximum of five seconds for the expression to be true...
$browser->waitUntil('App.dataLoaded');

$browser->waitUntil('App.data.servers.length > 0');

// Wait a maximum of one second for the expression to be true...
$browser->waitUntil('App.data.servers.length > 0', 1);
```
```
# Dusk 中的许多「等待」方法依赖于 waitUsing 方法。该方法可以等待一个回调返回 true。waitUsing 接受的参数为最大等待秒数、闭包的执行间隔、闭包以及一个可选的错误信息。
$browser->waitUsing(10, 1, function () use ($something) {
    return $something->isReady();
}, "Something wasn't ready in time.");
```
assert Vue
```
// HTML...

<profile dusk="profile-component"></profile>

// Component Definition...

Vue.component('profile', {
    template: '<div>{{ user.name }}</div>',

    data: function () {
        return {
            user: {
              name: 'Taylor'
            }
        };
    }
});
```
```
public function testVue()
{
    $this->browse(function (Browser $browser) {
        $browser->visit('/')
                ->assertVue('user.name', 'Taylor', '@profile-component');
    });
}
```

#### assert 相关方法
| Assertion                                                 | Description                                                  |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| `$browser->assertTitle($title)`                           | Assert the page title matches the given text.                |
| `$browser->assertTitleContains($title)`                   | Assert the page title contains the given text.               |
| `$browser->assertUrlIs($url)`                             | Assert that the current URL (without the query string) matches the given string. |
| `$browser->assertPathBeginsWith($path)`                   | Assert that the current URL path begins with given path.     |
| `$browser->assertPathIs('/home')`                         | Assert the current path matches the given path.              |
| `$browser->assertPathIsNot('/home')`                      | Assert the current path does not match the given path.       |
| `$browser->assertRouteIs($name, $parameters)`             | Assert the current URL matches the given named route's URL.  |
| `$browser->assertQueryStringHas($name, $value)`           | Assert the given query string parameter is present and has a given value. |
| `$browser->assertQueryStringMissing($name)`               | Assert the given query string parameter is missing.          |
| `$browser->assertHasQueryStringParameter($name)`          | Assert that the given query string parameter is present.     |
| `$browser->assertHasCookie($name)`                        | Assert the given cookie is present.                          |
| `$browser->assertCookieMissing($name)`                    | Assert that the given cookie is not present.                 |
| `$browser->assertCookieValue($name, $value)`              | Assert a cookie has a given value.                           |
| `$browser->assertPlainCookieValue($name, $value)`         | Assert an unencrypted cookie has a given value.              |
| `$browser->assertSee($text)`                              | Assert the given text is present on the page.                |
| `$browser->assertDontSee($text)`                          | Assert the given text is not present on the page.            |
| `$browser->assertSeeIn($selector, $text)`                 | Assert the given text is present within the selector.        |
| `$browser->assertDontSeeIn($selector, $text)`             | Assert the given text is not present within the selector.    |
| `$browser->assertSourceHas($code)`                        | Assert that the given source code is present on the page.    |
| `$browser->assertSourceMissing($code)`                    | Assert that the given source code is not present on the page. |
| `$browser->assertSeeLink($linkText)`                      | Assert the given link is present on the page.                |
| `$browser->assertDontSeeLink($linkText)`                  | Assert the given link is not present on the page.            |
| `$browser->assertInputValue($field, $value)`              | Assert the given input field has the given value.            |
| `$browser->assertInputValueIsNot($field, $value)`         | Assert the given input field does not have the given value.  |
| `$browser->assertChecked($field)`                         | Assert the given checkbox is checked.                        |
| `$browser->assertNotChecked($field)`                      | Assert the given checkbox is not checked.                    |
| `$browser->assertRadioSelected($field, $value)`           | Assert the given radio field is selected.                    |
| `$browser->assertRadioNotSelected($field, $value)`        | Assert the given radio field is not selected.                |
| `$browser->assertSelected($field, $value)`                | Assert the given dropdown has the given value selected.      |
| `$browser->assertNotSelected($field, $value)`             | Assert the given dropdown does not have the given value selected. |
| `$browser->assertSelectHasOptions($field, $values)`       | Assert that the given array of values are available to be selected. |
| `$browser->assertSelectMissingOptions($field, $values)`   | Assert that the given array of values are not available to be selected. |
| `$browser->assertSelectHasOption($field, $value)`         | Assert that the given value is available to be selected on the given field. |
| `$browser->assertValue($selector, $value)`                | Assert the element matching the given selector has the given value. |
| `$browser->assertVisible($selector)`                      | Assert the element matching the given selector is visible.   |
| `$browser->assertMissing($selector)`                      | Assert the element matching the given selector is not visible. |
| `$browser->assertDialogOpened($message)`                  | Assert that a JavaScript dialog with given message has been opened. |
| `$browser->assertVue($property, $value, $component)`      | Assert that a given Vue component data property matches the given value. |
| `$browser->assertVueIsNot($property, $value, $component)` | Assert that a given Vue component data property does not match the given value. |

#### 页面

    php artisan dusk:page Login
```
public function url()
{
    return '/login';
}

public function assert(Browser $browser)   // 不是必须的
{
    $browser->assertPathIs($this->url());
}


```
```
use Tests\Browser\Pages\Login;

$browser->visit(new Login);
```
```
use Tests\Browser\Pages\CreatePlaylist;

$browser->visit('/dashboard')
        ->clickLink('Create Playlist')
        ->on(new CreatePlaylist)
        ->assertSee('@create');
```
快捷方式
```
public function elements()
{
    return [
        '@email' => 'input[name=email]',
    ];
} 

# $browser->type('@email', 'taylor@laravel.com');
```
全局快捷方式 
```
# Class Page
public static function siteElements()
{
    return [
        '@element' => '#selector',
    ];
}
```
页面方法

    public function createPlaylist(Browser $browser, $name)
    {
        $browser->type('name', $name)
                ->check('share')
                ->press('Create Playlist');
    }

```
use Tests\Browser\Pages\Dashboard;

$browser->visit(new Dashboard)
        ->createPlaylist('My Playlist')
        ->assertSee('My Playlist');
```
#### 组件
生成组件

    php artisan dusk:component DatePicker

```
<?php

namespace Tests\Browser\Components;

use Laravel\Dusk\Browser;
use Laravel\Dusk\Component as BaseComponent;

class DatePicker extends BaseComponent
{

    public function selector()
    {
        return '.date-picker';
    }

    /**
     * Assert that the browser page contains the component.
     *
     * @param  Browser  $browser
     * @return void
     */
    public function assert(Browser $browser)
    {
        $browser->assertVisible($this->selector());
    }

    public function elements()
    {
        return [
            '@date-field' => 'input.datepicker-input',
            '@month-list' => 'div > div.datepicker-months',
            '@day-list' => 'div > div.datepicker-days',
        ];
    }

    public function selectDate($browser, $month, $year)
    {
        $browser->click('@date-field')
                ->within('@month-list', function ($browser) use ($month) {
                    $browser->click($month);
                })
                ->within('@day-list', function ($browser) use ($day) {
                    $browser->click($day);
                });
    }
}
```
使用 component

    public function testBasicExample()
    {
        $this->browse(function (Browser $browser) {
            $browser->visit('/')
                    ->within(new DatePicker, function ($browser) {
                        $browser->selectDate(1, 2018);
                    })
                    ->assertSee('January');
        });
    }


#### 连续集成
**Travis CI**
在 Travis CI 中运行 Dusk 时需要「sudo-enabled」的 Ubuntu 14.04 (Trusty) 环境。由于 Travis CI 不是图形环境，我们需要一些额外的步骤去启动 Chrome 浏览器，另外，我们需要使用 php artisan serve 命令去启动 PHP 的内置服务器。
```
sudo: required
dist: trusty

addons:
   chrome: stable

install:
   - cp .env.testing .env
   - travis_retry composer install --no-interaction --prefer-dist --no-suggest

before_script:
   - google-chrome-stable --headless --disable-gpu --remote-debugging-port=9222 http://localhost &
   - php artisan serve &

script:
   - php artisan dusk
```
**CircleCI 1.0**
在 CircleCI 1.0 中运行 Dusk 时需要使用以下配置进行启动。与 TravisCI 相同，我们需要使用 php artisan serve 命令去启动 PHP 的内置服务器。
```
dependencies:
  pre:
      - curl -L -o google-chrome.deb https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
      - sudo dpkg -i google-chrome.deb
      - sudo sed -i 's|HERE/chrome\"|HERE/chrome\" --disable-setuid-sandbox|g' /opt/google/chrome/google-chrome
      - rm google-chrome.deb

test:
    pre:
        - "./vendor/laravel/dusk/bin/chromedriver-linux":
            background: true
        - cp .env.testing .env
        - "php artisan serve":
            background: true

    override:
        - php artisan dusk
```
**CircleCI 2.0**
在 CircleCI 2.0 中运行 Dusk 时需要将以下 steps 添加至 build：
```
 version: 2
 jobs:
     build:
         steps:
            - run: sudo apt-get install -y libsqlite3-dev
            - run: cp .env.testing .env
            - run: composer install -n --ignore-platform-reqs
            - run: npm install
            - run: npm run production
            - run: vendor/bin/phpunit

            - run:
               name: Start Chrome Driver
               command: ./vendor/laravel/dusk/bin/chromedriver-linux
               background: true

            - run:
               name: Run Laravel Server
               command: php artisan serve
               background: true

            - run:
               name: Run Laravel Dusk Tests
               command: php artisan dusk
```
**Codeship**
```
phpenv local 7.1
cp .env.testing .env
composer install --no-interaction
nohup bash -c "./vendor/laravel/dusk/bin/chromedriver-linux 2>&1 &"
nohup bash -c "php artisan serve 2>&1 &" && sleep 5
php artisan dusk
```
## 四、数据库测试
#### 生成工厂

    php artisan make:factory PostFactory
    php artisan make:factory PostFactory --model=Post
#### 每次测试后重置数据库

    use RefreshDatabase;
#### 编写工厂
```
use Faker\Generator as Faker;

$factory->define(App\User::class, function (Faker $faker) {
    return [
        'name' => $faker->name,
        'email' => $faker->unique()->safeEmail,
        'email_verified_at' => now(),
        'password' => '$2y$10$TKh8H1.PfQx37YgCzwiKb.KjNyWgaHb9cbcoQgdIVFlYg7B77UdFm', // secret
        'remember_token' => Str::random(10),
    ];
});
```
工厂状态
```
$factory->state(App\User::class, 'delinquent', [
    'account_status' => 'delinquent',
]);

$factory->state(App\User::class, 'address', function ($faker) {
    return [
        'address' => $faker->address,
    ];
});

$factory->afterMaking(App\User::class, function ($user, $faker) {
    // ...
});

$factory->afterCreating(App\User::class, function ($user, $faker) {
    $user->accounts()->save(factory(App\Account::class)->make());
});


$factory->afterMakingState(App\User::class, 'delinquent', function ($user, $faker) {
    // ...
});

$factory->afterCreatingState(App\User::class, 'delinquent', function ($user, $faker) {
    // ...
});
```
#### 使用工厂
```
public function testDatabase()
{
    $user = factory(App\User::class)->make();

    // Use model in tests...
}
```
```
$users = factory(App\User::class, 3)->make();

$users = factory(App\User::class, 5)->states('delinquent')->make();

$users = factory(App\User::class, 5)->states('premium', 'delinquent')->make();
```
覆盖属性
```
$user = factory(App\User::class)->make([
    'name' => 'Abigail',
]);
```
create === make + save()
```
public function testDatabase()
{
    // Create a single App\User instance...
    $user = factory(App\User::class)->create();

    // Create three App\User instances...
    $users = factory(App\User::class, 3)->create();

    // Use model in tests...
}
```
```
user = factory(App\User::class)->create([
    'name' => 'Abigail',
]);
```
关联
```
$users = factory(App\User::class, 3)
           ->create()
           ->each(function ($user) {
                $user->posts()->save(factory(App\Post::class)->make());
            });
			
			
$user->posts()->createMany(
    factory(App\Post::class, 3)->make()->toArray()
);
```
```
$factory->define(App\Post::class, function ($faker) {
    return [
        'title' => $faker->title,
        'content' => $faker->paragraph,
        'user_id' => function () {
            return factory(App\User::class)->create()->id;
        }
    ];
});
```
```
$factory->define(App\Post::class, function ($faker) {
    return [
        'title' => $faker->title,
        'content' => $faker->paragraph,
        'user_id' => function () {
            return factory(App\User::class)->create()->id;
        },
        'user_type' => function (array $post) {
            return App\User::find($post['user_id'])->type;
        }
    ];
});


    public function testCreatingANewOrder()
    {
        // Run the DatabaseSeeder...
        $this->seed();

        // Run a single seeder...
        $this->seed(OrderStatusesTableSeeder::class);

        // ...
    }
```
#### assert 方法
```
$this->assertDatabaseHas($table, array $data);	
$this->assertDatabaseMissing($table, array $data);	
$this->assertSoftDeleted($table, array $data);
```

## 五、模拟
当测试某个控制器的时候，也许你并不想去测试他所触发的事件，因为该事件已经有了自己的测试方法。这时候你就可用在测试中模拟该时间了。

```php
use Mockery;
use App\Service;

$this->instance(Service::class, Mockery::mock(Service::class, function ($mock) {
    $mock->shouldReceive('process')->once();
}));

$this->mock(Service::class, function ($mock) {
    $mock->shouldReceive('process')->once();
});

$this->spy(Service::class, function ($mock) {
    $mock->shouldHaveReceived('process');
});
```
#### Bus 模拟
你可以使用 Bus facade 的 fake 方法来模拟任务执行，测试的时候任务不会被真实执行。使用 fakes 的时候，断言一般出现在测试代码的后面：
```
class ExampleTest extends TestCase
{
    public function testOrderShipping()
    {
        Bus::fake();

        // Perform order shipping...

        Bus::assertDispatched(ShipOrder::class, function ($job) use ($order) {
            return $job->order->id === $order->id;
        });

        // Assert a job was not dispatched...
        Bus::assertNotDispatched(AnotherJob::class);
    }
}
```
#### 事件模拟
测试的时候不会触发事件监听器运行。然后你就可以断言事件运行了，甚至可以检查它们收到的数据。
```
<?php

namespace Tests\Feature;

use Tests\TestCase;
use App\Events\OrderShipped;
use App\Events\OrderFailedToShip;
use Illuminate\Support\Facades\Event;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;

class ExampleTest extends TestCase
{
    /**
     * Test order shipping.
     */
    public function testOrderShipping()
    {
        Event::fake();

        // Perform order shipping...

        Event::assertDispatched(OrderShipped::class, function ($e) use ($order) {
            return $e->order->id === $order->id;
        });

        // Assert an event was dispatched twice...
        Event::assertDispatched(OrderShipped::class, 2);

        // Assert an event was not dispatched...
        Event::assertNotDispatched(OrderFailedToShip::class);
    }
	
	
	public function testOrderProcess()
	{
		Event::fake([
			OrderCreated::class,
		]);

		$order = factory(Order::class)->create();

		Event::assertDispatched(OrderCreated::class);

		// Other events are dispatched as normal...
		$order->update([...]);
	}
}

// After calling Event::fake(), no event listeners will be executed. So, if your tests use model factories that rely on events, such as creating a UUID during a model's creating event, you should call Event::fake() after using your factories.
```
```php

<?php

namespace Tests\Feature;

use App\Order;
use Tests\TestCase;
use App\Events\OrderCreated;
use Illuminate\Support\Facades\Event;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Foundation\Testing\WithoutMiddleware;

class ExampleTest extends TestCase
{
    /**
     * Test order process.
     */
    public function testOrderProcess()
    {
        $order = Event::fakeFor(function () {
            $order = factory(Order::class)->create();

            Event::assertDispatched(OrderCreated::class);

            return $order;
        });

        // Events are dispatched as normal and observers will run ...
        $order->update([...]);
    }
}
```
#### 邮件模拟
测试时不会真的发送邮件。然后你可以断言发送给了用户，甚至可以检查他们收到的数据。
```
class ExampleTest extends TestCase
{
    public function testOrderShipping()
    {
        Mail::fake();

        // Assert that no mailables were sent...
        Mail::assertNothingSent();

        // Perform order shipping...

        Mail::assertSent(OrderShipped::class, function ($mail) use ($order) {
            return $mail->order->id === $order->id;
        });

        // Assert a message was sent to the given users...
        Mail::assertSent(OrderShipped::class, function ($mail) use ($user) {
            return $mail->hasTo($user->email) &&
                   $mail->hasCc('...') &&
                   $mail->hasBcc('...');
        });

        // Assert a mailable was sent twice...
        Mail::assertSent(OrderShipped::class, 2);

        // Assert a mailable was not sent...
        Mail::assertNotSent(AnotherMailable::class);
    }
}
```
如果你是用后台任务队执行 mailables 的发送，你应该用 assertQueued 方法来代替 assertSent：
```
Mail::assertQueued(...);
Mail::assertNotQueued(...);
```
#### 通知模拟
测试的时候并不会真的发送通知。然后你可以断言已经发送给你的用户，甚至可以检查他们收到的数据。

    public function testOrderShipping()
    {
        Notification::fake();

        // Assert that no notifications were sent...
        Notification::assertNothingSent();

        // Perform order shipping...

        Notification::assertSentTo(
            $user,
            OrderShipped::class,
            function ($notification, $channels) use ($order) {
                return $notification->order->id === $order->id;
            }
        );

        // Assert a notification was sent to the given users...
        Notification::assertSentTo(
            [$user], OrderShipped::class
        );

        // Assert a notification was not sent...
        Notification::assertNotSentTo(
            [$user], AnotherNotification::class
        );

        // Assert a notification was sent via Notification::route() method...
        Notification::assertSentTo(
            new AnonymousNotifiable, OrderShipped::class
        );
    }

#### 队列模拟
测试的时候并不会真的把任务放入队列。然后你可以断言任务被放进了队列，甚至可以检查它们收到的数据。

    public function testOrderShipping()
    {
        Queue::fake();

        // Assert that no jobs were pushed...
        Queue::assertNothingPushed();

        // Perform order shipping...

        Queue::assertPushed(ShipOrder::class, function ($job) use ($order) {
            return $job->order->id === $order->id;
        });

        // Assert a job was pushed to a given queue...
        Queue::assertPushedOn('queue-name', ShipOrder::class);

        // Assert a job was pushed twice...
        Queue::assertPushed(ShipOrder::class, 2);

        // Assert a job was not pushed...
        Queue::assertNotPushed(AnotherJob::class);

        // Assert a job was pushed with a specific chain...
        Queue::assertPushedWithChain(ShipOrder::class, [
            AnotherJob::class,
            FinalJob::class
        ]);
    }

#### 存储模拟
你可以轻松地生成一个模拟的磁盘，结合 UploadedFile 类的文件生成工具，极大地简化了文件上传测试。
```
    public function testAvatarUpload()
    {
        Storage::fake('photos');

        $response = $this->json('POST', '/photos', [
            UploadedFile::fake()->image('photo1.jpg'),
            UploadedFile::fake()->image('photo2.jpg')
        ]);

        // Assert one or more files were stored...
        Storage::disk('photos')->assertExists('photo1.jpg');
        Storage::disk('photos')->assertExists(['photo1.jpg', 'photo2.jpg']);

        // Assert one or more files were not stored...
        Storage::disk('photos')->assertMissing('missing.jpg');
        Storage::disk('photos')->assertMissing(['missing.jpg', 'non-existing.jpg']);
    }
	// By default, the fake method will delete all files in its temporary directory. If you would like to keep these files, you may use the "persistentFake" method instead.
```
#### Facades

    public function testGetIndex()
    {
        Cache::shouldReceive('get')
                    ->once()
                    ->with('key')
                    ->andReturn('value');

        $response = $this->get('/users');
    
        // ...
    }

> 你不应该模拟 Request Facades。应该使用 HTTP 辅助函数 get， post 等来运行测试。模拟 Config Facades，应该使用 Config::set 方法。

