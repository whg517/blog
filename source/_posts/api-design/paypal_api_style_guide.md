---
title: PayPal API 设计原则
author: kevin
date: 2020-09-01 10:07:00
updated: 2020-09-01 10:07:00
tags:
- api-design
---

> 本文是 [API Design Guidelines](https://github.com/paypal/api-standards/blob/master/api-style-guide.md)
> 的翻译，仅供学习交流。如有翻译不当，请斧正。所有权归原作者，侵删。

## 一、介绍

PayPal 平台是可重用服务的集合，这些服务封装了明确定义的业务功能。鼓励开发人员通过应用程序编程接口（API）访问这些功能，以实现一致的设计模式和原理。通过将多个互补功能组合为构建块，以提供出色的开发人员体验以及快速组成复杂业务流程的能力。

PayPal API 尽可能地遵循 [RESTful](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) 架构风格。为了达到此目标，我们开发了一组适用于 RESTful API 设计的规则、标准和约定。以用来帮助设计和维护数百个 API ，并经过几年的发展以满足各种用例的需求。

我们分享这些指导方针是为了帮助传播良好的 API 设计实践。我们已经从更广泛的社区中广泛地吸取了经验，并尽力回馈。该文档尽可能通用，以便于和您在项目中正在使用的原则合并。如果您有任何更新、建议或您想贡献的补充，请随时提交PR或创建一个ISSUE 。

<!-- more -->

### 1. 文档语义，格式和命名

本文档中的关键字“必须(MUST)”，“不得(MUST NOT)”，“必需(REQUIRED)”，“应该(SHALL)”，“不应该(SHALL NOT)”，“应该(SHOULD)”，“不应该(SHOULD NOT)”，“推荐(RECOMMENDED)”，“可能(MAY)”和“可选(OPTIONAL)”按照[RFC 2119中的](https://www.ietf.org/rfc/rfc2119.txt)描述进行解释。

单词“ REST”和“ RESTful”必须按此处所示编写，代表所有大写字母的首字母缩写。“ JSON”，“ XML”和其他首字母缩写也是如此。

机器可读的文本（例如URL，HTTP动词和源代码）以固定宽度的字体表示。

包含可变块的URI是根据[URI模板RFC 6570](https://tools.ietf.org/html/rfc6570)指定的。例如，包含名为 account_id 的变量的URL将显示为https://foo.com/accounts/\{account_id\}/。

HTTP标头以 camelCase + 连字符的语法编写，例如 Foo-Request-Id 。

### 2. 贡献者

[Sanjay Dalal](https://www.linkedin.com/in/sanjaydalal)（前成员：PayPal API平台），[Jason Harmon](https://es.linkedin.com/in/jasonhnaustin)（前成员：PayPal API平台），[Erik Hogan](https://www.linkedin.com/in/erik-hogan-81431)（PayPal API平台），[Jayadeba Jena](https://www.linkedin.com/in/jayadeba-jena-1a6a0020)（PayPal API平台），[Nikhil Kolekar](https://www.linkedin.com/in/nikhil-kolekar-28627a2/)（PayPal API平台），[Gagan Maheshwari](https://www.linkedin.com/in/gaganmaheshwari)（前者）成员：PayPal API平台），[Michael McKenna](http://linkedin.com/in/mgmckenna)（PayPal全球化），[George Petkov](https://www.linkedin.com/in/gbpetkov)（前成员：PayPal API平台）和Andrew Todd（PayPal Credit）。



<h2>目录</h2>

[TOC]

## 二、指南解释

### 1. 关键术语

#### 1.1 资源

REST中信息的关键抽象是一种资源。根据[Fielding的论文5.2节](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2)，可以命名的任何信息都可以是资源：文档或图像，临时服务（例如“洛杉矶今天的天气”），其他资源的集合，非虚拟对象（例如一个人），等等。资源是到一组实体的概念映射，而不是在任何特定时间点对应于该映射的实体。更准确地说，资源R是随时间变化的隶属函数 `MR(t)` ，用于将时间 `t` 映射到等效的一组实体或值。集合中的值可以是资源表示和/或资源标识符。

资源也可以映射到空集，从而允许在存在任何概念之前就对该概念进行引用。

#### 1.2 资源标识符

REST 使用资源标识符来标识组件之间交互中涉及的特定资源实例。命名机构（例如提供 API 的组织）分配了资源标识符以使其有可能引用资源，它负责维护映射随时间的语义有效性（确保成员资格函数不变）。—— [菲尔丁的论文第5.2节](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2)

#### 1.3 表示

REST 组件通过使用表示形式来捕获资源的当前状态或预期状态并在组件之间传输该表示形式，从而对资源执行操作。一个表示形式是一个字节序列，外加描述这些字节的表示形式元数据 —— [Fielding论文第5.2节](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2)。

#### 1.4 域

根据 Wikipedia 所说，[领域模型](https://en.wikipedia.org/wiki/Domain_model)是一个抽象系统，描述了知识，影响或活动领域的选定方面。这些概念包括业务中涉及的数据，以及业务针对该数据使用的规则。例如，PayPal 域模型包括诸如支付，风险，合规性，身份，客户支持等域。

#### 1.5 功能

能力代表组织业务逻辑的面向业务和面向客户的视图。功能可用于组织 API 组合，作为其系统的稳定，业务驱动的视图，可由客户和经验消耗。能力的示例包括：合规性，信用，身份，零售和风险等。

功能驱动界面，而域则更粗糙，并且更接近代码和组织的结构。从服务的角度来看，能力和领域被视为正交的问题。

#### 1.6 命名空间

功能推动了 API 产品组合中的服务建模和名称空间问题。命名空间是业务能力模型的一部分。名称空间的示例包括：合规性，设备，转移，信用，限制等。

命名空间应反映逻辑上将一组业务功能分组的域。域定义应反映客户对平台功能组织方式的看法。请注意，这些可能不一定反映公司的层次结构，组织或（现有）代码结构。在某些情况下，从定义上可以反映目标，面向客户的平台组织模型的意义上说，领域定义是理想的。底层服务实现和组织结构可能需要迁移以反映这些边界。

#### 1.7 服务

服务提供了用于访问和操纵[资源](#resource)值集的通用 API  ，无论成员资格函数的定义方式或处理请求的软件类型如何。服务是可以执行许多功能的通用软件。因此，考虑存在的不同类型的服务具有启发性。

从逻辑上讲，我们可以将公开的服务和 API 分为两类：

1. **功能 API **是实现通用，可重用业务功能的服务所公开的公共API。
2. **特定**于**体验的 API **建立在功能API的基础上，并公开可能针对特定渠道的功能，或针对通用能力的特定于上下文的专业化而优化的功能。上下文信息可能与时间，位置，设备，通道，身份，用户，角色，特权级别等有关。

##### 1.7.1 基于功能的服务和 API

功能 API 是可重用业务功能的公共接口。*公开*暗示这些 API 仅限于供前端体验，外部使用者或来自不同域的内部使用者使用的接口。

##### 1.7.2 基于体验的服务和 API

特定于体验的服务在核心功能上提供了最少的附加业务逻辑，并且主要提供了转换和轻型编排，以根据特定体验，渠道或设备的需求量身定制交互。它们的输入/输出功能仅限于服务呼叫。

#### 1.8 客户端，API 客户端，API 使用者

调用API请求并使用API响应的实体。

## 三、服务设计原则

本节阐述指导服务设计的原则，这些服务向内部和外部开发人员，邻接，合作伙伴和关联公司公开API。服务是指与特定功能有关的功能，以API形式公开。

以下是服务的核心设计原则。

### 1. 松耦合

**服务和消费者必须彼此松散耦合。**

耦合是指两件事之间的联系或关系。耦合程度可与依赖程度相媲美。该原则提倡服务契约的设计，并始终强调减少（*放松*）服务契约，其实施和服务使用者之间的依赖性。

松散耦合的原理促进了服务逻辑和实现的独立设计和发展，同时仍然强调了与已经依赖该服务功能的消费者之间的基线互操作性。

该原则意味着以下几点：

- 服务契约不应公开实施细节
- 服务契约可以发展而不会影响现有消费者
- 特定域中的服务可以独立于其他域发展

### 2. 封装形式

**域服务只能通过其他服务契约访问其不拥有的数据和功能。**

服务公开的功能包括其拥有和实现的功能和数据，以及不依赖于它的功能和数据。该原则主张，服务依赖和不拥有的任何功能或数据都只能通过服务契约来访问。

该原则意味着以下几点：

- 服务具有明确的隔离边界-在功能和数据方面明确的所有权范围
- 服务无法公开其不直接拥有的数据

### 3. 稳定性

**服务契约必须稳定。**

服务的设计方式必须使其所暴露的契约对现有客户仍然有效。如果服务契约需要以与消费者不兼容的方式发展，则应清楚地传达这一点。

该原则意味着以下几点：

- 现有的客户服务必须在记录的时间内得到支持
- 必须以不影响现有消费者的方式引入其他功能
- 必须明确规定弃用和迁移政策以设定消费者的期望

### 4. 可重用

**服务必须开发为可在多个上下文中被多个使用者重用。**

API平台的主要目标是通过使用和组合服务来使应用程序能够快速，经济高效地开发。只有在为多个用例和多个使用者灵活地开发了服务契约的情况下才有可能。该原则主张服务的开发方式应使其能够被多个消费者和在多种环境中使用，其中某些环境可能会随着时间而发展。

该原则意味着以下几点：

- 服务契约不仅应针对即时环境而设计，还应具有支持和/或可扩展性，以供多个消费者在不同环境中使用
- 服务契约可能需要随着时间的推移逐步发展以支持多种环境和消费者

### 5. 基于契约

**功能和数据只能通过标准化服务契约公开。**

服务通过服务契约公开其目的和功能。服务契约包括功能方面，非功能方面（例如可用性，响应时间）和业务方面（例如每次通话费用，条款和条件）。*标准化*意味着服务契约必须符合契约设计标准。

该原则主张所有功能和数据都只能通过标准化服务契约公开。因此，服务的消费者只能通过服务契约来理解和访问功能和数据。

该原则意味着以下几点：

- 在服务契约之外无法理解或访问功能和数据
- 每条数据（例如在数据存储中管理的数据）仅由一项服务拥有

### 6. 一致性

**服务必须遵循一组通用的规则，交互样式，词汇和共享类型。**

一组规则规定了服务的定义，以便以一致的方式公开服务。通过减少新服务使用者的学习曲线，该原理提高了 API 平台的易用性。

该原则意味着以下几点：

- 为服务规定了一套标准
- 服务应使用通用和共享词典中的词汇
- 兼容的交互样式，服务粒度和共享类型是实现完全互操作性和简化服务组合的关键

### 7. 使用方便

**服务必须易于使用并在使用者（和应用程序）中组成。**

使用困难且耗时的服务会通过鼓励消费者找到访问相同功能的替代机制来降低微服务架构的优势。可组合性意味着可以轻松组合服务，因为服务合同和访问协议是一致的，并且不必对每个服务合同有不同的理解。 

该原则意味着以下几点：

- 服务合同易于发现和理解
- 服务合同和协议在可能的所有方面都是一致的，例如，标识和认证机制，错误语义，通用类型用法，分页等。
- 服务具有明确的所有权，因此消费者提供者可以就 SLA ，要求和问题与服务所有者联系
- 消费者提供者可以轻松集成，测试和部署使用此服务的消费者
- 消费者提供商可以轻松监控服务的非功能性方面

### 8. 可外部化

**服务的设计必须使其提供的功能易于外部化。**

开发了一种服务以供可能来自另一个域或团队，另一个业务部门或另一个公司的消费者使用。在所有这些情况下，公开的功能都是相同的。更改的是访问机制或服务执行的策略，例如身份验证，授权和速率限制。由于公开的功能是相同的，因此应设计一次服务，然后通过适当的策略根据业务需求将其外部化。

该原则意味着以下几点：

- 服务接口必须从域模型及其旨在支持的预期用例中得出
- 支持的服务合同和访问（绑定）协议必须满足消费者的需求
- 服务的外部化不得要求重新实现或更改服务合同

## 四、HTTP方法，标头和状态

### 1. 数据资源和HTTP方法

#### 1.1 业务能力和资源建模

组织的各种业务[功能](#功能)通过API公开为一组[资源](#资源)。功能不得在所有 API 之间重复；而是期望在各个用例之间按需重新使用资源（例如，用户帐户，信用卡等）。

#### 1.2 HTTP方法

大多数服务很容易落入标准数据资源模型中，在该模型中，主要操作可以用首字母缩写 CRUD （创建，读取，更新和删除）表示。这些很好地映射到标准 HTTP 动词。

| HTTP方法 | 描述                                               |
| -------- | -------------------------------------------------- |
| `GET`    | 要**获取**的资源。                                 |
| `POST`   | 要**创建**一个资源，或*执行*上的资源的复杂的操作。 |
| `PUT`    | 要**更新**的资源。                                 |
| `DELETE` | 要**删除**的资源。                                 |
| `PATCH`  | 对资源执行**部分更新**。                           |

调用的实际操作必须与上表中定义的HTTP方法语义相匹配。

- 该 **`GET`** 方法一定不能有副作用。它不得更改基础资源的状态。
- `POST` ：方法应用于在集合中创建新资源。
  - **示例：**要在文件上添加信用卡， `POST https://api.foo.com/v1/vault/credit-cards` 
  - 幂等语义：如果这是同一调用（包括 [`Foo-Request-Id`](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#http-custom-headers) 标头）的后续执行，并且已经创建了资源，则请求应该是幂等的。
- 该 `POST` 方法应该用于创建一个新的子资源并建立其与主资源的关系。
  - **示例：**要退还交易ID 12345的付款，请执行以下操作：`POST https://api.foo.com/v1/payments/payments/12345/refund` 
- 该 **`POST`** 方法可以与操作名称一起用于复杂的操作中。这也称为[*控制器模式*](https://github.com/paypal/api-standards/blob/master/patterns.md#controller-resource)，被认为是 RESTful 模型的例外。它更适用于以下情况：资源代表业务流程，而操作是作为其一部分执行的步骤或动作。欲了解更多信息，请参见[REST Web服务食谱](http://techbus.safaribooksonline.com/book/web-development/web-services/9780596809140)中的 [2.6节](http://techbus.safaribooksonline.com/9780596809140/chapter-identifying-resources)。
- 该 **`PUT`** 方法应该用于更新资源属性或建立从资源到现有子资源的关系；它使用对子资源的引用来更新主资源。

#### 1.3 处理中

在所有这些准则中，都假定必须使用 [JavaScript Object Notation（JSON）](http://json.org/) 发送请求主体和响应主体。JSON 是由无序键值对组成的对象的轻量级数据表示形式。 JSON 可以表示四种原始类型（字符串，数字，布尔值和null）和两种结构化类型（对象和数组）。处理 API 方法调用时，应遵循以下准则。

#### 1.4 数据模型

用于表示的数据模型必须符合 [RFC 7159中](https://tools.ietf.org/html/rfc7159) 所述的 JSON 数据交换格式。

#### 1.5 序列化

- 资源端点必须支持 `application/json` 作为内容类型。
- 如果 `Accept` 报头被发送并且 `application/json` 不是可接受的响应，则 `406 Not Acceptable` 必须返回错误。

#### 1.6 输入和输出严格度

API 在产生的信息中必须严格，并且在使用时也应严格。

由于我们正在处理编程接口，因此我们需要尽可能地避免猜测发送给我们的内容的含义。鉴于集成对于开发人员来说通常是一次性的任务，并且我们提供了良好的文档，所以我们必须严格使用接收到的数据。必须权衡 [Postel 定律](https://en.wikipedia.org/wiki/Robustness_principle)与许可分析的许多危险。

### 2. HTTP头

HTTP 头的是为了以统一，标准化和隔离的方式提供有关消息的正文或发送者的元数据信息。HTTP标头名称不区分大小写。

- HTTP标头应仅用于处理跨领域关注点。
- API实现不应引入或依赖标头。
- 标头不得包含API或特定于域的值。
- 如果可用，则必须使用HTTP标准标头，而不是创建自定义标头。

#### 2.1 假设条件

**服务使用者和服务提供者：**

- 不应该期望特定的HTTP标头可用。调用链中的中间组件可能会丢弃HTTP标头。这就是业务逻辑不应基于HTTP标头的原因。
- 不应该假设标头的值尚未更改为HTTP消息传输的一部分。

**基础结构组件**（HTTP消息传递中涉及的Web服务框架，客户端调用库，企业服务总线（ESB），负载平衡器（LB）等）：

- 可以基于特定报头的可用性和有效性返回错误，而不转发消息。例如，基于客户端身份和凭据的请求的身份验证或授权错误。
- 可以添加，删除或更改HTTP标头的值。

#### 2.2 HTTP标准标头

这些是从 [HTTP / 1.1 规范（RFC 7231）](http://tools.ietf.org/html/rfc7231#page-33)定义或引用的标头。许多基础架构组件都很好地定义和理解了它们的目的，语法，值和语义。

| HTTP标头名称         | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `Accept`           | 该请求标头指定 API 客户端能够在响应中处理的媒体类型。发出 HTTP 请求的系统应发送此标头。处理该请求的系统不应假定该请求可用。在所有这些 API 指南中都假定 API 支持 `application/json` 。 |
| `Accept-Charset`   | 该请求标头指定 API 客户端在响应中可以处理的字符集。该值 `Accept-Charset`应包括`utf-8`。 |
| `Content-Language` | 该请求/响应头用于指定内容的语言。默认语言环境为 `en-US` 。 API 客户端应使用 Content-Language 标头识别数据的语言。API 必须在响应中提供此标头。  例： `Content-Language：zh-CN` |
| `Content-Type`     | 该请求/响应头指示请求或响应主体的媒体类型。 <br/><ul><li>API 客户端必须与请求包括：如果请求包含一个主体，例如它是一个 `POST` ， `PUT` 或 `PATCH` 请求。</li><li>如果包含响应主体（不与 `204` 响应一起使用），API开发人员必须在响应中包含它。</li><li>如果内容是基于文本的类型，例如[JSON](http://json.org/)，则 `Content-Type` 必须包含一个字符集参数。字符集必须为 UTF-8 。</li><li>目前唯一支持的媒体类型是`application/json`。</li></ul><br /> 例：<br /><pre>（在HTTP请求中）Accept: application/json<br />                                         Accept-Charset: utf-8  <br />（在HTTP响应中）Content-Type: application/json; charset=utf-8</pre> |
| `Link`             | 根据 [Web连接RFC 5988](https://tools.ietf.org/html/rfc5988)，连接是由国际化资源标识符（IRI）标识的两个资源之间的类型化连接。 `Link` 实体头字段提供了在HTTP头中的序列化的一个或多个链接的装置。  应该在设计假设下构建 API ，即 API 或 API 客户端的业务逻辑都不应该依赖标头中提供的信息。标头只能用于携带跨领域的关注信息，例如安全性，可追溯性，监视等。  因此，`Link`响应代码`201`或禁止使用标头`3xx`。考虑在响应正文中使用[HATEOAS链接](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#hypermedia)。 |
| `Location`         | 此响应标头字段用于将收件人重定向到Request-URI以外的其他位置，以完成请求或标识新资源。  应该在设计假设下构建API，即API或API客户端的业务逻辑都不应该依赖标头中提供的信息。标头只能用于携带跨领域的关注信息，例如安全性，可追溯性，监视等。  因此，`Location`响应代码`201`或禁止使用标头`3xx`。考虑在响应正文中使用[HATEOAS链接](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#hypermedia)。 |
| `Prefer`           | [`Prefer`](https://tools.ietf.org/html/rfc7240)请求头字段表示客户端要求服务器的首选行为，但并比一定会生效。它是一个`end to end`字段，如果转发请求，则必须由代理转发，除非使用 `Connection` 头字段将 `Prefer` 明确标识为 `hop by hop` 。如果API文档中明确指出了对的支持，则以下标记值可以用于API `Prefer`。  <br />`respond-async`：API客户端希望API服务器异步处理其请求。<br /><pre>Prefer: respond-async</pre><br />服务器返回 `202 (Accepted)` 响应并异步处理请求。API 服务器可以随后使用 Webhook 通知客户端，或者客户端可以`GET`稍后调用以获取响应。有关更多详细信息，请参考[异步操作](https://github.com/paypal/api-standards/blob/master/patterns.md#asynchronous-operations)。  <br />`read-consistent`：API 客户端希望API 服务器从持久性存储返回具有一致数据的响应。对于不为其客户端提供任何优化首选项的API，此行为将是默认行为，并且不需要客户端设置此令牌。<br /><pre>Prefer: read-consistent</pre><br />`read-eventual-consistent`：API客户端希望API服务器从缓存或可能最终最终一致的数据存储返回响应（如果适用）。如果错过了从这两种类型的任何一种源中查找数据的机会，则API服务器可能会从一致的持久性数据存储中返回响应。<br /><pre>Prefer: read-eventual-consistent</pre><br />`read-cache`：API客户端希望API服务器从缓存中返回响应（如果有）。如果缓存命中未命中，则服务器可以返回其他来源的响应，例如最终的一致数据存储或一致，持久的数据存储。<br /><pre>Prefer: read-cache</pre><br />`return=representation`：API客户端更喜欢API服务器在对成功请求的响应中包含代表资源当前状态的实体。此首选项旨在通过消除在`GET`创建（`POST`）修改操作（`PUT`或`PATCH`）之后需要后续请求来检索资源的当前表示的方式，来提供一种优化客户端与服务器之间通信的方法。<br /><pre>Prefer: return=representation</pre><br />`return=minimal`：API客户端指示服务器仅返回对成功请求的最小响应。确定什么构成适当的“最小”响应完全由服务器决定。<br /><pre>Prefer: return=minimal</pre> |

#### 2.3 HTTP自定义标题

以下是这些准则中使用的一些自定义标头。这些不是HTTP规范的一部分。

| HTTP标头名称     | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `Foo-Request-Id` | API使用者可以选择发送带有唯一ID的此标头，以标识用于跟踪目的的请求标头。  这样的标头也可以在内部用于日志记录和跟踪目的。如果响应是同步的，则建议将此标头作为响应标头发送回去，或者作为适用的Webhook的请求标头发送回去。 |

#### 2.4 HTTP标头传递

当服务收到请求标头时，除了在分发给下游应用程序的请求/消息中的HTTP标准标头之外，它们还必须传递相关的自定义标头。

### 3. HTTP状态码

RESTful 服务使用 HTTP 状态代码来指定 HTTP 方法执行的结果。 HTTP 协议使用整数和消息指定请求执行的结果。该数字称为*状态码*，该消息称为*原因短语*。原因短语是人类可读的消息，用于阐明响应的结果。 HTTP 协议将范围内的状态代码分类。

#### 3.1 状态码范围

响应API请求时，必须使用以下状态代码范围。

| 范围  | 含义                                                         |
| ----- | ------------------------------------------------------------ |
| `2xx` | 成功执行。方法执行有可能以多种方式成功执行。此状态代码指定成功的方式。 |
| `4xx` | 通常，这些问题包括请求，请求中的数据，无效的身份验证或授权等。在大多数情况下，客户端可以修改其请求并重新提交。 |
| `5xx` | 服务器错误：由于站点中断或软件缺陷，服务器无法执行该方法。5xx 范围状态代码不应用于验证或逻辑错误处理。 |

#### 3.2 状态报告

成功和失败不仅适用于整个操作，还适用于SOA框架部分或代码执行的业务逻辑部分。

以下是状态代码和原因短语的准则。

- 必须使用`2xx`范围内的状态代码报告成功。
- `2xx`仅当完整的代码执行路径成功时，才必须返回范围内的HTTP状态代码。这包括任何容器/ SOA框架代码以及该方法的业务逻辑代码执行。
- 必须在`4xx`或`5xx`范围内报告失败。对于系统错误和应用程序错误都是如此。
- 如[`error.json`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/error.json)架构所定义，主体中必须存在一致的JSON格式的错误响应。该模式用于限定错误的种类。请参阅[错误处理](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#error-handling)指南以获取更多详细信息。
- 返回状态代码在`4xx`或`5xx`范围内的服务器必须返回`error.json`响应主体。
- 返回状态码`2xx`范围内的服务器`error.json`不得将响应后的响应或任何类型的错误代码作为响应主体的一部分返回。
- 对于`4xx`代码范围内的客户端错误，原因短语应为客户端提供足够的信息，以便能够确定导致错误的原因以及如何解决该错误。
- 对于`5xx`代码范围内的服务器错误，原因短语和后面的错误响应`error.json`应限制信息量，以避免向客户公开内部服务实现的详细信息。对于外部API和内部API都是如此。服务开发人员应使用日志记录和跟踪实用程序来提供其他信息。

#### 3.3 允许的状态代码列表

所有REST API务必仅使用以下状态代码。**`BOLD`**API开发人员应使用状态代码。其余的主要用于Web服务框架开发人员，报告与安全性，内容协商等有关的框架级错误。

- API绝不能返回此表中未定义的状态代码。
- API只能返回此表中定义的某些状态代码。

| 状态码                          | 描述                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| **`200 OK`**                    | 通用成功执行。                                               |
| **`201 Created`**               | 用作对`POST`方法执行的响应，以指示成功创建资源。如果已经创建了资源（例如，通过先前执行同一方法），则服务器应返回状态代码`200 OK`。 |
| **`202 Accepted`**              | 用于异步方法执行，以指定服务器已接受该请求并将在以后执行。有关更多详细信息，请参阅“ [异步操作”](https://github.com/paypal/api-standards/blob/master/patterns.md#asynchronous-operations)。 |
| **`204 No Content`**            | 服务器已成功执行该方法，但是没有要返回的实体主体。           |
| **`400 Bad Request`**           | 服务器无法理解该请求。使用此状态代码来指定： 作为有效负载一部分的数据不能转换为基础数据类型。数据不是预期的数据格式。必填字段不可用。简单数据验证类型的错误。 |
| `401 Unauthorized`              | 该请求需要身份验证，但未提供任何身份。请注意此与之间的区别`403 Forbidden`。 |
| **`403 Forbidden`**             | 客户端无权访问资源，尽管它可能具有有效的凭据。如果业务级别授权失败，API可以使用此代码。例如，避孕药持有人没有足够的资金。 |
| **`404 Not Found`**             | 服务器未找到与请求URI匹配的任何内容。这意味着URI不正确或资源不可用。例如，可能是该键处的数据库中没有数据。 |
| `405 Method Not Allowed`        | 服务器尚未实现请求的HTTP方法。这通常是API框架的默认行为。    |
| `406 Not Acceptable`            | 当服务器无法使用客户端请求的媒体类型返回响应的有效负载时，服务器必须返回此状态代码。例如，如果客户端发送了`Accept: application/xml`标头，而API只能生成`application/json`，则服务器必须返回`406`。 |
| `415 Unsupported Media Type`    | 当请求的有效负载的媒体类型无法处理时，服务器必须返回此状态代码。例如，如果客户端发送`Content-Type: application/xml`标头，但API仅接受`application/json`，则服务器必须返回`415`。 |
| **`422 Unprocessable Entity`**  | 请求的操作无法执行，可能需要与当前请求之外的API或进程进行交互。这与500响应不同，因为没有系统性的问题限制API执行请求。 |
| `429 Too Many Requests`         | 如果用户，应用程序或令牌的速率限制已超过预定义的值，则服务器必须返回此状态代码。在其他HTTP状态代码[RFC 6585中](https://tools.ietf.org/html/rfc6585)定义。 |
| **`500 Internal Server Error`** | 这可能是系统错误，也可能是应用程序错误，通常表明尽管客户端似乎提供了正确的请求，但服务器上出现了意外情况。一个`500`响应指示服务器端软件缺陷或站点停运。`500`不应用于客户端验证或逻辑错误处理。 |
| `503 Service Unavailable`       | 由于临时维护，服务器无法处理服务请求。                       |

#### 3.4 HTTP方法到状态码的映射

对于每种HTTP方法，API开发人员应仅在此表中使用标记为“ X”的状态代码。如果API需要返回标有的任何状态代码**`X`**，则应在API设计审核过程和成熟度评估中对用例进行审核。这些状态码大多数用于支持非常罕见的用例。

| 状态码   | 200成功 | 创建了201 | 202接受 | 204没有内容 | 400错误的要求 | 找不到404 | 422无法处理的实体 | 500内部服务器错误 |
| -------- | ------- | --------- | ------- | ----------- | ------------- | --------- | ----------------- | ----------------- |
| `GET`    | X       |           |         |             | X             | X         | **`X`**           | X                 |
| `POST`   | X       | X         | **`X`** |             | X             | **`X`**   | **`X`**           | X                 |
| `PUT`    | X       |           | **`X`** | X           | X             | X         | **`X`**           | X                 |
| `PATCH`  | X       |           |         | X           | X             | X         | **`X`**           | X                 |
| `DELETE` | X       |           |         | X           | X             | X         | **`X`**           | X                 |

- `GET`：该`GET`方法的目的是检索资源。成功后，将`200`期望状态码和包含资源内容的响应。如果资源集合为空（中的0个项目`/v1/namespace/resources`），`200`则状态为适当（资源将包含一个空`items`数组）。如果资源项目在基础数据中被“软删除”，`200`则不适合（`404`正确），除非打算公开“已删除”状态。
- `POST`：的主要目的`POST`是创建资源。如果资源不存在，而是在执行过程中创建的，则`201`应该返回状态码。
  - 期望在成功执行后，在响应主体中返回对创建的资源的引用（以链接或资源标识符的形式）。
  - 幂等语义：如果这是同一调用（包括[`Foo-Request-Id`](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#http-custom-headers)标头）的后续执行，并且已经创建了资源，`200`则应返回状态码SHOULD。有关API中的幂等性的更多详细信息，请参阅[idempotency](https://github.com/paypal/api-standards/blob/master/patterns.md#idempotency)。
  - 如果使用了子资源（“控制器”或数据资源），并且主要资源标识符不存在，`404`则是适当的响应。
- `POST`也可以在利用[控制器模式](https://github.com/paypal/api-standards/blob/master/patterns.md##controller-resource)时使用，`200`是适当的状态代码。

- `PUT`：此方法应该返回状态代码，`204`因为在大多数情况下，由于请求是要更新资源并且已成功更新，因此无需返回任何内容。来自请求的信息不应回显。
  - 在极少数情况下，可能需要在响应中提供服务器生成的值，以优化客户端流（如果客户端必须必须执行`GET`after `PUT`）。在这种情况下，应`200`设立一个响应机构。
- `PATCH`：此方法应遵循与`PUT`，`204`status和没有响应主体相同的状态/响应语义。
  - `200`+应该不惜一切代价避免响应主体，因为`PATCH`它会执行部分更新，这意味着每个资源多次调用是正常的。这样，对整个资源进行响应会导致大量带宽使用，尤其是对于带宽敏感的移动客户端。
- `DELETE`：此方法应该返回状态代码，`204`因为在大多数情况下无需返回任何内容，因为请求是删除资源并且已成功删除该资源。
  - 由于该`DELETE`方法也必须是幂等的，因此`204`即使资源已被删除，它仍应返回。通常，API使用者不关心资源是否作为此操作的一部分或之前的内容被删除了。这也是为什么`204`不`404`应该返回的原因。

## 五、超媒体

### 1. HATEOAS

超媒体是[超文本](https://en.wikipedia.org/wiki/Hypertext)一词的扩展，是一种非线性的信息介质，根据[Wikipedia的规定](https://en.wikipedia.org/wiki/Hypermedia)，它包括图形，音频，视频，纯文本和超链接。作为应用状态引擎的超媒体`HATEOAS`是Roy Fielding在[论文中](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)描述的REST应用体系结构的约束。

在RESTful API的上下文中，客户端可以完全通过服务动态提供的超媒体与服务交互。超媒体驱动的服务通过在响应中包括超媒体链接，向其客户端提供资源表示，以动态导航API。这不同于SOA的其他形式，在SOA中，服务器和客户端基于Web上某个位置定义的或基于带外交换的，基于WSDL的规范进行交互。

### 2. 符合Hypermedia的API

符合超媒体的API公开了服务的有限状态机。在这里，要求如`DELETE`，`PATCH`，`POST`和`PUT`通常在启动状态的转变，而响应指示状态的变化。让我们以一个API为例，该API公开一组操作来管理用户帐户生命周期并实现HATEOAS接口约束。

客户端通过固定的URI开始与服务进行交互`/users`。此固定URI支持`GET`和`POST`操作。客户端决定执行一项`POST`操作以在系统中创建用户。

**请求：**

```
POST https://api.foo.com/v1/customer/users

{
    "given_name": "James",
    "surname" : "Greenwood",
    ...

}
```

**响应：**

API从输入中创建一个新用户，并在响应中将以下链接返回到客户端。

- 检索用户完整表示的`self`链接（又称链接）（`GET`）。
- 更新用户的链接（`PUT`）。
- 部分更新用户的链接（`PATCH`）。
- 删除用户的链接（`DELETE`）。

```
{
HTTP/1.1 201 CREATED
Content-Type: application/json
...

"links": [
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "self",
        
    },
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "delete",
        "method": "DELETE"
    },
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "replace",
        "method": "PUT"
    },
    {
        "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
        "rel": "edit",
        "method": "PATCH"
    }
]

}
```

客户端可以将这些链接存储在其数据库中以供以后使用。

然后，在管理员决定删除其中一个用户之前，客户端可能希望显示一组用户及其详细信息。因此，客户端`GET`对相同的固定URI 执行a `/users`。

**请求：**

```
GET https://api.foo.com/v1/customer/users
```

该API通过相应的`self`链接返回系统中的所有用户。

**响应：**

```
{
    "total_items": "166",
    "total_pages": "83",
    "users": [
    {
        "given_name": "James",
        "surname": "Greenwood",
        ...
        "links": [
            {
                "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
                "rel": "self"
            }
        ]
    },
    {
        "given_name": "David",
        "surname": "Brown",
        ...
        "links": [
            {
                "href": "https://api.foo.com/v1/customer/users/ALT-MDFSKFGIFJ86DSF",
                "rel": "self"
            }
   },
   ...
```

客户端可以跟随`self`用户的链接并找出它可以在用户资源上执行的所有可能的操作。

**请求：**

```
GET https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI
```

**响应：**

```
HTTP/1.1 200 OK
Content-Type: application/json
{
        "given_name": "James",
        "surname": "Greenwood",
        ...
        "links": [
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "self",
        
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "delete",
            "method": "DELETE"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "replace",
            "method": "PUT"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "edit",
            "method": "PATCH"
        }


}
```

为了删除用户，客户端`delete`从其数据存储中检索链接关系类型的URI并对该URI执行删除操作。

**请求：**

```
DELETE https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI
```

综上所述：

- 客户端可以导航到API的入口点，以便访问所有其他资源。
- 客户端无需构建组成URI的逻辑来执行不同的请求或通过查看可能与URI和状态更改相关的响应详细信息（在后面的部分中进行更详细的描述）来编码任何种类的业务规则。 。
- 客户端承认创建URI的过程属于服务器这一事实。
- 客户端将URI视为不透明标识符。
- 在表示中使用超媒体的API可以无缝扩展。随着新方法的引入，可以通过相关的HATEOAS链接扩展响应。这样，客户可以增量方式利用该功能。例如，如果API开始支持新`PATCH`操作，则客户端可以使用它进行部分更新。

链接的存在本身不脱钩客户不必学会做一个过渡和所有相关链接语义请求所需的数据，特别是对`POST`/ `PUT`/ `PATCH`操作。API必须提供文档，以清楚地描述每个URI的所有链接，链接关系类型和请求响应格式。

后续部分提供有关链接结构以及不同关系类型的含义的更多详细信息。

### 3. 链接描述对象

必须使用*[链接描述对象（LDO）] [4](http://json-schema.org/latest/json-schema-hypermedia.html#anchor17)*模式*描述链接*。LDO在links数组中描述了单个链接关系。以下是链接描述对象的属性的简要描述。

`href`：

- `href`必须提供属性的值。
- `href`属性的值必须是*[URI模板] [6，](https://tools.ietf.org/html/rfc6570)*用于确定相关资源的目标URI。应根据[RFC 6570](https://tools.ietf.org/html/rfc6570)将其解析为URI模板。
- 仅将绝对URI用作`href`属性值。客户端通常将表示形式的链接关系类型的绝对URI标记为书签，以便以后发出API请求。开发人员必须使用*[URI组件命名约定](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#uri-component-names)*来构造绝对URI。传入`Host`报头（例如api.foo.com）中的值必须用作`host`绝对URI 的字段。

`rel`：

- `rel` 代表“链接关系类型”中定义的关系
- 该`rel`属性的值指示与目标资源的关系的名称。
- `rel`必须提供属性的值。

- `method`：
  - 该`method`属性标识必须用于向链接目标发出请求的HTTP动词。如果忽略该`method`属性，`GET`则默认值为。
- `title`：
  - 该`title`属性提供了链接的标题，并且是一个有用的文档工具，可帮助最终客户理解。不需要此属性。

##### 不为LDO使用HTTP标头

请注意，这些API准则不建议使用HTTP `Location`标头来提供链接。另外，他们也不建议使用[JAX-RS中](https://java.net/projects/jax-rs-spec/pages/Hypermedia)`Link`描述的标头。HTTP标头的范围仅限于客户端和服务之间的点对点交互。由于响应可能会传递到客户端上可能不直接与服务交互的其他层和组件，因此存储在标头中的任何信息可能都不可用。因此，我们建议在HTTP响应主体中返回链接描述对象。

### 4. 链接数组

`links`模式的array属性用于将链接描述对象与*[JSON hyper-schema draft-04] [3](http://json-schema.org/draft-04/hyper-schema#)*实例相关联。

- 此属性必须是一个数组。
- 数组中的项目必须为链接描述对象类型。

##### 指定链接数组

这是一个如何描述架构中的链接的示例。

- 必须在API资源模式定义的一部分中提供与下面的示例JSON模式中定义的链接数组相似的链接数组。请注意，links数组需要在`properties`对象的关键字内声明。这是代码生成器为所生成的对象中的links数组添加setter / getter方法所必需的。
- 必须使用URI模板在响应模式中声明API作为响应的一部分返回的所有可能的链接。URI模板的links数组必须在`properties`关键字之外声明。

```
{
    "type": "object",
    "$schema": "http://json-schema.org/draft-04/hyper-schema#",
    "description": "A sample resource representing a customer name.",
    "properties": {
        "id": {
	    "type": "string",
            "description": "Unique ID to identify a customer."
        },
        "first_name": {
            "type": "string",
            "description": "Customer's first name."
        },
        "last_name": {
            "type": "string",
            "description": "Customer's last name."
        },
        "links": {
            "type": "array",
            "items": {
                "$ref": "http://json-schema.org/draft-04/hyper-schema#definitions/linkDescription"
            }
        }
    },
    "links": [
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "self"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "delete",
            "method": "DELETE"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "replace",
            "method": "PUT"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/{id}",
            "rel": "edit",
            "method": "PATCH"
        }
    ]

}
```

以下是符合上述架构的示例响应。

```
{
    "id": "ALT-JFWXHGUV7VI",
    "first_name": "John",
    "last_name": "Doe",
    "links": [
        {
            "href": "https://api.foo.com/v1/cusommer/users/ALT-JFWXHGUV7VI",
            "rel": "self"
        },
        {
            "href": "https://api.foo.com/v1/customer/users/ALT-JFWXHGUV7VI",
            "rel": "edit",
            "method": "PATCH"
        }
    ]
}
```

### 5. 链接关系类型

 `Link Relation Type` 用作链接的标识符。API必须分配有意义的链接关系类型，该类型必须明确描述链接的语义。客户端使用相关的链接关系类型，以便从表示形式中识别要使用的链接。

当*[IANA的标准链接关系列表] [5](http://www.iana.org/assignments/link-relations/link-relations.xhtml)*中定义的链接关系类型的语义与您要定义的语义匹配时，则必须使用它。下表描述了一些常用的链接关系类型。它还列出了这些准则定义的一些其他链接关系类型。

| 链接关系类型     | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| `self`           | 传达链接上下文的标识符。通常是指向资源本身的链接。           |
| `create`         | 指可用于创建新资源的链接。                                   |
| `edit`           | 指编辑（或部分更新）链接标识的表示形式。使用它来表示`PATCH`操作链接。 |
| `delete`         | 指删除链接标识的资源。使用它`Extended link relation type`来表示`DELETE`操作链接。 |
| `replace`        | 指完全更新（或替换）链接标识的表示形式。使用它`Extended link relation type`来表示`PUT`操作链接。 |
| `first`          | 指结果列表的第一页。                                         |
| `last`           | 指提供的结果列表的最后一页`total_required`指定为查询参数。   |
| `next`           | 指结果列表的下一页。                                         |
| `prev`           | 指结果列表的前一页。                                         |
| `collection`     | 引用集合资源（例如/ v1 / users）。                           |
| `latest-version` | 指向包含最新（例如当前）版本的资源。                         |
| `search`         | 指可用于搜索链接上下文和相关资源的资源。                     |
| `up`             | 指资源层次结构中的父资源。                                   |

对于所有[`controller`](https://github.com/paypal/api-standards/blob/master/patterns.md#controller-resource)款式复杂的操作，控制器`action`名称必须使用中的链接关系类型（例如`activate`，`cancel`，`refund`）。

### 6. 用例

请参阅[HATEOAS用例](https://github.com/paypal/api-standards/blob/master/patterns.md#hateoas-use-cases)以查找可以在哪里使用HATEOAS。

## 六、命名约定

本节介绍了URI，查询参数，资源，字段和枚举的命名约定。在这里让我们强调一下，这些指南不是完全按照此处描述的那样遵循约定，而是更多地是在设计API时定义一些命名约定并以一致的方式遵守这些约定。例如，对于字段和文件名，我们跟随了[snake_case](https://en.wikipedia.org/wiki/Snake_case)，但是，您可以使用其他形式，例如[CamelCase](https://en.wikipedia.org/wiki/Camel_case)或您自己设计的其他形式。遵守已定义的约定很重要。

### 1. URI组件名称

URI遵循[RFC 3986](https://tools.ietf.org/html/rfc3986)规范。该规范简化了REST API服务的开发和使用。本节中的准则遵循RFC 3986约束来管理URI结构和语义。

#### 1.1 URI组件

根据RFC 3986，通用URI语法由组件的层次结构序列组成，称为方案，权限，路径，查询和片段，如下面的示例所示。

```
         https://example.com:8042/over/there?name=ferret#nose
         \___/   \_______________/\_________/\_________/\__/
          |           |            |            |        |
       scheme     authority       path        query   fragment
```

#### 1.2 URI命名约定

以下是对API的URI特定命名约定准则的简要说明。本规范使用括号“()”进行分组，星号 “ * ” 指定零个或多个出现，括号“[]”表示可选字段。

- URI必须以字母开头，并且只能使用小写字母。
- URI路径中的文字/表达式应使用连字符（-）分隔。
- 查询字符串中的文字/表达式应使用下划线（_）分隔。
- URI路径和查询字符串必须将数据编码成[UTF-8八位字节](https://en.wikipedia.org/wiki/Percent-encoding)。
- URI中应该使用多个名词来标识数据资源的集合。
  - `/invoices`
  - `/statements`
- 资源集合中的单个资源可以直接存在于集合URI的下面。
  - `/invoices/{invoice_id}`
- 子资源集合可以直接存在于单个资源之下。这应该传达与另一个资源集合的关系（在此示例中为发票项目）。
  - `/invoices/{invoice_id}/items`
- 子资源的单个资源可以存在，但应避免使用顶级资源。
  - `/invoices/{invoice_id}/items/{item_id}`
  - 更好： `/invoice-items/{invoice_item_id}`
- 资源标识符应遵循后续部分中描述的[建议](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#resource-identifiers)。

**例子**

- `https://api.foo.com/v1/vault/credit-cards`
- `https://api.foo.com/v1/vault/credit-cards/CARD-7LT50814996943336KESEVWA`
- `https://api.foo.com/v1/payments/billing-agreements/I-V8SSE9WLJGY6/re-activate`

**正式定义：**

| 术语                    | 定义                                                         |
| ----------------------- | ------------------------------------------------------------ |
| URI                     | [端点]' **/** '资源路径[' **？**'查询'                       |
| end-point               | [scheme“ **：//** ”] [host [' **：** 'port]]                 |
| schema                  | “ **http** ”或“ **https** ”                                  |
| 资源路径(resource-path) | “ **/ v** ”版本' **/** '名称空间名称' **/** '资源（' **/** '资源） |
| 资源(resource)          | 资源名称[' **/** '资源ID]                                    |
| 资源名称(resource-name) | Alpha（Alpha \|数字\|'-'）*                                  |
| 资源编号(resource-id)   | 值                                                           |
| query                   | 名称' **=** '值（' **＆** '名称=值）*                        |
| name                    | Alpha（Alpha \|数字\|'_'）*                                  |
| value                   | URI百分比编码值                                              |

说明

```
'   用单引号括住特殊字符
"	用双引号引起来的字符串
()	使用括号进行分组
[]	使用方括号指定可选表达式
*	一个表达式可以重复零次或多次
```

#### 1.3 资源名称

将服务建模为一组资源时，开发人员必须遵循以下原则：

- 必须使用名词，而不是动词。
- 资源名称对于单例必须为单数；集合的名称必须为复数。
  - 用户帐户上自动付款配置的说明
    - `GET /autopay` 返回完整的表示形式
  - 假设费用的集合：
    - `GET /charges` 返回已收取的费用清单
    - `POST /charges` 创建一个新的收费资源， `/charges/1234`
    - `GET /charges/1234` 返回一次充电的完整表示

- 资源名称必须为小写字母，并且只能使用字母数字字符和连字符。

- 连字符（-）必须用作URI路径文字中的单词分隔符。**请注意**，这是连字符用作单词分隔符的唯一位置。在几乎所有其他情况下，必须使用下划线字符（_）。

#### 1.4 查询参数名称

- 查询字符串中的文字/表达式应使用下划线（_）分隔。
- 查询参数值必须为百分比编码。
- 查询参数必须以字母开头，并且应全部小写。只能使用字母字符，数字和下划线（_）字符。
- 查询参数应该是可选的。
- 如[资源集合中](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#resource-collections)所示，某些查询参数名称是保留的。

有关查询参数用法的更多特定信息，请参阅[URI标准](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#uri-standards-query)。

### 2. 字段名称

表示形式的数据模型必须符合[JSON](http://json.org/)。这些值本身可以是对象，字符串，数字，布尔值或对象数组。

- 关键字名称必须为小写单词，并用下划线字符（_）分隔。
  - `foo`
  - `bar_baz`
- 前缀`is_`或`has_`不应用于布尔类型的键。
- 代表数组的字段应使用复数名词来命名（例如，authenticators-包含一个或多个Authenticator，products-包含一个或多个产品）。

### 3. 枚举名称

枚举的条目（值）应仅由大写字母数字字符和下划线字符（_）组成。

- `FIELD_10`
- `NOT_EQUAL`

如果有一个行业标准要求我们执行其他操作，则枚举可能包含其他字符。

### 4. 链接关系名称

表示的链接关系类型`rel`必须为小写。

- 例

```
"links": [
    {
        "href": "https://uri.foo.com/v1/customer/partner-referrals/ALT-JFWXHGUV7VI/activate",
        "rel": "activate",
        "method": "POST"

    }
   ]
```

### 5. 文件名称

API使用的各种类型的JSON模式都应包含在单独的文件中，并使用`$ref`语法（例如`"$ref":"object.json"`）进行引用。JSON模式文件应使用下划线命名语法，例如`transaction_history.json`。

## 七、URI

### 1. 资源路径

API的[资源路径](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#uri-components)由URI的路径，查询和片段组成。它将包括API的主要版本，后跟名称空间，资源名称以及一个或多个子资源（可选）。例如，考虑以下URI。

```
https://api.foo.com/v1/vault/credit-cards/CARD-7LT50814996943336KESEVWA
```

下表列出了上述URI资源路径的各个部分。

| 路径片                          | 描述               | 定义                                                         |
| ------------------------------- | ------------------ | ------------------------------------------------------------ |
| `v1`                            | 指定API的主要版本1 | API主版本用于区分同一API的两个向后不兼容的版本。API主版本是一个整数值，必须将其包含在URI中。 |
| `vault`                         | 命名空间           | 命名空间标识符用于提供资源的上下文和范围。它们由API平台实现的业务能力模型中的逻辑边界确定。 |
| `credit-cards`                  | 资源名称           | 如果资源名称表示资源的集合，则资源上的`GET`方法应检索资源列表。查询参数应用于指定搜索条件。 |
| `CARD-7LT50814996943336KESEVWA` | 资源ID             | 要从集合中检索特定资源，必须将资源ID指定为URI的一部分。子资源在下面讨论。 |

#### 1.1 子资源

子资源表示从一种资源到另一种资源的关系。子资源名称提供了关系的含义。如果基数为1：1，则不需要其他信息。否则，子资源应提供用于唯一标识的子资源ID。如果基数为1：许多，则将返回所有子资源。不应支持不超过两个级别的子资源。

| 例                                                           | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/documents` | 该调用应返回与争议ABCD1234相关的所有文档。                   |
| `GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/documents/102030` | 该调用应仅返回与此争议相关的特定文档的详细信息。请记住，这只是说明如何使用子资源ID的说明性示例。在实践中，应该避免两步调用。如果第二标识符是唯一的，`/v1/customer-support/documents/102030`则首选顶级资源（例如）。 |
| `GET https://api.foo.com/v1/customer-support/disputes/ABCD1234/transactions` | 以下示例应返回与此争议相关的所有交易及其详细信息，因此无需指定特定的交易ID。如果需要特定的事务ID，`/v1/customer-support/transactions/ABCD1234`则最好进行检索（假设ID是唯一的）。 |

#### 1.2 资源标识符

[资源标识符](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#resource-identifier)标识资源或子资源。这些**必须**符合以下准则。

- 资源标识符的生命周期必须由资源的域模型拥有，在其中可以保证它们唯一地标识单个资源。
- API不得使用数据库序列号作为资源标识符。
- 优选使用[UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier)哈希ID（基于[HMAC](https://en.wikipedia.org/wiki/Hash-based_message_authentication_code)）作为资源标识符。
- 出于安全性和数据完整性的原因，所有子资源ID必须仅在父资源内范围内。
  **示例：** `/users/1234/linked-accounts/ABCD`
  即使帐户“ ABCD”存在，也必须将其链接到用户1234，否则绝不能返回。
- 枚举值可用作子资源ID。应该使用枚举值的字符串表示形式。
- 不得有两个资源标识符，一个接一个。
  **例：** `https://api.foo.com/v1/payments/payments/12345/102030`
- 资源ID应该尝试使用资源标识符字符或ASCII字符。有**不应该**是使用UTF-8字符的任何ID。
- 资源ID和查询参数值必须对URI未保留字符以外的任何字符执行URI百分比编码。使用UTF-8字符的查询参数值必须进行编码。

### 2. 查询参数

查询参数是在资源路径后指定的名称/值对，如RFC 3986 所述。在应用以下部分时，还应遵循[命名约定](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#naming-conventions)。

##### 筛选资源集合

- 查询参数仅应用于限制资源收集或用作搜索或过滤条件。
- 集合中的资源标识符不应用于过滤集合结果，资源标识符应位于URI中。
- 分页参数应遵循[分页](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#pagination)准则。
- 默认的排序顺序应该被认为是不确定的和不确定的。如果需要明确的排序顺序，则查询参数`sort`应与以下常规语法一起使用：`{field_name}|{asc|desc},{field_name}|{asc|desc}`。例如：`/accounts?sort=date_of_birth|asc,zip_code|desc`

##### 在单个资源上查询参数

在使用一种资源的典型情况下（例如`/v1/payments/billing-plans/P-94458432VR012762KRWBZEUA`），不应使用查询参数。

##### 缓存友好的API

在极少数情况下，资源需要高度可缓存（通常是数据变化很小），可以使用查询参数，而不是`POST`+请求主体。至于`POST`会做出反应不可缓存，`GET`最好在这些情况下。这是唯一可能需要查询参数的情况。

##### 用POST查询参数

当`POST`用于操作时，通常不建议使用查询参数来支持请求正文字段。在`POST`提供分页结果的情况下（通常在`GET`不合适的复杂搜索API中），可以使用查询参数来提供指向下一页结果的超媒体链接。

##### 为同一查询参数传递多个值

在将查询参数用于搜索功能时，通常需要传递多个值。例如，它可能出现的情况是一个资源可以有许多国家，如`OPEN`，`CLOSED`和`INVALID`。如果API客户端想要查找所有`CLOSED`或的项目`INVALID`怎么办？

- 建议API通过重复查询参数来实现此功能。这是HTTP标准固有支持的，并且已经内置在大多数客户端库中。
  - 上面的查询将实现为`?status=CLOSED&status=INVALID`。
  - 在API规范中`"repeated": true`，必须使用参数的定义部分中的参数将其标记为可重复。
  - 参数名称应为单数。
- URI的实际长度限制非常低-最保守的是大约[2,000个字符](https://stackoverflow.com/questions/417142/what-is-the-maximum-length-of-a-url-in-different-browsers#417184)。因此，在某些情况下，API设计者可以选择使用单个查询参数来接受逗号分隔的值，以便在查询字符串中容纳更多的值。请记住，服务器和客户端库未始终提供此功能，这意味着实现者将需要编写其他字符串解析代码。由于额外的复杂性和与HTTP标准的差异，因此除非合理，否则不建议使用此解决方案。
  - 上面的查询将实现为`?statuses=CLOSED,INVALID`。
  - API规范中不得将参数标记为可重复。
  - 该参数必须`"type": "string"`在API规范中标记为，以容纳逗号分隔的值。`type`不得使用任何其他值。参数说明应指示接受逗号分隔的值。
  - 查询参数名称应为复数形式，以提示使用该模式。
  - 逗号（Unicode `U+002C`）应该用作值之间的分隔符。
  - API文档必须定义必要时如何转义分隔符。

## 八、错误处理

根据HTTP规范，可以使用整数和消息指定请求执行的结果。该数字称为*状态码*，该消息称为*原因短语*。原因短语是一种人类可读的消息，用于阐明响应的结果。`4xx`范围内的HTTP状态代码指示客户端错误（验证或逻辑错误），而范围内的HTTP状态代码`5xx`指示服务器端错误（通常是缺陷或中断）。但是，这些状态码和人类可读的原因短语不足以以机器可读的方式传达有关错误的足够信息。要解决错误，非人类的RESTful API使用者需要其他帮助。

因此，API必须返回符合[Common Types](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04)储存库中[`error.json`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/error.json)定义的架构的JSON错误表示。建议API所属的名称空间具有与其关联的错误目录。请参阅[错误目录](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#error-catalog)以获取更多详细信息。

### 1. 错误模式

`error.json`作为模式的错误响应必须包括以下字段：

- `name`：错误的易于理解的唯一名称。建议[`error_spec.json#name`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/error_spec.json)在发送错误响应之前从错误目录中检索此值。

- ```
  details
  ```

  ：包含错误的各个实例的数组，其具体内容如下。客户端错误（

  ```
  4xx
  ```

  ）必填。

  - `field`：[JSON指向](http://tools.ietf.org/html/rfc6901)错误的字段（如果位于正文中）的[指针](http://tools.ietf.org/html/rfc6901)，否则为path参数或查询参数的名称。
  - `value`：错误字段的值。
  - `issue`：错误原因。建议[`error_spec_issue.json#issue`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/error_spec_issue.json)在发送错误响应之前从错误目录中检索此值。
  - `location`：在错误的字段中，所述的位置`query`，`path`或`body`。如果不存在此字段，则默认值为`body`。

- `debug_id`：一个唯一的错误标识符，在服务器端生成并记录为关联目的。

- `message`：一条人类可读的消息，描述了错误。此消息必须是问题的描述，而不是有关解决方法的建议。建议[`error_spec.json#message`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/error_spec.json)在发送错误响应之前从错误目录中检索此值。

- `links`：特定于错误情况的[HATEOAS](https://github.com/paypal/api-standards/blob/master/hypermedia.md)链接。使用这些链接可提供有关错误情况以及如何解决该问题的更多信息。您可以插入来自[`error_spec.json#suggested_application_actions`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/error_spec.json)和/或[`error_spec.json#suggested_user_actions`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/error_spec.json)此处的链接，以及与API相关的其他HATEOAS链接。

以下字段是可选的：

- `information_link`：（`deprecated`）与此错误相关的扩展开发人员信息的URI；这应该是指向错误类型的公开文档的链接。使用`links`代替。

###### JSON指针的使用

如果您使用其他方法`field`在已经发布的API中标识，则可以继续使用现有方法。但是，如果您打算迁移到建议的方法，则希望提高API的主要版本并为客户提供迁移帮助，因为这可能对他们来说是一个潜在的重大变化。

JSON指针`field`应为[JSON字符串值](https://tools.ietf.org/html/rfc6901#section-5)。

#### 输入验证错误

在验证请求时，应按以下顺序解决各种问题：

| 要求验证问题                                                 | HTTP状态码                 |
| ------------------------------------------------------------ | -------------------------- |
| 格式不正确的JSON。                                           | `400 Bad Request`          |
| 包含客户端可以更改的验证错误。                               | `400 Bad Request`          |
| 由于请求主体之外的因素而无法执行。该请求格式正确，但由于语义错误而无法遵循。 | `422 Unprocessable Entity` |

### 2. 错误样本

本节提供一些示例来描述[`error.json`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/error.json)在各种情况下的用法。

##### i. 验证错误响应-单个字段

下面的示例`VALIDATION_ERROR`在一个字段中显示类型的验证错误。由于这是客户端错误，因此`400 Bad Request`应返回HTTP状态代码。

```json
{  
   "name":"VALIDATION_ERROR",
   "details":[  
      {  
         "field":"#/credit_card/expire_month",
         "issue":"Required field is missing",
         "location":"body"
      }
   ],
   "debug_id":"123456789",
   "message":"Invalid data provided",
   "information_link":"http://developer.foo.com/apidoc/blah#VALIDATION_ERROR"
}
```

##### ii. 验证错误响应-多个字段

以下示例`VALIDATION_ERROR`在两个字段中显示了相同类型的验证错误。请注意，这`details`是一个列出错误中所有实例的数组。由于这两个都是客户端错误，因此`400 Bad Request`应返回HTTP状态代码。

```json

{  
   "name": "VALIDATION_ERROR",
   "details": [  
      {  
         "field": "/credit_card/expire_month",
         "issue": "Required field is missing",
         "location": "body"
      },
      {  
         "field": "/credit_card/currency",
         "value": "XYZ",
         "issue": "Currency code is invalid",
         "location": "body"
      }
   ],
   "debug_id": "123456789",
   "message": "Invalid data provided",
   "information_link": "http://developer.foo.com/apidoc/blah#VALIDATION_ERROR"
}
```

##### iii. 验证错误响应-批量

对于如下所示`OBJECT_NOT_FOUND_ERROR`和的异类客户端错误`MULTIPLE_CORE_BUNDLES`，将`errors`返回一个名为的数组。每个错误实例在此数组中均表示为一项。由于这些是客户端验证错误，因此`400 Bad Request`应返回HTTP状态代码。

```json
   "errors": [  
      {  
         "name": "OBJECT_NOT_FOUND_ERROR",
         "debug_id": "38cdd677a83a4",
         "message": "Bundle is not found.",
         "information_link": "<link to public doc describing OBJECT_NOT_FOUND_ERROR error>",
         "details": [  
            {  
               "field": "/bundles/0/bundle_id",
               "value": "33333",
               "issue": "BUNDLE_NOT_FOUND",
               "location": "body"
            }
         ]
      },
      {  
         "name": "MULTIPLE_CORE_BUNDLES",
         "debug_id": "52cde38284sd3",
         "message": "Multiple CORE bundles.",
         "information_link": "<link to public doc describing MULTIPLE_CORE_BUNDLES error>",
         "details": [  
            {  
               "field": "/bundles/5/bundle_id",
               "value": "88888",
               "issue": "MULTIPLE_CORE_BUNDLES",
               "location": "body"
            }
         ]
      }
   ]
```

##### vi. 语义验证错误响应

如果客户端输入的格式正确且有效，但请求操作可能需要与此URI之外的API或进程进行交互，`422 Unprocessable Entity`则应返回HTTP状态代码。

```
{  
   "name": "BALANCE_ERROR",
   "debug_id": "123456789",
   "message": "The account balance is too low. Add balance to your account to proceed.",
   "information_link": "http://developer.foo.com/apidoc/blah#BALANCE_ERROR"
}
```

### 2. API规范中的错误声明

文档生成工具和客户端/服务器端绑定生成工具必须识别，这一点很重要[`error.json`](https://github.com/paypal/api-standards/blob/master/v1/schema/json/draft-04/error.json)。下一节显示了如何`error.json`在API规范中引用OpenAPI。

```
"responses": {
          "200": {
            "description": "Address successfully found and returned.",
            "schema": {
              "$ref": "address.json"
            }
          },
          "403": {
            "description": "Unauthorized request.  This error will occur if the SecurityContext header is not provided or does not include a party_id."
          },
          "404": {
            "description": "The requested address does not exist.",
            "schema": {
              "$ref": "v1/schema/json/draft-04/error.json"
            }
          },
          "default": {
            "description": "Unexpected error response.",
            "schema": {
              "$ref": "v1/schema/json/draft-04/error.json"
            }
          }
        }
```

## 九、API版本控制

本节介绍如何版本化API。它描述了API的生命周期状态，列举了版本控制策略，描述了与向后兼容的准则，并描述了生命周期终止策略。

### 1. API生命周期

以下是API生命周期状态的示例。

| 状态               | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| 计划(PLANNED)      | API已计划开发。已经建立了API发布计划。                       |
| 测试版(BETA)       | API可以运行，并且可以供生产中的选定新订户使用，以进行测试，验证和推出新API的目的。 |
| 服役(LIVE)         | API已投入使用，生产中的新订户均可使用。完全支持API版本。     |
| 已淘汰(DEPRECATED) | API是可操作的，并且在运行时可供现有订户使用。完全支持API版本，包括以向后兼容的方式解决的错误修复。API版本不适用于新订户。 |
| 退役(RETIRED)      | API未从生产中发布，并且在运行时不再对任何订阅者可用。进入此状态的所有已部署应用程序的占用空间必须从生产和阶段环境中完全删除。 |

### 2. API版本政策

API是版本化产品，必须遵守以下版本化原则。

1. API规范必须遵循版本控制方案，在此`v`版本中，引入版本时，主版本号是`1`第一个LIVE版本的序号，`0`而次要版本是任何主版本的第一个次要版本的序号。
2. 每当对API进行增量更改时，无论是否向后兼容，都必须对API规范进行版本控制。这样就可以对更改进行标记，记录，审查，讨论，发布和传达。
3. API端点只能反映主要版本。
4. API规范版本反映了接口更改，可以与服务实现版本控制方案分开。
5. 次要API版本必须与同一个主要版本中的所有先前次要版本保持向后兼容性。
6. 主要的API版本可以保持与先前主要版本的向后兼容性。

对于给定的功能集，`LIVE`所有主要版本和次要版本在任何给定时间都必须只有一个API版本处于状态。这样可以确保订阅者始终了解他们应该使用哪个版本的API产品。例如，v1.2 `RETIRED`，v1.3 `DEPRECATED`或v2.0 `LIVE`。

### 3. 向后兼容

API应该以向前和可扩展的方式进行设计，以保持兼容性并避免资源，功能和过多版本控制的重复。

API必须遵循以下原则才能被认为是向后兼容的：

1. 所有更改都必须加和。
2. 所有更改都必须是可选的。
3. 语义不得改变。
4. 查询参数和请求主体参数必须无序。
5. 现有资源的附加功能必须实现：
   1. 作为可选扩展，或
   2. 作为对新子资源的操作，或者
   3. 如果不能合理扩展现有操作（例如，资源创建），则通过更改请求主体，同时仍可以接受所有先前存在的现有请求变化。

#### 3.1 向后不兼容更改的非详尽清单

**URIs**

1. 资源URI可以支持其他查询参数，但是不能是必需的。
2. 没有新添加的查询参数，请求URI的API行为不得有任何变化。
3. 具有约束条件的新参数不得添加到请求中。
4. 现有参数，整个表示形式或资源的语义不应更改。
5. 服务必须识别参数的先前有效值，并且在使用时不应抛出错误。
6. URI返回的HTTP状态代码一定不能有任何变化。
7. 必须不存在的HTTP动词（如任何变化`GET`，`POST`，`PUT`或`PATCH`）由URI早期支持。URI可以支持一个新的HTTP动词。
8. URI的请求或响应标头的名称和类型不得有任何更改。可以添加其他头，只要它们是可选的。

**API仅支持媒体类型`application/json`。以下内容适用于JSON表示形式的稳定性。**

1. API响应的JSON对象中的现有属性必须继续以相同的名称和JSON类型（数字，整数，字符串，数组，对象）返回。
2. 如果响应字段的值是一个数组，则该数组中内容的类型不得更改。
3. 如果响应字段的值是一个对象，则兼容性策略必须整体应用于JSON对象。
4. 可以随时将新属性添加到表示中，但是不应改变现有属性的含义。
5. 添加的新属性一定不是必需的。
6. 确保在响应中包含必填字段。
7. 对于原始类型，除非在API文档中有描述的约束（例如，字符串的长度，ENUM类型的可能值），否则客户端不得假定这些值以任何方式受到约束。
8. 如果对象的属性是URI，则它必须具有与URI相同的稳定性。
9. 对于返回HATEOAS链接作为表示形式一部分的API，rel和href的值必须保持相同。
10. 对于ENUM类型，不得支持已支持的枚举值或这些值的含义。

### 4. 终止政策

终止（EOL）策略规定了API版本从过渡`LIVE`到`RETIRED`状态的方式。它旨在确保需要从旧版API迁移到新版API的API客户获得一致且合理的过渡期，同时使健康的流程可以消除技术债务。

**次要API版本EOL**

根据版本控制政策，次要API版本必须与同一主要版本中的先前次要版本向后兼容。因此，次要API版本是`RETIRED`紧随同一主要版本的较新次要版本之后`LIVE`。此更改应不会对现有订户产生影响，因此无需通过`DEPRECATED`状态转换即可促进客户端迁移。

**主要API版本EOL**

根据版本控制政策，主要API版本可以与先前的主要版本向后兼容。因此，退休主要API版本时，以下规则适用。

1. 在`DEPRECATED`替代服务`LIVE`为所有将继续保留的功能提供清晰的客户迁移路径之前，不得使用主要的API版本。此应包括文档以及适当的迁移工具和示例代码，这些代码可为客户提供进行干净迁移所需的内容。
2. 不推荐使用的API版本必须处于`DEPRECATED`最短状态，以便为客户客户提供足够的迁移通知。应根据具体情况考虑使用外部客户端弃用API版本，并且可能需要额外的弃用时间和/或约束，以最大程度地减少对业务的影响。
3. 如果处于`LIVE`或`DEPRECATED`状态的版本化API 没有客户端，则可以`RETIRED`立即移至该状态。

#### 4.1 生命周期终止政策–替代主要API版本简介

由于导致淘汰已存在的API版本的新的主要API版本是一项重大的商业投资决定，因此API所有者必须在开始大量的设计和开发工作之前，为新的主要版本辩护。API所有者应该探索引入新的主要API版本的所有可能替代方法，目的是在决定引入新版本之前将对客户的影响最小化。理由应包括以下内容：

商业案例

1. 新版本交付的客户价值是现有版本无法实现的。
2. 过时版本与新版本之间的成本效益分析。
3. 解释引入新的主要版本的替代方法，以及为何未选择这些替代方法。
4. 如果需要向后不兼容的更改来解决关键的安全问题，则不需要第1项和第2项。

API设计

1. 新API版本中所有资源的域模型，以及它们与先前主要API版本的域模型的比较方式。
2. API操作及其实现的用例的描述。
3. 定义性能和可用性的服务级别目标，该目标与要弃用的主要API版本相同或更好。

迁移策略

1. 受影响的现有客户数量；内部，外部和合作伙伴。
2. 沟通和支持计划，以告知现有客户新版本，价值和迁移路径。

## 十、弃用

本文档介绍了随着API的发展而弃用API部分的解决方案。它是API版本控制策略的扩展。

### 使用条款

*`API Element`*在本文档中使用该术语来指代可能在API中弃用的*事物*。的示例`API Element`包括：端点，查询参数，路径参数，JSON对象架构中的属性，类型的JSON对象架构或自定义HTTP标头。

该术语*`old API`*用于表示您的API所取代的现有次要或主要版本或现有的其他API。

该术语*`new API`*用于表示您的API的新的次要版本或主要版本，或替代的新的其他API `old API`。

*`API definition`*是遵循[OpenAPI](http://swagger.io/specification)的服务接口规范的形式。API的定义可以在中找到`swagger.json`。

### 1. 背景

定义API时，您必须做出许多具有长期影响的重大决策。目的是制作一个长寿，耐用和可重用的API。您正在尝试使其“正确”。但是实际上，您不会每次都成功。引入了新的需求。您对问题的理解会发生变化。当时看起来不错的决定，现在可能会限制您优雅地扩展API以满足新理解的能力。随着影响的逐渐集中，过去做出的轻量级决策现在似乎有些沉重。保持向后兼容性是一个持续的挑战。

一种选择是创建API的新主版本。这使您可以抛弃过去的决定而重新开始。不幸的是，这还意味着您的所有客户现在都需要迁移到新的端点，以便进行任何新工作来交付客户价值。这很难。没有良好的激励，许多客户就不会搬家。管理客户迁移有很多开销。您还需要在相当长的一段时间内支持两组接口。另一个考虑因素是您的API产品可能有多个端点，但是您要进行的重大更改只会影响一个端点。使您的客户为所有API端点迁移其应用程序只是为了“修复”一小部分，是一项非常沉重且昂贵的更改。从哲学和工程学的角度来看，虽然纯粹而简单，从投资回报率的角度来看，这通常是不合理的。本指南的目的是找到一个中间立场，该中间立场在需要进行微小更改时提供一条更实际的途径，但从本质上讲，这仍与[API版本控制政策](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#api-versioning-policy)。

### 2. 要求

以下是弃用的要求。

1. API开发人员应该能够弃用`API Element`次要版本的API。
2. API规范必须突出显示API的一个或多个已弃用的元素，以便API使用者知道。
3. API服务器务必在运行时通知客户端应用有关请求和/或响应中存在的不赞成使用的元素，以便工具可以识别此情况，记录警告并根据需要突出显示不赞成使用的元素的用法。
4. 已过时`API Elements`必须为主要版本的生活依然支持或直至客户不再使用它们（的手段，以确定这都留给了API所有者的自由裁量权，因为这是他们的客户谁最终会受到影响）。

### 3. 解决方案

下面介绍如何解决上面列出的要求。该解决方案涉及使用注释解决文档相关需求，并使用自定义标头解决运行时相关需求。

1. [文献资料](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#deprecation-documentation)
2. [运行](https://github.com/paypal/api-standards/blob/master/api-style-guide.md#deprecation-runtime)

#### 3.1 文献资料

在API的定义中，名为的可选注释`x-deprecated`用于将标记`API Element`为。

##### 注释：x弃用

`x-deprecated`可用于弃用任何类型的`API Element`。注释应在`API Element`定义的位置内联使用。期望通过反省API定义来生成文档的API工具会识别此注释，并突出显示`API Element`已弃用的相应注释。还假定该注释可以被包括生成实现绑定（POJO）的工具完全忽略。换句话说，将不会`@deprecated`为该`x-deprecated`注释生成任何特定于实现语言的构造（例如Java注解），也不在此解决方案的范围内。

据预计，该API文档将突出废弃的`API Elements`通过注释`x-deprecated`的API规范，清楚，并在适当的粒度。

##### x弃用的注释的架构

我们提供了特定的JSON对象类型，以用于弃用specific `API Elements`。本节列出了这些类型的架构。为的特定应用提供模式的目的`x-annotation`是使API开发人员易于注释`API definition`和API工具，以突出显示每个不推荐`API Element`使用的细节。

###### 通用架构元素

以下是在用于JSON的新JSON对象类型中使用的常见模式类型。

```
		"x-deprecatedValue": {
			"type": "string",
			"description": "Value of the element that is deprecated. Use to deprecate a particular value in parameter or schema property as applicable."
		},
		"x-deprecatedSee": {
			"type": "string",
			"description": "URI (indirect or absolute) or name of to new parameter, resource, method, api_element, as applicable."
		},
		"x-apiVersion": {
			"pattern": "^[1-9][0-9]*[.][0-9]+$",
			"minLength": 3,
			"maxLength": 8,
			"description": "This string should contain the release or version number at which this schema element became deprecated. Version should be in the format '{major}.{minor}' (no leading 'v')."
		}
```

###### 弃用的资源

以下模式必须用于弃用中的资源对象`API definition`。中的资源对象的示例`swagger.json`是：`operation`和`paths`。

```
		"x-deprecatedResource": {
			"type": "object",
			"title": "Schema for a deprecated resource.",
			"description": "Schema for deprecating a resource API element. A resource API element could be an operation or paths.",
			"properties": {
				"see": {
					"$ref": "#/definitions/x-deprecatedSee"
				},
				"since_version": {
					"$ref": "#/definitions/x-apiVersion"
				}
			}
		}
```

以下部分提供了几个示例，这些示例显示了资源级别的`deprecatedResource`for `x-deprecated`注释的用法。

###### 示例：不推荐使用的资源

下面的示例演示弃用命名的资源`commercial-entities`在`swagger.json`。

```
    "paths": {
       
        "/commercial-entities": {
             "x-deprecated": {
            		"see": "financial-entities",
            		"since_version": "1.4"
            	},
        ...
        }
```

###### 示例：不推荐使用的方法

下面的示例显示了不推荐使用的方法，`PUT /commercial-entities/{merchant_id}/agreements`并鼓励使用新方法`PATCH /commercial-entities/{merchant_id}/agreements`。

```
    "paths": {
        "/commercial-entities/{merchant_id}/agreements": {
            "put": {
                "summary": "Updates the Commercial Entity Agreements Details for a Merchant.",
                "operationId": "commercial-entity.agreement.update",
                "x-deprecated": {
            			"see": "patch",
            			"since_version": "1.4"
            	},
            	"parameters": [
                    {
                        "name": "merchant_id",
                        "in": "path",
                        "description": "The encrypted Merchant's identifier.",
                        "required": true,
                        "type": "string"
                    },
                    {
                        "name": "Agreements",
                        "in": "body",
                        "description": "An array of AgreementDetails",
                        "required": true,
                        "schema": {
                            "$ref": "./model/agreement_details.json"
                        }
                    }
                ],
            ...
         }
```

##### 不推荐使用的参数

`swagger.json`提供了一种为方法定义一个或多个参数的方法。参数的类型为：路径，查询和标头。通常，可以弃用查询和标头参数。`x-deprecated`对参数使用注释时，必须使用以下模式。

```json
		"x-deprecatedParameter": {
			"type": "object",
			"title": "Schema for a deprecated parameter.",
			"description": "Schema for deprecating an API element inline. The API element could be a custom HTTP header or a query param.",
			"properties": {
				"value": {
					"$ref": "#/definitions/x-deprecatedValue"
				},
				"see": {
					"$ref": "#/definitions/x-deprecatedSee"
				},
				"since_version": {
					"$ref": "#/definitions/x-apiVersion"
				}
			}
		}
```

以下部分提供了几个示例，显示`deprecatedParameter`了`x-deprecated`在参数级别使用for 注释的情况。

###### 示例：不建议使用的查询参数

下面的示例显示`x-deprecated`批注中`swagger.json`用于指示查询参数已弃用的用法`record_date`。

```json
        "/commercial-entities/{merchant_id}": {
            "get": {
                "summary": "Gets a Commercial Entity as denoted by the merchant_id.",
                "description": "Gets the Commercial Entity as denoted by the merchant_id.",
                "operationId": "commercial-entity.get",
                "parameters": [
                    {
                        "name": "record_date",
                        "in": "query",
                        "description": "The date to use for the query; defaulted to yesterday.",
                        "required": false,
                        "type": "string",
                        "format": "date",
                        "x-deprecated": {
                            "since_version": "1.5",
                            "see": "transaction_date"
                        }
                    },
                    {
                        "name": "transaction_date",
                        "in": "query",
                        "description": "The date to use for the query; defaulted to yesterday.",
                        "required": false,
                        "type": "string",
                        "format": "date"
                    },
                    ...
```

###### 示例：不建议使用的标题

以下示例显示了一个已弃用的自定义HTTP标头，名为`CLIENT_INFO`。

###### OpenAPI

```json
        "/commercial-entities/{merchant_id}": {
            "get": {
                "summary": "Gets a Commercial Entity as denoted by the merchant_id.",
                "description": "Gets the Commercial Entity as denoted by the merchant_id.",
                "operationId": "commercial-entity.get",
                "parameters": [
                    ...
                    {
                        "name": "CLIENT_INFO",
                        "in": "header",
                        "description": "Optional header for all the API's to pass on api caller tracking information. This header helps capture any input from the caller service and pass it along to analytics for tracking .",
                        "in" : "header",
                        "x-deprecated": {
                             "since_version": "1.5"
                        }

                    }
                    ...
```

###### 示例：不建议使用的查询参数值

以下示例显示了中的`x-deprecated`注释的用法，`swagger.json`以指示已弃用`y`名为的查询参数的特定值（）`fields`。

```json
        "/commercial-entities": {
            "get": {
                "summary": "Gets Commercial Entities.",
                "description": "Gets the Commercial Entities.",
                "operationId": "commercial-entity.get",
                "parameters": [
                    {
                        "name": "fields",
                        "in": "query",
                        "description": "Fields to return in response, default is x, possible values are x, y, z.",
                        "required": false,
                        "type": "string",
                        "x-deprecated": {
                            "since_version": "1.5",
                            "value": "y"
                        }
                    },
                    {
                        "name": "transaction_date",
                        "in": "query",
                        "description": "The date to use for the query; defaulted to yesterday.",
                        "required": false,
                        "type": "string",
                        "format": "date"
                    },
                    ...
```

##### 弃用的JSON对象架构

为了将JSON对象架构内不建议使用JSON对象本身或一个或多个属性的一个模式，我们建议使用一种叫做架构`deprecatedSchema`的`x-deprecated`注解。

```
		"x-deprecatedSchema": {
			"type": "array",
			"description": "Schema for a collection of deprecated items in a schema.",
			"items": {
				"$ref": "#/definitions/x-deprecatedSchemaProperty"
			}
		}
```

```
		"x-deprecatedSchemaProperty": {
			"type": "object",
			"title": "Schema for a deprecated schema property or schema itself.",
			"description": "Schema for deprecating an API element within JSON Object schema. The API element could be an individual property of a schema of a JSON type or an entire schema representing JSON object.",
			"required": ["api_element"],
			"properties": {
				"api_element": {
					"type": "string",
					"description": "JSON pointer to API element that is deprecated. If the API element is JSON Object schema of a type itself, JSON pointer MUST point to the root of that schema. If the API element is a property of schema, the JSON pointer MUST point to that property."
				},
				"value": {
					"$ref": "#/definitions/x-deprecatedValue"
				},
				"see": {
					"$ref": "#/definitions/x-deprecatedSee"
				},
				"since_version": {
					"$ref": "#/definitions/x-apiVersion"
				}
			}
		}
```

为了避免使用`keywords`所需的扩展JSON draft-04架构来通过在架构的`x-annotation`元数据部分中使用来弃用架构本身或在行内弃用单个属性，我们在中选择了`x-annotation`在JSON Object的引用旁边使用的破坏性较小的途径`API definition`。因此，应该在`API definition`“引用”模式中使用该注释。如果可以使用内联，则应该继续并在架构或架构本身中注释不赞成使用的属性。

截至2017年3月，[OpenAPI 3.0.0-rc0](https://github.com/OAI/OpenAPI-Specification/blob/OpenAPI.next/versions/3.0.md)引入了`deprecated`标志以应用于操作，参数和架构字段级别。`x-annotation`可以与`deprecated`标志一起使用，以为已弃用的标记提供其他有用的信息`API Element`。

###### 示例：响应中不建议使用的属性

下面的示例显示`x-deprecated`批注中的用法，`API definitions`以指示已弃用作为`address`响应命名的属性。

```
        "/commercial-entities/{merchant_id}": {
            "get": {
                "summary": "Gets a Commercial Entity as denoted by the merchant_id.",
                "description": "Gets the Commercial Entity as denoted by the merchant_id.",
                "operationId": "commercial-entity.get",
                "responses": {
                    "200": {
                        "description": "The Commercial Entity.",
                        "schema": {
                            "$ref": "./model/interaction/commercial-entities/merchant_id/get_response.json",
                            "x-deprecated": [
            					   {
            						  "api_element": "./model/interaction/commercial-entities/merchant_id/get_response.json#/address",
            						  "see": "./model/interaction/commercial-entities/merchant_id/get_response.json#/global_address",
            						  "since_version": "1.4"
            					   }
            				    ] 
                        }
                    },
                    "default": {
                        "description": "Unexpected error",
                        "schema": {
                            "$ref": "v1/schema/json/draft-04/error.json"
                        }
                    }
                }
```

###### 示例：响应中不推荐使用的枚举

以下示例显示了中的`x-deprecated`注释的用法，`API definitions`以指示已弃用`FAILED`名为的属性使用的枚举`state`。

```
       "/commercial-entities/{merchant_id}": {
            "get": {
                "summary": "Gets a Commercial Entity as denoted by the specified merchant identifier.",
                "description": "Gets the Commercial Entity as denoted by the specified merchant identifier.",
                "operationId": "commercial-entity.get",
                "responses": {
                    "200": {
                        "description": "The Commercial Entity.",
                        "schema": {
                            "$ref": "./model/interaction/commercial-entities/merchant_id/get_response.json",
                            "x-deprecated": [
            					   {
            						  "api_element": "./model/interaction/commercial-entities/merchant_id/get_response.json#/state",
            						  "value": "FAILED",
            						  "since_version": "1.4"
            					   }
            				    ] 
                        }
                    },
                    "default": {
                        "description": "Unexpected error",
                        "schema": {
                            "$ref": "v1/schema/json/draft-04/error.json"
                        }
                    }
                }
```