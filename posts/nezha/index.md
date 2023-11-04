# 哪吒监控 - 轻量服务器运维监控工具


[哪吒监控](https://nezha.wiki)，可以说是拿到VPS，更新完系统后，几乎第一个就要装的监控程序。

<!--more-->

几乎支持各种系统部署。

## 透明主题

```js
<style>
/* 屏幕适配 */
@media only screen and (min-width: 1200px) {
    .ui.container {
    width: 80% !important;
}
}

@media only screen and (max-width: 767px) {
    .ui.card>.content>.header:not(.ui), .ui.cards>.card>.content>.header:not(.ui) {
        margin-top: 0.4em !important;
    }
}

/* 整体图标 */
i.icon {
    color: #000;
    width: 1.2em !important;
}

/* 背景图片 */
body {
    content: " " !important;
    background: fixed !important;
    z-index: -1 !important;
    top: 0 !important;
    right: 0 !important;
    bottom: 0 !important;
    left: 0 !important;
    background-position: top !important;
    background-repeat: no-repeat !important;
    background-size: cover !important;
    background-image: url(https://gitee.com/darki/img/raw/master/1631081013043.webp) !important;
    font-family: Arial,Helvetica,sans-serif !important;
}

/* 导航栏 */
.ui.large.menu {
    border: 0 !important;
    border-radius: 0px !important;
    background-color: rgba(255, 255, 255, 55%) !important;
}

/* 首页按钮 */
.ui.menu .active.item {
    background-color: transparent !important;
}

/* 导航栏下拉框 */
.ui.dropdown .menu {
    border: 0 !important;
    border-radius: 0 !important;
    background-color: rgba(255, 255, 255, 80%) !important;
}

/* 登陆按钮 */
.nezha-primary-btn {
    background-color: transparent !important;
    color: #000 !important;
}

/* 大卡片 */
#app .ui.fluid.accordion {
    background-color: #fbfbfb26 !important;
    border-radius: 0.4rem !important;
}

/* 小卡片 */
.ui.four.cards>.card {
    border-radius: 0.6rem !important;
    background-color: #fafafaa3 !important;
}

.status.cards .wide.column {
    padding-top: 0 !important;
    padding-bottom: 0 !important;
    height: 3.3rem !important;
}

.status.cards .three.wide.column {
    padding-right: 0rem !important;
}

.status.cards .wide.column:nth-child(1) {
    margin-top: 2rem !important;
}

.status.cards .wide.column:nth-child(2) {
    margin-top: 2rem !important;
}

.status.cards .description {
    padding-bottom: 0 !important;
}

/* 小鸡名 */
.status.cards .flag {
    margin-right: 0.5rem !important;
}

/* 弹出卡片图标 */
.status.cards .header > .info.icon {
    margin-right: 0 !important;
}

.nezha-secondary-font {
    color: #21ba45 !important;
}

/* 进度条 */
.ui.progress {
    border-radius: 50rem !important;
}

.ui.progress .bar {
    min-width: 1.8em !important;
    border-radius: 15px !important;
    line-height: 1.65em !important;
}

.ui.fine.progress> .bar {
    background-color: #21ba45 !important;
}

.ui.progress> .bar {
    background-color: #000 !important;
}

.ui.progress.fine .bar {
    background-color: #21ba45 !important;
}

.ui.progress.warning .bar {
    background-color: #ff9800 !important;
}

.ui.progress.error .bar {
    background-color: #e41e10 !important;
}

.ui.progress.offline .bar {
    background-color: #000 !important;
}

/* 上传下载 */
.status.cards .outline.icon {
    margin-right: 1px !important;
}

i.arrow.alternate.circle.down.outline.icon {
    color: #21ba45 !important;
}

i.arrow.alternate.circle.up.outline.icon {
    color: red !important;
}

/* 弹出卡片小箭头 */
.ui.right.center.popup {
    margin: -3px 0 0 0.914286em !important;
    -webkit-transform-origin: left 50% !important;
    transform-origin: left 50% !important;
}

.ui.bottom.left.popup {
    margin-left: 1px !important;
    margin-top: 3px !important;
}

.ui.top.left.popup {
    margin-left: 0 !important;
    margin-bottom: 10px !important;
}

.ui.top.right.popup {
    margin-right: 0 !important;
    margin-bottom: 8px !important;
}

.ui.left.center.popup {
    margin: -3px .91428571em 0 0 !important;
    -webkit-transform-origin: right 50% !important;
    transform-origin: right 50% !important;
}

.ui.right.center.popup:before,
.ui.left.center.popup:before {
    border: 0px solid #fafafaeb !important;
    background: #fafafaeb !important;
}

.ui.top.popup:before {
    border-color: #fafafaeb transparent transparent !important;
}

.ui.popup:before {
    border-color: #fafafaeb transparent transparent !important;
}

.ui.bottom.left.popup:before {
    border-radius: 0 !important;
    border: 1px solid transparent !important;
    border-color: #fafafaeb transparent transparent !important;
    background: #fafafaeb !important;
    -webkit-box-shadow: 0px 0px 0 0 #fafafaeb !important;
    box-shadow: 0px 0px 0 0 #fafafaeb !important;
    -webkit-tap-highlight-color: rgba(0,0,0,0) !important;
}

.ui.bottom.right.popup:before {
    border-radius: 0 !important;
    border: 1px solid transparent !important;
    border-color: #fafafaeb transparent transparent !important;
    background: #fafafaeb !important
    -webkit-box-shadow: 0px 0px 0 0 #fafafaeb !important;
    box-shadow: 0px 0px 0 0 #fafafaeb !important;
    -webkit-tap-highlight-color: rgba(0,0,0,0) !important;
}

.ui.top.left.popup:before {
    border-radius: 0 !important;
    border: 1px solid transparent !important;
    border-color: #fafafaeb transparent transparent !important;
    background: #fafafaeb !important;
    -webkit-box-shadow: 0px 0px 0 0 #fafafaeb !important;
    box-shadow: 0px 0px 0 0 #fafafaeb !important;
    -webkit-tap-highlight-color: rgba(0,0,0,0) !important;
}

.ui.top.right.popup:before {
    border-radius: 0 !important;
    border: 1px solid transparent !important;
    border-color: #fafafaeb transparent transparent !important;
    background: #fafafaeb !important;
    -webkit-box-shadow: 0px 0px 0 0 #fafafaeb !important;
    box-shadow: 0px 0px 0 0 #fafafaeb !important;
    -webkit-tap-highlight-color: rgba(0,0,0,0) !important;
}

.ui.left.center.popup:before {
    border-radius: 0 !important;
    border: 1px solid transparent !important;
    border-color: #fafafaeb transparent transparent !important;
    background: #fafafaeb !important;
    -webkit-box-shadow: 0px 0px 0 0 #fafafaeb !important;
    box-shadow: 0px 0px 0 0 #fafafaeb !important;
    -webkit-tap-highlight-color: rgba(0,0,0,0) !important;
}

/* 弹出卡片 */
.status.cards .ui.content.popup {
    min-width: 20rem !important;
    line-height: 2rem !important;
    border-radius: 5px !important;
    border: 1px solid transparent !important;
    background-color: #fafafaeb !important;
    font-family: Arial,Helvetica,sans-serif !important;
}

.ui.content {
    margin: 0 !important;
    padding: 1em !important;
}

/* 服务页 */
.ui.table {
    background: RGB(225,225,225,0.6) !important;
}

.ui.table thead th {
    background: transparent !important;
}

/* 服务页进度条 */
.service-status .good {
    background-color: #21ba45 !important;
}

.service-status .danger {
    background-color: red !important;
}

.service-status .warning {
    background-color: orange !important;
}

/* 版权 */
.ui.inverted.segment, .ui.primary.inverted.segment {
    color: #000 !important;
    font-weight: bold !important;
    background-color: #fafafaa3 !important;
}
</style>

<!--Logo和版权-->
<script>
window.onload = function(){
var avatar=document.querySelector(".item img")
var footer=document.querySelector("div.is-size-7")
footer.innerHTML="SimplyMiss の 天眼"
footer.style.visibility="visible"
avatar.src="https://blog.870211.xyz/images/maple-transparent.png"
avatar.style.visibility="visible"
}
var faviconurl="https://b.izhi.ml/images/maple-transparent.png" ;            
var link = document.querySelector("link[rel*='icon']") || document.createElement('link');
link.type = 'image/x-icon';
link.rel = 'shortcut icon';
link.href = faviconurl;
document.getElementsByTagName('head')[0].appendChild(link);
</script>
```

## Custom主题 [转载](https://blog.amzayo.com/index.php/archives/28/)

![show](https://cdn.amzayo.top/static/imgs/21.png)

[下载地址](https://cdn.amzayo.top/static/public/amzayo_template_fin_2.91.zip)

使用方法
1.template里面的文件解压出来放到服务端/opt/nezha/dashboard/theme-custom/template目录里面

2.重启哪吒面板服务

3.在哪吒面板后台主题选择Custom(local)

4.将css.txt里面的CSS样式代码复制到哪吒面板后台的“自定义代码”文本框里

## 注意事项

- 将面板和数据域名换成https的过程中，如果套在CloudFlare上，建议使用`完全`或`完全（严格）`的方式，因为如果使用的是`灵活`，CloudFlare有可能回落使用80端口与服务器间的访问，导致连通性各种问题，时通时不通。


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/nezha/  

