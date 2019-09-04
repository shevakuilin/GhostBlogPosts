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

##### 优势

- 技术上，采用 Node.js，并发能力远超使用 [PHP](http://php.net/) 的 Wordpress，独立测试表明，Ghost 比 WordPress 快了 **1,900％**。什么意思呢？就是说，在 WordPress 响应1个请求所花费的时间内，Ghost 已经响应了其中的19个请求。博客的速度会影响一切，从搜索引擎排名到移动用户互动。


    <img src="https://i0.wp.com/wphive.com/wp-content/uploads/2019/01/ghost-speed.png?w=1024&ssl=1" width="800" height ="222" />


- 易用性上，没有 Wordpress 那么臃肿，可以专注于写作，完美支持 MarkDown，整合了评论系统，官方和开源社区都提供了很多精致的主题皮肤。
- 使用上，可随时随地编辑，简洁便利的可视化后台比 Hexo,  Jekyll 这类静态博客要书写方便，无需担心考虑多设备同步的问题。

##### 劣势

- 需要配套支持 Node 环境的虚拟机，一般免费的很少支持，有一定的开销。
- Ghost 后台提供的功能过于基础，不过基本满足日常写作。

##### 亮点

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



##### 1.安装 Node 环境

Node.js 有两种安装方式，一种是直接部署，另一种是编译部署。

前者安装快，干净简洁，适合新手，缺点是全局安装插件需要手动创建关联；后者全局安装 Node.js 模块，无需手动创建关联，缺点是安装慢，编译过程长达30分钟以上，需手动安装 gcc 等其他工具，部署完的 Node.js 也分别放在了好几个文件夹里，不适合新手。

这里我们使用直接部署安装：

```vim
$ curl -sL https://rpm.nodesource.com/setup_10.x | bash -

$ yum install -y nodejs
```

安装完成后，输入 node -v 后回车，如果显示 v10.16.3 则表示 Node.js 安装成功。


##### 2.安装 Nginx


2.1 首先在 `/etc/yum.repos.d/` 目录下创建一个源配置文件 `nginx.repo`：

```vim
$ vim /etc/yum.repos.d/nginx.repo
```

2.2 写入以下内容：

```vim
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
gpgcheck=0
enabled=1
```

完成后 `:wq` 保存退出。

2.3 安装 Nginx：

```vim
$ yum install nginx -y
```

2.4 设置 Nginx 服务器开机启动：

```vim
$ systemctl enable nginx.service
```

2.5 启动 Nginx 并查看 Nginx 服务状态：

```vim
$ systemctl start nginx.service

$ systemctl status nginx.service
```

到此 Nginx 就安装完成了，到浏览器地址栏输入你的云服务器或 VPS 的 IP 后回车，就能看到 "Welcome to Nginx!" 字样了。


##### 3.安装 MySQL (MariaDB)

Ghost 默认使用的是 `Sqlite3` 数据库，如果内容多的话，推荐改用 MySQL 数据库。如果不需要则跳过此步。

> 从 CentOS 7 开始，CentOS 使用 MariaDB 代替了 MySQL 数据库，MariaDB 系 MySQL 的一个分支，使用方法和 MySQL 基本一致，主要由开源社区维护，采用 GPL 授权许可。开发此分支的主要原因之一：Orcale 公司收购了 MySQL，所以 MySQL 有闭源的可能，因此社区采用了开源的 MariaDB 来规避此风险。


3.1 安装：

```vim
$ yum install mariadb mariadb-server #询问是否要安装，输入Y即可自动安装,直到安装完成

$ systemctl enable mariadb.service #设置开机启动
```

常用命令：

`systemctl start/stop/restart mariadb.service #启动/停止/重启 MariaDB`

3.2 修改 MySQL 密码：

直接输入 `mysql` 进入 MariaDB，然后输入：
```vim
$ set password for 'root'@'localhost'=password('');
```
这一行是设置本地密码，密码为空，本地就不需要每次都输密码了。

3.3 MySQL 授权远程连接（navicat等）：
```vim
$ grant all on *.* to root identified by 'root';
```

3.4 创建 ghost 数据库，用来存放博客的数据：
```vim
$ create database ghost;
```

3.5 新建用户：

```vim
$ grant all privileges on ghost.* to 'ghost'@'%' identified by '123456';
```
这里的意思是，新建一个用户 ghost，密码为 123456，用户名及密码自行更改。

3.6 重新读取权限表中的数据到内存，不用重启 MySQL 就可以让权限生效：
```vim
$ flush privileges;
```
到此数据库准备完成，输入 `exit` 退出MariaDB。


3.7 为避免数据库里的中文出现乱码，还需配置 MySQL 字符集编码：

通过 vim 指令打开 `my.cnf` 文件：
```vim
$ vim /etc/my.cnf
```

按 `i` 键进入编辑模式，在原有的基础上插入以下内容：
```vim
[client]
default-character-set=utf8  
[mysql]
default-character-set=utf8  
[mysqld]
character-set-server=utf8  
collation-server=utf8_general_ci
```
输入完成后按esc键，再输入:wq保存退出。

3.8 为了防止之后 Ghost 启动时无法访问我们的数据库，还需要在上面的基础上再添加一些内容来保证访问：
```vim
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
[mysqld]
skip-grant-tables # 就是它
character-set-server=utf8
collation-server=utf8_general_ci
datadir=/var/lib/mysql
```
:wq 保存后，重启数据库：`systemctl restart mariadb.service`


##### 4.安装 Ghost

###### 4.1 检查安装环境

4.1.1 检查 Node.js：
查看安装的 Node.js 版本号：
```vim
$ node -v
```
```vim
v10.16.3
```

查看 Node.js 安装路径：
```vim
$ sudo which node
```
```vim
/bin/node
```
**注意**：Node.js 需要安装在系统路径，比如 `/usr/bin/node` 或者 `/usr/local/bin/node` 等。

4.1.2 检查 Nginx

检查 nginx 版本：
```vim
$ nginx -v
```
```vim
nginx version: nginx/1.16.1
```
**注意**： 如果需要配置 SSL 使用 https，则 nginx 版本需要大于等于 1.9.5。

检查 nginx 服务是否正在运行：
```vim
$ systemctl is-active nginx
```
```vim
active
```

4.1.3 检查 MySQL

查看 MySQL 版本：
```vim
$ mysql --version
```
```vim
mysql  Ver 15.1 Distrib 5.5.60-MariaDB, for Linux (x86_64) using readline 5.1
```

检查 MySQL 服务是否正在运行：
```vim
$ systemctl is-active mariadb.service
```
```vim
active
```

###### 4.2 开始安装

4.2.1 安装 Ghost-CLI
Ghost-CLI 可以方便对未来的 Ghost 状态管理，升级。装上 Ghost-CLI 以后，之后的版本升级只需要 ghost update 一下就可以了。

使用 npm 全局安装 ghost-cli ：
```vim
$ sudo npm i -g ghost-cli
```

查看安装的 ghost-cli 版本：
```vim
$ ghost version
```
```vim
Ghost-CLI version: 1.11.0
```

4.2.2 创建新用户
**注意**： 如果已有拥有 sudo 权限的非 root 用户，跳过此步骤。

> 由于 Ghost-CLI 不允许在 root 用户下使用，会提示 `You can't run commands as the 'root' user.`，所以需要创建一个新用户。

新建一个用户：
```vim
$ adduser <user>
```
**注意**：`<user>` 为新建用户的用户名。注意这个名字不能为 ghost，因为 Ghost 会创建一个叫 ghost 的用户。Ghost-CLI 会创建一个用户名为 ghost 的系统用户和用户组来自动运行 Ghost。

设置用户密码：
```vim
$ passwd <user>
```
输入两遍用户密码。

赋予 /etc/sudoers 文件写权限：
```vim
$ chmod -v u+w /etc/sudoers
```

编辑文件：
```vim
$ vim /etc/sudoers
```

找到这个位置：
```vim
## Allow root to run any commands anywhere
root    ALL=(ALL)       ALL
```

在下面添加：
```vim
# 该用户在使用 sudo 命令时不需要输入密码
<user>    ALL=(ALL)       NOPASSWD:ALL
# or 该用户在使用 sudo 命令时需要输入密码
<user>    ALL=(ALL)       ALL
```

保存退出，并恢复 /etc/sudoers 文件权限：
```vim
$ chmod -v u-w /etc/sudoers
```

4.2.3 配置 Ghost
登录刚才创建好的用户，或已创建的`非 root 且拥有 sudo 权限的用户名不为 ghost `的用户
```vim
$ su - <user>
```

创建网站目录并设置权限:
```vim
$ sudo mkdir -p /var/www/ghost
$ sudo chown <user>:<user> /var/www/ghost
$ sudo chmod 775 /var/www/ghost
```
`<user>` 为刚才登陆的用户

进入到网站目录：
```vim
$ cd /var/www/ghost
```

4.2.4 安装 Ghost
官方文档推荐使用 Ubuntu，在安装过程中Ghost-CLI会对当前 OS 类型进行检查，为了避免报错，CentOS 需要加上 `--no-stack`
```vim
$ ghost install --no-stack
```
```vim
✔ Checking system Node.js version
✔ Checking logged in user
✔ Checking current folder permissions
ℹ Checking operating system compatibility [skipped]
Local MySQL install not found. You can ignore this if you are using a remote MySQL host.
Alternatively you could:
a) install MySQL locally
b) run `ghost install --db=sqlite3` to use sqlite
c) run `ghost install local` to get a development install using sqlite3.
? Continue anyway?
```
因为安装的是 MariaDB，所以直接输入 `y` 跳过，等待下载完成。

```vim
✔ Checking system Node.js version
✔ Checking logged in user
✔ Checking current folder permissions
ℹ Checking operating system compatibility [skipped]
Local MySQL install not found. You can ignore this if you are using a remote MySQL host.
Alternatively you could:
a) install MySQL locally
b) run `ghost install --db=sqlite3` to use sqlite
c) run `ghost install local` to get a development install using sqlite3.
? Continue anyway? Yes
MySQL check skipped
ℹ Checking for a MySQL installation [skipped]
✔ Checking memory availability
✔ Checking for latest Ghost version
✔ Setting up install directory
✔ Downloading and installing Ghost v2.30.2
✔ Finishing install process
? Enter your blog URL: (http://localhost:2368)
```
输入自己网站完整访问路径 https://shevakuilin.com 或 ip 地址（记得加上 http）。
回车：
```vim
? Enter your blog URL: [https://shevakuilin.com](https://shevakuilin.com)
? Enter your MySQL hostname: (localhost)
```

输入 MySQL 的登陆地址，本机登陆就是 localhost 直接回车即可（如果是远程数据库，输入数据库服务器的 ip 地址，不要带上 http）：
```vim
? Enter your MySQL hostname: localhost
? Enter your MySQL username:
```

输入 root：
```vim
? Enter your MySQL username: root
? Enter your MySQL password: [input is hidden]
```
```vim
输入 MySQL 的 root 用户密码：
按照我们之前设置的数据库密码，本地数据库密码为空，远程为 root
```vim
? Enter your MySQL password: [hidden]
? Enter your Ghost database name:
```

输入要创建的数据库的名称，回车会直接使用默认名称：
```vim
✔ Configuring Ghost
✔ Setting up instance
Running sudo command: chown -R ghost:ghost /var/www/ghost/content
✔ Setting up "ghost" system user
? Do you wish to set up "ghost" mysql user? (Y/n)
```

回车确认自动创建 MySQL 用户：
```vim
? Do you wish to set up "ghost" mysql user? Yes
✔ Setting up "ghost" mysql user
? Do you wish to set up Nginx? (Y/n)
```

直接回车确定自动设置 Nginx：
```vim
? Do you wish to set up Nginx? Yes
Nginx is not installed. Skipping Nginx setup.
ℹ Setting up Nginx [skipped]
Task ssl depends on the 'nginx' stage, which was skipped.
ℹ Setting up SSL [skipped]
? Do you wish to set up Systemd? (Y/n)
```
此时 Ghost-CLI 还无法识别 CentOS 上已安装的 Nginx，稍后手动设置

直接回车确实自动设置系统服务（一定不要选择 No）：
```vim
? Do you wish to set up Systemd? Yes
```

之后会看到配置和初始化完成的提示，然后输入密码：
```vim
✔ Configuring Ghost
✔ Setting up instance
? Sudo Password [input is hidden]
```

由于版本差异，你也可能看到这样的提示：
```vim
? Do you wish to set up Systemd? Yes
✔ Creating systemd service file at /var/www/ghost/system/files/ghost_shevakuilin-com.service
Running sudo command: ln -sf /var/www/ghost/system/files/ghost_shevakuilin-com.service /lib/systemd/system/ghost_shevakuilin-com.service
Running sudo command: systemctl daemon-reload
✔ Setting up Systemd
Running sudo command: /var/www/ghost/current/node_modules/.bin/knex-migrator-migrate --init --mgpath /var/www/ghost/current
✔ Running database migrations
? Do you want to start Ghost? (Y/n)
```
现在启动是不会成功的，还需要设置 Nginx 的反向代理。

4.2.5 设置 nginx
新建配置文件：
```vim
$ sudo vim /etc/nginx/conf.d/ghost.conf
```
将以下内容输入到 `ghost.conf` 中，将 `server_name` 后面的 `example.com` 改为你的 IP 或者域名：
```vim
server {  
     listen 80;
     server_name example.com;

     access_log /var/log/nginx/ghost.log;
     error_log /var/log/nginx/ghost_error.log;

     proxy_buffers 16 64k;
     proxy_buffer_size 128k;

     location / {
         proxy_pass http://127.0.0.1:2368;
         proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
         proxy_redirect off;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header Host $http_host;
         proxy_set_header X-NginX-Proxy true;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header X-Forwarded-Proto $scheme;
     }
}
```
输入完成后保存退出，重启 nginx 服务：
```vim
$ sudo systemctl restart nginx
```

4.2.6 设置 Ghost 生产模式配置文件
编辑生产模式配置文件：
```vim
$ vim /var/www/ghost/config.production.json
```

复制一下内容，将原内容覆盖掉：
```vim
{
     "url": "http://xx.xx.xx.xx",
     "server": {
         "port": 2368
     },
     "database": {
         "client": "mysql",
         "connection": {
             "host": "127.0.0.1",
             "user": "ghost数据库用户名",
             "password": "ghost数据库密码",
             "database": "ghost数据库名",
             "charset":"utf8"
         }
     },
     "paths": {
         "contentPath": "content/"
     },
     "logging": {
         "level": "info",
         "rotation": {
             "enabled": true
         },
         "transports": ["file", "stdout"]
     }
}
```
保存后退出。

### 六、测试

启动当前网站服务：
```vim
$ sudo systemctl start ghost_shevakuilin-com
```

查看服务状态：
```vim
$ sudo systemctl status ghost_shevakuilin-com
```

使用绑定的域名尝试访问自己的网站，访问 `http://<domain>/ghost` 注册管理员账号。

可正常访问，则将该服务设置为开机启动：
```vim
$ sudo systemctl enable ghost_shevakuilin-com
```

### 七、其他

在启动 Ghost 之后，如果你关闭终端窗口或者从 SSH 断开连接时，Ghost 就停止了。为了防止 Ghost 停止工作，我们需要使用 [PM2](https://github.com/Unitech/pm2) 让 Ghost 保持运行：
```vim
$ cd /var/www/ghost
$ npm install pm2 -g # 安装PM2
$ NODE_ENV=production
$ pm2 start index.js --name "ghost"
$ pm2 startup centos pm2 save
```
如果提示找不到 `index.js`，cd 到 `/var/www/ghost/current` 目录下继续执行命令即可。

如果 npm 安装依赖的时候无法安装，需要把镜像换成淘宝的，再试试：
```vim
$ npm install -g cnpm --registry= https://registry.npm.taobao.org
$ cnpm install pm2 -g
$ NODE_ENV=production pm2 start index.js --name "ghost"
$ pm2 startup centos
$ pm2 save
```

```vim
pm2 start/stop/restart/status ghost # 启动/停止/重启/查看状态
```

至此，Ghost 就全部搭建完毕，打开你的浏览器，在浏览器中输入 域名地址/ghost/，开始初始化用户名，密码，就可以开始愉快的Ghost之旅了。

以后每次升级，只需要在 Ghost 安装目录下执行 `ghost update [version]` 即可。

### 最后

为了搭建 Ghost 一路踩了很多坑，足足花了两三天的时间，不过在搭建完成进入网站的那一刹那，什么都值了，即便是默认的主题皮肤也帅爆了啊。

参考链接：

[Ghost 博客搭建日记](https://halfrost.com/ghost_build/)
[Ghost 博客升级指南](https://halfrost.com/ghost_update/)
[Ghost 博客系统搭建](https://myfirstwon.com/ghost-install/)
[CentOS 7下搭建Ghost 2.6.2博客的超详细教程](https://art3mis.info/chao-xiang-xi-centos-7da-jian-ghost-2-6-2bo-ke-jiao-cheng/)
[WordPress vs. Ghost – Which One is Better? [2019]](https://wphive.com/reviews/wordpress-vs-ghost-which-one-is-better/)
[Ghost CLI](https://ghost.org/docs/api/v2/ghost-cli/)
[基于阿里云ECS的Centos7搭建Ghost](https://segmentfault.com/a/1190000010158007)
[Centos7.0搭建lamp环境](http://blog.6le6le.com/?p=140)
[Install Ghost](https://ghost.org/docs/install/ubuntu/#create-a-directory)
