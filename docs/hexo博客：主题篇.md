# hexo博客：主题篇

[Tranquilpeak](https://github.com/LouisBarranqueiro/hexo-theme-tranquilpeak)（宁静）是 hexo 的一款相比于 next 更美观，更加图文并茂的主题。

<!-- more -->
<!-- toc -->

#### 主题\_config.yml 文件设置

```yml
# 图片资源位置
image_dir: assets/images
# 侧导航设置
sidebar:
  menu:
    home:
      title: global.home
      url: /
      icon: fa fa-home
    categories:
      title: global.categories
      url: /all-categories
      icon: fa fa-bookmark
    tags:
      title: global.tags
      url: /all-tags
      icon: fa fa-tags
    archives:
      title: global.archives
      url: /all-archives
      icon: fa fa-archive
    about:
      title: global.about
      url: /#about
      icon: fa fa-question
    github:
      title: global.github
      url: https://github.com/topseczbw
      icon: fab fa-github
    mail:
      title: global.mail
      url: mailto
      icon: fa fa-envelope
 # 作者相关
 author:
      email: 569119225@qq.com
      location: 北京-海淀
 # 个性化定制
 # 侧边栏状态123456
 sidebar_behavior: 2
 # 阅读时侧边栏状态是否收起
 clear_reading: true
 # 缩略图设置
 thumbnail_image: true
 thumbnail_image_position: left
 auto_thumbnail_image: true
 # 缩略图设置
 cover_image: http://pov6tpkt4.bkt.clouddn.com/bg4.jpg
 favicon: avatar.png
 image_gallery: true
 # 分类、标签、归档是否分页
 archive_pagination: true
 category_pagination: true
 tag_pagination: true
 # 评论系统
 disqus_shortname: zbwblog
```

#### 主题 languages/zh-cn 设置

```yml
author:
  # 你的个人简介 (支持 Markdown 和 HTML 语法)
  bio: "> 乐观看待未来<br> 不抱怨只自省<br> 超越常人的坚持"
  # 你的工作简介
  job: "TALFE"
```

#### 文章中 Front-matter 常用设置

```yml
# 隐藏所有文章页面上的侧边栏，让文章全宽以提高阅读，并享受宽图像和封面图像
clearReading: true
# 文章缩略图地址 主题/source/assets/images/ 或者 在线地址
thumbnailImage: image-1.png
# 缩略图定位：right left bottom
thumbnailImagePosition: bottom
# 如果没有缩略图，自动选择封面图像或文章图库中的第一张照片作为缩略图
autoThumbnailImage: yes
# 标题、日期、类别对齐方式
metaAlignment: center
# 在帖子视图（封面）中，以全尺寸显示在帖子顶部的图像
coverImage: image-2.png
# 给封面加标题
coverCaption: "A beautiful sunrise"
# 给封面加帖子元(标题、日期、类别)
coverMeta: out
# 封面占比：partial full
coverSize: full
# 禁用评论
comments: false
# 禁用帖子元
meta: false
# 禁用发布操作
actions: false
```

#### 常用操作

- 标签插件

  ```javascript
  {% alert danger no-icon %}
    警告内容
  {% endalert %}
  {% hl_text danger %}
    高亮文本
  {% endhl_text %}

  class类型： danger、success、warning、info
  ```

#### 踩过的坑

1. ##### 网站运行时样式丢失

* 背景
  * 博客搭建完成后，尝试通过 github 管理本 blog 项目  
  * `blog` 指本项目，并非 `hexo` 最终吐出来的    
  * `public` 静态文件
  * 当在另外一台电脑 `git clone` 本项目，希望更新博客时，本以为根据根目录和主题目录的 package.json 安装依赖完成后可以直接运行并编辑博客，但是运行起来后发现，网站的样式丢失了。
  * 定位到问题，是因为第一次搭建 Tranquilpeak 主题时，作者推荐直接下载生产版压缩包（里面包含压缩好的 css 和 node_modules），而不是从仓库 master 分支拉开发版本项目。是因为即使是在开发环境中，主题项目引用的仍然是压缩过后的 css。所以由于开发环境主题项目没有压缩 css，而经过 hexo 编译的 markdown 文件引入的又是生产环境压缩过后的 css，导致样式丢失。

* 解决方案
  * 进入主题目录，执行 `npm run build` 命令，再启动 hexo 就正常了。

2. ##### 如何格式化全站设置时间样式
   * 找到 `themes/tranquilpeak/languages/zh-cn.yml ` 文件 
   * 根据 `moment.js` 中的format 格式修改 `date_format`

3. ##### 增加博客评论功能
   * 踩坑 `gitalk` 后，决定使用 `来比力` 
   * 站在巨人的肩膀上吧！[链接地址](https://blog.csdn.net/qq_41923622/article/details/82966186)

#### 搭建博客相关资源

1. [GitHub+Hexo 搭建个人网站详细教程](https://zhuanlan.zhihu.com/p/26625249)
2. [Codeing Page](https://coding.net/pages)
3. [Hexo 从 GitHub 到阿里云](https://zhuanlan.zhihu.com/p/58654392)
4. [百度站点管理](https://ziyuan.baidu.com/site/index)
5. [如何配置 SSH 公钥访问 git 仓库？](https://coding.net/help/doc/git/ssh-key.html)
6. [Coding Pages 托管](https://blog.csdn.net/qq_36667170/article/details/79318665)
7. [Hexo博客提交百度和Google收录](https://www.jianshu.com/p/f8ec422ebd52)
8. [腾讯云开发平台 = coding ≈ github](https://dev.tencent.com/u/singlebridge/p/singlebridge/git/pages/settings)
