# Git

## 文件操作

### git restore

作用：恢复工作区的文件，当你不想要当前文件的修改了，可以使用 restore 恢复此文件到之前的版本

- `git resotre <file>`：将 file 恢复至暂存区或最后一次提交的状态
- `git restore --staged <file>`：将 file 恢复为暂存区状态
- `git restore --source=<commit> <file>`：从指定的 commit 中恢复文件

## 分支操作

### git rebase

作用：将一个分支的提交重新应用到另一个分支的基础上，从而创建一个线性的提交历史

- `git rebase <branch>`：将当前分支的 commit 重新应用到 branch 上
- `git rebase -i HEAD~x`：启动交互式 rebase，编辑最近的 x 次提交

应用场景：

1. 从 master 分支迁出之后，进行开发，之后 master 分支有新的提交，要将本次的 commit 接入 master，相当于以 master 最新的情况为基础进行相同的开发

![image-20240731195144888](C:\Users\zhengdaxu\AppData\Roaming\Typora\typora-user-images\image-20240731195144888.png)

2. 合并多个 commit，以保持提交历史整洁，避免冗余的 commit：使用 `git rebase -i HEAD~x`，对于需要合并的 commit，使用 squash 进行标记（按照对应提示进行）

```
pick commitA
pick commitB
pick commitC
```

```
pick commitA
squash commitB
squash commitC
```

如上编辑后，输入 `:wq` 保存，执行结果将三次 commit 合并为最后的 commitA，结果相同

![image-20240731195122170](C:\Users\zhengdaxu\AppData\Roaming\Typora\typora-user-images\image-20240731195122170.png)

### git revert

### git reset