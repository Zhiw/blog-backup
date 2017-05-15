---
title: 利用 Hexo 搭建个人博客
date: 2016-02-25 15:50:36
tags: Hexo
categories: Hexo
---
# 前言
本文主要介绍利用 Hexo 和 GitHub Pages 搭建个人博客，在此之前，默认您已经拥有 GitHub 账号以及了解 Hexo 和 GitHub Pages 的相关知识，若有疑问，可前往 [Hexo](https://hexo.io/zh-cn/) 和 [GitHub Pages](https://pages.github.com/) 进行了解。
# 安装 Node.js
 ## 安装 NVM(Node Version Manager)
 使用 [brew](http://brew.sh) 安装 NVM
``` bash
brew install nvm
```
將下列指令加入 `.bash_profile`
``` bash
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
```

重新载入 `.bash_profile` 设定
``` bash
source .bash_profile
```
## 使用 NVM 安装 Node.js
列出所有可安装的版本
``` bash
nvm ls -remote
```
安装对应版本的 Node
``` bash
nvm install "your version"
```
使用对应版本的 Node.js
``` bash
nvm use "your version"
```
预设使用你下载的版本
``` bash
nvm alias default "your version"
```

测试 Node.js和 npm
``` bash
node -v
npm -v
```
# 安装 Hexo
``` bash
npm install -g hexo-cli
```
部署 Hexo
hexo init [文件名]
``` bash
hexo init hexo
cd hexo
npm install
```
部署成功之后，Hexo 会自动在目标文件夹建立博客网站所需要的所有文件。此时可以通过输入以下命令在本地进行预览
``` bash
hexo generate
hexo server
```
此时，打开浏览器输入 <http://localhost:4000>，就可以看到博客的效果，按`Ctrl+C`可停止服务


## Hexo 的使用
``` plain
.
├── _config.yml    
├── package.json
├── scaffolds
├── public
├── source
├   ├── _drafts
├   └── _posts
└── themes
```
### 文件结构
** _config.yml **  
网站的配置信息，您可以在此配置大部分的参数。

** package.json **  
应用程序的信息。EJS, Stylus 和 Markdown renderer 已默认安装，您可以自由移除。

** scaffolds **  
模版文件夹。当您新建文章时，Hexo 会根据 scaffold 来建立文件。

** public **  
利用`hexo g`命令生成的静态文件，也就是最终部署到网站上的文件夹，可以根据需要进行修改，** 比如在 GitHub 上生成 README 文件，可以将文件命名为 `README.mdown` ，然后将文件移动到该文件夹下面，再进行部署。 **


** source **  
资源文件夹是存放用户资源的地方。除 `_posts` 文件夹之外，开头命名为 `_` (下划线)的文件 / 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 `public` 文件夹，而其他文件会被拷贝过去。

** themes **  
主题文件夹。Hexo 会根据主题来生成静态页面。

### Hexo 命令介绍
常用命令(括号内为简写)：
``` bash
hexo clean        清除文件缓存
hexo deploy(d)    部署你的网站
hexo generate(g)  生成静态文件
hexo init         新建一个hexo目录文件夹
new(n) "post name"      新建文章
new page "page name"    新建页面
server(s)        启动服务器
```
更多命令请输入`hexo help`查看
### 新建博文
我们可以使用命令新建博文，输入以下命令：
`hexo new "题目"`
这样在我们工程的`/source/_posts/`下面会新建一个“题目.md”的文件，这就是我们新建的博文
或者也可以手动在`/source/_posts/`下新建一个“题目.md”的文件。
``` bash
title: Hello World
date: 2016-02-25 19:56:29 
categories: hexo 
tags: [hexo,github] #多于一项时用这种格式
---
正文

```

### 新建页面
新建的博文默认是在文章页面，如果我们需要新建如分类，标签和关于这样的页面，需要执行以下命令：
`hexo new page "页面名称"`
执行后`source/`新建了一个以刚才页面名称命名文件夹，并多了一个”index.md“的文件，这便是我们新建的页面
### Hexo 部署
部署前要在网站配置文件下设置：
```
deploy:
  type: git
  repo: 你的github pages的地址.git
  branch: master
```
然后执行以下命令：
``` bash
hexo clean
hexo generate
hexo deploy
```

### 安装 Nex.T 主题
我的博客使用了Nex.T的主题，详情可到 [Nex.T](https://github.com/iissnan/hexo-theme-next) 进行查看，有详细的使用说明，你也可以使用其他你喜欢的主题。

折腾了好长时间终于把文章写完，也成功部署上去了，很久时间不写文章，有点生疏，在此记录下这个过程来也算是给博客开了一个头，同时也给后面有需要的一个参考，少走弯路，相互学习。

# 参考链接

[Hexo](https://hexo.io/zh-cn/)
[GitHub Pages](https://pages.github.com/)
[Node.js | Mac OSX 新手入门](http://mac-osx-for-newbie-book.kejyun.com/software/softwareWebDeveloperNodeJS.html)
[手把手教你建 github 技术博客](http://wuxiaolong.me/2015/07/31/build-blog-by-hexo/)