# Sublime Text - 个人认为最好用的文本编辑器


跨平台的文本编辑器。

&lt;!--more--&gt;

## 简介
[Sublime Text](http://www.sublimetext.com/)是付费软件，但非常之良心，只是偶尔弹出提示购买，并没有任何功能限制。作为一个跨平台的文本编辑器，支持基于Python的插件。可通过包（Package）扩充本身的功能。大多数的包使用自由软件授权发布，并由社区建置维护。

虽然Sublime Text 3一直在Beta，但很多插件已经开始放弃支持ST2了，所以推荐使用ST3。

* 下载地址戳这里：http://www.sublimetext.com/3

* 3143可用`Sublime Text`许可证一枚

    ```
    ----- BEGIN LICENSE -----
    TwitterInc
    200 User License
    EA7E-890007
    1D77F72E 390CDD93 4DCBA022 FAF60790
    61AA12C0 A37081C5 D0316412 4584D136
    94D7F7D4 95BC8C1C 527DA828 560BB037
    D1EDDD8C AE7B379F 50C9D69D B35179EF
    2FE898C4 8E4277A8 555CE714 E1FB0E43
    D5D52613 C3D12E98 BC49967F 7652EED2
    9D2D2E61 67610860 6D338B72 5CF95C69
    E36B85CC 84991F19 7575D828 470A92AB
    ------ END LICENSE ------
    ```

* 3126可用`Sublime Text`许可证一枚

    ```
    —– BEGIN LICENSE —–
    Michael Barnes
    Single User License
    EA7E-821385
    8A353C41 872A0D5C DF9B2950 AFF6F667
    C458EA6D 8EA3C286 98D1D650 131A97AB
    AA919AEC EF20E143 B361B1E7 4C8B7F04
    B085E65E 2F5F5360 8489D422 FB8FC1AA
    93F6323C FD7F7544 3F39C318 D95E6480
    FCCC7561 8A4A1741 68FA4223 ADCEDE07
    200C25BE DBBC4855 C4CFB774 C5EC138C
    0FEC1CEF D9DCECEC D3A5DAD1 01316C36
    —— END LICENSE ——
    ```

## 定制配置
正所谓，没有插件的编辑器不是好的美工刀。
要有一把顺手的美工刀，总少不了定（调）制（教）。

1. 安装插件管理器
    **ctrl&#43;\`** 打开调试窗口，在输入框内粘贴如下代码，然后回车即可自动安装，安装完成可能需要重启ST。

    ```
    import urllib.request,os,hashlib; h = &#39;2915d1851351e5ee549c20394736b442&#39; &#43; &#39;8bc59f460fa1548d1514676163dafc88&#39;; pf =     &#39;Package Control.sublime-package&#39;; ipp = sublime.installed_packages_path(); urllib.request.install_opener(  urllib.request.build_opener( urllib.request.ProxyHandler()) ); by = urllib.request.urlopen( &#39;http:// packagecontrol.io/&#39; &#43; pf.replace(&#39; &#39;, &#39;%20&#39;)).read(); dh = hashlib.sha256(by).hexdigest(); print(&#39;Error validating   download (got %s instead of %s), please try manual install&#39; % (dh, h)) if dh != h else open(os.path.join( ipp, pf),   &#39;wb&#39; ).write(by)
    ```

    使用方法：工具栏 Preferences – Package Control，或者 ctrl&#43;shift&#43;P ，插件的安装卸载都可以在这里操作。
    如何安装插件详见：https://packagecontrol.io/installation

2. 推荐安装的插件列表

    易用性：
    ChineseLocalization：完全汉化插件
    HTML5：HTML5标签拓展
    JsFormat：javascript格式化
    CSS Format：CSS格式化
    Tag：HTML格式化
    [Brackethighlighter](https://packagecontrol.io/packages/BracketHighlighter)：标签对标记，以高亮显示配对括号以及当前光标所在区域
    SideBarEnhancements：增强型侧边栏
    BufferScroll：代码折叠状态保留
    StyleToken：标记颜色代码
    [AutoPEP8](https://packagecontrol.io/packages/AutoPEP8)：格式化Python代码。
    [Alignment](https://packagecontrol.io/packages/Alignment)：进行智能对齐
    
    功能：
    [Markdown Editing](https://packagecontrol.io/packages/MarkdownEditing)：支持Markdown语法高亮；支持Github Favored Markdown语法；自带3个主题。本篇文章就是在此插件下完成
    Emmet：前端神器
    TortoiseSVN：SVN你懂的
    QuoteHTML：把HTML拼接成js插入字符串，神器
    Clipboard Manager：增强型剪贴板，可访问历史剪贴板记录
    FileHeader：文件模板，可自动更新修改时间
    AutoPrefixer：浏览器私有属性前缀补全 (Node.js依赖)
    ColorConvert：RGBA颜色转换，十六进制颜色转换为RGBA颜色
    Better Completion：全能代码提示
    LiveStyle：双向更改无刷新实时预览，包含chrome插件 Emmet LiveStyle
    SFTP：需要激活，看这里 http://mooring.iteye.com/blog/2067269
    jQuery：jQuery 代码提示（Better Completion已可替代此插件）
    
    其他：
    [ConvertToUTF8](https://packagecontrol.io/packages/ConvertToUTF8)：GBK编码兼容
    [IMESupport](https://packagecontrol.io/packages/IMESupport)：输入法不跟随时安装
    TrailingSpaces：多余空格标记，强迫症患者福音
    Hasher：符号转义，ctrl&#43;shift&#43;p 选择 Entity Encode
    PackageResourceViewer：插件修改必备，ctrl&#43;shift&#43;p 调用 Open Resource

3. 用户配置文件设置
    工具栏 Preferences – Settings-User 加入下面的代码：
    
    ```
    // 使光标更柔和
    &#34;caret_style&#34;: &#34;phase&#34;,
    // 自动移除行尾多余空格
    &#34;trim_trailing_white_space_on_save&#34;: true,
    // 文件末尾自动保留一个空行
    &#34;ensure_newline_at_eof_on_save&#34;: true,
    // 设置字体，推荐可以尝试 Yahei Consolas Hybrid
    &#34;font_face&#34;: &#34;Microsoft Yahei UI&#34;,
    // 设置字体大小
    &#34;font_size&#34;: 11,
    // 设置为 true ，禁用 Emmet 的 tab 键功能（请使用 ctrl&#43;e），系统自带的`tab`  功能还是可圈可点的。当然也可以不设置它，以完全使用 Emmet 的 tab 补全功能
    &#34;disable_tab_abbreviations&#34;: true,
    // 把代码 tab 对齐转换为空格对齐
    &#34;translate_tabs_to_spaces&#34;: true,
    // 配合设置空格数
    &#34;tab_size&#34;: 2,
    // 用于右侧代码预览时给所在区域加上边框，方便识别
    &#34;draw_minimap_border&#34;: true,
    // 窗口失焦立即保存文件
    &#34;save_on_focus_lost&#34;: true,
    // 当前行高亮
    &#34;highlight_line&#34;: true,
    // 设置自动换行
    &#34;word_wrap&#34;: &#34;true&#34;,
    // 默认显示行号右侧的代码段闭合展开三角号
    &#34;fade_fold_buttons&#34;: false,
    // 侧边栏文件夹显示加粗，区别于文件
    &#34;bold_folder_labels&#34;: true,
    // 高亮未保存文件
    &#34;highlight_modified_tabs&#34;: true,
    // 使用 unix 风格的换行符
    &#34;default_line_ending&#34;: &#34;unix&#34;,
    // 开启选中范围内搜索，而不是整个文档
    &#34;auto_find_in_selection&#34;: true
    ```

4. Windows系统将Sublime Text 添加到鼠标右键功能：
    把以下内容复制并保存到文件，重命名为：sublime_addright.reg，然后双击就可以了。

    ```reg
    Windows Registry Editor Version 5.00
    [HKEY_CLASSES_ROOT\*\shell\SublimeText3]
    @=&#34;Edit with Sublime Text3&#34;
    &#34;Icon&#34;=&#34;C:\\Program Files\\Sublime Text 3\\sublime_text.exe,0&#34;
    [HKEY_CLASSES_ROOT\*\shell\SublimeText3\command]
    @=&#34;C:\\Program Files\\Sublime Text 3\\sublime_text.exe %1&#34;
    [HKEY_CLASSES_ROOT\Directory\shell\SublimeText3]
    @=&#34;Edit with Sublime Text3&#34;
    &#34;Icon&#34;=&#34;C:\\Program Files\\Sublime Text 3\\sublime_text.exe,0&#34;
    [HKEY_CLASSES_ROOT\Directory\shell\SublimeText3\command]
    @=&#34;C:\\Program Files\\Sublime Text 3\\sublime_text.exe %1&#34;
    ```

    注意：
    1. 需要把代码中的Sublime的安装目录（C:\\Program Files\\Sublime Text 3\\sublime_text.exe），替换成自已实际的Sublime安装目录。
    2. 其中，@=&#34;Edit with Sublime Text3&#34; 引号中的内容为出现在鼠标右键菜单中的文字内容。

## 使用技巧

### 代码段（Code Snippets）
Sublime Text 支持代码段（Code Snippet），输入代码段名称后 Tab 即可生成代码段。
可以通过Package Control安装第三方代码段，也可以自己创建代码段，参考[这里](http://www.hongkiat.com/blog/sublime-code-snippets/)

### 全屏（Full Screen）
Sublime Text 有两种全屏模式：普通全屏和无干扰全屏。
个人强烈建议在开启全屏前关闭菜单栏（Toggle Menu），否则全屏效果会大打折扣。

`F11` 切换普通全屏
`Shift &#43; F11` 切换无干扰全屏

### 格式化（Formatting）
Sublime Text 基本的手动格式化操作包括： `Ctrl &#43; [` 向左缩进， `Ctrl &#43; ]` 向右缩进，此外 `Ctrl &#43; Shift &#43; V` 可以以当前缩进粘贴代码（非常实用）。

### 快捷键列表（Shortcuts Cheatsheet）
我把本文出现的Sublime Text按其类型整理在这里，以便查阅。

#### 通用（General）
* `↑↓←→`：上下左右移动光标，注意不是不是 KJHL ！
* `Alt`：调出菜单
* `Ctrl &#43; Shift &#43; P`：调出命令板（Command Palette）
* **Ctrl &#43; \`**：调出控制台

#### 编辑（Editing）
* `Ctrl &#43; Enter`：在当前行下面新增一行然后跳至该行
* `Ctrl &#43; Shift &#43; Enter`：在当前行上面增加一行并跳至该行
* `Ctrl &#43; ←/→`：进行逐词移动
* `Ctrl &#43; Shift &#43; ←/→`：进行逐词选择
* `Ctrl &#43; ↑/↓`：移动当前显示区域
* `Ctrl &#43; Shift &#43; ↑/↓`：移动当前行

#### 选择（Selecting）
* `Ctrl &#43; D`：选择当前光标所在的词并高亮该词所有出现的位置，再次 `Ctrl &#43; D` 选择该词出现的下一个位置，在多重选词的过程中，使用 `Ctrl &#43; K` 进行跳过，使用 `Ctrl &#43; U` 进行回退，使用 `Esc` 退出多重编辑
* `Ctrl &#43; Shift &#43; L`：将当前选中区域打散
* `Ctrl &#43; J`：把当前选中区域合并为一行
* `Ctrl &#43; M`：在起始括号和结尾括号间切换
* `Ctrl &#43; Shift &#43; M`：快速选择括号间的内容
* `Ctrl &#43; Shift &#43; J`：快速选择同缩进的内容
* `Ctrl &#43; Shift &#43; Space`：快速选择当前作用域（Scope）的内容

#### 查找&amp;替换（Finding&amp;Replacing）
* `F3`：跳至当前关键字下一个位置
* `Shift &#43; F3`：跳到当前关键字上一个位置
* `Alt &#43; F3`：选中当前关键字出现的所有位置
* `Ctrl &#43; F/H`：进行标准查找/替换，之后：
* `Alt &#43; C`：切换大小写敏感（Case-sensitive）模式
* `Alt &#43; W`：切换整字匹配（Whole matching）模式
* `Alt &#43; R`：切换正则匹配（Regex matching）模式
* `Ctrl &#43; Shift &#43; H`：替换当前关键字
* `Ctrl &#43; Alt &#43; Enter`：替换所有关键字匹配
* `Ctrl &#43; Shift &#43; F`：多文件搜索&amp;替换

#### 跳转（Jumping）
* `Ctrl &#43; P`：跳转到指定文件，输入文件名后可以：
    * `@` 符号跳转：输入 @symbol 跳转到 symbol 符号所在的位置
    * `#` 关键字跳转：输入 #keyword 跳转到 keyword 所在的位置
    * `:` 行号跳转：输入 :12 跳转到文件的第12行。
* `Ctrl &#43; R`：跳转到指定符号
* `Ctrl &#43; G`：跳转到指定行号

#### 窗口（Window）
* `Ctrl &#43; Shift &#43; N`：创建一个新窗口
* `Ctrl &#43; N`：在当前窗口创建一个新标签
* `Ctrl &#43; W`：关闭当前标签，当窗口内没有标签时会关闭该窗口
* `Ctrl &#43; Shift &#43; T`：恢复刚刚关闭的标签

#### 屏幕（Screen）
* `F11`：切换普通全屏
* `Shift &#43; F11`：切换无干扰全屏
* `Alt &#43; Shift &#43; 2`：进行左右分屏
* `Alt &#43; Shift &#43; 8`：进行上下分屏
* `Alt &#43; Shift &#43; 5`：进行上下左右分屏
* 分屏之后，使用 `Ctrl &#43; 数字键` 跳转到指定屏，使用 `Ctrl &#43; Shift &#43; 数字键` 将当前屏移动到指定屏。例如， `Ctrl &#43; 1` 会跳转到1屏，而 `Ctrl &#43; Shift &#43; 2` 会将当前屏移动到2屏。

## 常用插件
* [Markdown Editing](https://packagecontrol.io/packages/MarkdownEditing)
    * 常用快捷键（Key Bindings）
        `Ctrl &#43; 1...6` 1至6级标题
        `Ctrl &#43; Win &#43; V` 或 `Ctrl &#43; Win &#43; R` 插入链接
        `Shift &#43; Win &#43; K` 插入图片
        `Alt &#43; Shift &#43; 6` 插入标注
    * 代码段（Code Snippet）
        输入 `mdl &#43; tab` 会自动生成下面的链接标记

        ```
        [](link)
        ```

        输入 `mdi &#43; tab` 会自动插入下面的图片标记

        ```
        ![Alt text](/path/to/img.jpg &#34;Optional title&#34;)
        ```

## 参考资料
* 官方文档：http://www.sublimetext.com/docs/3/
* 官方论坛：http://www.sublimetext.com/forum/
* Package Control：https://packagecontrol.io/  大量的 Sublime Text 插件和主题。
* 《Sublime Text 全程指南》：http://zh.lucida.me/blog/sublime-text-complete-guide/
* 《Sublime Text：学习资源篇》：http://www.jianshu.com/p/d1b9a64e2e37


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/sublime-text/  

