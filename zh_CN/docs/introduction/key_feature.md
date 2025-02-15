# 2. 关键特性

标签：``关键特性``

从功能组件上来看，WeDPR隐私计算系统主要包括：隐私计算核心算法实现组件、统一网关、隐私计算管理平台。

隐私计算核心算法组件是隐私计算任务的核心实现；隐私计算网关负责所有隐私计算节点的通信、机构内的服务注册和服务发现等；隐私计算管理平台实现了跨机构元数据信息同步、细粒度的用户和权限管理体系以及丰富的异构数据源的接入等。

## 2.1 隐私计算任务

**丰富的隐私计算任务支持**

- **隐私求交集**: 包括两方隐私求交集任务和多方隐私求交集任务，并从性能、网络带宽、使用场景等多方面考虑，实现了多种隐私求交集算法，包括CM2020(性能高), RA2018(非平衡PSI算法，适用于CS模式), ECDH-PSI;
- **匿踪查询**: 基于OT算法构建匿踪查询，可将数据集发布为匿踪查询服务开放给相关用户使用;
- **联合建模**: 基于SecureLGB和SecureLR算法支持多方数据联合建模，并可将建模结果发布为模型用于预测，满足了80%多方数据联合建模需求;
- **联合分析**: 基于安全多方计算算法，提供了类SQL/Python的隐私数据联合分析语法，可在不引入额外学习成本的前提下，满足数据开发人员基于多方数据进行联合分析的需求;


**专家模式**

- 基于Jupyter构建了专家模式，并提供了联合建模Python toolkit: wedpr-ml-toolkit来支持数据开发人员在Jupyter中发起各类隐私计算任务，可在不升级隐私计算系统组件的前提下，实现数据开发人员各种定制化的数据统计、评估需求，提升了隐私计算系统的功能可扩展性
- 实现了用户维度的Jupyter管理，多用户的Jupyter环境通过linux用户体系完全隔离开


## 2.2 隐私计算网关

**统一网关**

- 支持基于最短路径的消息路由转发
- 支持按节点ID、服务名、机构名进行路由寻址
- 支持服务注册和服务发现

**统一网关SDK**

- 提供Java/Python网关SDK，支持接入网关与其他节点、服务或者机构进行通信
- 可向网关注册服务
- 可从网关拉取服务信息


## 2.3 隐私计算管理功能

**灵活可信的数据同步服务**

- 基于区块链进行跨机构元数据信息同步

**丰富的数据源管理**

- 支持HDFS, Hive, MYSQL和国产数据库等多种数据源接入

**支持DAG的任务调度模块**

- 任务调度模块支持DAG工作流
- 各类隐私计算节点多活可扩展


**细粒度的用户和权限体系**

- 支持多用户模式，并支持用户维度的数据、服务权限管理
- 实现了通用的审批模块，支持数据、服务授权

**服务发布功能**

- 数据集可发布为匿踪查询服务，供授权的用户或机构查询
- 联合建模训练过程中产生的模型可发布为服务，供联合预测使用

**API接入**

- 支持应用方通过申请的凭证AccessKey接入到管理平台

