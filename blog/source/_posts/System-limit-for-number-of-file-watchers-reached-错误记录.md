title: 错误记录：System limit for number of file watchers reached 
author: Jon
tags:

  - 不是特别坑
categories:
  - 错误记录
date: 2019-10-03 22:49:00
---
### 我在执行hexo server -d 时报错
Error: ENOSPC: System limit for number of file watchers reached, watch '/root/blog/Treeeeeeee.github.io/blog/themes/inside/node_modules/hexo/lib/plugins/tag/index.js'
    at FSWatcher.start (internal/fs/watchers.js:165:26)
    at Object.watch (fs.js:1258:11)
    at createFsWatchInstance (/root/blog/Treeeeeeee.github.io/blog/node_modules/chokidar/lib/nodefs-handler.js:38:15)
    at setFsWatchListener (/root/blog/Treeeeeeee.github.io/blog/node_modules/chokidar/lib/nodefs-handler.js:81:15)
    at FSWatcher.NodeFsHandler._watchWithNodeFs (/root/blog/Treeeeeeee.github.io/blog/node_modules/chokidar/lib/nodefs-handler.js:233:14)
    at FSWatcher.NodeFsHandler._handleFile (/root/blog/Treeeeeeee.github.io/blog/node_modules/chokidar/lib/nodefs-handler.js:262:21)
    at FSWatcher.<anonymous> (/root/blog/Treeeeeeee.github.io/blog/node_modules/chokidar/lib/nodefs-handler.js:495:21)
    at FSReqWrap.oncomplete (fs.js:154:5)

**文中说我达到文件监视程序数量的系统限制，不是很明白**

