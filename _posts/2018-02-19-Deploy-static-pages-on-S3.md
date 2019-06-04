---
title: 部署HEXO静态页面到S3上
date: 2018-02-19 
tags: 
- S3
- Hexo
- Route53
author: Faushine
layout: post
---
之前一直是用[Github Pages](http://pages.github.com/)搭博客，虽然写得不多但是渐渐也能感受到没有数据库的痛，于是在过年期间就把它搬到了[S3](https://amazonaws-china.com/cn/s3/?nc2=h_m1)上面，考虑到一年的免费期还是很划算的。根据Amazon的免费套餐，AWS新客户在一年内的每个月中可以获得标准存储类中的 5GB Amazon S3 存储、20000 个 Get 请求、2000 个 Put 请求以及 15GB 的对外数据传输量。这个量对于一般个人博客来说绰绰有余了。
>什么是 AWS S3

Amazon S3 是专为从任意位置存储和检索任意数量的数据而构建的对象存储，这些数据包括来自网站和移动应用程序、公司应用程序的数据以及来自 IoT 传感器或设备的数据。它旨在提供 99.999999999% 的持久性，并存储每个行业的市场领导者使用的数百万个应用程序的数据，属于AWS底层存储服务。同时提供Host静态网站的功能

> AWS S3 定价标准（未指定请求）

||定价
| --- | :-:
|PUT、COPY、POST 或 LIST 请求 | 每 1000 个请求 0.005 USD 每 GB 
|GET 及所有其他请求 | 每 1000 个请求 0.0004 USD 每 GB 

下面会简单介绍一下部署的过程，期间没有什么值得一提的操作，就是碰上的坑比较清新脱俗。

## 准备数据
因为之前用Jekyll感觉很不顺手，了解下来发现Hexo很人(zhu)性(ti)化(duo)就果断换成了Hexo。以本站为例，选用的主题是[NexT](http://theme-next.iissnan.com/)，用户可以通过修改主题的`_config.yml`去增删前端的元素。具体步骤请参见[开始使用NexT](http://theme-next.iissnan.com/getting-started.html)。

## 创建S3 Bucket
存储桶名称必须与托管的网站名称匹配，所以这个部分需要创建两个bucket以支持来自 faushine.com 以及 www.faushine.com 的请求。首先创建名为 www.faushine.com 的bucket。

 - 进入S3界面，点击CreateBucket，取名为 "www.faushine.com"。
 - 选择一个Region，你可以在[这里](https://s3-accelerate-speedtest.s3-accelerate.amazonaws.com/en/accelerate-speed-comparsion.html)找到延迟最低的Region用于存放你的博客，点击创建。

 ![alt text](/img/in-post/2018-02-19/1.png "创建bucket")

 - 在存储桶策略中设置Policy，确保外部可访问：
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadForGetBucketObjects",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::www.faushine.com/*"
        }
    ]
}
```
> www.faushine.com 是bucket的名字，作为资源标识符

 - 在属性中开启静态网站托管，设置索引文档和错误文档（假如有）。

 ![alt text](/img/in-post/2018-02-19/2.png "开启静态网站托管")

接下来创建名为faushine.com的bucket

 - 进入S3界面，点击CreateBucket，取名为faushine.com。
 - 选择属性
 - 选择静态网站托管
 - 选择重定向请求，键入 www.faushine.com
 - 保存

## 创建用户用于上传静态文件
 - 进入IAM服务->创建策略->JSON
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetBucketLocation"
            ],
            "Resource": "arn:aws:s3:::www.faushine.com"
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject",
                "s3:GetObject",
                "s3:DeleteObject"
            ],
            "Resource": "arn:aws:s3:::www.faushine.com/*"
        }
    ]
}
```
- 创建新组->附加策略（将上文创建的策略添加到该组）->创建完成
- 创建用户->访问类型选择‘编程访问’->将用户添加到刚刚创建的组->创建用户->’下载.cvs‘，将用户的Access key下载到本地

![alt text](/img/in-post/2018-02-19/3.png "添加用户")

## 安装deploy插件

- 运行命令
```
npm install --save hexo-deployer-s3
```
- 编辑站点配置文件`_config.yml`
```
deploy:
  type: s3
  bucket: www.faushine.com
  aws_key: xxxxxx
  aws_secret: xxxxxx
  region: xxxx
  delete_removed: true
```
> delete_removed为true时，部署完成后将从S3删除`_config.yml`文件

- 部署静态文件
```
hexo d -g
```

- 现在就能通过 
http://www.example.com.s3-website-us-east-1.amazonaws.com/ 
或者 http://example.com.s3-website-us-east-1.amazonaws.com/ 访问站点啦。

## 域名解析
S3相当于是一个虚拟的机器，他现在只有AWS内网IP，所以只能通过内部域名解析找到他，有两种方法可以用于域名解析。

- EIP
- Route 53

第一种方式是申请一个EIP，把EIP和S3绑定起来，因为公网上可以用EIP访问S3，所以之后把域名的A记录换成这个IP就好。
第二种方式是申请AWS的DNS服务，再把万网（或者其他域名服务商）的DNS服务器换掉。
本站采用第二种方式解析域名。

需要创建名为 faushine.com 以及 www.faushine.com 的两个Hosted Zone，并分别添加名为 faushine.com 以及 www.faushine.com 的两条A记录。

- 进入DNS Management
- Create Hosted Zone
- Create Record Set 添加A记录，在Alias Target中选择S3资源地址，保存
- 域名服务商的DNS服务器修改为如下图所示的Name Servers

![alt text](/img/in-post/2018-02-19/4.png "Name Servers")
![alt text](/img/in-post/2018-02-19/5.png "DNS修改")
按理来说等一会就能通过你自己的域名访问博客了，但是，但是，偶尔也会出现DNS解析错误的彩蛋，据可靠人士分析是因为用了境外的DNS服务器，所以被GFW拦的不要不要的。这个问题至今没想明白，欢迎指教。

<br>本文完。
<br><br><center>`附彩蛋`</center>
![alt text](/img/in-post/2018-02-19/7.png)
 