title: "windows脚本转移Linux上git命令出现错误"
author: Jon
tags:

  - 巨坑
categories:
  - 错误记录
date: 2019-10-03 23:18:00
---
## 场景

在我将整个hexo博客源码从windows10转移到linux（centos7）中时，脚本中的git命令没有例外的出现了这个错误

```bash
git: 'pull' is not a git command. See 'git --help'.  
Did you mean one of these? 
		pull

git: 'push' is not a git command. See 'git --help'.  
Did you mean one of these? 
		push

```

这是什么人间疾苦，于是我Google，找到最接近的答案就是这个

[https://stackoverflow.com/questions/1465398/git-pull-broken](https://stackoverflow.com/questions/1465398/git-pull-broken)

这是十年前的一个issue，很明显不是这个问题。git早已经修正了这个issue。

**那么问题来了，这到底是什么原因导致git认为你认为完全正确的命令是有问题的呢：**

* 首先我保证绝对不是git版本的问题，我反复在服务器上更替了不同版本的git，无论是直接安装编译好的版本还是自己编译都多次尝试过，但是无一例外出错了。还是相同的错误。
* 其次脚本绝对是可运行的，在windows上可以正确无误的执行。
* 服务器镜像问题？没钱换服务器，况且服务器上运行了不少东西，不好更换，但我觉得的应该不是，其他脚本命令是可以运行的。

那到底是哪的问题呢？

## 解决

最后我几乎快完全丧失解决它的希望时，我进行了最后一次尝试。

**用在linux  vim重写脚本，发现在写的途中有些命令写出现和原来的颜色是不一样的**

最后脚本运行成功。

**所以千万别用windows10的记事本写代码，应该是编码的问题，还是建议用专业的编辑器写脚本**