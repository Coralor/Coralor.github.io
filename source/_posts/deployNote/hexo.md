---
title: hexo多终端写blog
date: 2022-11-21 04:14:34
tags: hexo
---

hexo 上传部署到github的其实是hexo编译后的文件（即部署时自动生成的.deploy_git文件），是用来生成网页的，不包含源文件，配置文件，主题文件等。所以一旦换了电脑，从github上git clone下来的代码是并不能让我们实现多终端写blog的。

因此利用git的分支管理，把源文件进行备份，但是并非最优解。

# 在源电脑处：
1. 创建一个hexo分支，并设置为默认的分支，便于pull与push时不用指定分支
2. 在本地任意目录，git clone + 项目地址
3. 把文件中除.git文件以外的所有文件删除，**如果没有.git文件，在查看出点击显示隐藏文件**
4. 把原电脑中除了.deploy_git文件的所有源文件拷贝过来
5. copy过来后，添加一个.gitignore文件，用来忽略一些git的提交文件，
   文件内容如下：
   .DS_Store
   Thumbs.db
   db.json
   *.log
   node_modules/
   public/
   .deploy*/
6. 你之前克隆过theme中的主题文件，那么应该把主题文件中的.git文件夹删掉
7. git add .
   git commit –m "add branch"
   git push 

# 在新电脑处
1. 没有git就install git，然后配置全局的邮箱和用户名
2. 设置电脑的公钥ssh, ssh-keygen -t rsa -C "youremail", 按三次回车
3. 执行命令cat ~/.ssh/id_rsa.pub，得到公钥
4. 将公钥粘贴到github的ssh配置中（ssh配置在account setting中）
5. 安装node
6. 安装hexo, npm install hexo-cli -g
7. 在任意目录git clone git@……  (使用ssh可以避免网络不好的情况，如果先git clone 了https的，也可以在.git的config文件中修改成ssh方式)
8. cd xxx.github.io
9. npm install
10. npm install hexo-deployer-git --save
11. 执行hexo命令。（hexo clean / hexo g /  hexo s /  hexo  d）

# hexo d部署时报错的解决方法

hexo部署链接不上github,code为128，最初可以多尝试几次提交，有可能是连接不上github服务器，毕竟国内连接比较不稳定，要是不行可以考虑更新公钥再重试。
```
fatal: unable to access 'https://github.com/Coralor/Coralor.github.io.git/': Failed to connect to github.com port 443: Timed out
FATAL {
  err: Error: Spawn failed
      at ChildProcess.emit (node:events:513:28)
      at ChildProcess.cp.emit (D:\BlogOfCoralPapy\copyBlog\Coralor.github.io\node_modules\cross-spawn\lib\enoent.js:34:29)
      at Process.ChildProcess._handle.onexit (node:internal/child_process:293:12) {
    code: 128
  }
} Something's wrong.
```
1. 更新windows的公钥，添加到github中
2. 删除.deploy_git文件，然后重新进行 hexo clean  hexo g  hexo d
