---
title: Git-fork分支整理
date: 2017-12-04 14:23:19
categories: "git"
tags:
	-- "git"
---

<font size=4>

## 1.Fork    

在github上你要贡献的repo(eg.http://github/remote/test.git)之后称上游仓库。点击fork，将上游仓库fork到你的github，之后称为远程库(eg.http://github/chercher/test.git)

## 2.Clone    

选择本地文件夹，之后称为本地库

git clone http://github/chercher/test.git


## 3.创建dev分支

进入文件夹中，创建dev分支作为你的开发分支，当你完成了这个开发分支的时候直接将这个分支的内容push到你的远程库。一般一个分支对应一个issue，开发完毕后即可销毁

git checkout -b dev 创建并切换至dev分支，是git branch dev + git checkout dev


## 4.创建upstream分支

upstream分支是用于同步上游仓库的，可以同步其他人对上游仓库的更改

git remote add upstream http://github/remote/test.git

这时候用git remote 可以查看远程分支，git remote -v 可以查看具体路径

这时候应该有origin、upstream两种分支且分别有fetch和push的路径，origin是你的远程库，upstream是你的上游仓库

tips. 如果远程分支路径出错了，git remote set-url branch_name new_url 替换为具体的你的出错的分支名和新的路径即可

## 5.同步上游仓库

在提交自己的修改之前，先同步上游仓库到master

git fetch('拿来、取来') upstream（'上游、上行'） 

git remote update upstream

git rebase upstream/master

注意：这个同步仓库是upstream的仓库同步到自己本地的远端仓库，git fetch upstream 之后需要 git pull 一下，将自己本地远端的仓库更新到本地代码

注：公司的项目用的是GitLab,这个和GitHub有所不同。GitLab更新的话，需要 git fetch upstream, 然后 git merge upstream/master(master可指定为某一具体分支)



## 6.修改文件push到远程库

对本地库进行修改后，git add changed_file & git commit -m"message" 添加文件到暂存区然后提交，写入相应信息。

git push origin dev:dev 这时你的远程库将会多出一个dev分支


## 7.提出pull request

这时候在你的远程库中点击create pull request，就可以等待别人review你的代码后merge入上游仓库了



## 8.合并commit

一个issue有时候并不是一次commit就可以完成的，这时候就涉及到洁癖患者们用rebase合并commit的过程了

第一次commit的时候并不需要做rebase的操作，rebase是将之后的多次commit合并到之前的一个commit当中

以第二次修改为例，在commit之后进行  git rebase -i HEAD~2 

tips. 如果出现"Could not execute editor"  则设置 git config即可，git config --global core.editor /usr/bin/vim

之后再执行rebase命令，可以看到这两次的提交，现在状态都是pick。只选择第一个commit为p，其他的都为s，即把新的commit并入之前的。

修改完成后写入，然后自动跳转至另一个页面修改commit的信息

之后继续按照第6步push --force到远程库的流程进行就可以了~

参考：

[Git从fork分支开始的过程整理-chercher](https://www.cnblogs.com/chercher/p/5587979.html)

[github使用-一只小菜鸟](http://www.jianshu.com/p/79454cf00945)

[GitLab Fork项目同步源项目更新-熊皮皮](http://www.jianshu.com/p/3053b25a50e2)