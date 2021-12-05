# note

Git 小记

## git checkout filename

命令 `git checkout readme.txt` 意思就是，把 readme.txt 文件在工作区的修改全部撤销，这里有两种情况：

- 一种是 `readme.txt` 自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

- 一种是 `readme.txt` 已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次 `git commit` 或 `git add` 时的状态。
