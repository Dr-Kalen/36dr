title: Hexo 静态博客 部署Github
date: 2015-11-03 13:42:11
tags: 技术文章
---

## Hexo简介
Hexo 是一个简单地、轻量地、基于Node的一个静态博客框架。通过Hexo我们可以快速创建自己的博客，仅需要几条命令就可以完成。<br>
Hexo可以部署在自己的服务器上面，也可以部署github上面。对于个人用户来说，部署在github上好处颇多，不仅可以省去服务器的成本，还可以减少各种系统运维的麻烦事(系统管理、备份、网络)。
Hexo的官方网站：http://hexo.io/ ，也是基于Github构建的网站。

## Hexo 安装步骤

Hexo 安装
``` text
# npm install -g hexo
```
<!--more-->
查看hexo的版本,验证安装成功
``` text
# hexo version
hexo: 2.5.5
os: Windows_NT 6.1.7601 win32 x64
http_parser: 1.0
node: 0.10.5
v8: 3.14.5.8
ares: 1.9.0-DEV
uv: 0.10.5
zlib: 1.2.3
modules: 11
openssl: 1.0.1e
```
安装好后，我们就可以使用Hexo创建项目了。
``` text
# hexo init nodejs-hexo
[info] Creating file: source/_posts/hello-world.md
[info] Creating file: package.json
[info] Creating file: .gitignore
[info] Copying file: _config.yml
[info] Copying file: scaffolds/draft.md
[info] Copying file: scaffolds/page.md
[info] Copying file: scaffolds/photo.md
[info] Copying file: scaffolds/post.md
[info] Creating folder: source/_drafts
[info] Creating folder: scripts
[info] Copying theme data...
[info] Initialization has been done. Start blogging with Hexo!
```
我们看到当前在目录下，出现了一个文件夹，包括初始化的文件。

进入目录，并启动Hexo服务器。
``` text
# 进入目录
# cd nodejs-hexo

# 启动hexo服务器
# hexo server
[info] Hexo is running at http://localhost:4000/. Press Ctrl+C to stop.
```

这时端口4000被打开了，我们能过浏览器打开地址，http://localhost:4000/ 。

## Hexo 使用

1 Hexo 目录结构
* scaffolds 脚手架，也就是一个工具模板
* scripts 写文件的js，扩展hexo的功能
* source 存放博客正文内容
* source_drafts 草稿箱
* source_posts 文件箱
* themes 存放皮肤的目录
* themes_landscape 默认的皮肤
* config.yml 全局的配置文件
* db.json 静态常量

在这里，我们每次用到的就是_posts目录里的文件，而_config.yml文件和themes目录是第一次配置好就行了。
<br>
_posts目录：Hexo是一个静态博客框架，因此没有数据库。文章内容都是以文本文件方式进行存储的，直接存储在_posts的目录。Hexo天生集成了markdown，我们可以直接使用markdown语法格式写博客，例如:hello-world.md。新增加一篇文章，就在_posts目录，新建一个xxx.md的文件。
<br>
themes目录：是存放皮肤的，包括一套Javascript+CSS样式和基于EJS的模板设置。通过在themes目录下，新建一个子目录，就可以创建一套新的皮肤，当然我们也可以直接在landscape上面修改。
<br>

2 全局配置
_config.yml是全局的配置文件：很多的网站配置都在这个文件中定义。

* 站点信息: 定义标题，作者，语言
* URL: URL访问路径
* 文件目录: 正文的存储目录
* 写博客配置：文章标题，文章类型，外部链接等
* 目录和标签：默认分类，分类图，标签图
* 归档设置：归档的类型
* 服务器设置：IP，访问端口，日志输出
* 时间和日期格式： 时间显示格式，日期显示格式
* 分页设置：每页显示数量
* 评论：外挂的Disqus评论系统
* 插件和皮肤：换皮肤，安装插件
* Markdown语言：markdown的标准
* CSS的stylus格式：是否允许压缩
* 部署配置：github发布
查看文件：_config.yml

``` text
# Hexo Configuration
## Docs: http://hexo.io/docs/configuration.html
## Source: https://github.com/tommy351/hexo/

# 站点信息
title: Hexo博客
subtitle: 新的开始
description: blog.fens.me
author: bsspirit
email: bsspirit@gmail.com
language: zh-CN

# URL
## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
url: http://blog.fens.me
root: /
permalink: :year/:month/:day/:title/
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code

# 文件目录
source_dir: source
public_dir: public

# 写博客配置
new_post_name: :title.md # File name of new posts
default_layout: post
auto_spacing: false # Add spaces between asian characters and western characters
titlecase: false # Transform title into titlecase
external_link: true # Open external links in new tab
max_open_file: 100
multi_thread: true
filename_case: 0
render_drafts: false
post_asset_folder: false
highlight:
  enable: true
  line_number: true
  tab_replace:

# 目录和标签
default_category: uncategorized
category_map:
tag_map:

# 归档设置
## 2: Enable pagination
## 1: Disable pagination
## 0: Fully Disable
archive: 2
category: 2
tag: 2

# 服务器设置
## Hexo uses Connect as a server
## You can customize the logger format as defined in
## http://www.senchalabs.org/connect/logger.html
port: 4000
server_ip: 0.0.0.0
logger: false
logger_format:

# 时间和日期格式
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: MMM D YYYY
time_format: H:mm:ss

# 分页设置
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page

# 评论
disqus_shortname:

# 插件和皮肤
## Plugins: https://github.com/tommy351/hexo/wiki/Plugins
## Themes: https://github.com/tommy351/hexo/wiki/Themes
theme: landscape
exclude_generator:

# Markdown语法
## https://github.com/chjj/marked
markdown:
  gfm: true
  pedantic: false
  sanitize: false
  tables: true
  breaks: true
  smartLists: true
  smartypants: true

# CSS的stylus格式
stylus:
  compress: false

# 部署配置
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type:
3 命令行使用

查看命令行帮助


D:\> hexo help
Usage: hexo

Commands:
  help      Get help on a command
  init      Create a new Hexo folder
  migrate   Migrate your site from other system to Hexo
  version   Display version information

Global Options:
  --config   Specify config file instead of using _config.yml
  --debug    Display all verbose messages in the terminal
  --safe     Disable all plugins and scripts
  --silent   Hide output on console

For more help, you can use `hexo help [command]` for the detailed information
or you can check the docs: http://hexo.io/docs/
```
命令行解释：

* help 查看帮助信息
* init 创建一个hexo项目
* migrate 从其他系统向hexo迁移
* version 查看hexo的版本
* –config参数，指定配置文件，代替默认的_config.yml
* –debug参数，调试模式，输出所有日志信息
* –safe参数，安全模式，禁用所有的插件和脚本
* –silent参数，无日志输出模式

创建新文章
``` text
D:\workspace\javascript\nodejs-hexo>hexo new 新的开始
[info] File created at D:\workspace\javascript\nodejs-hexo\source\_posts\新的开始.md
```

3 发布到Github
写完了文章，我们就可以发布了。要说明的一点是hexo的静态博客框架，那什么是静态博客呢？静态博客，是只包含html, javascript, css文件的网站，没有动态的脚本。虽然我们是用Node进行的开发，但博客的发布后就与Node无关了。在发布之前，我们要通过一条命令，把所有的文章都做静态化处理，就是生成对应的html, javascript, css，使得所有的文章都是由静态文件组成的。

``` text
D:\workspace\javascript\nodejs-hexo>hexo generate
[info] Files loaded in 0.895s
[create] Public: js\script.js
[create] Public: css\fonts\fontawesome-webfont.svg
[create] Public: css\fonts\FontAwesome.otf
[create] Public: css\fonts\fontawesome-webfont.ttf
[create] Public: css\fonts\fontawesome-webfont.eot
[create] Public: css\fonts\fontawesome-webfont.woff
[create] Public: fancybox\blank.gif
[create] Public: fancybox\fancybox_loading@2x.gif
[create] Public: fancybox\fancybox_overlay.png
[create] Public: css\images\banner.jpg
[create] Public: fancybox\fancybox_sprite.png
[create] Public: fancybox\jquery.fancybox.css
[create] Public: fancybox\fancybox_loading.gif
[create] Public: fancybox\fancybox_sprite@2x.png
[create] Public: fancybox\jquery.fancybox.js
[create] Public: fancybox\jquery.fancybox.pack.js
[create] Public: fancybox\helpers\jquery.fancybox-buttons.js
[create] Public: fancybox\helpers\fancybox_buttons.png
[create] Public: fancybox\helpers\jquery.fancybox-buttons.css
[create] Public: fancybox\helpers\jquery.fancybox-media.js
[create] Public: fancybox\helpers\jquery.fancybox-thumbs.css
[create] Public: fancybox\helpers\jquery.fancybox-thumbs.js
[create] Public: archives\index.html
[create] Public: images\fens.me.png
[create] Public: archives\2014\index.html
[create] Public: archives\2014\05\index.html
[create] Public: css\style.css
[create] Public: index.html
[create] Public: categories\日志\index.html
[create] Public: categories\日志\第一天\index.html
[create] Public: 2014\05\07\abc\index.html
[create] Public: 2014\05\07\hello-world\index.html
[create] Public: tags\开始\index.html
[create] Public: tags\我\index.html
[create] Public: tags\日记\index.html
[info] 35 files generated in 0.711s
```
发布到Github

接下来，我们把这个博客发布到github。<br>
在github中创建一个项目nodejs-hexo，项目地址：https://github.com/bsspirit/nodejs-hexo<br>
编辑全局配置文件：_config.yml，找到deploy的部分，设置github的项目地址。<br>
``` text
deploy:
  type: git
  repo: git@github.com:Dr-Kalen/36dr.git
```
然后，通过命令进行部署。
``` text
D:\workspace\javascript\nodejs-hexo>hexo deploy
[info] Start deploying: github
[info] Setting up GitHub deployment...
Initialized empty Git repository in D:/workspace/javascript/nodejs-hexo/.deploy/.git/
[master (root-commit) 43873d3] First commit
 1 file changed, 0 insertions(+), 0 deletions(-)
 create mode 100644 placeholder
[info] Clearing .deploy folder...
[info] Copying files from public folder...

// 省略部分输出

Branch gh-pages set up to track remote branch gh-pages from github.
To git@github.com:bsspirit/nodejs-hexo.git
 * [new branch]      gh-pages -> gh-pages
[info] Deploy done: github
```
4 模板安装<br>
将Git Shell 切到Hexo目录下，然后执行下面的命令，将pacman主题下载到 themes/pacman 目录下。
``` text
#git clone https://github.com/A-limon/pacman.git themes/pacman
```
修改你的博客根目录Hexo下的config.yml配置文件中的theme属性，将其设置为pacman。
<br>
更新pacman主题
``` text
cd themes/pacman
git pull
```
注意：先备份_config.yml 文件后再升级<br>
模板详细的配置和升级请参照 [2]


## 参考文献
1. [Hexo在github上构建免费的Web应用](http://blog.fens.me/hexo-blog-github/)
2. [Hexo 优化和模板安装](http://www.cnblogs.com/zhcncn/p/4097881.html)
3. [Hexo 官方文档](https://hexo.io/docs/deployment.html)
4. [Hexo 非常简约的设计Maupassant](https://www.haomwei.com/technology/maupassant-hexo.html)
