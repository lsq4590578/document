[TOC]

# 1.0.0 git常用操作

```
git status filename
git status
```

```bash
git add . 写入缓冲期
```

```bash
git clean -df    #git add的内容不会被清除掉，否则会被自动删除
```

```bash
git reset HEAD #修改暂存区删除文件，工作区则不作出改变,被 master 分支指向的目录树所替换
```

```

```

![img](D:\技术文档\git\061510341401056.png)

当前状态为master | MERGING：

需要解决冲突，然后再次git add .  git commit git push

```
# 显示commit历史，以及每次commit发生变更的文件
$ git log --stat

# 搜索提交历史，根据关键词
$ git log -S [keyword]

# 显示某个commit之后的所有变动，每个commit占据一行
$ git log [tag] HEAD --pretty=format:%s


git blame 命令是以列表形式显示修改记录，如下实例：


```

