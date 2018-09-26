---
title: Hexo 上传 github 
date: 2018-01-18 16:21:20
tags: [hexo, amber, blog, github]
---
## _config.yml 修改配置


```
deploy:
  type: git
  repo: https://github.com/amberyang/amberyang.github.io
  branch: master
```
**XXX.github.io, XXX 一定要是你的 github 名称，否则提示页面找不到**

## 上传 github

清除缓存

```
➜  hexo clean
```

生成静态文件

```
➜  hexo generate
```

部署网站

```
➜  hexo deploy
```

PS：报错

### npm install hexo-deployer-git --save

执行

```
npm install hexo-deployer-git --save
```

### Error: Permission denied (publickey). fatal: Could not read from remote repository

删除 /node_modules/hexo-deployer-git 文件夹，再执行一次 hexo-deployer，正常情况下需要输入用户名和密码。

## 分类和标签无法显示或者无法点击

1.在 source 文件夹下创建文件夹 tags 和 categories。

2.在 tags 目录下添加 index.md 文件：
       
```
---
title: 标签
date: 2017-05-27 15:37:47
type: "tags"
---
```

3.在 categories 目录下添加 index.md 文件：


```
---
title: 分类
date: 2017-05-27 15:38:46
type: "categories"
---
```
4.修改主题的 _config.yml


```
menu:
  home: /
  archives: /archives
  tags: /tags
```


## 参考

https://hexo.io/zh-cn/docs/commands.html

https://www.zhihu.com/question/29017171

