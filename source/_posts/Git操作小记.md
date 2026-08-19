title: Git操作小记
date: 2015-12-27 20:56:18
tags: Blog
---

本博客采用创作共用版权协议, 要求署名、非商业用途和保持一致. 转载本博客文章必须也遵循[署名-非商业用途-保持一致](http://creativecommons.org/licenses/by-nc-sa/3.0/deed.zh)的创作共用协议.

## Git操作清单


> 持续更新...

### 提交和删除

```
# 使用一次新的commit，替代上一次提交
# 如果代码没有任何新变化，则用来改写上一次commit的提交信息
$ git commit --amend -m [message]

# 重做上一次commit，并包括指定文件的新变化
$ git commit --amend   ...

# 删除暂存文件, 不删除本地
$ git rm --cached filename

# 本地的dev分支推送的远程dev分支(远程没有则创建)
$ git push origin dev:dev 

# 从远程仓库抓取所有本地没有的内容
$ git fetch origin

# 推送, 本地的修改推送到远程remote中的branch分支上
$ git push (remote) (branch)

# 选择一个commit，合并进当前分支
$ git cherry-pick [commit]
```

<!--more-->

### 分支

```
# 创建并切换分支
$ git checkout -b dev

# 合并分支
$ git merge dev

# 删除分支
$ git branch -d dev

# 删除远程分支
$ git push origin --delete <branchName>

# 查看各分支及追踪的远程分支
$ git branch -vv

# 强行删除未合并的分支
$ git branch -D feature-vulcan

# 查看详细的远程仓库信息
$ git remote show (remote-name)

# 创建并切换分支, 并与远程分支关联, 作用是git pull的时候会从远程remotename/branch分支拉取数据
$ git checkout -b [branch] [remotename]/[branch]

# 改变当前本地仓库追踪的远程分支
$ git branch -u origin/serverfix

# 删除远程名为serverfix分支
$ git push origin --delete serverfix

# 新建一个分支，与指定的远程分支建立追踪关系
$ git branch --track [branch] [remote-branch]

# 建立追踪关系，在现有分支与指定的远程分支之间
$ git branch --set-upstream [branch] [remote-branch]

# 重命令远程分支(devel 分支重命名为 develop 分支)
# 1. 删除远程分支
$ git push --delete origin devel
# 2. 重命名本地峰值
$ git branch -m devel develop
# 3. 推送本地分支
$ git push origin develop

```

- [git 分支](http://www.zhihu.com/question/21995370)
- [ Git 分支 - 远程分支](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E8%BF%9C%E7%A8%8B%E5%88%86%E6%94%AF)

### 查看信息

```
# 查看分支合并状况
$ git log --graph --pretty=oneline --abbrev-commit

# 查看提交记录, 并能看到文件的修改
$ git log --stat

# 显示指定文件相关的每一次diff
$ git log -p [file]

# 显示指定文件是什么人在什么时间修改过
$ git blame [file]

# 显示暂存区和工作区的差异
$ git diff

# 显示暂存区和上一个commit的差异
$ git diff --cached []

# 显示工作区与当前分支最新commit之间的差异
$ git diff HEAD

# 显示两次提交之间的差异
$ git diff [first-branch]...[second-branch]
```

### 标签

```
# 新建一个tag在当前commit
$ git tag [tag]

# 新建一个tag在指定commit
$ git tag [tag] [commit]

# 提交指定tag
$ git push [remote] [tag]

# 提交所有tag
$ git push [remote] --tags

# 新建一个分支，指向某个tag
$ git checkout -b [branch] [tag]

# 删除远程标签
$ git push origin --delete tag <tagname>

# 获取远程的tag
$ git fetch origin tag <tagname>
```


### 撤销

```
# 版本回退 只需要添加commit版本号的前几位
git reset --hard 3628164

# 恢复暂存区的指定文件到工作区
$ git checkout [file]

# 恢复某个commit的指定文件到工作区
$ git checkout [commit] [file]

# 恢复上一个commit的所有文件到工作区
$ git checkout .

# 对某个文件进行版本回退
$ git reset commit号 filename 

# 重置暂存区的指定文件，与上一次commit保持一致，但工作区不变
$ git reset [file]

# 重置暂存区与工作区，与上一次commit保持一致
$ git reset --hard

# 重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变
$ git reset [commit]

# 重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致
$ git reset --hard [commit]

# 重置当前HEAD为指定commit，但保持暂存区和工作区不变
$ git reset --keep [commit]

# 新建一个commit，用来撤销指定commit
# 后者的所有变化都将被前者抵消，并且应用到当前分支
$ git revert [commit]
```

- [git如何删除远程仓库的某次错误提交](http://zhiwei.li/text/2015/08/16/git%E5%A6%82%E4%BD%95%E5%88%A0%E9%99%A4%E8%BF%9C%E7%A8%8B%E4%BB%93%E5%BA%93%E7%9A%84%E6%9F%90%E6%AC%A1%E9%94%99%E8%AF%AF%E6%8F%90%E4%BA%A4/)


### 保持主分支同步更新

多人协作时, 首先fork后自己仓库, 然后clone到本地, 创建自己的分支

```
# fork该仓库git@github.com:tornadoweb/tornado.git
$ git clone git@github.com:Dinosaurliu/tornado.git
# 创建并切换到开发分支
$ git checkout -b dev
```

持续在dev分支开发新功能, 当在分支开发功能时可能原仓库推送了新的功能, 需要更新master主分支

```
# 切换到master主分支
$ git checkout master
# 增加新的远程仓库, 命名为upstream
$ git remote add upstream git@github.com:tornadoweb/tornado.git
# 查看当前的远程仓库
$ git remote -v

# 同步远程仓库的更新, 拉取upstream对应的远程仓库的更新, 并解决冲突, 注意fetch后会存储在upstream/master本地分支
$ git fetch upstream  # git查找upstream对应服务器, 从中抓取本地没有的数据，并且更新本地数据库，移动 origin/master 指针指向新的、更新后的位置。
# 合并远程master分支
$ git merge upstream/master  # 将最新的origin/master合并到本地master分支上
```


## 参考链接

- [Fork A Repo](https://help.github.com/articles/fork-a-repo/)
- [Syncing a fork](https://help.github.com/articles/syncing-a-fork/)
