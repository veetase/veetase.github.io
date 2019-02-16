---
layout: "post"
title: “OTP in Elixir”
date: "2018-06-03 15:04"
---

### 什么是OTP
OTP最早是爱立信公司于1988年开源的用于用Erlang编写分布式、高容错的应用程序，很长一段时间里OTP就是Erlang专属的，WikiPedia也是这么描述的：
> OTP is a collection of useful middlewares, libraries, and tools written is Erlang programming language. —WikiPedia
Elixir的诞生让OTP不在是Erlang的专属，虽然Elixir也是会被编译称Erlang字节码执行，但显然上面的描述已经不太规范。就连Erlang的作者对OTP的描述也特意去掉了Erlang的限制：
> OTP is an application operating system and a set of libraries and procedures used for building large-scale, fault-tolerant, distributed applications. — Joe ArmStrong

### 为什么需要OTP
想要弄清楚为什么需要OTP就要从Erlang本身开始说起。爱立信公司开发Erlang的目的是为了能用它来开发一个高容错、分布式、易拓展的应用程序来满足电信业务。可以说，Erlang就是为分布式而生。支持其分布式特性的基石是Erlang Virtual Machine，Virtual Machine提供了不同于操作系统进程的另一种更加轻量的Process，Process有以下特性：
- Process之间互相独立互不影响
- Process之间以MailBox的形式进行通信（下文会详细讲）
- 由Virtual Machine自动调度
- 轻量，一个新的Process只有约1～2kb大小，一台普通的电脑可以轻易同时跑几十万甚至上百万个Process
- 在Erlang中Process是一等公民，操作Process就像在面向对象语言中操作一个类一样简单。

**OTP就是管理Process的工具**





