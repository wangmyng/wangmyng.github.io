## Git
### git rm ###

当文件已被暂存时无法直接 rm，-f 强制删除 使暂存区与本地同时删除，--cached 只删除暂存区的文件信息

### git  log ###

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

### git commit ###

--amend 修改提交信息 重新提交 覆盖上一次提交，用于遗落文件或描述信息补充等

