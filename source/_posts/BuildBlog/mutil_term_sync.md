---

title: hexo 多终端同步博客
date: 2017-07-27
comments: false
categories: 博客折腾
tags: Hexo

---

## A电脑:

- 添加 /.deploy_git, /public 到.gitignore文件末尾,一个一行
- 初始化本地仓库  
`git init`
- 将本地仓库与远程仓库对接  
`git remote add origin git@github.com:joahcy/joahcy.github.io.git`
- 添加所有本地文件到git  
`git add . `  
**注意:如果自己的主题是克隆下来的,先把主题下的.git文件夹和.gitignore文件删除,不然上传不上去**
- git提交  
`git commit -m "提交hexo配置文件"`
- 新建分之并切换  
`git branch hexo`  
`git checkout hexo`
- push到Github项目的hexo分支上  
`git push origin hexo`

## B电脑:

- 安装好必备环境:git,nodejs,hexo
- 将hexo分支克隆岛本地:  
`git clone -b git@github.com:joahcy/joahcy.github.io.git joahcyBlog`
- 安装依赖  
`npm install`

## 环境装好就可以开心的多端同步更新博客了

- 先从远程仓库pull  
`git pull git@github.com:joahcy/joahcy.github.io.git hexo`
- 写博客  
`hexo n "test多端同步"`
- 利用Markdown编辑器编辑
- 添加source文件夹到本地git,因为只更改了source文件夹  
`git add source`
- 提交  
`git commit -m "添加多终端同步更新博客"`
- push到远程仓库hexo分支  
`git push origin hexo`
- 部署  
`hexo g -d`
