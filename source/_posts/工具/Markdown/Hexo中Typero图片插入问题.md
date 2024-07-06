---
title: Hexo中Typero图片插入问题
tags:
  - Hexo 
  - Typero
  - Markdown
  - Bug
categories:
  - 工具
  - Markdown
date: 2024-07-06 14:51:55
---

**[Hexo 官网](https://hexo.io/zh-cn/)**

**Question :** 

​	hexo 处理 本地编写的带有图片的markdown时, 无法正确加载图片的路径

**Solution :**

1. 安装插件 `hexo-typora-img`

   

   ```bash
   npm install hexo-typora-img --save
   ```

   ![image-20240706153100037](Hexo中Typero图片插入问题/image-20240706153100037.png)

2. 修改_config.yml

   ![image-20240706153226789](Hexo中Typero图片插入问题/image-20240706153226789.png)

   ```yml
   post_asset_folder: true #启用 资源文件夹
   relative_link: true #把链接改为与根目录的相对位址
   ```

   

3. 修改typora图片配置

![image-20240706145214903](Hexo中Typero图片插入问题/image-20240706145214903.png)
