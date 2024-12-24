# git常用命令整理
## 基本原理

![4c3afd13fcc2fba20f3f63c96cadbbd9](img\4c3afd13fcc2fba20f3f63c96cadbbd9.webp)



## 基础命令

1. 生成仓库
    `git init [仓库地址]`

2. 将文件从工作区添加到缓存区
    `git add [文件名]`
    `git add .`    (将修改文件全部添加)

3. 提交文件到仓库

  `git commit -m "提交说明“ `

4. 查看各个文件的状态
    `git status`



## 分支相关

1. 切换分支
    `git switch 分支名`

2. 切换并生成分支
    `git switch -c 分支名`

3. 查看分支
    `git branch`

4. 删除分支：

​	`git branch -d 分支名`

5. 将其他分支合并到本分支

​	`git merge 其他分支`

6. 建立本地分支和远程分支的联系：

   `git branch --set-upstream-to <branch-name> origin/<branch-name>`

   示例：`git branch --set-upstream-to=origin/dev dev`



## 版本回退

1. 移除所有暂存区的修改，但不会修改工作区。

   `git reset`

2. 移除所有暂存区的修改，**强制删除所有工作区的修改**。（没有被git托管的文件不受影响）

​		`git reset --hard`

3. 将分支状态回滚到指定commit，清除暂存区的修改，但保持工作区状态不变。

​	` git reset <commit>`

4. 将分支状态回滚到指定commit，清除暂存区的修改，**强制删除所有工作区的修改**。

​	` git reset <commit> --hard`



## 配置

1. 将一些操作取**别名**，例如，`git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"`



## 日志

1. 以美化的图查看日志

​	`git log --graph --pretty=oneline --abbrev-commit`



## 远程仓库

1. 克隆仓库
`git clone 仓库地址`

2. 连接远程仓库并命名为origin
`git remote add origin 远程仓管地址`

3. 查看远程仓库地址
`git remote -v`

4. 向远程仓库推送
`git push origin`





## 好的笔记参考

1. [创建与合并分支 - Git教程 - 廖雪峰的官方网站 (liaoxuefeng.com)](https://liaoxuefeng.com/books/git/branch/create/index.html)

