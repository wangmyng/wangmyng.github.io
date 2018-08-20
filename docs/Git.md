## Git 三棵树结构 ##

这里的树不是准确地代表数据结构，可以简单理解为 “文件的集合”

**HEAD**	上一次提交的快照，下一次提交的父节点

**Index**	下一次提交的快照

**Working Directory**	本地工作目录，接下来简称 work

一个简单的操作循环如下：

**Work**  stage files -->  **Index**  commit -->  **HEAD**  checkout file/project --> **Work**

HEAD 相当于一个指针，始终指向当前分支，并保证分支引用指向最顶端的 commit 快照

每次 commit 形成一个快照，链接于当前分支的最顶端快照，并移动 HEAD 指向的分支指向这个快照

## git reset ##

* --soft，第一级：移动 HEAD

reset 移动 HEAD 指向的分支，使分支指向上一个快照，相当于撤销了上一次的 commit

* --mixed，第二级：更新索引（默认级别，省略参数）

在第一步基础上，取消暂存的内容，相当于撤销了 commit 和 add 两次操作

* --hard，第三级：更新工作目录

***危险*** 在第二步基础上，撤销了本地工作的更新内容，强制覆盖了本地目录的文件

* git reset file.txt 是 git reset --mixed HEAD file.txt 的简写形式，本质是将 file.txt 复制到索引当中

* git reset eb43f file.txt 将文件恢复到 commit eb43f 的版本

* git status 推荐使用 git reset 来取消暂存


## git checkout ##

与 git reset 一样，也操纵三棵树

* 带路径参数

***危险*** 

* 不带路径参数




## git stash ##

## git diff ##

显示本地与暂存区的区别，即尚未暂存的更新内容

--cached 或 --staged 显示暂存区与 HEAD 内容的区别，即要被提交的更新内容

## git rm ##

当文件已被暂存时无法直接 rm，-f 强制删除 使暂存区与本地同时删除，--cached 只删除暂存区的文件信息

## git commit ##

--amend 修改提交信息 重新提交 覆盖上一次提交，用于遗落文件或描述信息补充等

## git  log ##

-p 显示每次提交的内容，

-(n) 显示最近的 n 条日志，如 -p -2 显示最近两次提交的内容

--stat 显示提交的统计信息，如加减或更改的文件数量

--pretty= 以指定格式显示信息，可选值包括 online short full format 等，其中 format 通过自己指定格式来输出信息

--graph 以 ASCII 图形展示分支 合并历史

//组合使用实例1：    git log --pretty=format:"%h %s" --graph

--since= 和 --after= 限定起始时间

--until= 和 --before= 限定截止时间

时间有多种表现形式，如 2018.07.22 或 06-22 或 3.days 2.minute等

//组合使用实例1：    git log --since=2017.09.17 --until=07.23

--author 指定作者，--committer 指定提交人，--grep 指定提交时 -m 信息中的关键字

//当两者同时使用得到并集，再加 --all-match 得到交集

-S 指定提交的文件中的文字内容中的字符串， 如方法名、类名、注释等字符串

