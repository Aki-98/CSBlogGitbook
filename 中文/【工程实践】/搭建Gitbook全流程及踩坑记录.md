**笔记仓库转Gitbook的好处：**

- 全局搜索文本
- 树状图结构浏览笔记



# 第一步 准备md笔记仓库

下面所有操作已经写成了自动化cli工具，可以直接在cmd下运行

> 代码位置：https://github.com/Aki-98/Pre-Gitbook-Cli



**1.将所有文件处理成md，或引用到md文件中**

Gitbook是只能够渲染markdown页面的。

那有些PDF文件，txt文件，或者源码，也希望在Gitbook网页中可以看到，怎么做呢？

对于txt文件，可以直接转换成md格式。

对于PDF文件和源码，可以以引用方式写在md文件中。



**2.规整图片**
在md文件中引用网络图片，在生成的Gitbook网页中也是能够看到的，但网络图片可能失效，如果担心这点的话，可以下载保存到仓库中。

在各个电脑上操作同一个笔记仓库，可能导致图片引用的绝对路径不一致，导致Gitbook也渲染不到图片的问题，如果有这种情况，需要重写绝对路径或改成相对路径。

如果笔记是拍摄的实体书，图片会过大，如果移动仓库内图片的位置，.git文件也会增加不必要的体积，为此可以压缩图片，对于书籍来说不会丢失多少有效数据。



**3.准备SUMMARY.md**

SUMMARY.md是Gitbook右侧的树状导航。

gitbook init 可以生成SUMMARY.md，但有各种bug，且没办法根据所有md文件生成summary



**4.准备book.json**

book.json用于配置Gitbook网页的标题、版权、插件（全局搜索器）等

下面是我的配置，可以参考下

https://github.com/Aki-98/CSBlogGitbook/blob/main/book.json

配置详解：

```
{
    "root": "/",
    "gitbook": "3.2.3",
    "title": "CS Blog", // 网页标题
    "page": {
        "title": "CS Blog" // 网页标题
    },
    "author": "aki", // 作者
    "variables": {
        "authorName": "aki" // 作者
    },
    "plugins": [
        "accordion", // 一个插件，用于折叠内容或创建可展开的段落（手风琴效果）
        "search-pro", // 一个改进的全局搜索插件，提供更强大的搜索功能。
        "-search", // 表示禁用默认的搜索插件（GitBook 自带的 search 插件）
        "tbfed-pagefooter" // 一个用于页面底部添加版权声明和修改时间的插件
    ],
    "pluginsConfig": {
        "tbfed-pagefooter": { // 每个文件底部加上
            "copyright": "Copyright &copy Aki 2024", // 版权声明
            "modify_label": "modified time:", 
            "modift_format": "YYYY-MM-DD HH:mm:ss" // 修改时间格式
        }
    }
}
```



# 第二步 准备本地Gitbook搭建环境

*也可部署Gitbook Cicd跳过此步



直接看的网络上的教程，踩了一堆坑。

> 这个教程太简单了，可以先看一遍了解一下大概，但不要按里面的流程走
>
> https://tonydeng.github.io/gitbook-zh/
>
> 这个更详细更好
>
> https://1927344728.github.io/fed-knowledge/tools/Gitbook/



**0.踩坑前置**

刚开始直接去官网下载安装的

> 官网：https://nodejs.org/en

但是官网这个版本(v18.x)太新了，gitbook有各种适配问题，后来降级成(v.10.x才正常使用gitbook)

讲一下踩坑过程吧，使用v18.x版本的nodejs，安装gitbook-cli后，在命令行调用gitbook init，报错：

```shell
/home/travis/.nvm/versions/node/v12.18.3/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287
      if (cb) cb.apply(this, arguments)
                 ^
TypeError: cb.apply is not a function
    at /home/travis/.nvm/versions/node/v12.18.3/lib/node_modules/gitbook-cli/node_modules/npm/node_modules/graceful-fs/polyfills.js:287:18
    at FSReqCallback.oncomplete (fs.js:169:5)
```

> 解决方法：https://stackoverflow.com/questions/64211386/gitbook-cli-install-error-typeerror-cb-apply-is-not-a-function-inside-graceful

需要更新下graceful-fs版本

我使用npm更新的时候还踩了个坑，调用任何命令都是报错 ERR! Cannot read property 'insert' of undefined

> 解决方法：https://github.com/npm/cli/issues/4876

就是需要修改镜像到https://registry.npmmirror.com

如果还需要设置proxy的话，这里有教程

> npm设置代理：https://blog.csdn.net/yanzi1225627/article/details/80247758

更新完之后，gitbook 调用任何命令没有反应，没有报错没有任何信息，本地目录没有任何变化，之后就将nodejs降级成了10.x

期间还有一个坑，gitbook和gitbook-cli是不能共存的，使用gitbook-cli(gitbook的命令行工具)得把gitbook卸载掉，安装完gitbook-cli第一次调用gitbook命令时，gitbook -V这种看版本号的也算，gitbook-cli就会去安装对应版本的gitbook

因为我每次调用gitbook命令，都会卡在install gitbook这一步，查了下网上的方法，可以克隆gitbook的github仓库然后自己install自己link

```shell
git clone https://github.com/GitbookIO/gitbook.git
cd gitbook
npm install
npm link
```

首先gitbook仓库里面的说明是使用bun安装而不是npm，其次这样安装还是相当于npm install gitbook，gitbook和gitbook-cli是冲突的，所以这个方法是错的。

下面说正确的方法



**1.安装nvm(nodejs的版本管理工具)和10.x版本的nodejs**

根据自己系统的情况二者选一

```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

bash
Copy code
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash
```

安装nodejs 10.x版本并使用

```shell
nvm install 10
nvm use 10
```

这里还踩了个坑，上面的步骤中会安装nodejs 10.x版本适配的npm，但是我之前使用的18.x版本，切换使用nodejs 10.x版本后，npm的版本没有切换，调用npm任何命令都会报错，后来且切换回使用18版本，再卸载npm，这里的卸载操作会卸载最新版本npm，切换使用10.x版本后，就能正常使用到较低版本的npm了

```shell
nvm use 18
npm uninstall npm -g
```

-g的意思是将操作实施到全局，这个命令之后也会用到



**2.安装使用gitbook-cli&gitbook**

```shell
npm install gitbook-cli -g
```

`npm install -g` 命令用于全局安装 Node.js 模块。这意味着安装的模块将被放置在一个特定的目录中，通常是你的系统的全局 `node_modules` 目录下，而不是当前项目的 `node_modules` 目录下。

如果在 `npm install` 命令中不使用 `-g`（全局）参数，那么该命令将会在当前项目目录下的 `node_modules` 文件夹中安装指定的包，而不是在全局环境中安装。

```shell
gitbook -V
```

查看gitbook版本，第一次调用任何gitbook命令会去安装gitbook

再次运行gitbook -V可以看到gitbook-cli和gitbook的版本

注意不要自己npm install gitbook，要通过gitbook-cli



**3.预览gitbook网站，进行修复**

将命令行切换到自己的自己的md源文件根目录下，运行下面的命令自动生成README.md、SUMMARY.md

建议跑脚本，gitbook init生成文件有各种bug，且没办法根据所有md文件生成summary

```shell
gitbook init
```

注意目录下md文件和子文件不要包含“#”这个字符，也就是说生成的SUMMARY.md引用中不要有“#”字符，如果有的话会导致生成的gitbook页面点击左部的相应导航条目无法跳转

另外md文件中任何位置不要有连续的大括号，比如{{ 和 }}

> 相关说明：https://wkcom.gitee.io/gitbook%E7%9A%84%E4%BD%BF%E7%94%A8/3%E3%80%81gitbook%E8%BF%9E%E7%BB%AD%E5%A4%A7%E6%8B%AC%E5%8F%B7%E7%9A%84%E8%A7%A3%E5%86%B3%E6%96%B9%E5%BC%8F.html

大括号中间加个空格就能解决

预览gitbook网站

```shell
gitbook server
```

在浏览器打开本地端口page查看

```shell
http://localhost:4000/
```



# 第三步 搭建.io在线网站

*也可部署Gitbook Cicd跳过此步



有很多解决方案，我选择直接在Github上传静态网站。



生成gitbook静态网站，在自己的md源文件根目录下运行：

```shell
gitbook build
```

会生成个_book目录，里面放置的就是静态网站代码



只要仓库里有gh-pages分支，就可以根据下面的连接访问Gitbook Pages

```shell
https://githubusername.github.io/reponame
```

所以步骤就是：

1. 创建个空的github仓库
2. 切出个gh-pages分支，上传静态网站
3. 我需要多个gitbook，所以我在根目录下创建个子文件夹，在这个子文件夹下上传对应的静态网站

那我就可以根据下面路径访问多个Gitbook

```shell
https://githubusername.github.io/reponame/subfoldername1
https://githubusername.github.io/reponame/subfoldername2
```



# 第四步 Gitbook CICD

可以直接复制下面的文件到自己的笔记仓库

https://github.com/Aki-98/CSBlogGitbook/blob/main/.github/workflows/deploy.yml

如果你的笔记仓库分支不是main, 上面yml中第一部分触发条件需要重写

```
on:  push:    branches: {your_branch}
```

注意放置路径也需要是/.github/workflows/deploy.yml

在个人账户的Settings>Developer Settings>Personal access tokens>Fine-grained tokens>Generate New Token

权限我是直接放到最大的，不清楚最低权限怎么配置，各位需要可以自己调查下

点击自己笔记仓库的Settings > Secrets and variables > Actions 创建PERSONAL_TOKEN, 将上面生成的token复制过来

之后每次push，都会自动生成Gitbook网页并发布到笔记仓库的gh-pages分支

通过下面的链接格式就可以访问到网页了

```
https://{your_github_name}.github.io/{repo_name}
```



下面是deploy.yml的详解，谁能想到写个这么简单的yml也踩了一堆坑...

```
# CICD 任务的名称
name: Deploy GitBook

# 定义了触发条件：当用户push到本仓库的main分支时
on:
  push:
    branches:
      - main

# CICD任务
jobs:
  deploy:
    # 运行CICD任务的平台：最新版Ubuntu
    runs-on: ubuntu-latest
    steps:
      # 检出笔记仓库分支
      - name: Checkout code
        uses: actions/checkout@v3
      # 准备Node.js 12的环境, 为了兼容Gitbook选择了一个比较低的版本
      - name: Setup Node.js 12
        uses: actions/setup-node@v3
        with:
          node-version: '12'
	  # 先行安装graceful-fs, 不然会报错, 详见第二步
      - name: Install Latest graceful-fs
        run: npm install graceful-fs@4.2.0 -g
      # 安装Gitbook Cli工具
      - name: Install GitBook CLI
        run: npm install gitbook-cli@2.1.2 -g
      # 利用Gitbook Cli工具安装gitbook
      - name: Install GitBook
        run: gitbook install
      # 根据笔记渲染生成Gitbook网页
      - name: Build GitBook
        run: gitbook build
      # 利用peaceiris/action-gh-pages发布Gitbook网页到当前仓库的ph-pages分支中
      # action-gh-pages更多使用方式可见: https://github.com/peaceiris/actions-gh-pages
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.PERSONAL_TOKEN }} # 使用本仓库的PERSONAL_TOKEN
          publish_dir: ./_book # 使用_book目录下的文件
          publish_branch: gh-pages # 发布到gh-pages分支
```

