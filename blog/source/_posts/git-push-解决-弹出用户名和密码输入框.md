title: git push 解决 弹出用户名和密码输入框
author: Jon
tags:
  - 知识就是力量
categories:
  - Git学习备忘
date: 2019-10-04 02:32:00
---

## 首先查看你的远程仓库版本

```bash
git remote -v 

result：
origin  https://github.com/username/username.github.io.git (fetch)
origin  https://github.com/username/username.github.io.git (push)
```

若你没有添加别人的仓库，通常只有这两个

## 重新设置对应远程仓库成ssh的方式:

```
git remote rm origin
git remote add origin git@github.com:username/repository.git
git push -u origin master
```

