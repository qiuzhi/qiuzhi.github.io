# 宝塔面板安装Tiny Tiny RSS


物尽其用，刚好雨云免费的宝塔面板可以绑定两个域名，一个用作博客备站，另一个刚好可以玩下Tiny Tiny RSS。

<!--more-->

官网最新教程已然推荐使用docker进行安装了，但雨云的虚拟主机无法安装docker，只能使用源码安装，好在基础环境宝塔面板还是简化了些操作，但是由于多年更新，此种安装方式官方wiki也不再更新，导致还是碰上了很多坑，一路坑一路填，居然成功了。

## 宝塔面板新建站点

1. 面站 -> PHP项目 -> 添加站点，确定文件目录，选用PHP7.4版本

2. 软件商店 -> PHP-7.4 -> 安装扩展`fileinfo`

## Tiny Tiny RSS

[wiki](https://tt-rss.org/wiki.php)

1. 按[教程](https://tt-rss.org/wiki/InstallationNotesHost)，先下载源码。

```bash
git clone https://git.tt-rss.org/fox/tt-rss.git tt-rss
```

2. 配置`config.php`

网上某些教程说可以访问`/install`路径进行数据库配置安装，但访问后报了404，那些教程已然落后了。

```bash
cp config.php-dist config.php

# 追加
putenv('TTRSS_DB_TYPE=mysql');
putenv('TTRSS_DB_PORT=3306');
putenv('TTRSS_DB_HOST=dbhost');
putenv('TTRSS_DB_NAME=dbname');
putenv('TTRSS_DB_USER=dbuser');
putenv('TTRSS_DB_PASS=dbpassword');
putenv('TTRSS_SELF_URL_PATH=https://example.com/tt-rss');
```
3. 安装数据库

```bash
php ./update.php --update-schema
```

4. 理论上，做完这些，你就可以在相应网址看到Tiny Tiny RSS首页了。

but，WTF！！！

我遇到了下列一系列的坑。

## 踩坑过程

### 坑一：PHP多版本共存

{{< admonition question "宝塔面板有两个版本的PHP，默认是更高的8以上版本，可能不支持！">}}

```txt
PHP Fatal error:  Uncaught Error: Call to undefined function putenv() in /www/wwwroot/rss.izhi.tk/config.php:3
Stack trace:
#0 /www/wwwroot/rss.izhi.tk/include/functions.php(29): require_once()
#1 /www/wwwroot/rss.izhi.tk/update.php(11): require_once('...')
#2 {main}
  thrown in /www/wwwroot/rss.izhi.tk/config.php on line 3
```

指定php命令版本

```bash
/www/server/php/74/bin/php ./update.php --update-schema
```

{{< /admonition >}}

### 坑二：函数 `putenv` 过期

{{< admonition question "Exception while creating PDO object:could not find driver">}}

```txt
PHP Warning:  putenv() has been disabled for security reasons in /www/wwwroot/xxx.tk/config.php on line 3
```

从日志可以看出，配置方式过时了，未生效，tt-rss使用了默认的PostgreSQL数据库，由于PHP未安装pgsql扩展而报未找到驱动错误。

两种处理方式，我使用1可以解决，第2种后边在网上找到，未验证：

1. 直接在`classes/Confing.php`中配置默认值

```php
private const _DEFAULTS = [
                Config::DB_TYPE => [ "pgsql",                                                                   Config::T_STRING ],
                Config::DB_HOST => [ "db",                                                                              Config::T_STRING ],
                Config::DB_USER => [ "",                                                                                        Config::T_STRING ],
                Config::DB_NAME => [ "",                                                                                        Config::T_STRING ],
                Config::DB_PASS => [ "",                                                                                        Config::T_STRING ],
                Config::DB_PORT => [ "5432",                                                                            Config::T_STRING ],
                Config::MYSQL_CHARSET => [ "UTF8",                                                              Config::T_STRING ],
                Config::SELF_URL_PATH => [ "https://example.com/tt-rss", Config::T_STRING ],
                Config::SINGLE_USER_MODE => [ "",                                                               Config::T_BOOL ],
                Config::SIMPLE_UPDATE_MODE => [ "",                                                             Config::T_BOOL ],
                Config::PHP_EXECUTABLE => [ "/usr/bin/php",                                     Config::T_STRING ],
                Config::LOCK_DIRECTORY => [ "lock",                                                             Config::T_STRING ],
```

2. 去掉禁用函数`putenv()` ,改用`define()`定义,详见[Error: PDO object:SQLSTATE[08006] [7] could not connect to server](https://community.tt-rss.org/t/error-pdo-object-sqlstate-08006-7-could-not-connect-to-server/4369)

```php
<?php
	// *******************************************
	// *** Database configuration (important!) ***
	// *******************************************

	define('DB_TYPE', "pgsql"); // or mysql
	define('DB_HOST', "localhost");
	define('DB_USER', "fox");
	define('DB_NAME', "fox");
	define('DB_PASS', "XXXXXX");
	define('DB_PORT', ''); // usually 5432 for PostgreSQL, 3306 for MySQL

	define('MYSQL_CHARSET', 'UTF8');
	// Connection charset for MySQL. If you have a legacy database and/or experience
	// garbage unicode characters with this option, try setting it to a blank string.

	// ***********************************
	// *** Basic settings (important!) ***
	// ***********************************

	define('SELF_URL_PATH', 'https://example.org/tt-rss/');
	// This should be set to a fully qualified URL used to access
	// your tt-rss instance over the net.
	// The value should be a constant string literal. Please don't use
	// PHP server variables here - you might introduce security
	// issues on your install and cause hard to debug problems.
	// If your tt-rss instance is behind a reverse proxy, use the external URL.

	define('SINGLE_USER_MODE', false);
	// Operate in single user mode, disables all functionality related to
	// multiple users and authentication. Enabling this assumes you have
	// your tt-rss directory protected by other means (e.g. http auth).

	define('SIMPLE_UPDATE_MODE', false);
	// Enables fallback update mode where tt-rss tries to update feeds in
	// background while tt-rss is open in your browser.
	// If you don't have a lot of feeds and don't want to or can't run
	// background processes while not running tt-rss, this method is generally
	// viable to keep your feeds up to date.
	// Still, there are more robust (and recommended) updating methods
	// available, you can read about them here: http://tt-rss.org/wiki/UpdatingFeeds

	<blah, etc, blah>
```

{{< /admonition >}}

### 坑三：非root用户运行

{{< admonition question "* Please don't run this script as root.">}}

不能使用root运行，通过`su`切换指定`www`用户运行。

```bash
sudo -u www /www/server/php/74/bin/php ./update.php --update-schema
```
以下[备用](https://community.tt-rss.org/t/cannot-run-update-php-by-any-means/2516/2)

```bash
sudo -u www-data php ./update.php --daemon
```

{{< /admonition >}}


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/bt-install-tt-rss/  

