#GIT 入门命令

## 创建与删除库
	1. 设置自己的身份	
		> git config --global user.name "Your Name" <br>	
		> git config --global user.email "email@example.com"
	2. 创建与删除库
		> mkdir：普通linux命令创建目录
		> git init 将当前目录建立为仓库
		> ls -a:找到仓库中的.git目录，rm -rf 彻底删除后手动删除仓库内文件即可完全删除整个仓库

## 提交数据
	1. 查看当前库的状态
		> git status 
	2. 将修改的文件提交到暂存区
		> git add xxx.cpp;
	3. 将暂存区的文件提交到当前分支
		> git commit -m "提交注释"
	4. 说明
		> git库中分为 工作区：也就是实际工作的文件；暂存区：也就是每次add之后的中间态空间，而此时再去修改工作区的文件后，直接commit不会提交后一次修改的数据。工作区的修改需要再次add后一次的修改，然后再次commit即可提交全部两次修改
		> [详情可见](https://www.liaoxuefeng.com/wiki/896043488029600/897884457270432)
	5. 撤销修改
		>  git checkout -- <filename>
		>  这里的丢弃修改是指将工作区的文件恢复到最近一次add或者commit的状态，即：修改后没有add，此时会恢复到当前版本库的状态,修改且提交到了暂存区，则会恢复到暂存区的状态，
		>  后者想恢复到当前版本库则需要线运行git reset HEAD <filename>命令将暂存区清空，HEAD表示当前版本，或许可以设置成别的历史版本
		>  git reset HEAD <filename>,撤销在暂存区的修改

## 查看提交记录 与 回退版本
	1. 查看完整记录
		> git log
	2. 查看简易记录
		> git log --pretty=oneline  以下是运行结果，用于后面的命令
		> 1094adb7b9b3807259d8cb349e7df1d4d6477073 (HEAD -> master) append GPL
		> e475afc93c209a690c39c13a46716e8fa000c366 add distributed
		> eaadf4e385e865d25c48e7ca9c8395c3f7dfaef0 wrote a readme file
	3. 回退版本
		> git reset
		> git reset --hard HEAD~? ?是任意阿拉伯数据，表示从当前版本回退多少次
		> git reset --hard XXXXX  xxxxx是提交日志中的前面的id列表，一般输入5-8个值即可找到对应版本。
	4. 历史命令
		> git reflog 记录了每次使用的命令

	5. 小结
		> HEAD指向的版本就是当前版本，因此，Git允许我们在版本的历史之间穿梭，使用命令git reset --hard commit_id。
	    > 穿梭前，用git log可以查看提交历史，以便确定要回退到哪个版本。
    	> 要重返未来，用git reflog查看命令历史，以便确定要回到未来的哪个版本。

## 总结
	1. git checkout
		> 此命令是将版本库里的版本替换工作区的版本
	2. git rm
		> 类似linux的删除文件命令，与add一样需要提交，递归删除需要加上-rf 命令应该