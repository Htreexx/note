# git

- 本地创建仓库

  ```git
  git init						// 初始化仓库
  git add README.md				// 将工作区的文件添加到缓冲区
  git commit -m "first commit"	// 将缓冲区的文件提交到归档区
  ```

- 添加远程仓库

  ```
  git remote add origin https://github.com/Htreexx/note.git
  ```

  

- 回滚
  - `git reset --mixed <commit id>`  回滚到指定版本，此时缓冲区也重置了，但是保存该版本的修改。mixed是默认模式。
  
  - `git reset --soft <commit id>` 回滚到指定版本，保留缓冲区。
  - `git reset --hard <commit id>` 回滚到指定版本，不保留该版本的的修改。

- 分支
  - `git branch` 查看当前所在的分支
  - `git checkout -b <分支名>` 创建分支，并切换到该分支
  - 打发士大夫犯得上发生
