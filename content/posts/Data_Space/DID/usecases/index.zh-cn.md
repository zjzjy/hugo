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

## 1 概要
参考资料：[DSBA Technical Convergence v2.0 chapter 6](https://data-spaces-business-alliance.eu/wp-content/uploads/dlm_uploads/Data-Spaces-Business-Alliance-Technical-Convergence-V2.pdf)

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

## 2 各部分详解
#### 2.1 包裹递送公司

对应的架构图如下：
![image](delivery.png)
Packet Delivery是一家提供全球包裹递送服务的物流公司。它提供两种包裹递送服务：
- 标准包裹递送服务
    - 客户可以指定包裹的发件人、收件地址、包裹准备取件的日期和时间，以及包裹的收件人姓名和地址。
    - 当包裹递送公司收到客户的包裹递送订单时，它会返回包裹计划送达的目标日期。
    - 在定义的条款和条件下（terms and conditions）【例如：无海关违规，地址有效等】，它承诺在同一国家内48小时内交付包裹，国际运输则需5-6天。
    - 然而，客户不能调整具体的送达日期（例如，将其延迟到更适合的日期）或在选定的送达日期内微调具体的送达时间。
- 金牌包裹递送服务
    - 客户享受标准包裹递送的所有好处，同时还可以在提供的时间段内调整具体的送达地址、送达日期，以及在选定的送达日期内微调具体的送达时间，前提是这些调整是可行的（例如，提前足够时间提出请求，并且不会产生额外费用）。


包裹递送公司以电子方式向不同的零售商提供服务，让他们通过REST API访问其包裹交付信息系统(P.Info)来发布包裹递送订单、追踪订单位置、允许被他们授权的顾客执行调整地址、日期和计划交付时间的请求。

###### 2.1.1 DELIVERYORDER
P.Info通过上下文代理（context broker）提供的NGSI-LD接口实现的。DELIVERYORDER是一个具有以下属性的实体：
- issuer
- pickingAddress（取件地址）
- pickingDate（取件日期）
- pickingTime（取件时间）
- destinee（收件人）
- deliveryAddress（送达地址）
- PDA（planned date of arrival，计划送达日期）
- PTA（planned time of arrival，计划送达时间）
- EDA（expected date of arrival，预期送达日期）
- ETA（expected time of arrival，预期送达时间）
  
Packet Delivery为其P.Info系统定义了两个角色：P.Info.standard和P.Info.gold，并根据这些角色定义了通过上下文代理服务可以对上述属性执行的操作。

为了简化场景描述，我们将关注送达地址、计划送达日期和计划送达时间这三个属性，因为我们可以假设其他属性在订单创建时将被分配值，始终可读，但用户无法通过已定义的角色进行更改。对于这三个属性，一旦订单创建后，以下策略适用于已定义的角色：
| Path      | Verb | Verb    |
|/ngsild/v1/entities/{entityID}/attrs/{attrName} | GET|PATCH|
| :-------        |    :----:   |          ---: |
|   deliveryAddress   | P.Info.standard/gold      | P.Info.gold   |
| EDA   | P.Info.standard/gold       | ---     |
| ETA   | P.Info.standard/gold       | ---     |
| PDA   | P.Info.standard/gold       | P.Info.gold     |
| PTA   | P.Info.standard/gold       | P.Info.gold     |

注意，订单将使用不同的路径（/ngsi-ld/v1/entities/）通过POST创建。为了发出此类请求，定义了一个额外的角色P.Create，该角色仅分配给Happy Pets和No Cheaper。

包裹递送公司针对潜在零售商和其他类型公司发布两种产品：
- 基础（basic）递送：允许获取该服务的公司向其客户提供标准包裹递送服务。
- 高级（premium）递送：允许获取该服务的公司向其客户提供标准和金牌包裹递送服务。

需要注意的是，Packet Delivery**不应了解任何零售商应用程序（application）用户的身份**。它只需要能够在接收到请求时：

a）**识别**该请求来自于与获取其平台产品（offering）的零售商公司相关联的用户

b）**确定**用户在零售商公司为其分配的P.Info应用程序中的角色（role，即P.Info.standard或P.Info.gold）

c）**检查**该角色是否是该零售商公司可以分配的角色，考虑到它在平台上获得的产品（offering）

在完成这些步骤后，Packet Delivery将只需检查拥有该角色的用户是否可以执行所请求的操作。

由No Cheaper创建的应用程序，无论其是否为用户分配了P.Info.gold角色，都无法成功更改某个订单的PTA属性的值，因为它已获取的标准包裹递送服务不允许更改这些值。（因为No Cheaper是廉价型公司，所以没有分配P.Info.gold角色的权限）。　

#### 2.2 数据服务消费者：HappyPets Inc.
![image](pets.png)
HappyPets Inc.（简称HappyPets）是一家销售宠物用品的公司。它将在市场上获得“高级递送”服务。这将使其能够通过HappyPets的商店应用程序向客户提供标准和金牌配送服务。此外，公司内部可能有某些员工，即公司提供的电话帮助台服务的主管和代理，他们可能会使用内部应用程序（HappyPetsBackOffice）更改给定订单的送货地址、PDA和PTA。

当客户在HappyPetsStore注册时，它可以作为“普通”客户或“主要”客户（支付年费）。“普通”客户将获得标准包裹递送服务，而“优质”客户将获得金牌包裹递送服务。这意味着他们分别在HappyPetsStore应用程序中被分配了P.Info.standard角色和P.Info.gold角色。

另一方面，不同的员工在HappyPetsBackOffice应用程序中被分配了不同的角色，因此在实体店（physical shop）担任监督员（supervisor）角色的员工或在中央帮助台工作的代理也被分配了P.Info.gold角色。

Happy Pets员工：
- 在平台上获取高级包裹递送产品。
Happy Pets客户：
- 在Happy Pets商店系统中注册并被分配为优质客户角色
    - 为简化，假设已经有一个Happy Pets客户已经注册为优质客户
- 在商店系统下单，其本质是在P.Info系统中创建了一个包裹递送订单
    - 为简化，假设已经为该客户在P.Info系统中创建了一个订单
- 成功通过Packet Delivery门户（portal）更改订单的PTA
    - 我们将在4. Detailed Workflow描述执行此操作的详细流程。

### 2.3 数据服务消费者：No Cheaper Ltd.
![image](nocheaper.png)
No Cheaper是一家以较大折扣销售各种产品的公司。它将在平台上获取Packet Delivery公司的基础包裹递送产品。

No Cheaper 员工：
- 在平台上获取基础包裹递送产品
No Cheaper 客户：
- 在No Cheaper商店系统中注册并被分配为普通客户角色
    - 为简化，假设已经有一个No Cheaper客户已经注册为普通客户
- 在商店系统下单，其本质是在P.Info系统中创建了一个包裹递送订单
    - 为简化，假设已经为该客户在P.Info系统中创建了一个订单
- 当尝试通过Packet Delivery门户（portal）更改订单的PTA时，被拒绝
- 还可以展示，当No Cheaper员工在其自己的身份提供者系统中为No Cheaper客户分配优质客户角色时，该请求也将被拒绝

### 2.4 数据市场平台
市场是基于[FIWARE BAE(Business Application Ecosystem) component](https://business-api-ecosystem.readthedocs.io/en/latest/)
组件构建的，该组件由FIWARE 业务框架和[TMForum](https://www.tmforum.org/)提供的一组API组成。它允许在整个服务生命周期（life cycle）内对不同类型的资产进行货币化（monetization），从创建产品（offering creation）到收费（charging）、记账（accounting）和涉及参与方的收入结算（revenue settlement required for billing and payment）。
![image](FIWARE.png)
创建offer时的包裹递送订单参数，以及市场在获取和激活阶段执行的必要步骤的实现，都由安装在Charging Backend组件中的专用插件提供。

您可以在这里找到[Marketplace UI](https://github.com/i4Trust/bae-i4trust-theme)的专用主题。
### 2.5 信任锚