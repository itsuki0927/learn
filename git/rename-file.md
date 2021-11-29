# 重命名文件的两种方式

## 第一种

1. `mv a.txt b.txt` 先使用 `mv` 命令将 a 文件重命名 b 文件
2. `git add b.txt` 使用 `git add` 将 b 文件添加到暂存区
3. `git rm a.txt` 使用 `git rm` 将 a 文件从暂存区删除

## 第二种

1. `git mv a.txt b.txt` 直接使用`git mv old-file-name new-file-name` 进行重命名
