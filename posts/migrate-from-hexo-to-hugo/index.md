# 从Hexo迁移到Hugo


无意网上看到大神使用Hugo建立的Blog，升起了自建的念头。

<!--more-->

## 为什么要迁移

1. 可能是一颗喜欢折腾的心。
2. 评估原来Hexo博客的情况，迁移工作不难。
3. Hugo优势更吸引，独立二进制文件，简洁高效。
4. Blog原有的文章数不多，希望有个新的开始，是时候积累点，无论是否流水账。

## 对博客发展的顾虑，或是考虑过多

1. 还是不喜欢为博客配图，这个无法从根本上改善博客的观感。也可能是未找到靠谱的图库解决方案。
2. 总想着以后可能又不会继续书写下去，插入太多的图片，使用框架程序内特定的语法会影响后续迁移可能，带来的不确定性。
3. 所以有点感觉是放不开手脚去做，畏手畏尾，从而什么都没有做。

## 迁移发现惊喜

1. 原来的Hexo博客已经折腾出后台管理解决方案了，通过Qoex可解决，但因为部署在Vercel上，大陆访问有一定的卡顿。
2. 由于Qoex也支持Hugo，本计划也使用其进行后台管理，但总有不得力的感觉，谈不上喜欢或讨厌，也没有书写的兴致。
3. 最后我也找到了合适的解决方案，通过VS Code可以远程管理部署在Debian上的Hugo程序代码，也方便在工作环境中远程管理，完美。

## 网站建设思路

1. 通过ZeroTier建立虚拟局域网，把家庭环境、工作环境、外出场景打通。
2. Hugo程序部署在Debian服务器上，开放SSH访问远程管理，生成环境唯一部署，减少出错可能性。
3. 利用VS Code插件Remote SSH连接Debian系统上的Hugo源码目录，撰写文章。
4. VS Code终端运行命令发布预览，自带端口转发部署，可实时调试页面展示。
5. VS Code直接源码提交Github管理，使用Vercel监控提交记录，实现实时生成部署。

## 问题解决

{{< admonition question "Github：通过sshkey的方式拉取代码报错kex_exchange_identification: Connection closed by remote">}}
近来家庭网络进行了调整，增加了科学上网，应该是网络引发问题产生；网上查资料，通过编辑 `~/.ssh/config` 文件（没有就新增），在用户目录下的.ssh目录，添加如下内容解决：
```md
Host github.com
    HostName ssh.github.com
    User git
    Port 443
```
{{< /admonition >}}


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/migrate-from-hexo-to-hugo/  

