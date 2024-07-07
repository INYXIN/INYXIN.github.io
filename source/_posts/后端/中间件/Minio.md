---
title: Minio
date: 2024‎-‎06-‎10 ‏‎23:02:34
tags: [ Minio]
categories: [ 后端 , 中间件]
index_img: /img/default.png
---



## 安装

### 通过 1panel 快速安装 minio (略)

### 使用! [image-20240617162518125](

##### 创建桶

![image-20240611102448868](Minio/image-20240611102448868.png)

##### 设置为公开, 私有无法访问

##### ![image-20240611102707639](Minio/image-20240611102707639.png)

##### 配置访问规则, 前缀

![image-20240611102812724](Minio/image-20240611102812724.png)

##### 创建服务账号

![image-20240611102921408](Minio/image-20240611102921408.png)

![image-20240611102939916](Minio/image-20240611102939916.png)

##### 代码中使用

![image-20240611102958006](Minio/image-20240611102958006.png)

## 2.通过 Docker 自己安装

```sh
docker pull docker://minio/minio # 拉取minio镜像
```

![image-20240612091412567](Minio/image-20240612091412567.png)



