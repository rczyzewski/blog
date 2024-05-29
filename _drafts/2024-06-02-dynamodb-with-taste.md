---
title: 'Accessing AWS dynamoDB with java with a style'
subtitle: "Object Document Maper with type safety for queries and conditions."
author:
  - rczmdp
categories: [ java, dynamoDB, AWS ]
tags: [ productivity ]
description: |
  The concept of Object Relational Mapping(ORM) is well known. The analogous concepts, Object Document Mapper,  
  could be applicable for non relational databases like AWS DynamoDB. 
---

## Intro

Working with databases might be easy or hard: depending on data structures we are storing, and how our code around it is
organized. Those data structures are changing, the names of properties are changing, then the names in mapper are not
the names from conditions and code will become complex, on its own.

I love dynamo, it's very powerful, in my previous project this key-value databaase was a perfect fit. Amazon sdk 2.0
provide like a low level interace to it. But in the world of fast changing requirements, I want to keep in my `.java`
files things related to business logic and limit the boilerplate to the minimum. 


