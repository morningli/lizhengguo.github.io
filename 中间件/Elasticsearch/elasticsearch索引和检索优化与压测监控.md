# elasticsearch索引和检索优化与压测监控

[原文链接](https://www.jianshu.com/p/901651b81788)

## Overview

先来看看es的整体架构图，上面有多个重要模块，今天主要写在lucene上面的index模块与search模块的优化经历，力求简要写出改变了configuration之后，会给es cluster带来什么样的影响。

![](https://upload-images.jianshu.io/upload_images/2189341-1811c15a476060eb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/914/format/webp)

## Index Optimization

![](https://upload-images.jianshu.io/upload_images/2189341-2c58d57d74b19bac.jpeg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp)

上图展示了一个doc index/write请求过来，es为其建立倒排的过程，而index opt.的优化点就主要集中在该posting list building过程，先认识4个组件（heap buff, os cache, transLog, disk）