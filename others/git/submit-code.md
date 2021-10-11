# 提交代码

## 一、fork - MR

fork的仓库为小仓，原仓库为大仓，fork大仓代码进行开发，如何同步？

1、给本地仓库增加一个remote

```
➜  next git:(master) git remote -v
origin  git@github.com:luodaoyi/hexo-theme-next.git (fetch)
origin  git@github.com:luodaoyi/hexo-theme-next.git (push)

// 将大仓的远程仓库添加到remote
➜  next git:(master) git remote add upstream https://github.com/iissnan/hexo-theme-next.git

// 查看是否添加成功
➜  next git:(master) git remote -v
origin  git@github.com:luodaoyi/hexo-theme-next.git (fetch)
origin  git@github.com:luodaoyi/hexo-theme-next.git (push)
upstream    https://github.com/iissnan/hexo-theme-next.git (fetch)
upstream    https://github.com/iissnan/hexo-theme-next.git (push)

```

2、同步fork，从大仓 fetch 分支和提交点，传送到本地，并会被存储在一个本地分支 upstream/master

```
➜  next git:(master) git fetch upstream From https://github.com/iissnan/hexo-theme-next
 * [new branch]      dev        -&gt; upstream/dev
 * [new branch]      master     -&gt; upstream/master
 * [new branch]      pisces     -&gt; upstream/pisces
```

3、切换到本地开发分支，防止出错

```
➜  next git:(master) git checkout master
Already on &#39;master&#39;
Your branch is up-to-date with &#39;origin/master&#39;.
```

4、把 upstream/master 分支合并到本地开发分支上，这样就完成了同步，并且不会丢掉本地修改的内容

```
➜  next git:(master) git merge upstream/master
Already up-to-date
```

5、解决冲突，提交到小仓，然后提交MR。

## 二、压缩合并（squash merge）的方式提交代码

采用压缩合并的方式提交代码，有利于保证主干代码的质量（线性提交、每次commit都有构建成功记录），具体的代码提交工作流程如下：

1. clone主仓库到本地
2. 创建特性分支（推荐分支名规范：feature\_{需求ID}），在特性分支进行代码修改并commit
3. push本地特性分支的修改到远端，并从远端特性分支向主仓库master发起MR请求
4. 采用压缩合并（squash and merge）的方式合并代码到主仓库，并填写commit内容
5. 删除特性分支
6. 同步远端master的更新到本地master
7. 重复2～6，每次都采用新的特性分支进行开发

远端仓库的特性分支和本地仓库的特性分支，在完成压缩合并之后就不能再利用了，可以删除掉，若需要继续开发新特性，需要从本地仓库master重新拉分支，即重复流程规范的 2～6。

