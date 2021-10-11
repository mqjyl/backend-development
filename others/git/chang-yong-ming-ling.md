# 常用命令

1、删除远程分支： `git push origin --delete <branch>` 

2、删除本地分支：`git branch -d <branch>`，-D是强制删除

3、查看远程分支与本地分支的对应关系：`git remote show orign`

4、已经删除的远程分支使用 `git branch -a` 仍然可以看到，可以使用 `git remote prune origin` 进行清理

5、`git log --graph --oneline`  显示前六位log码和对应的message

6、`git checkout -b NewFix`  在主分支上建立一个新的分支
