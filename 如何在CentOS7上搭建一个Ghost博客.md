### 前言

此前我的个人博客采用 [Hexo](https://hexo.io/zh-cn/index.html) + [Github Pages](https://pages.github.com/) 的方案，原因很简单：免费，静态博客部署方便、以及我喜欢的极简风格。Hexo 是用 [Node.js](https://nodejs.org/en) 编写的，页面生成及渲染速度都相当快，支持 Markdown、一键部署以及丰富的插件系统，使用体验一直都很不错，在相当长的一段时间里，它都是我认为最好的博客框架。

最初的邂逅总是美好，直到经历了几次设备更替，每次更换新电脑，或者从公司到家里的环境切换，都需要对本地的 Hexo 进行繁琐的迁移同步，三番两次下来，渐渐令我对 Hexo 失去耐心，我意识到，是时候去寻找一个代替品了。

同样身为静态博客框架的 [Jekyll](https://jekyllcn.com) 是我想到的第一个方案，它虽然可以解决 Hexo 上那糟糕的的多设备同步问题，但缺点同样明显：

- Github 对 Jekyll 的插件有很多限制
- Jekyll 本身是由 Ruby 编写的，渲染速度上完全不及 Hexo
- 与 Hexo 比起来，两者的关系更像互补，而不是代替

就在纠结当中，无意间逛到了这个 [博客](https://halfrost.com)， 由于长期以来都只用过静态博客框架，突然想趁着这次机会从头搭一个动态博客，折腾一下，于是便有了这篇文章。

在 [WordPress](https://wordpress.org/) 和 [Ghost](https://ghost.org/) 之间略微挣扎了一会，决定用 Ghost 开始我新的博客搭建之旅。

### 一、Ghost 简介

<img src="/content/images/2019/09/11b8145be285a17777ee4d809bfd4d09_XL.jpg" width="900" height ="450" />

Ghost 是一套基于 Node.js 构建的开源博客平台（Open source blogging platform），具有易用的书写界面和体验，博客内容默认采用 Markdown 语法书写，不过原生的不支持 Markdown 的表格和 [LaTeX](http://www.baidu.com/link?url=K_4JBAR3uS6xnec99adJX0IzXiUT0ANv52nbth0WUNaf3Z52ob2qSLRPX0KjGNEBBBXE5d8hEvOhKdgBR77EDa)，如果需要使用，需要在服务器端安装插件。

Ghost 目标是取代臃肿的 Wordpress，界面简洁，专注写作，支持在线预览和在线写作。最大的优势就是它是从头重新写起的 (Ghost 最初的团队都来自 Wordpress)。原来的 Wordpress 无论是代码架构、具体实现、设计、决策过程到开源协作模式都已经变得过时而臃肿，加上大量第三方插件对特定实现的依赖，任何大改动都牵一发而动全身，开发效率变得极低。

#### 优势

- 技术上，采用 Node.js，并发能力远超使用 [PHP](http://php.net/) 的 Wordpress，独立测试表明，Ghost 比 WordPress 快了 **1,900％**。什么意思呢？就是说，在 WordPress 响应1个请求所花费的时间内，Ghost 已经响应了其中的19个请求。博客的速度会影响一切，从搜索引擎排名到移动用户互动。


    <img src="https://i0.wp.com/wphive.com/wp-content/uploads/2019/01/ghost-speed.png?w=1024&ssl=1" width="800" height ="222" />


- 易用性上，没有 Wordpress 那么臃肿，可以专注于写作，完美支持 MarkDown，整合了评论系统，官方和开源社区都提供了很多精致的主题皮肤。
- 使用上，可随时随地编辑，简洁便利的可视化后台比 Hexo,  Jekyll 这类静态博客要书写方便，无需担心考虑多设备同步的问题。

#### 劣势

- 需要配套支持 Node 环境的虚拟机，一般免费的很少支持，有一定的开销。
- Ghost 后台提供的功能过于基础，不过基本满足日常写作。

#### 亮点

- 采用 Mysql 作为数据库， 通用快速上手，同样支持其他数据库比如 [MariaDB](https://mariadb.org/)， Sqlite。
- 支持主流的 [Apache](https://www.apache.org/)、[Nginx](https://www.nginx.com/) 作为反向代理，配置多个 Ghost 博客， 同时也能增加了网站的负载。
- 支持多种环境 Ubuntu（官方推荐），Docker，CentOS等。
- 支持第三方代码高亮，如 [highlight.js](http://highlightjs.org/)， [Prism.js](http://prismjs.com/) 和 [Rainbow](https://craig.is/making/rainbows)。
- 整合 [Disqus](https://disqus.com/) 评论系统, 建立属于自己的 Discuss 圈。
- 采用 [Font Awesome](http://fontawesome.io/icons/) 作为社交按钮，也可以自定义图标。
- 国外优秀免费 [Ghost 主题](http://themeforest.net/category/blogging/ghost-themes) 资源分享。

### 二、搭建方式

Ghost 主要分为 **手动搭建** 和 **自动搭建**（仅支持 Ghost 版本在 1.0 以后）两种方式：

- 手动搭建通常采取 zip 解压安装的方式，手动安装 Ghost，后续的 Ghost 状态管理，升级等都需要自己处理。
- 自动搭建通过 Ghost 1.0 版本后提供的工具 [Ghost-CLI](https://ghost.org/docs/api/v2/ghost-cli/) 来完成，非常方便管理，推荐使用，本文的教程也是以自动搭建的方式进行。

### 三、准备清单

- 一台服务器（本文使用阿里云ECS，服务器系统 CentOS 7.6 64位）
- 一个可用的域名（前期可使用服务器 ip 地址代替）

### 四、配置环境

- Node v10.16.3 （注意 Ghost 版本要求的对应 Node 版本号）
- Nginx 1.16.1
- Mysql （我实际安装的是 MariaDB，它是 Mysql 的开源分支，因为 Mysql 被收购了，可能有潜在的收费风险，所以推荐使用 MariaDB 了，完全兼容 Mysql）
- Ghost-CLI 1.11.0
- Ghost 2.30.2

### 五、搭建步骤

- 安装 Node 环境
