```
git 初始化提交项目
Git初始化本地已有项目,并推送到远端Git仓库操作
1. 创建本地项目，在项目根目录执行git init命令
git init

2. 在git服务器上创建一个仓库，这里使用GitHub创建一个仓库。
例如这个git仓库
https://github.com/ios-zhouyu/VueDemo.git

3. 执行git remote add origin ，git path 可以在【Clone or download】中获取。
git remote add origin https://github.com/ios-zhouyu/VueDemo.git

4. 从远程分支拉取master分支并与本地master分支合并。
git pull origin master:master

5. 提交本地分支到远程分支
git push -u origin master

6. 将现有项目添加并提交上传
git add -A
git commit -m '初始化git项目'
git push --set-upstream origin master

```

```
git init
git remote add origin +远程仓库地址
git pull origin master:master
git push -u origin master
git add -A
git commit -m '初始化git项目'
git push --set-upstream origin master
```

