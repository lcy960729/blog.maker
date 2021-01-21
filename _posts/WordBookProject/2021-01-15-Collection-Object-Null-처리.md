---
title: 'Collection Object Null 처리'
tags: WordBookProject
categories: wordbook-project
author: CY
key: Collection Object Null 처리
---
# 2021-01-15 :: Collection Object Null 처리

Collection은 맵핑할때나 객체를 받아올때나 null 때문에 문제가 많았다. (내가 잘 처리한 방법을 모르는건가??)

그래서 일단은 내 프로젝트에서 약속으로 Entity에서 사용되는 Collection객체는 final로 정의하고

DTO에서는 final로 정의하지않고 맵핑할때 새로 만든 Collection 객체로 set하도록 하였다.

null로 인해 Business로직에서 에러가 나지 않도록 좋은 방법을 생각해봐야겠다.