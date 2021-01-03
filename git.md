## git

```shell
创建版本库
git init

将文件加入仓库(其实是暂存区)
git add

把文件提交到仓库(将暂存区的所有内容提交到当前分支)
git commit -m "xxx"

查看结果
git status

查看日志
git log

回退
git reset --hard HEAD^     (HEAD^^,,,HEAD~100)

又不想回退了
git reset --hard commitId		(可以用git reflog 查看，相当于重返未来)

记录每一次命令
git reflog

将readme.txt文件在工作区的修改全部撤销
git checkout -- readme.txt

创建SSH Key
ssh-keygen -t rsa -C "email"

添加远程仓库(默认是origin)
git remote add origin xxx.git

将本地库的内容推送到远程(master分支)(由于第一次远程库是空的，所以加-u)
git push -u origin master
git push origin master

从远程库克隆一个本地库
git clone xxx.git

查看当前分支
git branch

创建分支
git branch dev
git switch -c dev (也可以)

切换分支
git checkout dev
git switch dev

合并分支
git merge dev

删除分支
git branch -d dev
```

