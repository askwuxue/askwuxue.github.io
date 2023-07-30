---
title: Git命令
date: 2018-12-04 21:16:04
tags: Git
categories: Git
---
_Git是一个版本控制系统，已经成为程序员必备的，必会的的工作软件。还没有入门的小伙伴，推荐大家学习廖雪峰老师的[Git教程](https://www.liaoxuefeng.com/wiki/896043488029600)，该教程非常适合新入门的同学学习，通俗易懂。因为基础的概念已经清楚了，但是对于常用的命令还没有很熟练，做一个常用命令的整理_

### 常用命令
1. 添加远程仓库(本质就是为远程仓库添加一个别名，方便以后操作)
```git
git remote add origin(仓库别名) (git@github.com:askwuxue/Users.git)远程仓库地址
```
2. 查看远程仓库
```git
git remote -v 
```
3. 删除远程仓库
```git
git remote rm origin
```
4. 新建并切换分支
```git
git checkout -b dev(新分支名)
```
5. 切换分支
```git
git branch dev(要切换的分支名)
```
6. 删除分支
```git
git branch -d dev(要删除的分支名)
```
7. 查看所有标签
```git
git tag
```
8. 查看某标签
```git 
git show <tagName>
```
9. 新建标签
```git
git tag <tagName> <commitId>
```
10. 新建标签并添加描述
```gut
 git tag -a <tagName> -m "tag message" <commitId>
```
 11. 推送至远程库
 ```git
 // 将分支内容推送至远程库，远程库就是github仓库地址的别名。-u是记住本次的对应关系，后续可直接使用 git push
 git push -u origin(远程仓库) master(本地分支)
 ```
 12. 撤销暂存区修改
 ```git
 git checkout --filename 
 ```
 13. 版本回退
 ```git
 // 第一步查看要回退到哪一个版本
 git log    // 查看提交历史
 git log --pretty=oneline // 查看提交历史的主要信息，比git log信息简洁
 
// 第二步
git reset --hard commitID 	// commitId 在查看版本历史的时候，会清楚的显示
 ```
### 常见问题
1. 解决冲突
    _冲突产生：一个文件，没有在同一个版本的基础上顺序修改就产生了冲突。如：A,B 拿了同一个版本的文件进行编辑，A编辑好了，并且push了。B随后编辑好了，也要push。但是这个时候A已经进行过编辑，B又编辑，就产生了冲突。本质是没有拿同一个版本的文件进行编辑。所以在正常情况下，最好拿最新版本进行编辑，可以有效避免冲突。如果冲突已经产生，课按照下面的方式进行处理_
    1.1 手动解决冲突，在merge的时候，Git会告诉我们有冲突，如下
    ![conflict](https://github.com/askwuxue/askwuxue.github.io/assets/32808762/df50bfba-476f-46f2-a1e8-cbd4e4bfb0c0)

  Git 告诉我们有冲突，需要我们手动进行合并。合并后如下

  ![resolve_conflict](https://github.com/askwuxue/askwuxue.github.io/assets/32808762/1beb3c51-181e-42e4-bcd2-3cc2c8b3175a)

手动合并结束，接下来，只需要像处理普通文件`git add`，`git commit`进行处理即可，至此，冲突解决完毕。
