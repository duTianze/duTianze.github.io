---
title:  Cafe Themed Hexo Add Gitalk
date:  2019-09-27 20:15:00
tags: hexo
---

## Gitalk 介绍
Gitalk是一个基于GitHub Issue和Preact开发的评论插件。

### 特性
* 使用GitHub登录
* 支持多语言[en, zh-CN, zh-TW, es-ES, fr, ru]
* 支持个人或组织
* 无干扰模式(设置distractionFreeMode为 true 开启)
* 快捷键提交评论(cmd|ctrl + enter)

## 创建 OAuth Application
打开[这个链接](https://github.com/settings/developers)，点击New OAuth App，填入以下信息：
* `Application name`：应用名称，可以随便填
* `Homepage URL`：主页 URL，一般填你的博客地址
* `Application description`：应用描述，选填
* `Authorization callback UR`L：授权回调地址，必须填你的博客地址
接着你就可以得到 `Client ID` 和 `Client Secret`。

## 新建 github 仓库保存评论
gitalk评论采用的是github仓库issues的方式，所以你需要将它与你的仓库联系起来。建议新建一个仓库，专门用于保存评论。
但是我的直接用的是blog仓库 dutianze.github.io。

<!--more-->
## 修改主题
拿到`Client ID`和`Client Secret`后，接下来，我们要到我们主题配置文件下，新增如下配置：

`_config.yml`
```yaml
# Gitalk
gitalk:
    enable: true     # 是否开启评论功能
    clientID: xxxxxxxxxx    # 刚才生成的 Client ID
    clientSecret: xxxxxxxxxxxxxx    # Client Secret，同上
    repo: blog-comment  # 刚才新建的用于保存评论的仓库名
    owner: isJaakko # 这个项目的拥有者（填你的账户名）
    admin: ['isJaakko'] # 管理员用户，可以填多个
    ID: location.pathname   # 页面 ID默认值，用于生成的 Issues 的 Labels 的值
    perPage: 15 # 每页最大评论数
    pagerDirection: last    # 排序方式是从旧到新（first）还是从新到旧（last）
    createIssueManually: true   #如果当前页面没有相应的 isssue ，且登录的用户属于 admin，则会自动创建 issue。如果设置为 true，则显示一个初始化页面，创建 issue 需要点击 init 按钮。
    distractionFreeMode: false  #是否启用快捷键(cmd|ctrl + enter) 提交评论.
```

* 找到你所使用的主题中，渲染文章的页面，如我使用的主题cafe中，文件目录为cafe/layout/_third-part/comments/
添加`gitalk.ejs`
```html
<div id="gitalk-container"></div>

// 引入 gitalk
<link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
<script src="https://unpkg.com/gitalk@latest/dist/gitalk.min.js"></script>
// 引入 MD5 加密算法
<script src="https://cdn.bootcss.com/blueimp-md5/2.10.0/js/md5.min.js"></script>

<script type="text/javascript">
    var gitalk = new Gitalk({
      clientID: '<%= theme.gitalk.clientID %>',
      clientSecret: '<%= theme.gitalk.clientSecret %>',
      repo: '<%= theme.gitalk.repo %>',
      owner: '<%= theme.gitalk.owner %>',
      admin: '<%= theme.gitalk.admin %>',
      id: md5(location.pathname),   // 这里需要注意！稍后会说
      createIssueManually: '<%= theme.gitalk.createIssueManually %>',
      distractionFreeMode: '<%= theme.gitalk.distractionFreeMode %>'
    });

    gitalk.render('gitalk-container');
</script>
```

* 目录cafe/layout/_third-part/comments/，引入刚才写的js
修改`index.ejs`，添加一行
```html
<% include ./gitalk.ejs %>
```

* 目录cafe/layout/_partial/
`comments.ejs`，末尾添加
```html
      <% if (theme.gitalk.enable){ %>
           <div id="gitalk-container"></div>
    <% } %>
```

## 提示
1. 每篇文章都需要管理员(刚才admin字段所填内容)手动打开初始化一下，才能使用评论.
2. 为gitalk的id使用MD5加密生成唯一32位字符串，否则当文章名过长时，评论会初始化错误。
3. 文章名称中如果使用了中文字符，会导致登陆出错，所以避免在文章名中使用中文字符。所以我以后的标题都会是英文 微笑。
4. 不同hexo主题配置文件会有不同，但是基本上是这三段代码，gitalk的代码，引入的代码，开启的if代码