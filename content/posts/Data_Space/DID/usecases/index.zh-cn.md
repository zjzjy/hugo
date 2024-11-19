---
title: "Workflows based on usecases"
date: 2024-11-19T09:31:35+08:00
lastmod: 2024-11-19T09:31:35+08:00
categories: ["Data Space"]
summary: "Usecases"
---

## Summary
基于详细用例的数据空间工作流程。
## To be translated

Oh Sorry!

This blog has't been translated to English, please wait for a little while...

## 等待被翻译

非常抱歉，看起来这篇博文还没有被翻译成中文，请等待一段时间

## 概要
参考：[DSBA Technical Convergence v2.0 chapter 6](https://data-spaces-business-alliance.eu/wp-content/uploads/dlm_uploads/Data-Spaces-Business-Alliance-Technical-Convergence-V2.pdf)
本节实现了一个场景，其中数据服务提供者在公共平台上提供服务，以便服务消费方能够获取该服务的访问权限。此外，这些消费方可以将获取的服务访问权限委托给其用户/客户。

本节用例：数据服务提供者是一个包裹递送公司，支持创建和管理包裹递送订单，并且提供查看和修改订单的特定属性的服务；消费方是向其客户提供商品系统的零售商，他们可以通过数据空间市场获得对包裹递送公司服务的访问权，并将访问权委托给客户。

在参考用例中，涉及到多个参与方，每个参与方都有自己的基础设施。具体包括：

1. 数据空间市场： 用于创建服务产品和获取访问权限的公共市场。
2. 信任锚： 担任方案管理员角色，持有有关每个参与方的信息【（包括通用唯一识别码，称为欧盟经济体注册和识别号（Economic Operators Registration and Identification: EORI number）】，并允许它检查每个参与方的准入情况。
3. 包裹递送公司：提供检索和更新包裹交付订单数据的服务。
4. Happy pets: 高级宠物零售商。此外，还涉及两个人类角色：Happy Pets 员工和Happy Pets 客户。
5. No Cheaper： 提供大幅折扣的零售商。此外，还涉及两个人类角色：No Cheaper 员工和 No Cheaper 客户。

总体架构如下：
![image](architecture.png)

+ 包裹递送公司和商店系统的服务提供者各自拥有自己的身份提供者（identity provider: IdP）和授权注册表（authorization registry）。
+ 包裹递送公司托管了一个门户（portal），允许用户查看和修改包裹递送订单的属性。
+ 订单实体（entity）存储在上下文代理（context broker）的实例（instance）中。
+ 对包裹递送订单实体的读写访问（read-and-write access）由策略执行点（policy enforcement point: PEP proxy）代理和策略决定点（policy decision point: PDP）控制，这会在之后的各个参与方详细介绍到。