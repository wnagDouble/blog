---
title: "Github Pages 评论系统 Giscus"
date: 2023-12-24T14:30:08+08:00
categories: []
draft: false
---
## Github配置  
登录[github网站](https://github.com/)，找到自己要集成giscus的仓库。点击`Settings`，找到`Discussions `。勾选复选框。  
具体操作可参考[官方文档](https://docs.github.com/zh/discussions/quickstart)。  
## Giscus配置
打开[giscus](https://giscus.app/zh-CN)网站。在仓库输入框中，输入指定仓库的url地址，如下图。  
![giscus](../../images/20231224144225.png)  
如果Github仓库已配置好，会提示`该仓库满足所有条件`。往下滚动,找到`启用 giscus`，会自动生成`script`代码。
```javascript
<script src="https://giscus.app/client.js"
        data-repo="owner/repo_name"
        data-repo-id="R_key"
        data-category="category_name"
        data-category-id="DIC_key"
        data-mapping="og:title"
        data-strict="0"
        data-reactions-enabled="1"
        data-emit-metadata="0"
        data-input-position="bottom"
        data-theme="preferred_color_scheme"
        data-lang="zh-CN"
        crossorigin="anonymous"
        async>
</script>
```  
将`script`添加到你的网站中即可。  
## 番外  
一开始没发现[giscus网站](https://giscus.app/zh-CN)会自动帮忙生成好`script`代码，所以`repo-id`和`category-id`是用
[Github Explorer](https://docs.github.com/zh/graphql/overview/explorer)手动查询的。具体可看[隔壁](/post/github_explorer)。