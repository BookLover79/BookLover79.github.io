---
title: Hello World
date: 2024-12-06
categories: 工具
tags: ['杂项']

---
## 快速开始

### 本地运行

``` bash
$ hexo g = hexo generate  # 创建静态页面
$ hexo s = hexo server  # 本地部署
```

### 远程部署

``` bash
$ hexo g = hexo generate 
$ hexo d = hexo deploy
```

## 分类和标签 Categories & Tags


- 分类：结构化、层次感强，用来划分大的内容模块。分类不宜过多，也不要太乱，一般来说，10个左右的大分类比较合适，最好一眼就能看懂。每个大分类可以有几层子分类，这样也更有条理。
- 标签：灵活，用来描述文章的细节和具体内容，通常用来补充分类无法覆盖到的多维度信息。标签没有数量限制，可以根据每篇文章的内容灵活添加。类似于关键词




|一级分类|slug|内容描述|
|:---------:|:------:|:-----------:|
|编程技术|programming |前端开发、后端开发、移动开发、数据库和编程语言的技术知识。|
|效率工具	|productivity	|办公工具、开发工具、自动化脚本、时间管理等，提高工作效率的实用工具和方法。|
|资源干货	|resources |开源框架、开源库、开源项目和插件的使用及推荐。|
|产品经理	|productmanager	|产品规划、需求管理、项目管理、数据驱动，以及市场与运营相关的管理知识，帮助产品经理优化产品流程和策略。|
|人工智能	|ai	|涵盖机器学习、自然语言处理 (NLP)、知识图谱等技术，探讨人工智能领域的前沿技术和应用案例。|
|数据科学	|datascience |数据挖掘、数据分析与建模等与数据处理相关的技术和方法。|
|实践作品	|projects	|个人项目日志、代码实践以及各种实战作品展示。|
|建站记录	|webdevelopment|网站搭建、前端优化、SEO优化和服务器配置的完整记录。|
|日志随笔	|journal|涵盖旅行与探索、阅读与思考、个人成长、兴趣爱好等生活感悟和随笔，分享个人生活与成长经验。|

## 文章头部

- title: Title

- date: YYYY-MM-DD HH:MM:SS

- categories: category

- tags: [tag1, tag2, ...]

- 加密码：
  - password: xxx

  - abstract: 你可以看到我吗

  - message: who am i



## 后台管理

https://github.com/jaredly/hexo-admin

## 博客风格（排名前10）

https://github.com/Ailln/awesome-hexo-theme?tab=readme-ov-file

## 解压

### 多卷压缩文件

给定四个文件

- `04.zip.001`（第一个分割文件）
- `04.zip.002`（第二个分割文件）
- `04.zip.003`（第三个分割文件）
- `04.zip.004`（第四个分割文件）

解压步骤：

1. 确保所有的分割文件都在同一个文件夹中。

2. 使用`7z x 04.zip.001`，7-Zip将自动识别这是一个多卷压缩文件

## GPT提示词

### 英文

指出下面英文存在的问题，使其更加符合英文论文规范（要求：没有错的语句和词语尽量保留原文）

提示：如果我希望将论文发表在BAMS期刊上，请按照BAMS文章的风格，对上面的内容进行润色。

Hint: If I wish to publish the paper in the BAMS journal, please polish the above abstract in the style of the BAMS article.

### 中文

润色下文，使其更加通顺、学术，符合论文规范（要求：没有错的语句和词语尽量保留原文）

精简下文，使其保持原意的情况下，更加通顺、学术，符合论文规范（要求：没有错的语句和词语尽量保留原文）

