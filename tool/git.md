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

- 注意事项

  - 频繁 Commit：在自己的分支里，不要憋大招（写了一周代码才 commit 一次）。写好一个独立的小函数，或者解决了一个小 Bug，就 commit 一次。这样即使写崩了，也能快速回滚到上一个健康的状态。

  - 不要在本地 merge 远程的分支：在实际的企业协同中，通常不建议在本地直接使用 `git merge` 合并到 main 然后 push。标准做法是：把你的 feature 分支 push 到远程仓库，然后在 GitLab/GitHub/Gitee 的网页端发起一个 Pull Request (PR) 或 Merge Request (MR)，让同事 Code Review 后，由系统合并到 main。