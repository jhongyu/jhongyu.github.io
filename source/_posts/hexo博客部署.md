---
title: hexo+next+github一把梭
tags:
  - Hexo
  - Next
  - Github
categories: 博客
date: 2020-04-30 17:48:50
---


整了两天终于用Github actions完成了博客在Github上的自动部署，故写作此文记录一下操作。
<!--more-->

## 前提条件

安装hexo的前提是你的电脑中已安装过[Node.js](nodejs.org)和[Git](git-scm.com)，若你安装过上述软件，即可跳过此步骤。

1. 安装Git
    - Windows系统：直接下载并安装[git](https://git-scm.com/download/win)，具体细节可在其[官网](https://git-scm.com/)查询
    - Linux(Ubuntu,Debian)：使用`sudo apt-get install git-core`命令安装
2. 安装Node.js
    可直接进其[官网](nodejs.org)查看各平台安装方法，在此不多做介绍。

## 安装Hexo

安装完上述软件后，即可使用npm安装Hexo。

{% code lang:bash line_number:false %}
    $ npm install -g hexo-cli
{% endcode %}

**PS：国内由于众所周知的原因，使用npm的官方镜像较慢，可以使用淘宝的npm镜像替代**
使用下列命令安装淘宝的cnpm替代npm后即可使用cnpm命令安装模块：
{% code lang:bash line_number:false %}
    $ npm install -g cnpm --registry=https://registry.npm.taobao.org
{% endcode %}


具体信息可查看[淘宝官方网站](https://developer.aliyun.com/mirror/NPM?from=tnpm)

安装以后，可以使用以下两种方式执行Hexo：

1. `npx hexo <command>`
2. 将Hexo所在的目录下的`node_modules`的路径添加到环境变量之中即可直接使用`hexo <command>`(鉴于此种方法会污染环境变量，故不推荐使用，可在Hexo所在目录下的`package.json`文件中添加脚本执行hexo命令)：
    `echo 'PATH="$PATH:./node_modules/.bin"' >> ~/.profile`

## 使用Hexo建站

安装Hexo完成后，可在选定的文件夹下执行以下命令：

{% code lang:bash line_number:false %}
    $ hexo init <folder>
    $ cd <folder>
    $ npm install
{% endcode %}

完成后新建的文件夹的结构如下：

{% code line_number:false %}
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
{% endcode %}

上述目录中themes所在文件夹即是存放主题文件的地方，此时该文件夹中应该包含了Hexo的初始主题，由于本文主要介绍Next主题的安装及配置，故不对Hexo做过多介绍，若想了解具体信息，可查看[官方文档](https://hexo.io/zh-cn/docs/)

## 安装Next主题

Next主题是Hexo生态中一款使用人数较多的主题，其具有完善的文档及社区，且在GitHub上开源，具体信息可查看其[文档](https://theme-next.org/docs/getting-started/)或[Github仓库](https://github.com/theme-next/hexo-theme-next)

下面介绍Next的安装方式：

{% tabs Tab one %}
<!-- tab 最新版本@text-width -->
利用Git直接clone其仓库：

{% code lang:bash line_number:false %}
    $ cd hexo
    $ git clone https://github.com/theme-next/hexo-theme-next  themes/next
{% endcode %}

<!-- endtab -->

<!-- tab 稳定版本@text-width -->
在Next仓库的[Release Page](https://github.com/theme-next/hexo-theme-next/releases)界面选择最新的stable版本下载，下载完成后解压放在上面生成的themes文件下。
<!-- endtab -->
{% endtabs %}

完成上述操作后，接下来需要在Hexo的配置文件(即`_config.yml`文件)中将主题设置为next

{% code lang:yml hexo/_config.yml line_number:false %}
    theme: next
{% endcode %}

设置完之后可以运行`hexo s --debug`命令启动hexo，访问默认地址`http://localhost:4000/`查看是否正常工作。此时展示的是Next的默认配置，若想更改配置，可根据其[文档](https://theme-next.org/docs/getting-started/)进行配置。

## 部署

完成Hexo和Next的安装之后，接下来就是将其部署到Github上。该步骤有两种方式实现，第一种为一键式部署，第二种为利用CI(持续集成)服务部署，接下来分别介绍这两种方式。
**由于两种方式都是将网站部署到Github Pages上，故需要提前注册Github账户并且新建一个名称为`用户名.github.io`的仓库。**

{% tabs Tab two %}
<!-- tab 一键式部署@text-width -->
1. 安装hexo-deployer-git：
{% code lang:bash line_number:false %}
$ npm install hexo-deployer-git --save
{% endcode %}
2. 在`_config.yml`文件中修改以下配置：
{% code lang:yml hexo/_config.yml line_number:false %}
deploy:
    type: git
    repo: //仓库地址
    branch: //分支
    message: //提交信息(可选)
{% endcode %}
3. 执行`hexo clean && hexo deploy`命令生成站点文件并推送至远程库
4. 登入GIthub，在推送的仓库中将默认分支设置为`_config.yml`文件中配置的分支名称。之后应该就可以在Github Pages的页面看到所生成的页面了
<!-- endtab -->

<!-- tab 持续集成方式@text-width -->
*Hexo和Next的官方文档中使用的CI工具均为Travis CI，但我尝试后均未成功，故选择了Github提供的Github Actions服务*
1. 使用`git init`命令将hexo所在目录初始化为一个本地仓库。
>👉注意：该步骤需要注意的一点是将网站部署到Github Page上时网站文件需要位于master分支上，而此时本地目录中的文件不是最终生成的网站文件，故在用git初始化本地仓库后需要使用`git checkout -b 分支名`命令新建并切换至新分支(如新建source分支，则命令为`git checkout -b source`)。
2. 将目录内未被忽略的文件提交至仓库
>👉注意：如果安装的Next主题为最新版本(即使用`git clone`命令克隆的仓库)，此时建议将Next主题所在文件夹加入`.gitignore`文件，然后使用`git submodule add`命令将其作为该仓库的子模块。
3. 在Github上新建一个名称为`用户名.github.io`的仓库，并按照其指示将本地代码推送至远程仓库。
4. 点击仓库中的Actions按钮新建一个Github Actions，此时Github会自动建立一个YAML格式的配置文件，在其中输入以下内容：
{% code lang:yaml line_number:false %}
name: Deploy

on:
    push:
     branches: [source]

jobs:
  hexo-deploy:
    runs-on: ubuntu-latest
    name: deploy blog.
    env:
      TZ: Asia/Shanghai
    steps:
      - name: Checkout
        uses: actions/checkout@v2
        with:
          submodules: true # Checkout private submodules(themes or something else).

      - name: Use Node.js
        uses: actions/setup-node@v1
        with:
          node-version: '12.x'

      # - name: Get yarn cache directory path
      #   id: yarn-cache-dir-path
      #   run: echo "::set-output name=dir::$(yarn cache dir)"

      # Caching dependencies to speed up workflows. (GitHub will remove any cache entries that have not been accessed in over 7 days.)
      # - name: Yarn cache
      #   uses: actions/cache@v1
      #   id: yarn-cache # use this to check for `cache-hit` (`steps.yarn-cache.outputs.cache-hit != 'true'`)
      #   with:
      #       path: ${{ steps.yarn-cache-dir-path.outputs.dir }}
      #       key: ${{ runner.os }}-yarn-${{ hashFiles('**/yarn.lock') }}
      #       restore-keys: |
      #           ${{ runner.os }}-yarn-

      # - name: Install Dependencies
      #   if: steps.cache.outputs.cache-hit != 'true'
      #   run: npm install

      - name: Install dependencies & Generate static files
        run: |
          node -v
          npm install -g hexo-cli
          npm run build

      # Deploy hexo blog website.
      - name: Deploy
        env:
          GIT_NAME: jhongyu
          GIT_EMAIL: ${{secrets.GIT_EMAIL}}
          REPO: github.com/jhongyu/jhongyu.github.io.git
          GH_TOKEN: ${{secrets.GH_TOKEN}}
        run: |
          cd ./public && git init && git add .
          git config --global user.name $GIT_NAME
          git config --global user.email $GIT_EMAIL
          git commit -m "Site deployed by GitHub Actions"
          git push --force --quiet "https://$GH_TOKEN@$REPO" master:master

      # Use the output from the `deploy` step(use for test action)
      - name: Get the output
        run: |
          echo "${{ steps.deploy.outputs.notify }}"

{% endcode %}
5. 之后在本地写完博客推送之后Github Actions便会自动运行将生成的网站文件提交到master分支上。
<!-- endtab -->
{% endtabs %}
