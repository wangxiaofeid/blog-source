---
title: postman使用技巧
date: 2018-10-11 14:57:51
tags: 工具
---

## 问题1：请求的时候如何携带cookie（不想拷贝）
1. 安装chrome插件--Postman Interceptor
2. 在postman里面打开Postman Interceptor选项
只有在浏览器上登录了，发送请求会自动携带上浏览器下该域名下cookie。（仅在chrome store下载的postman支持）
[Postman](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop/related)
[Postman Interceptor](https://chrome.google.com/webstore/detail/postman-interceptor/aicmkgpgakddgnaphhhpliifpcfhicfo?hl=zh-CN)

## 问题2：如果后端有XSRF-TOKEN安全机制访问还是不通
* 解决方式1：手动从拷贝到header里面
* 解决方式2：[地址](https://blog.csdn.net/lishanleilixin/article/details/82153350)
方法中提到的pm只有在postman app中可使用，可是app中Interceptor不能使用，希望后续能增加[issues](https://github.com/postmanlabs/postman-app-support/issues/1667)