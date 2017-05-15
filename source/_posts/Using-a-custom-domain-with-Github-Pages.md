---
title: 为你的 GitHub Pages 设置个性化域名
date: 2016-02-27 08:24:54
categories: GitHub
tags: 
---

上一篇文章[利用 Hexo 搭建个人博客](http://yuzhiw.com/2016/02/25/built-your-blog-by-hexo/)介绍了如何在 GitHub Pages 上利用 Hexo 搭建个人博客，本文主要介绍如何为你的博客设置个性化域名

# 购买域名

推荐 [Godaddy](https://www.godaddy.com/) 上购买，安全放心，而且支持支付宝付款
目前官网支持中文，所以以下教程只对需要特别注意的地方进行详细说明。
![Godaddy](http://7xrac3.com1.z0.glb.clouddn.com/godaddy.png)

1.搜索你想要的域名
2.选择你的域名后，点击购物车进入到付款页面，注意：会外加 ICANN 管理费 ¥1.14/年
3.付款界面选择年限，同时支持优惠码，网上可以搜到很多，随便搜一个填进去
4.然后去结算，填完必要的信息后，选择支付宝付款
5.注意事项：
- 注册信息的邮箱要填正确，否则修改很麻烦
- 购买完以后记得去邮箱激活域名

# 将域名与GitHub Pages绑定
## GitHub Pages 设置
在你该 Repository 的根目录下新建名为 CNAMA 的文本文件，内容为你的域名地址，如 yuzhiw.com
## DNS设置
推荐 [DNSPod](https://www.dnspod.cn)，快速，稳定，免费
注册或者登录 DNSPod，添加域名解析，参考下图
![DNSPod](http://7xrac3.com1.z0.glb.clouddn.com/DNSPOD.png)
两条A记录指向的是 GitHub Pages 指定的地址，参考[Customizing GitHub Pages ](https://help.github.com/articles/troubleshooting-custom-domains/)
## 去 Godaddy 修改 DNS
登录账户，点击域名->管理 DNS ->设置，将域名服务器修改为 f1g1ns1.dnspod.net 和 f1g1ns2.dnspod.net 
![DNS setting](http://7xrac3.com1.z0.glb.clouddn.com/dns-setting.png)
修改保存完以后，等待 DNS 服务器刷新，一般约十分钟就可以访问了

# 参考连接
- [Godaddy](https://www.godaddy.com/)
- [Customizing GitHub Pages ](https://help.github.com/articles/troubleshooting-custom-domains/)
- [Godaddy 注册商域名修改 DNS 地址](https://support.dnspod.cn/Kb/showarticle/tsid/42/)
- [如何搭建一个独立博客——简明 GitHub Pages 与 Hexo 教程](http://blog.csdn.net/poem_of_sunshine/article/details/29369785)



