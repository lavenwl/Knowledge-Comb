git撤销的集中情况

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



