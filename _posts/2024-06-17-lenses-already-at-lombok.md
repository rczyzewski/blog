---
title: '`Lenses` aka `@WithBy` already in lombok'
subtitle: "unknown lombok feature"
author:
  - rczmdp
categories: [ java, lombok ]
tags: [ lombok, immutability, java ]
description: |
  The lenses pattern is already implemented in lombok! What a great news! 
---

"A rose by any other name would smell as sweet"
William Shakespeare's play Romeo and Juliet

The quote is to ilustrate, that
what [Functional And Reactive Domain Modeling](https://www.manning.com/books/functional-and-reactive-domain-modeling)
describes as a lenses pattern, is available already in lombok, with a different name, and a different implementation,
but serving the same purpose.

The annotation is named `@WithBy`, and is similar to `@With` annotation - with one exception: it works better for
hierarhical immutable data structures.
`@WithBy` is part of the lombok since version `1.18.14`.

I have never seen this feature used: there is no documentation of the feature, no support for it from intellij lombok plugin. 

* The source code : https://github.com/projectlombok/lombok/blob/master/src/core/lombok/experimental/WithBy.java
* Java docs: https://javadoc.io/doc/org.projectlombok/lombok/latest/lombok/experimental/WithBy.html
* No documentation for the @WithBy:  https://projectlombok.org/features/experimental/WithBy
* GitHub issue: https://github.com/projectlombok/lombok/issues/2368
* lombok intellij plugin: https://plugins.jetbrains.com/plugin/6317-lombok/versions
* github repo for intellij plugin https://github.com/mplushnikov/lombok-intellij-plugin


