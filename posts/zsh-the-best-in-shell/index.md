# Zsh - Shell中极品


zsh固然是好工具，但配置使用也同样重要，一定要“就手”。

<!--more-->

## 前言

[zsh](https://en.wikipedia.org/wiki/Z_shell)是来自普林斯顿大学的学生Paul Falstad于1990年发布的，这名字来自于当时也是普林斯顿大学的一名助教Zhong Shao（后来成为耶鲁大学教授）的login-id。zsh一开始的设计初衷是为Commodore公司于1985年出的个人电脑Amiga设计一个开源的csh的子集，结果一不小心就暴走了；它在发布的时候，已经成为了ksh和tcsh的交集——既吸收了ksh在命令式可编程脚本语言上的良好逻辑设计，又吸收了tcsh丝滑般的交互体验，集众家之长，自然牛逼了。

2019年，Apple的macOS Catalina将zsh作为默认shell，取代了bash；2020年，Kali Linux（一个著名的安全领域的Linux操作系统）从2020.4版本后，也将zsh作为默认shell。

### 简介

zsh 是一个 Linux 下强大的 shell, 由于大多数 Linux 产品安装，以及默认使用bash shell, 但是丝毫不影响极客们对 zsh 的热衷, 几乎每一款 Linux 产品都包含有 zsh，通常可以用 apt-get、urpmi 或 yum 等包管理器进行安装。

zsh 具有以下主要功能：

- 开箱即用、可编程的命令行补全功能可以帮助用户输入各种参数以及选项；
- 在用户启动的所有 shell 中共享命令历史；
- 通过扩展的文件通配符，可以不利用外部命令达到 find 命令一般展开文件名；
- 改进的变量与数组处理；
- 在缓冲区中编辑多行命令；
- 多种兼容模式，例如使用 / bin/sh 运行时可以伪装成 Bourne shell；
- 可以定制呈现形式的提示符；包括在屏幕右端显示信息，并在键入长命令时自动隐藏；
- 可加载的模块，提供其他各种支持：完整的 TCP 与 Unix 域套接字控制，FTP 客户端与扩充过的数学函数；
- 完全可定制化。

可见，zsh缺点就是配置太麻烦，好在有一个叫做oh-my-zsh的开源项目，很好的弥补了这一缺陷，只需要修修改改配置文件，就能很顺手。

## 安装配置

### zsh

在 CentOS7 下，执行`yum`命令来安装需要的`zsh`原始程序与`git`程序来`pull`代码。

```bash
yum install -y zsh git

# 查看系统shell
cat /etc/shells
# 或
chsh -l

# 更换
chsh -s /bin/zsh
# 或
chsh -s $(which zsh)
```

### oh-my-zsh

安装[oh-my-zsh](https://github.com/ohmyzsh/ohmyzsh)脚本 (这一步需要先安装 git)。

```sh
sh -c "$(wget -O- https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Powerline字体

oh-my-zsh配置中采用的是Powerline字体，本地环境中若不包含该系列字体，则会出现乱码的问题。

```sh
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
# uninstall
./uninstall.sh
```

### 配置 ~/.zshrc

```
# 配置主题
ZSH_THEME="agnoster"
```

## 插件

使用 ZSH 替换原有的 SHELL 最主要的原因就是要使用其功能强大的插件，这里只推荐安装三个插件，它们分别是 wd、zsh-syntax-highlighting、zsh-autosuggestions 。

获取安装：

```sh
cd ~/.oh-my-zsh/custom/plugins
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git
git clone https://github.com/zsh-users/zsh-autosuggestions.git
```

编辑 ~/.zshrc 增加配置：

```
plugins=(git wd zsh-syntax-highlighting zsh-autosuggestions)
source ~/.oh-my-zsh/custom/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
source ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions/zsh-autosuggestions.zsh
```

## 更多

### vscode中设置终端

在`setting`中搜索`terminal`选项，然后修改`Integrated: Font Family`的值为powerline字体即可。例如本人采用的是 Source Code Pro for Powerline 。

---

> 作者:   
> URL: https://blog.iqzhi.com/posts/zsh-the-best-in-shell/  

