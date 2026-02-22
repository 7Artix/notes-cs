Tag动态管理系统. 用于方便地筛选, 管理各类内容. 是一套去中心化, 自动检索, 引用计数的思路, 满足动态生长需求.

需要自动生成(添加), 唯一性校验, 动态清理, 需要一套预处理机制.

# 核心逻辑

## Tag的定义

不应该预先定义任何Tag, 而是直接在Project或Post的YAML中直接写:

```yaml
title: "7Artix Site"
direction: "Web"      # 必填项: 约定某些字段
tags: ["Vue", "GSAP"] # 可选项: 自由发挥
```


## 自动化索引器

设计一个Node.js脚本, 在服务器启动或构建时运行:

1. 遍历: 递归扫描所有 `config.yaml` .
2. 提取: 收集所有符合要求的字段.
3. 生成ID (Hashing).
4. 计数与清理.
5. 生成索引.


# 分离式管理

页面本身通过Vue设计, 提升页面设计的自由度.

Tag系统通过管理员面板完成, 设计管理页面, 通过自动化脚本, 对YAML进行修改.


# 数据与程序分离

将数据与程序完全分离, 将项目抽象出模板, 便于迁移和复用.

将数据抽离, 保存在 `./data/` 目录.

新建 `./server/` 目录, 用于存放后端程序. Vue本身没有访问磁盘的权限, 通过Node.js脚本处理, 并与Vue建立数据连接. 衔接网页和硬盘数据.

Vue项目在部署之后, 浏览器里没有 `/data/` 目录, 因此无法访问, 必须转换成网络请求.

为Node.js运行安装包

```shell
npm install express cors js-yaml
```

`express` web框架, 相当于轻量级HTTP库, 可以处理HTTP请求.
`cors` Cross-Origin Resource Sharing, 解决跨域安全限制, 允许服务跨端口访问数据.
`js-yaml` YAML解析器

