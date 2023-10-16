1. `git commit --date`, `git commit --author`
2. `git ls-files`
3. `git update-index`
4. `git add -p`
5. `git merge --squash`
6. `git rebase --onto`
	- 参数 1: `目标分支`
         - 参数 2: `剔除的分支`
         - 参数 3: `需要的分支`
         - 应用后，需要的分支上的修改在目标分支上重演(只影响需要的分支)
7. `git bisect start`, `git bisect good`, `git bisect bad`, `git bisect reset`
	- `git bisect start 参数1 参数2`：指定范围
	- `git bisect reset`：从分离头指针状态，恢复到原先开始分支时的状态；即退出二分查找