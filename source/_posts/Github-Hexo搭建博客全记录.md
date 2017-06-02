---
title: Github+Hexo搭建博客全记录
categories: 博客搭建
tags: 
 - Hexo
 - YeLee
date: 2016-08-10 16:53:22
description: "断断续续花了差不多3天时间，终于搭好了这个博客，心情大好，决定记录下本人搭建过程的一点心得，以供同样在摸索的你参考"
---

## 1. Github 配置
### Github 账号注册
*要在Github 上面搭建博客当然需要有个账号了，没有的赶紧注册一个，现在的程序员没有一个Github还怎么愉快装逼（误）。
[传送门](https://github.com/)。怎么注册就不用细说了吧。

### 新建仓库
新建一个仓库，用来放博客相关的配置和文章。名字可以随意，反正以后也能改。博客仓库一般都会起这个名字：
**github账号名字.github.io**。
其他配置也随意，按个人喜好填写，如果实在懒直接下一步。╭(╯^╰)╮
<!-- more -->
### 仓库配置
* 进入仓库，仓库标题下方有7个导航，点击最右侧的**Setting**进入仓库属性设置
* 然后点击GitHub Pages 项下的**Launch automatic page generator**按钮
* 接着如果在新建仓库的时候又生成README.md文件可以选择**Load README.md**
* 没有README.md文件则直接点击**Continue to layouts**进入主题选择，随便选一个就好，反正接下来会用更好的Hexo主题
* 再接着点击**Publish page**搞定

这时候可以在setting页面找到刚刚创建的静态站点的访问地址**Your site is published at xxx**，如果仓库名字是**github账号名字.github.io**，那么访问 **http://github账号名字.github.io** 就可以看到自己创建好的站点了。

## 2.域名绑定
实际上无需购买域名也可以搭建博客，因此如果不打算购买域名可以跳过此步。不过我觉得使用自己的个性域名会更加酷...
### 域名购买
我的域名在[万网](https://wanwang.aliyun.com/)买的，当然你可以选择[dnspod](https://www.dnspod.cn/)。不过个人感觉在万网购买会便宜点。根据个人喜好和经济情况选择域名和购买年限。
* 在**万网**购买域名后，登录到控制台，进入域名选项页面，给你所购买的域名添加解析，因为博客使用的是Github提供的github pages，因此将**github账号名字.github.io**这个网站的IP地址设置为解析IP，可以使用Ping命令查看github账号名字.github.io的IP地址。
* 设置完成后，还需要在你新建的博客仓库下设置**Custom domain**，即把你的Github Pages和你购买的域名绑定，这样访问你所购买的域名就会重定向到博客所在的Github Pages。
如果还有疑问可以参考阿里云的云解析[解析设置入门指南](https://help.aliyun.com/knowledge_detail/29716.html?spm=5176.product29697.3.1.n5T5E9)。

## 3.使用Hexo博客框架
> Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

* Hexo相比于原生的Github Pages，无论是写作和部署的便捷性上还是主题美观程度上都好太多，所以使用Hexo博客框架是现在的潮流。
要使用Hexo，首先需要安装好[Git](https://git-scm.com/download/win)和[NodeJs](http://nodejs.cn/),都是exe安装包，如果没有什么特殊需求一路默认设置下一步就可以了。
* 打开命令行窗口，输入**npm install -g hexo-cli**，耐心等待下载完成，即可安装好Hexo。
* 在你喜欢的目录下，按住Shift+鼠标右键，选择**在此处打开命令行窗口**，然后输入命令**hexo init xxx**,其中xxx名字可以随便起。耐心等待hexo初始化和依赖组件下载完成，等到命令行窗口输出**INFO  Start blogging with Hexo!**，初始化才是真正完成。

![enter description here][1]

* 一般来说，只需关注**网站配置文件_config.yml**，**博客文章存放目录source/_posts**，**网站主题目录theme**。

### 网站配置文件_config.yml
具体config.yml配置请参考[Hexo官方说明文档](https://hexo.io/zh-cn/docs/configuration.html).这里我只说我修改的地方。
*  **title** 博客和网页标签显示的标题，按个人喜好随意
*  **subtitle** 博客副标题标题，一般写简介或者自己喜欢的话
*  **author** 作者名字，随便起
*  **language** 作者名字，随便起
*  **language**除非你英文特牛逼，否则一般都会改成zh-CN
*  **url** 设置为你购买的域名，如果没有购买域名可以设置为gtihub博客仓库地址
* **deploy**  需要进行如下设置：
　　type: git
　　repo:博客仓库的git地址，比如我的是https://github.com/LittleLiByte/littlelibyte.github.io.git。
其他配置属性留到主题设置时再细说。

### 添加博文
* 使用命令 **hexo new xxx**   可以新建一篇博文,其中xxx是博文标题。新建完成后，可以在/source/_post目录下找到新建的文章，**.md**格式的文件，后缀名不言而喻，博文使用markdown语法来写。
看到这里不知道markdown的小白也许一惊--写个博客还要去学一种新语法?然而markdown语法真的很简单很简单，掌握基础够用的语法只要五分钟。不信？请看[Markdown 语法的简要规则](http://www.jianshu.com/p/1e402922ee32/)。了解这几个规则就足够写出条理清晰、排版美观的博文了。
写完博文后，就可以在博客目录下打开命令行窗口，通过**hexo server**启动hexo本地服务，进行文章预览调试，在浏览器输入**http://localhost:4000**就可以预览效果了。

* 在将博文部署之前，如果你的博客购买了域名，那么还需要在博客目录的source文件夹下新建一个**CNAME**文件，只需要添加域名上去即可，例如我的域名是little-byte.com，则**CNAME**文件的内容只要添加little-byte.com即可。

### 将博文部署到Ｇithub
在博客目录下打开命令行窗口，输入命令**hexo deploy**即可部署.
```bash
hexo deploy
```
这时候打开你所绑定的域名或者GitHubPages页面，就可以看到你新写的文章了。

## 4.更换主题
Hexo默认的主题是landscape，个人感觉除了背景图比较酷炫之外，其他地方都很一般。所以爱折腾的我就要折腾出自己喜欢的主题。诸多比较之后，选择了YeLee.

### NexT主题下载
进入到博客目录内，然后通过git将NexT主题文件下载到该文件夹内

``` vim
git clone https://github.com/MOxFIVE/hexo-theme-yelee.git themes/yelee
```
### 使用NexT主题
修改网站配置文件_config.yml，将**theme: landscape**改成**theme: Yelee**，只需这么一句话就可以启用Yelee主题了。
可以通过 `hexo clean && hexo s` 命令来在本地启用调试，查看效果。
一般来说，效果类似下图：

![enter description here][2]

然后就可以开始按照自己的喜好进行配置了。YeLee 详细配置文档见此：[传送门][3]
**YeLee**说明文档已经说明得非常详细清晰了，这里就不再赘述了。
如果搭建过程有疑问，欢迎留言讨论。

  [1]: http://om2bpqram.bkt.clouddn.com/1488416894394.jpg
  [2]: http://om2bpqram.bkt.clouddn.com/1488435176017.jpg
  [3]: http://moxfive.coding.me/yelee/2.Basic-Usage/
