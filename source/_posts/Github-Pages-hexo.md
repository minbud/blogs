---
title: Github-Pages&hexo
date: 2018-06-10 18:18:24
tags: 搭建博客
---

使用 *github-pages+ hexo* 搭建个人博客-Mac环境
======================================
<p align="right">--搭建&主题更换&博客备份
--------------------------------------

## 1. 准备工作-github仓库
　　首先到github的官网注册账号（...）。然后，新建仓库，在仓库Setting页面：
　　![img](Github-Pages&hexo/1.png)
　　<!-- more -->
　　在setting页面下修改自己仓库的名字，注意格式要是 *yourusername* .github.io,比如我的就是minbud.github.io, 这样之后才能启用Pages并用 *yourusername* .github.io访问自己的博客，当然也可以绑定自己已有的域名
　　![img](Github-Pages&hexo/2.png)
　  找到Github Pages页面，选择publish，完成后就是这样：
　  ![img](Github-Pages&hexo/3.png)

## 2. 安装hexo
　  由于hexo基于node.js，需要先从[官网](https://nodejs.org/)下载nodejs安装包。
　  
　  安装好后，用下面的命令可以查看版本（npm是nodejs的安装包管理器）
> npm -v

　  然后，使用npm命令安装hexo
> sudo npm install -g hexo-cli 　  　  *-安装(-g表示全局安装)-*
> sudo npm uninstall -g hexo-cli 　  　  *-卸载-*

　  可以用hexo -v命令查看是否安装成功
    
　  新建一个文件夹，并在该文件夹下执行接下来的操作(以我的为例)
> sudo mkdir ~/Documents/hexo
> cd ~/Documents/hexo
> hexo init　  　  -进行hexo的初始化工作，会在该文件夹下生成一系列必要文件-
> npm install
> npm install hexo-deployer-git -save        -安装部署工具-

　  配置hexo，修改更目录下的_config.yml文件
> \#Site
> title: MinBud的博客
> subtitle: Walk Slow & Think Deep
> description:
> keywords:
> author: Minbud
> language:
> timezone:

　  最主要的是下面的配置项，在最末尾添加如下配置，可以设置部署时的push地址
> deploy:
>type: git
>repo: http://github.com/minbud/minbud.github.io.git
>branch: master

　  博客文件使用markdown格式，放在./source/_post文件夹中，可以在终端新建博客文件
> hexo new "myFirstBlog"

　  编写完博客后，使用以下命令可以生成静态网页并部署到github中

> hexo clean
> hexo generate　  　  -生成博客网页的各种静态文件-
> hexo server　  　  -在本地启动服务器，在浏览器访问http://localhost:4000/ 预览-
> hexo deploy 　  -将博客网页推送到服务器-

## 3. 更换主题
　  经过上面步骤后可以得到一个拥有默认主题的hexo博客，另外还有许多第三方制作的精美的主题可以供我们使用，我用的是Litten的yilia
　  
> cd ~/Documents/hexo
> git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia

　  会在./themes目录下拷贝yilia主题，需要在_config.yml中修改主题名称从而应用主题
　  另外需要对主题进行一定的修改，实现一定程度的定制，可以修改yilia文件夹下的_congfig.yml文件
> favicon: img/favicon.png　  　  -修改头像-
> avatar: img/minbud.png　  　  -修改网页标签栏图标-

　  由于生成的网页文件在hexo文件夹根目录下，可以将图片放在hexo/img/路径下，并如上设置
　  另外主题右下角有作者Litten的标签，有人可能不喜欢（比如我）,可以修改脚本文件将其去掉。index.html是生成的静态文件，直接修改只能生效一次或以后每次都得改。可以修改hexo/themes/yilia/layout/_partial/footer.ejs文件，将footer-right相关项删除就好了

　  在首页不显示博客的全部内容，需要在文章中需要截断的地方加入
> <\!\-\- more \-\-\>

　  
## 4. 备份博客
　  在本地文件夹中存有你的各种博客文件（.md），但是部署到github上的时候实际上只是将生成的静态博客网页和各种使用到的资源文件push上去了，万一本地文件丢失了，靠github上的文件难以恢复原来的博客文件。
　  可以在原仓库中创建分支，作为备份分支来解决这个问题
　  
　  在github网页上创建新分支，可以命名为hexo，在setting选项卡上找到branch，将默认分支设为hexo
　  新建文件夹，将博客仓库clone到里面，并将之前文件夹中的_config.yml，themes/，source，scffolds/，package.json，复制进来
　  将themes/yilia/.git/ 文件夹删除，否则无法push该文件夹
　  执行
> npm install
> npm install hexo-deployer-git -save
> git checkout hexo
> git remote add origin https://github.com/minbud/minbud.github.io.git
> git add .
> git commit -m "备份博客"
> git push origin hexo

　  成功后，在github网页上切换到hexo分支，可以看到文件夹下的文件都被传上来了，以后需要备份的时候使用git命令就可以了
　  
## 5. 备份恢复
　　备份的作用就是要用作将来的恢复，这里说一下在新的PC中利用hexo分支恢复备份的过程。
　　新建一个文件夹（假设新PC上已经安装上面的步骤装了git和hexo），在该文件目录下执行以下命令：
```
git init
git config –global user.mail yourmail —-如果没配置过，则是必须的步骤，不然会报错,/home/.gitconfig中可以看到相关配置
git config –global user.name “yourname”
git remote add origin https://github.com/minbud/minbud.github.io.git
git fetch origin hexo　　　　　　—-拉取远程分支hexo到本地
git checkout hexo
git pull origin hexo　　　　　—-拉取远程分支并和本地分支merge
```
　　通过上面的步骤就已经将hexo分支的文件拉去到本地了，可以用ls查看，但是此时如果发现目录下有.deploy_git目录，则还不能直接在该文件目录下进行工作，继续以下步骤：
```
rm -rf .deploy_git/
npm install
npm install hexo-deployer-git –save
```
　　至此，就可以在恢复的文件上执行和以前一样的操作了orz。 　　


## 6. 在博客中添加图片

　  编辑_config.yml文件，确认 *post_asset_folder:true*
　  在根目录下安装
> npm install https://github.com/CodeFalling/hexo-asset-image --save

　  之后使用hexo new命令时，source/_post/文件夹中会增加一个与博客同名的文件夹
　  将使用到的图片放在这个文件夹中，并在引用图片时，按如下：
　  
> \!\[img\](blogname/1.png)

　  图片正常显示  
　  
　  可能遇到需要设置图片大小和居中的情况，此时需要使用html代码：  
　  &lt; div align=center &gt; &lt; img src="./p/p.png" /&gt; &lt;/div &gt;
　  
## 7. 补充
　　国内使用git速度比较慢，有时只有十几kb，博客内容多了之后，发布或拉取工程的速度就难以忍受了，修改hosts文件可以在一定程度上缓解问题，可以达到200~300kb。

```
151.101.72.249 http://global-ssl.fastly.net
192.30.253.112 http://github.com
```
　　修改/etc/hosts文件，加入上面的内容即可。
　　
　　向仓库中的master分支添加README.md文件的方法：
```
1. 在 /source/ 目录下添加README.md文件
2. 在_config.yml文件添加: skip_render:README.md
```
