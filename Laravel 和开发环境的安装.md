> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=2](https://www.bilibili.com/video/av70545323?p=2 "https://www.bilibili.com/video/av70545323?p=2")

----

1. **先安装两个运行库**
	php, mysql 运行必备的库，不安装会报错。

2. **安装 wampserver**
	包含开发所需的 php、mysql、apache 和一些控制面板（比如 phpMyAdmin）。

3. **安装 composer**
	查看是否安装成功：`composer –V`

4. **更改 composer 的源**
	`composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/`

5. **安装 laravel**
	`composer create-project --prefer-dist laravel/laravel blog`

6. **修改 apache 配置文件**
	修改完记得重启 wampserver。

```conf
<VirtualHost *:80>
  ServerName qianjinyike.com
  ServerAlias qianjinyike.com
  DocumentRoot "C:\Users\leonz\blog\public"
  <Directory "C:\Users\leonz\blog">
    Options +Indexes +Includes +FollowSymLinks +MultiViews
    AllowOverride All
    Require local
  </Directory>
</VirtualHost>
```

7. **安装 notepad++**

8. **修改 hosts**
	`C:\Windows\System32\drivers\etc`

安装包中，wampserver 太大了无法上传。我上传到百度网盘中，大家可以下载
[https://pan.baidu.com/s/11Yl1FGZpYS7S9CJMoETdAA](https://pan.baidu.com/s/11Yl1FGZpYS7S9CJMoETdAA "https://pan.baidu.com/s/11Yl1FGZpYS7S9CJMoETdAA")
密码：67cc

其他软件，QQ 群文件可以下载。群号：375462817。
