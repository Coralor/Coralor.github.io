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

(4) ssh连接方式下，推送代码到github报错：
kex_exchange_identification: Connection closed by remote host
Connection closed by 20.205.243.166 port 22
fatal: Could not read from remote repository.
Please make sure you have the correct access rights
and the repository exists.

解决方法：
1.ssh -T -p 443 git@ssh.github.com 修改端口

2.在C:\Users\admin\.ssh目录下添加config文件
文件内容为： 
Host github.com
User git
Hostname ssh.github.com
PreferredAuthentications publickey
IdentityFile ~/.ssh/id_rsa
Port 443

3.ssh -vT git@github.com 测试是否可连接github
出现以下即连接成功：
debug1: pledge: network
debug1: client_input_global_request: rtype hostkeys-00@openssh.com want_reply 0
debug1: client_input_channel_req: channel 0 rtype exit-status reply 0
Hi Coralor! You've successfully authenticated, but GitHub does not provide shell access.
debug1: channel 0: free: client-session, nchannels 1
Transferred: sent 3016, received 2776 bytes, in 0.6 seconds
Bytes per second: sent 5332.4, received 4908.1or.github.io> git pull origin hexo  
debug1: Exit status 1