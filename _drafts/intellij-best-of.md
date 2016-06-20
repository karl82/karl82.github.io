---
title: IntelliJ best of tricks
layout: post
category: java
tags: [java, ide, intellij]
---

Several tips for using (IntelliJ IDEA)[].

# Plugins

# How to

## Skipping integration tests

If you have your integration tests named `*IntegrationTest` then they are
executed with your regular unit tests. Which takes ages and maybe you even do
not have environment prepared for your integrtion tests.

You can set regex [pattern for unit tests
configuratio](https://www.jetbrains.com/help/idea/2016.1/run-debug-configuration-junit.html#configTab).
How should such regex look like?

```regex
(.(?!Integration))*Test
```
