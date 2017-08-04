---
title: Hexo+GitHubPages搭建个人博客
date: 2017-07-12 16:00:04
categories: 博客折腾
tags:
- Hexo
- Github
---

## 前言
首先,欢迎各位来参观我的博客: [Joahcy's Blog][1].搭建博客的目的呢?一方面总结自己学习各种技术过程中踩过的坑以及心得,另一方面希望通过博客这个平台与大家一起交流,共同进步.其实博客搭建完成已有一段时间了,但是之后一直没有时间写这个教程,今天天清气爽,闲来无事正好有时间来完成这个博文.废话不多说,下面就让我们开始搭建属于自己的博客,保证小白白都能看懂,开始吧!
如果你还不知道你为什么要做一个博客，推荐你看：[<<我为什么写博客>>][2]
![此时风景正好][3]

## 关于GitHub

1.Github的优点

- [GitHub官网][4]
- GitHub是基于git实现的代码托管.git可能是目前最好用的版本控制系统了,非常受欢迎.
- GitHub可以免费使用,并且快速稳定.
- Github上面的世界很精彩,用久了你的眼界会开阔很多.

2.什么是Github Pages?

- Github Pages可以被认为是用户编写的,托管在github上的静态网页.

3.为什么要使用Github Pages?

- 可以绑定你的域名(但暂时貌似只能绑定一个).
- 简单快捷,使用Github Pages可以为你提供一个免费的服务器,免去了自己搭建服务器和写数据库的麻烦.

## 关于Hexo
1.Hexo是什么?

- [Hexo官网][5]
- 正如官网所介绍的: A fast, simple & powerful blog framework.Hexo 是一个快速、简洁且高效的博客框架。

2.它有什么优点?

- 速度快: 几十篇博文,只需数秒就可以生成静态网页
- 一键部署: 只需一条命令就可以完成部署
- 支持MarkDown语法
- 有丰富的插件库

想要更多了解,还需各位自行上官网或Google.

## 安装和使用Git

1. 安装
     - Windows
    [官网下载][6](可能有些慢)或[点此下载][7],之后就是小白安装,这是[安装详细教程][8]
     - Linux
	` apt install git 或 yum install git `
2. 怎么打开和使用git呢?
![右击鼠标点击GitBash][9]
![命令行使用git][10]

3. Git基础操作
 ![GIt基础操作][11]

4. Git教程: [ProGit(中文版)][12]  [廖雪峰GIt教程][13]

## 安装nodejs
- Windows
根据自己系统版本在[官网下载][14]安装即可.
- Linux
	apt install nodejs

## 安装淘宝cnpm
- npm 速度会有点慢,所以可以使用淘宝的cnpm代替npm,用法和npm几乎一样.
- 安装:  `npm install -g cnpm --registry=https://registry.npm.taobao.org`
    - Windows 桌面右击鼠标,点击Git Bash Here,进入命令行,执行以上命令
    - Linux 直接在命令行执行以上命令

## Hexo
- 安装
```
cnpm install -g hexo-cli
npm install hexo --save
```
- 创建博客文件夹(比如:E:/blog/joahcy或/home/blog/joahcy)并初始化配置
    - Windows 进入博客文件夹(如我的是joahcy文件夹),右击鼠标,点击Git Bash Here,进入命令行,执行以下操作
    - Linux 进入博客文件夹,执行以下操作
    ```
    hexo init
    npm install
    ```
初始化并且依赖安装完成后,博客目录大概如下:
![初始化后目录][15]

- 安装Hexo插件,保证后面操作不出错,在博客文件夹执行以下操作
```
npm install hexo-generator-index --save
npm install hexo-generator-archive --save
npm install hexo-generator-category --save
npm install hexo-generator-tag --save
npm install hexo-server --save
npm install hexo-deployer-git --save
npm install hexo-deployer-heroku --save
npm install hexo-deployer-rsync --save
npm install hexo-deployer-openshift --save
npm install hexo-renderer-marked@0.2 --save
npm install hexo-renderer-stylus@0.2 --save
npm install hexo-generator-feed@1 --save
npm install hexo-generator-sitemap@1 --save
```

- 生成静态文件
`hexo g`
- 测试,本地查看效果
    - 执行 `hexo s`
	- 浏览器访问: http://localhost:4000 测试是否成功(端口默认4000)
	![博客雏形][16]

## 将博客部署到Github上
- 创建github账户,[Github首页][17]点击右上角Sign up进行注册
- 创建项目代码库  
	注册完成后,点击New repository. 注意名字一定是 yourname.github.io,如图所示:
![创建仓库][18]

- 配置SSH秘钥
配置Github的SSH密钥可以让本地git项目与远程的github建立联系,让我们在本地写了代码之后直接通过git操作就可以实现本地代码库与Github代码库同步.操作如下：
1.查看本机用户目录是否存在SSH秘
2.若不存在则创建新的秘钥,需填写自己的Github注册时的邮箱
`ssh-keygen -t rsa -C "your_email@example.com"`
相关提示:

        Generating public/private rsa key pair.
        Enter file in which to save the key (/c/Users/you/.ssh/id_rsa): [Press enter]
直接回车,直接回车,则将密钥按默认文件进行存储.
然后根据提示,输入密码和确认密码(其实可以不用密码,就是到输密码的地方,都直接回车,所以后面每次push就只需回车就可以).相关提示如下：

        Enter passphrase (empty for no passphrase): [Type a passphrase]
        Enter same passphrase again: [Type passphrase again]

    输入完成之后，屏幕会显示如下信息：

        Your identification has been saved in /c/Users/you/.ssh/id_rsa.
        Your public key has been saved in /c/Users/you/.ssh/id_rsa.pub.
        The key fingerprint is:
        01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
3.在GitHub账户中添加你的公钥,将公钥文件id_rsa.pub的内容复制
4.登录Github,右上角进入settings,然后左侧点击`SSH and GPG keys`,接着点击`New SSH key`,如图:
![New SSH key][19]
5.测试
可以输入下面的命令，看看设置是否成功
`ssh -T git@github.com`
如果出现下面的信息,则表示设置成功:

        Hi joahcy! You've successfully authenticated, but GitHub does not provide shell access.
6.配置用户信息
    ```
     git config --global user.name "joahcy"//用户名
     git config --global user.email  "joahcy915@gmail.com"//填写自己的邮箱
    ```
至此秘钥配置完成,本机已与Github建立了联系.

- 修改博客目录(E:/blog/joahcy)下的_config.yml配置文件 (注意: 配置时,每个键值对之间都要加个空格)
	1.根据个人情况修改Site,Url和Pagination部分
	2.修改Deploy部分
	登录Github打开项目username.github.io
    ![打开项目][20]

    复制项目HTTPS路径
    ![复制项目HTTPS路径][21]

    找到关键字deploy,进行如图修改,repository为刚才复制的路径
    ![修改deploy][22]
- 部署
进入博客目录(E:/blog/joahcy),执行以下命令.

    ```
    hexo g
    hexo d
    ```
或者直接执行`hexo g -d`
执行完之后会让你输入github的账号和密码,输入完后就可以登录我们自己的部署在Github Pages服务器上的博客了.对应的地址是 username.github.io(我的是：joahcy.github.io)
如果觉得输入用户名和密码麻烦,可以修改deploy部分的repository,如图:
![修改为SSH连接][23]
![修改][24]
这样部署的时候就不用输入用户名和密码,是不是方便多了?

## 浏览器访问
通过浏览器访问自己的博客,地址为http://yourname.github.io
![博客雏形][25]
## 根据个人喜好修改博客主题
- 1.在[hexo官网][26]找到自己喜欢的主题,然后克隆下来.  (博客的主题都保存在博客根目录的themes文件夹下)
如克隆Next主题,进入博客目录,执行一下命令:
`git clone https://github.com/iissnan/hexo-theme-next themes/next`
- 2.找到全局_config.yml文件的theme关键字,改为next.(注:值是themes文件夹下主题文件夹的名字)
- 3.详细修改主题样式配置
修改主题目录下的_config.yml文件,在这就不详细解释了,可以找相应的教程来修改(注:简洁大方next主题有很多教程,当然也有其他的主题)

## 通过域名访问博客
- 通过域名商购买自己喜欢的的域名(万网,西部数码,GoDaddy等等),我用的万网.
搜索
![搜索][27]
找到可注册的域名
![找到可注册的域名][28]
如果用的万网,进入个人中心,点击域名
![进入个人中心][29]
点击域名后面的管理
![点击管理][30]
修改DNS 为f1gns1.dnspod.net	和 f1gns2.dnspod.net
![修改DNS][31]
- 在DNSPod官网注册账号,并添加DNS解析
进入域名解析
![进入域名解析][32]
添加域名
![添加域名][33]
获取博客IP地址:  CMD 里面  `ping  yourname.github.io ` 得到IP
点击域名进去,添加两条记录,IP为刚才ping 得的ip
![添加记录][34]
- 在博客source目录下添加CNAME文件,内容为你的域名,然后再一次部署
`hexo g -d`
- 耐心等待......大约十几分钟就可以通过域名来访问了

## 写博客
- 用hexo发表新文章
`hexo n "文章标题"`
会在项目 source/_posts 中生成 对应的.md文件,用编辑器打开编写即可。
当然,也可以直接在source/_posts中新建一个md文件,写完后,推送到服务器上,执行以下命令即可在我们的站点看到新的文章.
`hexo g -d`
- 用Markdown写文章
    - 什么是markdown?
    Markdown 是一种轻量级的「标记语言」,它的优点很多,目前也被越来越多的写作爱好者,撰稿者广泛使用.看到这里请不要被「标记」,「语言」所迷惑,Markdown 的语法十分简单.常用的标记符号也不超过十个,这种相对于更为复杂的HTML 标记语言来说,Markdown可谓是十分轻量的,学习成本也不需要太多,且一旦熟悉这种语法规则,会有一劳永逸的效果.
    - markdown有以下优点:
        - 专注你的文字内容而不是排版样式.
        - 轻松的导出 HTML、PDF 和本身的 .md 文件.
        - 纯文本内容,兼容所有的文本编辑器与字处理软件.
        - 可读,直观.适合所有人的写作语言.
    - 用什么工具写?
        - Windows : MarkdownPad2
        - Mac :用Mou 或 Atom
        - Web端: 用简书
    - 更多资料可以查看: [认识与入门 Markdown][35]

## 后记
至此,博客基本搭建完成.搭建了2天很累,但是还是有一定的成就感的.以后的路还很长,好好写文章提高自己吧.


  [1]: http://www.joahcy.com
  [2]: http://www.cnblogs.com/jhzhu/p/3893297.html
  [3]: http://oqwn6kueb.bkt.clouddn.com/sky-cloud.jpg
  [4]: https://github.com/
  [5]: https://hexo.io/
  [6]: https://git-for-windows.github.io/
  [7]: http://pan.baidu.com/s/1c1FTy5U
  [8]: http://blog.csdn.net/renfufei/article/details/41647875/
  [9]: http://oqwn6kueb.bkt.clouddn.com/git-bash.jpg
  [10]: http://oqwn6kueb.bkt.clouddn.com/git-bash2.jpg
  [11]: http://oqwn6kueb.bkt.clouddn.com/git-cmds.png
  [12]: http://git.oschina.net/progit/
  [13]: http://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000
  [14]: https://nodejs.org/en/download/
  [15]: http://oqwn6kueb.bkt.clouddn.com/blog-dir.jpg
  [16]: http://oqwn6kueb.bkt.clouddn.com/blog-original.jpg
  [17]: https://github.com/
  [18]: http://oqwn6kueb.bkt.clouddn.com/new-repo.jpg
  [19]: http://oqwn6kueb.bkt.clouddn.com/new-sshkey.jpg
  [20]: http://oqwn6kueb.bkt.clouddn.com/setup1.jpg
  [21]: http://oqwn6kueb.bkt.clouddn.com/setup2.jpg
  [22]: http://oqwn6kueb.bkt.clouddn.com/setup4.jpg
  [23]: http://oqwn6kueb.bkt.clouddn.com/setup3.jpg
  [24]: http://oqwn6kueb.bkt.clouddn.com/setup5.jpg
  [25]: http://oqwn6kueb.bkt.clouddn.com/blog-original.jpg
  [26]: https://hexo.io/themes/
  [27]: http://oqwn6kueb.bkt.clouddn.com/domain1.jpg
  [28]: http://oqwn6kueb.bkt.clouddn.com/domain2.jpg
  [29]: http://oqwn6kueb.bkt.clouddn.com/domain3.jpg
  [30]: http://oqwn6kueb.bkt.clouddn.com/domain4.jpg
  [31]: http://oqwn6kueb.bkt.clouddn.com/domain5.jpg
  [32]: http://oqwn6kueb.bkt.clouddn.com/DNS1.jpg
  [33]: http://oqwn6kueb.bkt.clouddn.com/DNS2.jpg
  [34]: http://oqwn6kueb.bkt.clouddn.com/DNS4.jpg
  [35]: https://sspai.com/post/25137
