### Git撤销的几种情况

* 提交后, 发现提交的备注写错了, 或者少提交了一些内容.

```shell
git commit --amend -m '添加了--amendc参数后的提交内容将于第一次内容合并, 备注信息将覆盖第一次的备注信息'
```

* 当对本地文件进行修改后, 发现修改错误, 想撤销到上一次提交的版本.

```shell
// 如果工作区修改了两个文件, 想分两次提交, 但不小心执行了命令
git add * 
// 暂存了所有的文件, 想要对其中一个文件执行取消暂存的操作, 执行
git restore --staged <fileName>
// 或者执行下面命令
git reset HEAD <fileName>
```

* 撤销本地的修改, 想回到上一次提交的内容

```shell
git restore <fileName>
// 或者使用, 返回到上一次暂存的内容
git checkout -- <fileName>
```

### 相关命令的第一印象功能

* git reset \[--soft, --mixed, --hard\] : 撤销到某一次提交到\[所有修改已保存暂存区, 所有修改未保存暂存区, 丢弃所有修改\]
* git checkout: 在保留所有提交的情况下, 切换分支HEAD到某一次提交
* git restore: 操作工作区域暂存区的文件来回切换
* git rebase: 已一种更优雅的方式合并代码, 但是好 丢弃时间等提交信息, 不会有分支的提交,而是合并成一条线提交
* git revert: 撤销某一次提交的内容, 并形成一次新的提交记录作为当前状态的基础
* git stash: 保存工作区的临时文件, 方便切换分支后返回重新应用修改
* git cherry-pick: 挑选某一次提交合并到特定分支

rebase 相当于多次 cherry-pick操作.

add 与 reset\(restore --staged\) 操作相反  reset HEAD &lt;fileName&gt; \(restore &lt;fileName&gt;\)可以删除本地的修改

详情参考[官网](https://git-scm.com/book/zh/v2)



### Intellj中的revert, 与revert Commit

revert : 类似取消本地修改, 同: git reset HEAD --hard

revert commit: 则是git 中的revert 同: git revert -n 提交版本号



