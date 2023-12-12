# Hexo版本更新


记录升级Hexo的注意事项。

<!--more-->

## 升级Hexo

### npm的全局软件更新

```shell
# 清理NPM缓存
$ npm cache clean -f

# 全局安装版本检测、版本升级工具
$ npm install -g npm-check
$ npm install -g npm-upgrade

# 全局检测哪些模块可以升级，这里可以根据打印的提示信息，手动安装最新版本的模块
$ npm-check -g

# 全局更新模块
$ npm update -g

# 全局安装或更新Hexo的最新版本
$ npm install --global hexo
```

### hexo当前目录的软件更新

```shell
# 进入博客的根目录
$ cd /blog-root

# 检测Hexo哪些模块可以升级
$ npm-check

# 删除package-lock.json
# rm -rf package-lock.json

# 更新package.json，一直回车即可
$ npm-upgrade

# 删除整个模块目录，这样可以避免很多坑
$ rm -rf node_modules

# 更新Hexo的模块
$ npm update --save

# 若出现依赖的问题，用以下命令检查一下，然后把报错的统一修复一下即可
$ npm audix

# 或者强制更新
$ npm update --save --force
```

### 检查

做完上述步骤完成后，可`package.json`检查文件中版本信息。

### 完成

至此Hexo的升级就结束了，但是不要着急将源文件上传到仓库，先在本地三连一下`hexo clean && hexo g -d`，如果在执行`Hexo d`的时候报错了，可以尝试删除`.deploy_git`文件夹里面的内容，这个是前面生成的网站项目内容，与当前的不兼容。

### 参考链接

* [Hexo版本更新](https://blog.sianx.com/posts/f16f368c/)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/hexo-upgrade/  

