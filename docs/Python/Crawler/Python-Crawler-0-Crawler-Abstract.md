---
title: Python【Crawler】爬虫总叙
tags:
  - Python
  - 爬虫
categories:
  - Python
  - 爬虫
  - 基础
thumbnail: 'https://xcdn.loli.top/gh/TCP404/Picgo/blog/thumbnail/python.png'
abbrlink: 28223
date: 2020-08-20 12:17:48
---

工具人，啊不，工具蜘蛛。

<!--more-->

## 使用场景
- **通用爬虫**：抓取系统重要组成部分，抓取的是**一整张页面数据**。
- **聚焦爬虫**：建立在通用爬虫之上，抓取的是页面中**特定的局部内容**。
- **增量爬虫**：检测网站中数据更新的情况，只会抓取网站中**最新的数据**。

## 矛与盾
- 反爬机制：门户网站，可以通过指定相应的策略或技术手段，防止爬虫程序进行爬取
    - 检查User-Agent
    - IP屏蔽
- 抗反爬策略：通过制定相关策略或技术手段破解门户网站的反爬机制。
    - UA伪装
    - IP代理
- robots.txt 协议 —— 君子协议：规定了网站中哪些数据可以被爬虫爬取，哪些数据不可以被爬取

- http
  - 概念：服务器与客户端数据交互的一种形式
  - 常用请求头信息：
    - User-Agent：请求载体的身份载体
    - Connection：请求完毕后，是否断开连接
  - 常用响应头信息：
    - Content-Type：服务器响应回客户端的数据类型

- https
    安全的http协议