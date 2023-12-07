# 使用github的webhook自动触发宝塔部署的hugo博客更新


博客部署到vercel上，有时终究是会遇到网络抽风，听说雨云线路还不错，也有免费用度，于是搞了台雨云的香港宝塔面板，研究如何可以达到像vercel上的自动部署效果。

<!--more-->

## 解决方案

本地提交 hugo 源码到 Github，自动触发构建并同步到宝塔指定的网站目录。

### Github自动构建

在 Github 的 Hugo 源码仓库根目录，新建".github/workflows/xxx.yml 文件，复制以下代码。作用：借助 Github Action 实现自动构建，并同步到另外一个仓库。其中 `PERSONAL_TOKEN` 为另外仓库的访问密钥；`external_repository` 为另外仓库地址。

```yml
name: Githubblog

on:
  workflow_dispatch:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          submodules: true
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify

      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.PERSONAL_TOKEN }}
          external_repository: koobai/koobai.github.io
          publish_dir: ./public
          
```

### 宝塔面板终端生成 git 公钥

```bash
# Git全局配置和单个仓库的用户名邮箱配置
git config --global user.name  "username"
git config --global user.email  "your@email.com"

# 生成git公钥用于自动拉取
ssh-keygen -t rsa -C "你的@email.com"

# 查看git公钥
cat ~/.ssh/id_rsa.pub
```
### 添加公钥到到 Github

头像–Settings–SSH and GPG keys–New SSH key

### 宝塔面板安装 WebHook 触发同步

打开宝塔面板商店，安装 WebHook 插件–添加执行脚本 (复制以下代码)。其中"gitHttp 为需同步的 github 仓库地址"，“gh-pages"为仓库分支名称。

```bash
#!/bin/bash
echo ""
#输出当前时间
date --date='0 days ago' "+%Y-%m-%d %H:%M:%S"
echo "Start"
#git分支名称
branch="gh-pages"
#git项目路径
gitPath="/www/wwwroot/$1"
#git 仓库地址
gitHttp="git@github.com:yourname/yourname.github.io.git"
echo "Web站点路径：$gitPath"
#判断项目路径是否存在
if [ -d "$gitPath" ]; then
        cd $gitPath
        #判断是否存在git目录
        if [ ! -d ".git" ]; then
                echo "在该目录下克隆 git"
                sudo git clone $gitHttp gittemp
                sudo mv gittemp/.git .
                sudo rm -rf gittemp
        fi
        echo "拉取最新的项目文件"
        #sudo git reset --hard origin/$branch
        git remote add origin $gitHttp
        git branch --set-upstream-to=origin/$branch $branch
        sudo git reset --hard origin/$branch
        sudo git pull $gitHttp  2>&1
        echo "设置目录权限"
        sudo chown -R www:www $gitPath
        echo "End"
        exit
else
        echo "该项目路径不存在"
                echo "新建项目目录"
        mkdir $gitPath
        cd $gitPath
        #判断是否存在git目录
        if [ ! -d ".git" ]; then
                echo "在该目录下克隆 git"
                sudo git clone $gitHttp gittemp
                sudo mv gittemp/.git .
                sudo rm -rf gittemp
        fi
        echo "拉取最新的项目文件"
        #sudo git reset --hard origin/$branch
        sudo git pull gitHttp 2>&1
        echo "设置目录权限"
        sudo chown -R www:www $gitPath
        echo "End"
        exit
fi
```

### Github 网页仓库配置 WebHook

查看 WebHook 插件密钥，复制密钥地址。添加到 Github 需同步的仓库–Settings–Webhooks–Add webhook。其中 Content type 选择 application/json。

```
格式如：https://面板地址:面板端口/hook?access_key=密钥&param=需同步到的目录名称
```

至此，步骤全部完成。当本地提交新文件到 Github hugo 源码 main 分支，就会自动触发（hugo 生成静态文件——同步到另一个仓库——同步到宝塔网站指定目录）。如果域名指定境外访问路径是 vercel 或 cloudflare 服务，当 hugo 源码更新的时候也会自动触发构建更新。

## 参考

[Github 自动构建 Hugo, 并通过 Webhook 同步到宝塔指定目录](https://koobai.com/hugo_action_webhooks/)
[GitHub Pages Action](https://github.com/peaceiris/actions-gh-pages)


---

> 作者:   
> URL: https://blog.iqzhi.com/posts/github-webhook-bt-hugo/  

