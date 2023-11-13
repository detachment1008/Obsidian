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
	- `git bisect start 参数1 参数2`：指定范围(第一个是后，第二个是前)
	- `git bisect reset`：从分离头指针状态，恢复到原先开始分支时的状态；即退出二分查找

---

1. `git checkout -b 分支名 仓库名/分支名`
	- 创建并设置跟踪分支(也会根据跟踪分支，创建新分支)
2. `git branch -u 仓库名/分支名 分支名`
	- 将给定分支设置为跟踪对应的远程分支
	- 如果省略最后的分支名，就是使用当前所在的分支
3. `git pull`
	- 拉取远程仓库的对应跟踪分支，然后根据跟踪分支更新当前分支(即使有多个分支跟踪他，也只更新当前分支)
	- `git push` 同理，可以将当前分支推送到远程的对应分支，并同时更新跟踪分支(即使跟踪分支和当前分支名不同)
 4. `git push <remote> <place>`
	 - `不指定任何参数`: 默认以当前的分支为参数进行推送
	 - `<remote>`: 远程仓库名
	 - `<place>`
		 - `单个的分支名`: 将本地的该分支推送到远端的该分支
		 - `<source>:<destination>`: 指定源和目的，上文的单个分支名，就是默认的源和目的相同
			 - 源可以不是一个分支，可以只是一个提交记录，只要是 fast-forward 就可以了
			 - 如果远程仓库中没有这个分支名，他就会自动创建这个分支然后进行同步操作
5. `git fetch <remote> <place>`
	- `<place>`
		- `单个的分支名`: 获取远程仓库的该分支，然后更新到本地的 `origin/` 分支上
		- `<source>:<dstination>`: 指定源和目的，此时源变成了远程仓库中的某个提交了，而目的就是本地的某个分支
			- 指定了目的，就不会去设置默认的只读的远程跟踪分支了
			- 同理，如果本地没有这个分支名，会自动的在本地创建这个分支，然后再进行更新
	- `不指定任何参数`：下载所有分支的所有提交，更新到默认的远程跟踪分支上
6. `git push 或 git fetch 时将 <source> 留空`
	- `git push origin :foo`: 将远程仓库中对应的分支删除
	- `git fetch origin :bar`: 在本地创建一个新分支(在当前 HEAD 的位置)
7. `git pull`
	- 就是 `git fetch` + `git merge` 的命令依次执行，`merge` 的参数就是 `git fetch` 中的 `<destination>`