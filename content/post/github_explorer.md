---
title: "Github GraphQL API"
date: 2023-12-24T15:00:53+08:00
categories: []
draft: false
---
# Github GraphQL API  
Github GraphQL API是Github提供的一种API接口。它可以让你更方便的获取Github上的数据。
请求该API最直观最简单的方式就是使用[Github GraphQL API资源管理器](https://docs.github.com/zh/graphql/overview/explorer)。  

比如，假如你想知道你的仓库ID和Discussions分类ID。就可以使用Github GraphQL API查询。   
假设你的仓库url地址是`https://github.com/owner/repo_name` 在Github GraphQL API资源管理器中输入如下查询语句。  
```GraphQL
{
  repository(owner: "owner", name: "repo_name") {
    id
    name
    discussionCategories(first:1) {
      nodes{
        id
        name
      }
      pageInfo{
        hasNextPage
        hasPreviousPage
      }
      totalCount
    }
  }
}
```  
就会得到如下结果。  
```GraphQL
{
  "data": {
    "repository": {
      "id": "R_key",
      "name": "repo_name",
      "discussionCategories": {
        "nodes": [
          {
            "id": "DIC_key",
            "name": "Announcements"
          }
        ],
        "pageInfo": {
          "hasNextPage": true,
          "hasPreviousPage": false
        },
        "totalCount": 6
      }
    }
  }
}

```
更多其他信息查询及资源管理器的使用可参考[官方文档](https://docs.github.com/zh/graphql/reference/queries)。  
## 番外
GraphQL是一种用于API的查询语言，它提供了一种更高效、强大和灵活的替代方案，相比于传统的RESTful架构。
GraphQL不仅允许客户端精确地指定它们需要的数据，还使得客户端能够聚合多个API调用到一个单一的GraphQL请求。  
[GraphQL官网](https://graphql.org/)  
[GraphQL中文网](https://graphql.cn/)