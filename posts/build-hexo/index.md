# Hexo - 借助GitHub搭建个人博客


Hexo静态博客框架。

<!--more-->

## 前言
`2017-09-05更新` 一年多后，由于不再在封闭环境及云桌面办公，所以把Hexo得重新启用，打算用来做技术博客，或者叫Wiki，写一些自己的技术总结，按此前搭建的记录，部署起来。

有些日子没玩博客了，其实以前也一直很少去输出一些什么，但就是喜欢有一个属于个人的地方，向互联网展示一下自己的存在感！
I'm here，嗯，是这个意思！
以前和网友一起合租过服务器，也在Godady买过域名，但最后还是没坚持下来，后来折腾完之后，最实用的反而是搭梯翻墙而已。
好了，再多的就不扯了，这篇就是为了记录下在Windows下用Hexo在GitHub上搭建个人博客的过程，希望可以对其他朋友有所帮助。

## 简介

### Hexo是什么？
Hexo是一个很好的开源静态博客生成器，基于Node.js开发，作者是台湾大学生tommy351。
Hexo依赖于Git与Node.js环境，就是说，你要玩Hexo，就必须先得把环境搭好，才能一起玩耍！

## 环境搭建
在Windows上装Git和Node.js都没什么好说的，和其它正常软件是一样的，就是下个安装包，双击打开，无脑下一步，默认选项即可，就完成了，下边附上下载包链接：

### 安装Git
[https://git-scm.com/][id1]

### 安装Node.js
[https://nodejs.org/][id2]
说明一下，下的是V6.11.2 LTS，长期支持版，而非最新的V8.4.0；非前端开发，没必要安装最新版。

### 安装Hexo
正主来了，这个开始就要学会使用Git Bash来运行了！
如果安装的时候选择了集成到Windows的CMD中，打开CMD运行也是可行的。
带上了参数-g就是全局安装，所以不用纠结是在哪个目录下运行命令。

```bash
$ npm install hexo-cli -g
```

#### 初始化（若非首次安装，不需要）
初始化有2种方式，下边以blog文件夹为例，初始化完还需要在文件夹里安装包依赖：
1. 直接指定目录

	```bash
	$ hexo init blog
	```

2. 进入指定目录

	```bash
	$ cd blog
	$ hexo init
	```

终端执行命令后显示的结果：

```
[info] Copying data[info] You are almost done! Don't forget to run `npm install` before start blogging with Hexo!
```

#### 依赖安装
* 安装包依赖

	```bash
	$ cd blog
	$ npm install
	```

## 部署
* 生成静态文件

	```bash
	$ hexo generate
	```

* 启动本地调试环境

	```bash
	$ hexo server
	```

至此，本地的环境算是完工了，你可以访问`http://localhost:4000`查看你的页面了！

## 定制配置

* 在Hexo 3.0版本后`deploy-git`[^1]被分开的，所以需要安装，安装命令如下：

	```bash
	$ npm install hexo-deployer-git --save
	```

* 增加RSS订阅[^2]，需要安装，命令如下：

	```bash
	$ npm install hexo-generator-feed --save
	```

	站点配置文件`_config.yml`增加配置如下：

	```yml
	# Feed
	feed:
	  type: atom
	  path: atom.xml
	  limit: 20
	  hub:
	```

* GitHub Pages 自定义404页面非常容易，直接在source根目录下创建自己的404.html就可以，代码如下：

	```html
	<!DOCTYPE HTML>
	<html>
	<head>
	  <meta http-equiv="content-type" content="text/html;charset=utf-8;"/>
	  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1" />
	  <meta name="robots" content="all" />
	  <meta name="robots" content="index,follow"/>
	</head>
	<body>
	  <script type="text/javascript" src="http://www.qq.com/404/search_children.js" charset="utf-8" homePageUrl="http://qiuzhi.github.io/" homePageName="回到我的主页">
	  </script>
	</body>
	</html>
	```

* 增加[disqus](https://disqus.com/)评论
	1. 到官网注册并配置账号，得到shortname
	2. 在主题配置文件`_config.yml`相应位置修改如下：

	```yml
	# Disqus
	disqus:
	  enable: true
	  shortname: shortname
	  count: true
	```

* 文章链接唯一化
	也许你会数次更改文章题目或者变更文章发布时间，在默认设置下，文章链接都会改变，不利于搜索引擎收录，也不利于分享。唯一永久链接才是更好的选择。

	安装

	```shell
	npm install hexo-abbrlink --save
	```

	在站点配置文件中查找代码permalink，将其更改为:

	```
	permalink: posts/:abbrlink/  # “posts/” 可自行更换
	```

	这里有个知识点：
		百度蜘蛛抓取网页的规则: 对于蜘蛛说网页权重越高、信用度越高抓取越频繁，例如网站的首页和内页。蜘蛛先抓取网站的首页，因为首页权重更高，并且大部分的链接都是指向首页。然后通过首页抓取网站的内页，并不是所有内页蜘蛛都会去抓取。

	搜索引擎认为对于一般的中小型站点，3层足够承受所有的内容了，所以蜘蛛经常抓取的内容是前三层，而超过三层的内容蜘蛛认为那些内容并不重要，所以不经常爬取。出于这个原因所以permalink后面跟着的最好不要超过2个斜杠。

	然后在站点配置文件中添加如下代码:

	```yml
	# abbrlink config
	abbrlink:
	  alg: crc32  # 算法：crc16(default) and crc32
	  rep: hex    # 进制：dec(default) and hex
	```
	
	可选择模式：
		crc16 & hex
		crc16 & dec
		crc32 & hex
		crc32 & dec

	可参照样例以选择：

	```
	crc16 & hex
	https://post.zz173.com/posts/66c8.html
	crc16 & dec
	https://post.zz173.com/posts/65535.html
	crc32 & hex
	https://post.zz173.com/posts/8ddf18fb.html
	crc32 & dec
	https://post.zz173.com/posts/1690090958.html
	```

## 发布到互联网
1. 到[GitHub][id3]注册
	新建repository(仓库)，填写格式如下图所示: [yourusername].github.io
	将[yourusername]换成你在GitHub注册的用户名！
1. 生成SSH Keys
	如何让本地git项目与远程的github建立联系？需要用到SSH Keys！
	1. 使用ssh-keygen命令生成密钥对

	```bash
	$ ssh-keygen -t rsa -C "这里是你申请Github账号时的邮箱"
	```

	然后系统会要你输入密码：（我们输入的密码会在你提交项目的时候使用）

	```bash
	Enter passphrase (emptyforno passphrase):<输入加密串>
	Enter same passphrase again:<再次输入加密串>
	```

	从终端提示生成的文件路径中找到你生成的密钥找到`id_rsa.pub`用终端进入编辑，复制密钥。
	1. 添加SSH Key到GitHub
	登陆GitHub网站，找到[Setting-SSH and GPG keys](https://github.com/settings/keys)，点击`New SSH key`按钮，将复制的密钥粘贴到`Key`栏；`Title`栏填上你喜欢的标识随意标记，然后保存即可。
	1. 验证SSH是否链通

	```bash
	$ ssh -T git@github.com
	```

	终端执行命令后显示的结果：

	```
	Hi xxx! You've successfully authenticated, but GitHub does not provide shell access.
	```

	现在就已经可以通过SSH链接到Github了！
	1. 别忘记设置提交信息

	```bash
	$ git config --global user.name "xxx"
	$ git config --global user.email "xxx@xxx.com"
	```

1. 打开站点配置文件，Hexo根目录下_config.yml：

	```yml
	# Deployment
	## Docs: https://hexo.io/docs/deployment.html
	deploy:
	  type: git
	  repository: https://github.com/[yourusername]/[yourusername].github.io.git
	  branch: master
	```

	将[yourusername]换成你在GitHub注册的用户名！  
1. 发布到Github

	```bash
	$ hexo deploy
	```

至此，GitHub的环境算是完工了，你可以访问`http://[yourusername].github.io`查看你的页面了！

## 维护

### 环境信息
* 查看node.js的版本

	```bash
	$ node -v
	v6.11.2
	```

* 查看node.js的版本

	```bash
	$ npm ls --depth 0
	hexo-site@0.0.0 \Hexo
	├── hexo@3.3.8
	├── hexo-deployer-git@0.1.0
	├── hexo-generator-archive@0.1.4
	├── hexo-generator-category@0.1.3
	├── hexo-generator-feed@1.2.0
	├── hexo-generator-index@0.2.1
	├── hexo-generator-tag@0.2.0
	├── hexo-renderer-ejs@0.2.0
	├── hexo-renderer-marked@0.2.11
	├── hexo-renderer-stylus@0.3.3
	├── hexo-server@0.2.2
	└── hexo-util@0.6.1
	```

### 依赖更新
在hexo site（`_config.yml`及`package.json`的资料夹）下  
* 查看是否存在新旧版本

	```bash
	$ npm outdated
	```

* 更新版本

	```bash
	$ npm update -S
	```

更多请参考 [npm文档](https://docs.npmjs.com/) 及 [hexo文档](https://hexo.io/docs/)

## 过程中遇到的问题及解决

### 问题一：Cannot find module 'hexo-util'
* 问题描述：

```bash
ERROR Script load failed: themes/hexo-theme-next/scripts/tags/exturl.js  
Error: Cannot find module 'hexo-util'  
    at Function.Module._resolveFilename (module.js:336:15)  
    at Function.Module._load (module.js:286:25)  
    at Module.require (module.js:365:17)  
    at require (/data/github/hexo/node_modules/hexo/lib/hexo/index.js:213:21)  
    at /data/github/hexo/themes/hexo-theme-next/scripts/tags/exturl.js:7:12  
    at /data/github/hexo/node_modules/hexo/lib/hexo/index.js:229:12  
    at tryCatcher (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/util.js:16:23)  
    at Promise._settlePromiseFromHandler (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:512:31)  
    at Promise._settlePromise (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:569:18)  
    at Promise._settlePromise0 (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:614:10)  
    at Promise._settlePromises (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:693:18)  
    at Promise._fulfill (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:638:18)  
    at Promise._resolveCallback (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:432:57)  
    at Promise._settlePromiseFromHandler (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:524:17)  
    at Promise._settlePromise (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:569:18)  
    at Promise._settlePromise0 (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:614:10)  
    at Promise._settlePromises (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:693:18)  
    at Promise._fulfill (/data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/promise.js:638:18)  
    at /data/github/hexo/node_modules/hexo/node_modules/bluebird/js/release/nodeback.js:42:21  
    at /data/github/hexo/node_modules/hexo/node_modules/hexo-fs/node_modules/graceful-fs/graceful-fs.js:78:16  
    at FSReqWrap.readFileAfterClose [as oncomplete] (fs.js:380:3)
```

* 解决方案
提示hexo-util找不到, 执行下面命令后就可以了

	```bash
	$ npm install hexo-util --save
	```

### 问题二：could not read Username for 'https://github.com': Invalid argument
* 问题描述：Windows下更换Comder作为CMD终端后，遇到

```bash
nothing to commit, working tree clean
bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': Invalid argument
FATAL Something's wrong. Maybe you can find the solution here: http://hexo.io/docs/troubleshooting.html
Error: bash: /dev/tty: No such device or address
error: failed to execute prompt script (exit code 1)
fatal: could not read Username for 'https://github.com': Invalid argument

    at ChildProcess.<anonymous> (E:\Dropbox\Hexo\node_modules\hexo-util\lib\spawn.js:37:17)
    at emitTwo (events.js:106:13)
    at ChildProcess.emit (events.js:191:7)
    at ChildProcess.cp.emit (E:\Dropbox\Hexo\node_modules\cross-spawn\lib\enoent.js:40:29)
    at maybeClose (internal/child_process.js:891:16)
    at Process.ChildProcess._handle.onexit (internal/child_process.js:226:5)
```

* 解决方案
	由于新手还不明白运行的机制，使用了Cmder自带的git工具，好像只能走ssh通道去更新。进行下边两步即可
	1. 那就先把hexo根目录下.deploy_git目录删除
	2. 修改_config.yml文件：
	```yml
	repository: https://github.com/username/username.github.io.git
	```

	修改为

	```yml
	repository: git@github.com:username/username.github.io.git
	```

[id1]: https://git-scm.com/ "git-scm"
[id2]: https://nodejs.org/ "Node.js"
[id3]: https://github.com/ "GitHub"

[^1]: https://github.com/hexojs/hexo-deployer-git
[^2]: https://www.npmjs.com/package/hexo-generator-feed


---

> 作者:   
> URL: https://blog.iqzhi.com/posts/build-hexo/  

