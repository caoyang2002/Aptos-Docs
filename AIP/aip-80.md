---
aip: 81
title: 无密钥账户的Pepper服务
author: 
  - name: Zhoujun Ma
    email: zhoujun@aptoslabs.com
discussions-to: <指向官方讨论帖的URL>
status: 草稿
last-call-end-date: <mm/dd/yyyy 最后留下反馈和评论的日期>
type: <标准 (核心, 网络, 接口, 应用, 框架) | 信息 | 过程>
created: 2024-03-17
updated: <mm/dd/yyyy>
requires: <AIP number(s)>
---

[TOC]

# AIP-80 - 标准化私钥

## 一、概要

本 AIP 定义了私钥在链下应如何表示。它提供了一种区分私钥和地址的方法，并定义了一种确定与哪种密钥方案相关联的方式。

### 1. 目标

> 目标是什么，范围是什么？有哪些指标？

私钥在链上的账户地址上是无法区分的。这个 AIP 的目标是改变这一点，提供一种区分私钥和链上地址的解决方案。

主要目标是防止人们在链上意外泄露他们的私钥。这通常是通过向私钥发送资金而导致的。今天，私钥、公钥和账户地址都以 32 字节或 64 十六进制字符的形式（带有或不带有 0x 前缀）表示。之后，有人可能会发现这个私钥，并在链上旋转账户的密钥。

其次，我们的目标是让这一变更过程尽可能平滑

> 讨论此更改将带来的业务影响和业务价值。

这将减少链上意外丢失密钥的数量。

### 2. 不在范围内

> 我们承诺不做什么，以及为什么它们被排除在范围之外？

在静态状态下加密私钥，因为这与私钥被误认为是地址是不相关的。

## 二、动机

> 描述这一变化的动机。它实现了什么？

这一变化应该减少用户在链上意外泄露私钥的普遍情况。

> 如果我们不接受这个提案会发生什么？

用户将继续以正常速度丢失他们的私钥及其关联的账户。



## 三、影响

> 哪些受众会受到这一变化的影响？受众需要采取什么样的行动？

这影响到用户和他们使用的钱包。钱包是私钥的主要存储地点，任何一个钱包应用（移动端、浏览器扩展或内置应用）或者带有钱包功能的工具（Aptos 命令行工具）都需要采纳关于私钥的任何新标准，同时要保持向后兼容。

## 四、规范

> 我们将如何解决问题？详细描述应该如何实施此提案。包括应该遵循的预期设计原则。使提案具体到足以让他人在其基础上构建，并可能推导出竞争性的实现。

### 1. 为私钥添加前缀

让我们从一个假设的私钥开始：`0x0000000000000000000000000000000000000000000000000000000000000001`

钱包、命令行工具和应用程序输出的私钥格式将具有前缀，例如：`ed25519-priv-0x0000000000000000000000000000000000000000000000000000000000000001`。  
然后，钱包、命令行工具和应用程序将接受私钥的两种可能输入：

1. 带有前缀，例如：`ed25519-priv-0x0000000000000000000000000000000000000000000000000000000000000001`
2. 不带前缀，例如：`0x0000000000000000000000000000000000000000000000000000000000000001`

前缀将由两部分组成：密钥方案和密钥类型。这将允许区分具有相同值和相同密钥方案的密钥。

1. 密钥方案
   1. ed25519 -> Ed25519
   2. secp256k1 -> Secp256k1
2. 密钥类型
   1. priv -> 目前为止私钥
   2. pub -> 目前为止公钥

这样做有以下好处：

1. 标识与私钥关联的签名方案，以实现在不同钱包和应用程序之间的可移植性。
2. 拒绝对已验证为 AccountAddress (`0x123...456`) 的参数输出的新私钥。
3. 对之前保存的私钥进行向后兼容。
4. 仅影响钱包和类似钱包应用的输入和输出。

## 五、参考实现

> 这是一个可选但非常鼓励的部分，您可以在此处包含一个示例，展示您在此提案中所寻求的内容。这可以是代码、图表，甚至是纯文本。理想情况下，我们可以链接到一个展示标准的存储库的实时链接，或者对于更简单的情况，可以包含内联代码。

待定

## 六、测试（可选）

> - 测试计划是什么？（除了负载测试外，所有测试都应作为实现细节的一部分，并且不需要另行调用）
> - 我们可以何时期待结果？
> - 测试结果如何？是否与预期相符？如果不符，解释差距。

...

## 七、风险和缺点

> - 在接受此提案可能带来的潜在负面影响。有哪些危险？

这里的风险是私钥在不同钱包之间不匹配。但这个风险是可以缓解的，因为私钥很容易去掉前缀，而不带前缀的密钥仍然会被接受。

> - 我们应该注意的任何向后兼容性问题？

没有，因为旧密钥仍然允许被导入。

> - 如果存在问题，我们如何缓解或解决它们？

我不认为更改后出现的问题会比当前导入和保存私钥时出现的问题更多。



## 八、替代方案

> 解释为什么您提交了此提案，而不是选择其他替代方案。为什么这是最好的可能结果？

### 1. 更改地址格式

地址格式可以从 64 个十六进制字符更改为其他编码方式。其他区块链使用 bech32 等编码方案来编码地址。这将更改长度和字符集，以自动阻止私钥用于地址。

不建议采用这种解决方案，因为它需要每个应用程序、索引器、链上表示、交易所和用户都更改为新格式。此外，旧地址格式需要与新地址格式具有向后兼容性，这样就无法解决此问题。

### 2. 将私钥编码更改为 Base64

私钥格式可以从 64 个十六进制字符更改为密钥的 base64（或其他编码）形式。这样可以让代码检查它，同时保持一种常用的格式。

不建议采用这种解决方案，因为这可能会导致密钥泄露到 hex 到 base64 转换器上。私钥绝不能发布到任何网站上。

### 3. 只使用助记词和路径

对于未来的私钥，只使用助记词和路径，以便与私钥区分开。派生路径将区分不同的私钥。

这种解决方案不允许自定义私钥，并且不允许私钥的简单向后兼容性。大多数钱包已经支持助记词，但私钥更具体，必须使用特定的派生路径。

这不允许对当前已保存的私钥进行向后兼容性，但可以在未来使用。

## 九、未来潜力

> 仔细思考这项提议在未来的演变。您认为这将如何发展？一年后会有什么结果？五年后呢？

在未来，这应该能够区分不同密钥方案和不同密钥类型的密钥。它将非常明确地表明钱包是否支持该密钥。

展望未来，可以更好地探索加密密钥的加密存储。

## 十、时间表

### 1. 建议的实施时间表

> 描述您期望的实施工作需要多长时间，可以将其分解为阶段或里程碑。

- 里程碑 1：新密钥格式作为输入和输出
  - 实施应该需要 2 周时间，更新 Rust、Python、Unity 和 TypeScript SDK 以接受新的密钥格式。
- 里程碑 2：
  - 这需要与每个钱包和应用程序合作，以输出新的标准。这可能需要 2-3 个月，因为大多数钱包使用的是传统的 TypeScript SDK。

### 2. 建议的开发者平台支持时间表

> 描述对于此功能，SDK、API、CLI、索引器的支持计划。

这种改变完全是在开发平台上的，应该只需要 2 周的支持。

### 3. 建议的部署时间表

> 指示将来的发布版本作为对此在我们的三个网络上部署的*粗略*估计（例如，发布 1.7）。
> 在 devnet 上？
> 在 testnet 上？
> 在 mainnet 上？

不需要在特定时间表上部署到所有系统。但是，做以下事情会很好：

1. 第一个月：4 个钱包接受新的私钥方案作为输入
2. 第二个月：将这些钱包更改为输出新的私钥方案
3. 第三个月：发起活动，修复其他应用程序，比如可能会接受/输出私钥的游戏。

## 十一、安全考虑

> - 这是否导致了安全假设或我们的威胁模型的更改？

这不应该改变任何安全假设或威胁模型。

> - 有任何潜在的骗局吗？有什么应对策略？

我不认为有任何潜在的骗局，因为这只是一个简单的前缀。

> - 有任何安全影响/考虑吗？

没有。

> - 是否有可以共享的任何安全设计文档或审计材料？

没有。

## 十二、开放问题（可选）

> Q&A 在这里，其中一些可以有答案，其中一些问题可能是我们还没有想清楚的事情，但我们应该思考。