DDDLIB-DOMAIN
======

[dddlib-domain模块是DDDLib的核心模块。这是用户项目中所必须的唯一编译时dddlib依赖（由于dddlib-domain也依赖于dddlib-utils，用户项目对dddlib-utils有传递性依赖）。IoC模块和持久化模块只是在集成时选择部署，用户项目不直接依赖于这些模块。

## 目标
dddlib-domain的主要目标是为基于领域驱动设计（Domain Driven Design, 简称DDD）范式的开发提供支持。在分层企业应用架构（表示层、应用层、领域层、基础设施层，其中应用层和领域层往往合称业务逻辑层）中，dddlib-domain直接为领域层服务，为领域对象提供统一的接口和基类，并通过依赖倒置（DI）的方式隔离领域层对基础设施层的IoC容器和持久化技术的依赖。

## API概述
dddlib-domain的API大概可分为三大类：

### 实体接口和基类

实体相关的接口和基类包括：

* [Domain](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/Entity.html)接口：代表DDD中的“实体”概念，等价于JPA中的Entity。实体是领域模型中的最重要的组成部分，对应到业务领域中的业务实体，例如员工、机构、岗位、账户、商品、财产，等等。所有的实体都必须实现Entity接口。同一种类型的实体通过唯一性的id属性相互区分。

* [BaseEntity](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/BaseEntity.html)抽象类：BaseEntity为实体类提供一个抽象基类，它提供了持久化的能力，使得实体能够通过仓储（[EntityRepository](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/EntityRepository.html)）接口持久化到数据库中。

* [AbstractEntity](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/AbstractEntity.html)抽象类：AbstractEntity抽象类继承[BaseEntity](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/BaseEntity.html)抽象类，为实体类提供一个更具体的、功能更多的基类。具体而言，它将id属性的数据类型限定为Long类型，提供version属性以支持持久化框架进行乐观锁并发控制，并提供了将自身保存到仓储，从仓储中删除自身的实例方法，以及根据id获取实体、查找本实体类型的所有实例、根据一个或多个属性查找实体等等静态方法。

* [ValueObject](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/ValueObject.html)接口：代表DDD中的“值对象”概念，等价于JPA中的Embeddable。值对象于实体的根本区别是：实体具有自己的标识（ID），可以独立存在，有自己独立的生命周期；而值对象没有自己的标识，不能独立存在，它是实体的一个组成部分，其生命周期依附于其所属的实体。

之所以区分BaseEntity和AbstractEntity两个实体基类，是为了满足软件开发中的两种不同的需求。如果严格遵循DDD的范式，那么只有聚合根实体才能够更改聚合内的实体的持久化状态，并不是每一个实体都能够自由save()和remove()自身的，否则可能打破聚合的规则不变性。在这种情况下，最好是实体类都从BaseEntity继承，由开发者精细定义实体类的持久化行为。

同时为了方便起见，也有很多开发者愿意在实体基类上实现很多公共行为，以最小化具体实体子类需要编写的方法接口的数量。他们或者没有聚合的概念，或者不忌讳打破聚合业务规则不变性。这种情况下，让实体类从AbstractEntity继承是个好的起点。

### 持久化接口和类

* [EntityRepository](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/EntityRepository.html)：这是持久化的核心接口，代表DDD中的“仓储”概念。实体以及从属于实体上的值对象通过仓储接口持久化到仓储（一般是数据库）里面，通过仓储接口可以以各种各样的方式查找实体。仓储接口功能上约等于JPA的EntityManager和Hibernate的Session。dddlib-persistence-jpa和dddlib-persistence-hibernate两个模块分别为该仓储接口提供了不同的实现。

dddlib支持四种查询方式：条件查询、命名查询、JPQL查询和原生SQL查询。这些查询分别由下面的类和接口支持：

* [CriteriaQuery](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/CriteriaQuery.html)：条件查询。条件查询是通过面向对象而不是面向查询语言的方式进行查询。为了构造条件查询，我们提供了下面的接口和类：用[QueryCriterion](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/QueryCriterion.html)接口代表单个的查询条件（internal子包中有它的各种各样的实现类，分别代表不同的查询条件），用[CriterionBuilder](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/CriterionBuilder.html)作为QueryCriterion的工厂来生成各种查询条件，用[OrderSetting](http://www.dayatang.org/dddlib/apidocs/org/dayatang/domain/OrderSetting.html)代表单个的排序字段。