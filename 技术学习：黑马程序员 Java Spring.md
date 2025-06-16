---
title: 技术学习：黑马程序员 Java Spring
date: 2022-05-02
tags: 
- Java
- Spring
categories:
- 算道求索（课外IT技能学习）
- Lecture
description: 黑马程序员Java Spring入门课程学习笔记
---

## 介绍

Spring：分层的 Java SE/EE 全栈轻量级开源框架，以 控制反转（Inverse of Control IOC）和面向切面编程（Aspect Orient Programming AOP）为内核。

提供了展现层 SpringMVC 和持久层 Spring JDBCTemplate 以及业务层事务管理等众多企业级应用技术，是使用最多的 Java EE 企业应用开源框架。

优势：

- 方便解耦，简化开发
  通过 Spring提供的loC容器，可以将对象间的依赖关系交由Spring 进行控制，避免硬编码所造成的过度耦合。
  用户也不必再为单例模式类、属性文件解析等这些很底层的需求编写代码，可以更专注于上层的应用。
- AOP编程的支持
  通过 Spring的AOP功能，方便进行面向切面编程，许多不容易用传统OOP实现的功能可以通过AOP轻松实现。
- 声明式事务的支持，让事务管理更简单。
- 方便调试。
- 方便集成框架。
- 降低 Java EE API 使用难度（如 JDBC，远程调用等）。
- 其源码是经典学习范例。

![ ](https://raw.githubusercontent.com/Jingqing3948/FigureBed/main/mdImages/202506111711463.png)