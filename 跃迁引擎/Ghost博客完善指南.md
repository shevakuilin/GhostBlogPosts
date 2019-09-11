# 域名备案
为了让网站的域名能够正常的访问到，我们需要申请管局的备案，由于此前买的是阿里云的 ECS，所以直接使用阿里云来进行备案，步骤非常简单。而且最爽的是，你在阿里云进行备案，花费多长时间，就会送你多长时间的 ECS 使用时间，相当于免费的续费服务。

<img src="https://github.com/shevakuilin/GhostPostsImages/raw/master/WechatIMG484.jpeg" width="800" height ="550" />

## 备案流程

### 1.验证备案类型：
填写部分主体和网站信息，系统将根据你所填写的信息，自动验证你要办理的备案类型。
<img src="http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/190022/156439620646090_zh-CN.png" width="800" height ="435" />

### 2.产品验证：
对搭建备案网站的云服务器进行验证。
<img src="http://static-aliyun-doc.oss-cn-hangzhou.aliyuncs.com/assets/img/157603/156439614350083_zh-CN.png" width="800" height ="435" />

### 3.填写主体信息和网站信息：
填写网站信息以及办理备案的个人或者单位的真实信息，按照内容填写即可，详情[参考](https://help.aliyun.com/knowledge_detail/36948.html#section-e7u-ig4-58s)。

### 4.上传资料及真实性核验：
下载阿里云 APP，然后上传证件照片或证件彩色扫描件，并通过人脸识别完成真实性核验。完成这一步之后，就不需要再进行幕布拍照然后寄回去审核的繁琐步骤了。

### 5.信息确认：
完成备案信息填写及资料上传、真实性核验后，需要对所有信息做最终确认，以保证信息真实准确，避免备案申请被驳回。

### 6.备案初审：
备案申请信息提交后，阿里云将在 1 个工作日进行初审。需要保持备案信息中的联系电话畅通以便阿里云的工作人员与你核实信息，一般第二天就会给你打电话进行确认。

### 7.邮寄资料：
根据你提交的资料及各地管局的要求，有可能需要你按照系统指示邮寄资料至指定地点，不过一般都不需要。

### 8.短信核验：
天津、甘肃、西藏、宁夏、海南、新疆、青海、浙江、四川、福建、陕西、重庆、广西、云南、山东、河南、安徽、湖南、山西、黑龙江、内蒙古、湖北，这些省份的备案信息需要进行短信核验，你会在初审通过后的几小时内收到由 `工信部备案系统` 发送的验证码，进入短信提供的链接地址，输入验证码进行确认，需要在 24 小时之内完成验证，否则审核会被驳回。

短信验证完成后，备案申请流程自动提交至管局审核。

### 9.管局审核：
初审完成后，阿里云备案审核专员会将备案申请转交至对应管局处做最终的管局审核。管局审核通过后你的备案即已完成，审核结果会发送 `短信`、`邮箱`通知，通知内会告诉你备案号及备案密码等信息。

备案申请信息成功提交至管局系统后，管局审核一般为 `3~20 个工作日`，不过一般都是一周左右，我大概是 8 天就收到了短信和邮件。

### 10.备案号显示在网站页脚：
备案通过后，需要在网站的页脚显示备案号，并链接到[工信部的网站](http://beian.miit.gov.cn/)。

复制下面代码，替换后插入网站页脚：
```html
<a href="http://beian.miit.gov.cn/" style="color: #ffffff">  你的备案号</a>
```

# Disque
如果你需要一个快速的方法来获得 Ghost 网站上的全功能评论，Disqus 是最适合的。

> 注意：虽然 Disqus 是免费的，但它[在 2017 年被一家广告技术公司收购](https://techcrunch.com/2017/12/05/zeta-global-acquires-commenting-service-disqus/)，可能会在您的网站上注入广告和广告跟踪器。
> 

### 1.复制Ghost-Disqus评论代码

首先，将此 Ghost-Specific Disqus 评论代码复制到剪贴板上。这与 Disqus 通用代码不同，这是专门针对 Ghost 主题进行的定制代码段：
```html
<div id="disqus_thread"></div>
<script>
var disqus_config = function () {
this.page.url = "{{url absolute="true"}}";  
this.page.identifier = "ghost-{{comment_id}}"
};
(function() {
var d = document, s = d.createElement('script');
s.src = 'https://EXAMPLE.disqus.com/embed.js';
s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
```

### 2.将注释代码粘贴到 post.hbs 中

进入主题文件夹，找到 `post.hbs` ，进入编辑：
```vim
$ vim post.hbs
```

如果你使用的是默认主题 **Casper**，会看到专门为插入注释而保留的一行代码。这就是你要粘贴 Disqus 代码的地方：
<img src="https://mainframe.ghost.io/content/images/2018/10/image-15.png" width="800" height ="200" />

注意：需要将第 65 行和第 69 行的注释删除。


### 3.找到你的 Disqus shortname

访问 [Disqus Admin](https://disqus.com/admin/)，创建一个站点或选择一个现有站点。从站点的设置区域中，找到shortname并复制它。
<img src="https://mainframe.ghost.io/content/images/2018/10/image-16.png" width="800" height ="200" />

### 4.插入 shortname 到 Disqus 代码中

将下面代码中的 `EXAMPLE` 替换为你的 `shortname`
```html
s.src = 'https://EXAMPLE.disqus.com/embed.js';
```
保存退出，重启 Ghsot 即可生效。
# 全站 HTTPS
[Certbot (Let's Encrypt)](https://github.com/certbot/certbot) 是一个公共的免费 SSL 项目，使用简单方便，由 Mozilla、思科、Akamai、IdenTrust 和 EFF 等组织发起，Facebook 等大公司赞助，目的就是向网站自动签发和管理免费证书，以便加速互联网由 HTTP 过渡到 HTTPS。

## 选择反向代理服务器 & 系统

进入 [官网](https://certbot.eff.org/)，按下图箭头所示选择你的反向代理服务器和系统，我选择的是 Nginx 和 CentOS 7，请参照你自己的具体情况进行选择。

<img src="https://github.com/shevakuilin/GhostPostsImages/raw/master/WechatIMG473.png" width="1000" height ="535" />

选择完成后会自动跳转到对应的文档，然后一步步按照文档说明操作，即可完成`安装`和 `HTTPS 证书自动续订`的工作。

### 1.SSH进入服务器

以具有 sudo 权限的用户身份 SSH 进入运行 HTTP 网站的服务器。

### 2.启用EPEL

按照 Fedora wiki 上的这些说明 [启用EPEL](https://fedoraproject.org/wiki/EPEL#How_can_I_use_these_extra_packages.3F)。

EPEL 包含在 CentOS Extras 存储库中，默认情况下已启用，为了保险起见还是安装检查一下：
```vim
$ yum install epel-release # 安装 EPEL
```


### 3.启用可选通道 (CentOS 系统跳过这一步)

只有使用的是 RHE L或 Oracle Linux，才需要启用可选通道。在 EC2 上，RHEL 用户可以通过运行以下命令启用可选通道，在命令中将 EC2 区域替换为 REGION：
```vim
$ yum -y install yum-utils
# yum-config-manager --enable rhui-REGION-rhel-server-extras rhui-REGION-rhel-server-optional
```

### 4.初始化 Certbot
```vim
$ sudo yum install certbot python2-certbot-nginx
```
一路选择 `y`，直到出现 `Complete!` 

这时你可能会看到一个 `Failed` 的失败提示：
```vim
Failed:
python-urllib3.noarch 0:1.10.2-5.el7
```
暂时忽略，在下一步中解决。

### 5.选择如何运行 Certbot & 获取并安装 HTTPS 证书
```vim
$ sudo certbot --nginx
```
该命令会获取一个 HTTPS 证书，并让 Certbot 自动编辑你的 Nginx 配置来支持 HTTPS。

这个时候你会收到一个报错信息：
```vim
ImportError: No module named 'requests.packages.urllib3'
```
这是 Python 的模块版本的问题，卸载现有版本的 `requests` 和 `urllib3`，然后重新安装：
```vim
$ pip uninstall requests
$ pip uninstall urllib3
$ yum remove python-urllib3
$ yum remove python-requests
$ yum install python-urllib3
$ yum install python-requests
$ sudo yum install certbot python2-certbot-nginx
```
再次执行：
```vim
$ sudo certbot --nginx
```

ok 没有了刚才的问题，继续刚才被中断的流程:
```vim
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: shevakuilin.com
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 1
```
选择你准备配置 HTTPS 的域名，输入域名前的数字。

看到下列信息表示你的 HTTPS 已经全部配置完成了：
```vim
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://shevakuilin.com
```
在这段信息下面的 `IMPORTANT NOTES` 中，会列出你的证书安装路径，以及到期时间。

为了安全起见，Certbot 默认的证书过期时间是 `90` 天，通过第 5 步的设置，就不需要担心这一点了，会自动续订我们的 HTTPS 证书。

### 6. 测试

现在你就可以测试你的 HTTPS 新网站了，测试配置地址：
```vim
https://www.ssllabs.com/ssltest/analyze.html?d=shevakuilin.com
```
shevakuilin.com 换成你自己的域名。

然后直接打开浏览器，输入你自己的域名，就能看到 HTTPS 的网站了。

# DNS CAA
Todo...
# CDN 图床
Todo...
# 汉化
Todo...
# 代码高亮
Ghost 默认是不提供代码高亮支持的，但是在后台提供了一个代码插入的配置入口 `Code injection`，可以在其中嵌入第三方插件的高亮代码。

以 `highlight.js` 为例，我们使用[这个网站](https://www.bootcdn.cn/highlight.js/
)提供的 CDN，进入该网站后，选择你想要高亮的语言，**注意：每个语言的高亮必须独立支持，也就是说，你没有相应语言的高亮代码，该语言就不会有高亮效果**。

## 设置 Code injection

首先进入你的 Ghost 后台，在浏览器中输入 `域名/ghost`，在设置里找到 `Code injection`。

以我的高亮代码为例，在 `Site Header` 中填入：
```html
<link href="https://cdn.bootcss.com/highlight.js/9.15.10/styles/monokai-sublime.min.css" rel="stylesheet">
```

在 `Site footer` 中填入：
```html
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/highlight.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/swift.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/objectivec.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/cpp.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/ruby.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/java.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/javascript.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/php.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/python.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/go.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/json.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/css.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/vim.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/shell.min.js"></script>
<script src="https://cdn.bootcss.com/highlight.js/9.15.10/languages/bash.min.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```

然后保存设置。

## 使用

以 `Swift` 代码高亮为例，你只需要

``` Swift
代码 
``` 
# 修改字体
在浏览器中输入 `域名/ghost`进入后台，在设置里找到 `Code injection`。

将你的字体添加到 `Site Header` ：
```html
<!--  加载google字体 -->
<link rel="stylesheet" type="text/css" href="//fonts.googleapis.com/css?family=Droid+Sans+Mono">
<!--  设置字体 -->
<style>
body,p,code,h1, h2, h3, h4, h5, h6,.hljs {
font-family: 'Droid Sans Mono', monospace;
}
.hljs {
font-size: 0.5em
}
</style>
```
字体库可以在谷歌的[字体库](http://www.googlefonts.cn/)里找。

# 自定义主题
Todo...

参考链接：
[Certbot](https://certbot.eff.org/)
[Centosrhel7-nginx](https://certbot.eff.org/lets-encrypt/centosrhel7-nginx)
[How_can_I_use_these_extra_packages](https://fedoraproject.org/wiki/EPEL#How_can_I_use_these_extra_packages.3F)
[centos7部署certbot 出现 requests.packages.urllib3 错误 的处理](https://www.52zheteng.info/服务器/centos7-使用certbot-出现-requests-packages-urllib3-错误-的处理/)
[Certbot提示requests.packages.urllib3解决](https://zoco.me/post/certbot-requests-packages-urllib3-resolve)
[首次备案](https://help.aliyun.com/knowledge_detail/36922.html)
