---
title: 开发中常用git命令
date: 2022-11-22 09:27:23
tags: git
---

# 常用的git命令

## 开发未完成时需切换分支

+ git stash 使用
将本地没提交的内容(git commit的内容不会被缓存 但git add的内容会被缓存)进行缓存并从当前分支移除，缓存的数据结构为堆栈，先进后出
git stash save "xxx"   加上自己的注解进行缓存
git stash与git stash save是一样的，将没有提交的内容缓存并移除，而这条缓存名称为最新一次提交的commit -m的内容，如果没有本地提交则是拉远程仓库是的commit内容

需要特别注意的是，**git stash 只会操作被git追踪的文件**。stash后新增的文件并没有进入缓存，就是因为git还没有追踪这些新增的文件，所以需要进行git add [文件名]让git追踪这文件，再stash
!['stash'](/images/stash.png)

1. git stash pop 
将堆栈中最新的内容pop出来应用到当前分支上，且会删除堆中的记录。pop后面有一行是Dropped，在堆栈中删除了这个缓存。如果pop出来的内容有冲突，git会中断此次pop并告知你需要进行冲突解决
也可以指定堆栈中的记录通过在git stash pop后面加上git stash list中的名称(eg：git stash pop stash@{0})

2. git stash apply
与pop相似，但他不会在堆栈中删除这条缓存，适合在多个分支中进行缓存应用
也可以进行指定git stash apply stash@{0}

3. git stash list
返回缓存的列表

4. git stash drop/git stash clear
git stash drop [名]删除单个缓存 举例git stash drop stash@{0}
git stash clear 全部删除

5. git stash show
git stash show [名]显示与当前分支差异。(eg: git stash show stash@{0}) 加上-p可以看详细差异


## 回退本地代码和线上代码

1. git log 查看commit提交记录

2. git reset --hard  number 这里的number是获取的commit对应的版本号
eg: commit1 __ 1223ddff    最新的提交（出错的提交，需要撤销掉改修改）
    commit2 __ 13445dd     需要回退到此处的提交。
    git reset --hard 13445dd 
    此时本地代码可以回退到指定的版本号

3. git push -f origin branchName  强推到线上分支，提交后线上代码与本地同步，回退成功。


## 修改未提交的commit记录

+ 修改最后一次提交
1. git commit --amend。会进入对最后一次提交信息vim编辑器界面
（普通模式下，按i进入编辑模式，编辑模式下按esc退出到普通模式，普通模式下按:进入命令模式，输入wq即可保存修改并退出vim编辑器）
2. git commit --amend -m 'new commit message'

+ 修改前边某次提交， **若是中途修改出错，或是不想继续修改，可以执行git rebase --abort终止**
1. git rebase -i HEAD~2，进入commit的选择界面，其中2代表最后2次提交，需要注意的是该值必须要小于提交总数，否则会报错
2. 将需要改变的提交的pick改为edit，保存并退出（选中多个，则会将多个提交合并成一个）
3. 执行命令git commit --amend进入vim编辑器，修改对应提交信息
4. 修改成功后，执行git log，查看修改结果
5. 确认无误后，执行git rebase --continue，至此，修改成功 Successfully rebased and updated。


## 修改已提交的commit
如果需要修改的commit已被提交到远程仓库，那么在基于前面修改的基础上，还需要强推本地仓库到远程分支
git push origin <branch nama> -f


## 撤销commit
1. git reset HEAD^n  n代表最后的n次commit
2. 各参数的意思：
（1）--mixed（默认参数）：**不删除工作空间改动代码**，撤销commit，并且撤销git add . 操作。
  git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。
（2）--soft：**不删除工作空间改动代码**，撤销commit，不撤销git add . 如果需要撤销git add，可以点击unstage changes移除暂存
（3）--hard：**删除工作空间改动代码**，撤销commit，撤销git add . **慎用！！！！**

完成这个操作后，就恢复到了上一次的commit状态。