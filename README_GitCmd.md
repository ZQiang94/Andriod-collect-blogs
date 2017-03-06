<div align=center><img src="https://dn-myg6wstv.qbox.me/fea7461e85aee80bbe96.png"/></div>
#####常用
提交代码
```javascript
git status //查看工作区相比于暂存区有改动的文件

git diff //查看具体改动文件内容

git add . /git add file某个文件 //将工作区的改动添加到暂存区

git commit –m “desc 描述” //添加注释并将暂存区所有改动提交到当前分支

git push origin HEAD:refs/for/develop //推送本地develop分支到远程origin/develop分支
```

重新打patch进行提交（修改上次提交的内容）
```javascript
git add .
git commit –amend //对上次提交做出修改
git push origin HEAD:refs/for/develop
```

场景：当本地代码已经有改动但是不能提交，又要修复一个紧急bug（目的是为了隐藏本地改动，修复bug并提交之后，将之前隐藏的本地改动显示出来）
```javascript
git stash //将工作区修改的部分隐藏起来

修复紧急bug

提交流程执行完之后

git stash pop //恢复之前隐藏的文件
```

tag命令
```javascript
//创建tag
git tag -a tag-name -m "tag信息"
//例如：git tag -a 3.9.5_2016090901_release -m "修复登录bug" 

//推送版本信息到远程仓库
git push origin tag-name
//例如：git push origin 3.9.5_2016090901_release
```

将暂存区指定文件撤销到工作区
```javascript
git reset HEAD file-src
```

#####基本指令
创建本地仓库
```javascript
git init
```

获取远程仓库
```javascript
git clone [url]
//例如 
git clone https://github.com/GeniusVJR/LearningNotes.git
```

创建远程仓库
```javascript
//添加一个新的remote远程仓库
git remote add [remote-name] [url]
//例：git remote add origin https://github.com/GeniusVJR/LearningNotes.git  origin:相当于改远程仓库的别名

//列出所有remote的别名
git remote

//列出所有remote的url
git remote -v

//删除一个remote
git remoterm [name]

//重命名remote
git remote rename [old-name] [new-remote]
```

添加
```javascript
git add [file1] [file2]          //添加指定文件到暂存区

git add [dir]                    //添加指定文件夹以及子文件到暂存区

git add .                        //添加当前所有文件至暂存区
```

删除
```javascript
git rm file.txt          //从版本库中移除，删除文件

git rm --cached          //停止追踪指定文件，但该文件会保留在工作区
```

更名
```javascript
git mv [file-original] [file-renamed]     //更改文件名，并将此操作加入暂存区
```

代码提交
```javascript
git commit -m "备注desc"                    //提交暂存区到仓库区

git commit [file1] [file2] -m "备注desc"    //提交暂存区的指定文件到仓库区

git commit -a                              //提交工作区自上次commit之后的变化到仓库区

git commit -v                              //提交时显示所有diff信息

git commit --amend                         //修改上一次提交
```

分支
```javascript
git branch                                  //列出本地所有分支

git branch -r                               //列出所有远程分支

git branch (branch-name)                    //创建一个新的分支

git branch -d [branch-name]                 //删除一个分支

git push (remote-name) :(remote-branch)     //删除remote分支

git checkout [branch-name]                  //切换到一个分支

git checkout -b [branch-name]               //创建并切换到该分支

git branch --track [branch] [remote-branch] //创建一个分支，并将该分支与指定远程分支建立追踪关系（该分支在此操作之前不存在）

git branch --set-upstream [branch] [remote-branch]  //将现有分支与指定远程分支建立追踪关系（该分支在此操作之前就已存在）

git merge [branch]                          //合并指定分支到当前分支

git merge [remote-name]/[branch]            //合并下载的改动到分支

git cherry-pick [commitId]                  //选择一个commit（通过commitId）合并至当前分支

从远程库中下载新的改动并合并到当前分支
pull = merge + fetch
git pull [remote-name] [branch]
例如：git pull origin master
```

tag
```javascript
git tag                                   //列出所有tag

git tag [tag-name] -m"tag-desc 备注信息"   //创建一个tag，并添加备注信息

git tag [tag-name] [commitId]             //在指定的commit上新建一个tag

git show [tag-name]                       //查看tag信息

git push [remote-name] [tag-name]         //提交指定tag至指定分支

git push [remote-name] --tags             //提交所有tag

git checkout -b [branch] [tag]            //新建一个分支，指向某个tag  ？？？完全不懂能用在什么场景？？？
```

查看信息
```javascript
git status                                //显示有变更的文件

git log                                   //显示当前分支的版本历史

git log --stat                            //显示commit历史，以及每次commit发生变更的文件

git log --follow [file]                   //显示某个文件的版本历史，包括文件改名
git whatchanged [file]

git log -p [file]                         //显示指定文件相关的每一次diff

git blame [file]                          //显示指定文件是什么人在什么时间修改过

git diff                                  //显示暂存区和工作区的差异

git diff --cached [file]                  //显示暂存区和上一个commit的差异

git diff HEAD                             //显示工作区与当前分支最新commit之间的差异

git diff [first-branch]...[second-branch] //显示两次提交之间的差异

git show [commit]                         //显示某次提交的元数据和内容变化

git show --name-only [commit]             //显示某次提交发生变化的文件？？？？？

git show [commit]:[filename]              //显示某次提交时，某个文件的内容

git reflog                                //显示当前分支的最近几次提交
```

远程同步
```javascript
git fetch [remote-name]/[branch]                          //下载 远程仓库/分支 的所有变动

git remote -v                                             //显示所有远程仓库

git remote show remote-name                               //显示指定远程仓库的信息

git pull [remote-name]/[branch-name]                      //拉去指定 远程仓库/分支 的变化，并与当前分支合并

git push [remote] [branch]                                //上传本地指定分支到远程仓库

git push [remote] --force                                 //强行推送当前分支到远程仓库，即使有冲突

git push [remote] --all                                   //推送所有分支到远程仓库
```

撤销
```javascript
git checkout [file-name]                                  //恢复暂存区的指定文件到工作区

git checkout [commitId] [file]                            //恢复指定的commit的文件到工作区

git checkout .                                            //恢复上一次commit的所有文件到工作区

git reset [file-name]                                     //重置暂存区的指定文件，与上一次commit保持一致，但工作区不变

git reset --hard                                          //重置暂存区与工作区，与上一次commit保持一致

git reset [commitId]                                      //重置当前分支的指针为指定commit，同时重置暂存区，但工作区不变？？？什么场景下用到？？？

git reset --hard [commitId]                               //重置当前分支的HEAD为指定commit，同时重置暂存区和工作区，与指定commit一致

git reset --keep [commit]                                 //重置当前HEAD为指定commit，但保持暂存区和工作区不变

git revert [commit]                                       //新建一个commit，用来撤销指定commit,后者的所有变化都将被前者抵消，并且应用到当前分支
```

其他
```javascript
git archive                                              //生成一个可供发布的压缩包
```

配置
git的设置文件为.gitconfig，它可以在用户主目录下（全局配置），也可以在项目目录下（项目配置）
```javascript
git config --list //显示当前的Git配置

git config -e[--globel] //编辑git配置文件

//设置用户信息
git config [--global] user.name "[name]"
git config [--global] user.email "[email address]"
```

参考：
######[git官方文档](https://git-scm.com/docs)
######[merge详解](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%88%86%E6%94%AF%E7%9A%84%E6%96%B0%E5%BB%BA%E4%B8%8E%E5%90%88%E5%B9%B6)
