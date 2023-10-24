# Cmder - 替代Cmd，打造Windows命令行军刀


Windows下终端工具。

<!--more-->

## 简介
[Cmder](http://cmder.net/)是一款Windows下终端工具，增强命令行的存在，把conemu，msysgit和clink打包在一起，让你无需配置就能使用一个真正干净的Linux终端！她甚至还附带了漂亮的monokai配色主题。官网下载的安装包其实是压缩档, 可即压即用。你甚至可以放到USB就可以虽时带着走，连调整过的设定都会放在这个目录下，不会用到系统机码(Registry)，所以也很适合放在Dropbox/Google Drive/OneDrive共享于多台电脑。

下载的时候，有两个版本，分别是mini与full版；唯一的差别在于有没有内建msysgit工具，这是Git for Windows的标准配备；全安装版cmder自带了msysgit, 压缩包23M, 除了git本身这个命令之外, 里面可以使用大量的linux命令；比如grep，curl(没有wget)；像vim，grep，tar，unzip，ssh，ls，bash，perl对于爱折腾的Coder更是痛点需求。

## 定制配置
为了军刀更锋利，更好用，少不了折腾一下配置，让它更适合个人的使用习惯。

- 打开管理员权限终端
    输入`Ctrl + t`，或者点击下面控制条的绿色加号，勾选"Run as administrator"

- 添加 cmder 到右键菜单
    打开管理员权限的终端，去到Cmder根目录，输入以下语句即可：
    ```shell
    Cmder.exe /REGISTER ALL
    ```

- 解决文字重叠问题
    `Win + ALT + P` 唤出设置界面 > mian > font > monospce,去掉那勾勾即可。

## Cmder常用快捷键
- 跟一般浏览器页签操作习惯一致：
- 可以利用Tab，自动路径补全(爽,赞！)；
- 可以利用Ctrl+T建立新页签；
- 利用Ctrl+W关闭页签;
- 还可以透过Ctrl+Tab切换页签;
- Alt+F4：关闭所有页签
- Alt+Shift+1：开启cmd.exe
- Alt+Shift+2：开启powershell.exe
- Alt+Shift+3：开启powershell.exe (系统管理员权限)
- Ctrl+1：快速切换到第1个页签
- Ctrl+n：快速切换到第n个页签( n值无上限)
- Alt + enter：切换到全屏状态；
- Ctr+r：历史命令搜索;
- End, Home, Ctrl：Traversing text with as usual on Windows

## 其他功能
- Cmder还增加了alias功能；他让你用短短的指令执行一些常见但指令超长又难以记忆的语法；比如 ls cls等等；在其控制台输入alias可以查看（新版本已废除）。
- 主控台文字自动放大缩小功能，你只要按下Ctrl+滑鼠滚轮就可以办到；如果你用支援两点触控的笔电，也可以在触控板上用两指放大的手势调整文字大小。还有：up，向上翻历史命令;
- Cmder有极为简单的复制粘贴操作。Ctr+V直接粘贴；用鼠标选中你想拷贝的内容，自动就复制到剪切板；
- 自定义aliases：打开Cmder目录下的config文件夹，里面的`user-aliases.cmd`文件就是我们可以配置的别名文件。
    ```cmd
    l=ls --show-control-chars -alrtF --color $*
    ```

## 问题及解决
### 问题一：中文乱码
一般的乱码在加了下边的软件环境变量后都可以解决，在`Settings > Startup > Environment`中添加：
```
set LC_ALL=zh_CN.UTF8
```

### 问题二：git status 显示中文乱码
场景：在中文情况下`Git status`是这样的:
```
Your branch is up-to-date with 'origin/master'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   "\345\274\200\351\242\230\346\212\245\345\221\212/\345\267\245\347\250\213\347\241\225\345\243\253\347\240\224\347\251\266\347\224\237\345\255\246\344\275\215\350\256\272\346\226\207\345\274\200\351\242\230\346\212\245\345\221\212\342\200\224\346\235\250\345\263\273\351\271\217.doc"
```

解决方法：
```shell
git config --global core.quotepath false
```

## 参考资料
- 《Cmder--Windows下命令行利器》：http://www.cnblogs.com/zqzjs/p/6188605.html
- 《Win下必备神器之Cmder》：https://segmentfault.com/a/1190000004408436
- 《Windows 命令行增强 cmder chocolatey 配置指南》：http://www.jianshu.com/p/479d974078a7


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/cmder/  

