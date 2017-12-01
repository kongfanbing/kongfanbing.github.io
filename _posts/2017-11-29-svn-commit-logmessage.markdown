---
layout: post
title:  svn提交日志不能为空的配置
date:   2016-08-02 11:44:00 +0800
categories: svn
---

>  温故而知新，~~可以为师矣~~
>
>  ——孔子

**svn提交的时候，不写日志不是个好习惯···**

1. 登录svn服务器后，进入对应项目的hooks目录。

2. copy一份pre-commit.tmpl为pre-commit。

3. 修改pre-commit文件。

4. 修改文件的可执行权限`chmod u+x pre-commit`。

   以下是修改过后的示例：

```
REPOS="$1"
TXN="$2"

# Make sure that the log message contains some text.
SVNLOOK=/usr/bin/svnlook
#$SVNLOOK log -t "$TXN" "$REPOS" | \
#   grep "[a-zA-Z0-9]" > /dev/null || exit 1
LOGMSG=`$SVNLOOK log -t "$TXN" "$REPOS" | grep "[a-zA-Z0-9]" | wc -c`
if [ "$LOGMSG" -lt 5 ];
then
  echo -e "说明不能少于5个字" 1>&2
  exit 1
fi

# Check that the author of this commit has the rights to perform
# the commit on the files and directories being modified.
#commit-access-control.pl "$REPOS" "$TXN" commit-access-control.cfg || exit 1

# All checks passed, so allow the commit.
exit 0

```



**允许修改提交的日志**

1. copy一份pre-revprop-change.tmpl为pre-revprop-change文件。
2. 修改文件的可执行权限。`chmod u+x pre-revprop-change`