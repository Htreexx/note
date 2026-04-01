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
  - `git reset --mixed <commit id>`  回滚到指定版本，此时缓冲区也重置了