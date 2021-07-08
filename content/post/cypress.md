---
title: "Cypress 实践总结"
date: 2021-07-08T09:52:31+08:00
tags: ["cypress"]
categories: ["DevOps","总结"]
toc: false
---



# 前言

最近接手了一个前端的项目，这个项目使用宜搭低代码平台进行搭建，由于每次修改代码或者调整组件时都有可能造成其他问题，影响整体的项目质量，然而又不想每次都进行手动的“点点点”操作，于是就打算研究一下如今流行的前端自动化框架 cypress。



### 下载

使用 npm 进行下载并运行

```bash
mkidr automation-test
cd automation-test
npm init 
...
npm install cypress --save-dev
./node_modules/.bin/cypress open
```



## 脚本编写

使用 vscode 打开刚才创建的 `automation-test` 文件夹，然后在 integration 文件夹下新建文件夹名称为 `coupon-ui` 并在此文件夹下新建 demo.spec.js 文件。



### 基本语法

```
describe('场景',function(){
    it('场景明细1',function (){
        expect(true).to.equal(true)
    })
    it('场景明细2',function (){
        expect(true).to.equal(true)
    })
})
```

例如

```
describe('登录',function(){
    it('打开页面',function (){
        // 逻辑
    })
    it('输入信息',function (){
        // 逻辑
    })
    it('登录',function (){
        // 逻辑
    })
    it('首页加载',function (){
        // 逻辑
    })
})
```

#### 方法举例

查询

用法和 jQuery 的 css 选择器类似

###### cy.get()

```js
cy.get('#query-btn').should('contain', 'Button') 
```

###### cy.contains()

```js
cy.get('.query-list')
  .contains('bananas').should('have.class', 'third')
```



更多的方法可以访问 https://example.cypress.io/ ，非常全面，而且简单易懂，在此不做赘述。

下面放一个简单的效果

<img src="https://ahian-blog.oss-cn-beijing.aliyuncs.com/images/2021-07-08-080013.gif" alt="Jietu20210708-155918" style="zoom:200%;" />



