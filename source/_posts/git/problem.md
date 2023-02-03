---
title: 常见各种git报错汇集和解决
date: 2022-11-22 09:27:23
tags: git
---

1. 提交代码时遇到的报错
```js
    warning: | TLS certificate verification has been disabled! |
    warning: ---------------------------------------------------
    warning: HTTPS connections may not be secure. See https://aka.ms/gcmcore-tlsverify for more information.
    warning: ----------------- SECURITY WARNING ----------------
    warning: | TLS certificate verification has been disabled! |
    warning: ---------------------------------------------------
    warning: HTTPS connections may not be secure. See https://aka.ms/gcmcore-tlsverify for more information.
```

解决方法:  开启ssl认证
git config --global http.sslVerify true

2. 把本地项目推送到新的远程仓库遇到的问题
   
推送过程：
git remote -v 查看本地关联的远程仓库
git remote rm origin  (移除当前的origin)
git remote add origin git仓库地址  
git push origin master --force 或 git pull origin master --allow-unrelated-histories 
**（允许不相关历史提交，并强制合并,会导致git history有新仓库的init commit记录, 要是强推就不会有）** 

遇到的报错：   
(1) git remote add origin + 地址
fatal: remote origin already exists.（报错远程起源已经存在。）

解决方法:  git remote rm origin

(2) git push -u origin master
fatal: Updates were rejected because the remote contains work that you do

解决方法: git pull origin master
但是又会可能出错fatal:refusing to merge unrelated histories
解决方法: git pull origin master --allow-unrelated-histories.

(3) Your local changes to the following files would be overwritten by merge(文件有冲突，没有更新，直接重来一遍)