# 第 2 章 异常检测中的碰撞过程

这不是一本关于异常检测的总体广度和深度的书。 它专门用于应用异常检测来解决DevOps社区在尝试监控我们管理最多的系统类型时所面临的常见问题。

其中一个含义是本书主要是关于时间序列异常检测。 这也意味着我们专注于广泛使用的工具，如Graphite，JavaScript，R和Python。 基于我们正在做出的假设，这些选择有几个原因。

- 我们假设我们的受众与我们自己很像：开发人员，系统管理员，数据库管理员以及使用大多数开源工具的DevOps从业者。
- 我们都没有统计学或操作研究等领域的博士学位，我们假设您也没有。
- 我们假设您正在进行时间序列监控，就像我们一样。

由于这些假设，本书非常有偏见。 这完全是关于指标的异常检测，我们不会涉及配置异常检测，相互比较机器，日志分析，将类似事物聚类在一起，或许多其他类型的异常检测。 我们还专注于检测异常，因为它们发生，因为这是通常我们试图用我们的监控系统做的。

## 2.1 异常检测的一个实例

大约在2008年，Evan Miller发表了一篇论文，描述了IMVU运行中的实时异常检测。这是Baron首次暴露于异常检测：

在星期五凌晨5点左右，它首先检测到[邀请他们的Hotmail联系人开设帐户的IMVU用户数量]的问题，这一问题在当天的大部分时间都持续存在。 实际上，外部服务提供商在周五早上更改了界面，影响了一些但不是所有用户。

该论文的以下图像显示了指标及其与通常行为的偏差。

[图 2.1]()

他们发现一个非常不稳定的信号发生异常变化。 Mind. Blown. Magic!

异常检测方法是Holt-Winters预测。 根据某些标准，它相对粗糙，但是可以应用良好的结果来精心选择遵循可预测模式的指标。 米勒继续提到其他例子，其中相同的技术帮助工程师找到问题并迅速解决问题。

如何在系统上获得类似的结果？ 要回答这个问题，首先我们需要考虑异常检测是什么，不是什么，以及它的好坏。

## 2.2 什么是异常检测？

异常检测是一种帮助在噪声指标中查找信号的方法。 “异常”的通常定义是一种不寻常或意外的事件或价值。 在监控指标的异常检测环境中，我们关注这些指标的意外值。

异常可能有很多原因。 重要的是要认识到我们观察到的度量标准中的异常与生成度量标准的系统中的条件不同。 通过假设指标中的异常表明系统存在问题，我们正在进行精神和实际的飞跃，这可能是也可能不是正确的。 异常检测不了解您的系统。 它只是理解你对异常值或异常值的定义。

值得注意的是，大多数异常检测方法都将“不寻常”和“意外”替换为“统计上不可能的”。这是常见的做法并且通常是隐含的，但您应该意识到这种差异。

一种常见的混淆是认为异常与异常值相同（与典型值非常相近的值）。 实际上，异常值很常见，应该被认为是正常的和预期的。 异常是异常值，至少在大多数情况下，但并非所有异常值都是异常。

## 2.3 它是什么好处？

异常检测具有多种用例。 即使在我们之前指出的本书的范围内，异常检测也可以做很多事情：

- 它可以找到不寻常的指标值，以显示未检测到的问题。 例如，在IMVU示例中，服务器会在一段时间内出现可疑繁忙或空闲，或者事件数量小于预期的事件。
- 它可以找到重要指标或过程的变化，以便人类可以调查并弄清楚原因。
- 尝试诊断检测到的问题时，它可以减少表面积或搜索空间。 在数百万个指标的世界中，能够找到在问题发生时表现异常的指标是一种有用的方法来缩小搜索范围。
- 它可以减少在各种不同机器或服务上校准或重新校准阈值的需要。
- 它可以增强人类的直觉和判断力，有点像钢铁侠的套装增强了他的力量。

异常检测不能做人们有时认为可以做的很多事情。 例如：

- 它不能提供根本原因分析或诊断，尽管它当然可以提供帮助。
- 它不能提供关于是否存在异常的肯定或否定答案，因为充其量只限于是否存在异常的可能性。 （即使是人类也常常无法确定某个值是异常的。）
- 它无法证明系统中存在异常，只是您正在观察的度量标准存在异常。 请记住，度量标准不是系统本身。
- 它无法检测到实际的系统故障（故障），因为故障与异常不同。 （再次见上一点。）
- 它不能取代人类的判断和经验。
- 它无法理解指标的含义。
- 一般而言，它不能在所有系统，所有指标，所有时间范围和所有频率范围内一般地工作。

最后一项非常重要。 有一些病理学案例，每种已知的异常检测方法，每种统计技术，每种测试，每种假阳性过滤器，一切都将崩溃并失败。 对于大型数据集，例如在现代应用程序中以高分辨率监视大量计算机的指标时，您会发现这些病态情况得到保证。

特别是，在诸如一秒钟度量分辨率的高分辨率下，大多数机器生成的度量标准都非常嘈杂，并且会导致大多数异常检测技术摒弃大量误报。

### 2.3.1 异常是罕见的吗？

根据您的观察方式，异常情况很少见或常见。 异常的通常定义使用概率作为异常的代理。 经常出现的经验法则是距平均值三个标准偏差。 这是我们稍后将深入讨论的一种技术，但是现在只要说我们假设数据的行为完全符合预期就足够了，99.73％的观测值将落在三西格玛之内。 换句话说，每千人略少于三次观测将被视为异常。

这听起来非常罕见，但考虑到每天有14,400分钟，你仍然会将每天观察到的38个观测结果标记为异常，即使是一分钟的粒度也是如此。 如果你使用一秒钟的粒度，你可以将这个数字乘以60.突然间，这些罕见的事件似乎非常普遍。 有人甚至会称他们为吵闹，不是吗？

这是您在每个服务器上的每个指标上想要的吗？ 你可以自己决定自己的感受。 问题的关键是，很多人可能认为异常检测发现罕见的事件，但在现实中这种假设并不总是成立。

## 2.4 你如何使用异常检测？

要在实践中应用异常检测，通常有两种选择，至少在本书考虑的范围内。 选项一是生成警报，选项二是记录事件以供以后分析，但不要对它们发出警报。

从指标中的异常生成警报有点危险。 部分原因是因为异常罕见的假设并不像您想象的那么真实。 请参阅侧栏。 一种天真的异常警报方法几乎肯定会引起很多噪音。

我们的建议是不要警惕大多数异常现象。 这直接来自异常并不意味着系统处于不良状态的事实。 换句话说，有一个指标异常的观察和实际系统的故障有很大的区别。 如果您可以保证异常可靠地检测到系统中的严重问题，那就太好了。 继续并提醒它。 但另外，我们建议您不要对可能没有影响或后果的事情发出警报。

相反，我们建议您记录这些异常观察结果，但不要对它们发出警报。 现在，您已经基本上为指标中最不寻常的数据点创建了一个索引，供以后使用以防万一。 例如，在诊断您检测到的问题时。

此建议中嵌入的假设之一是异常检测足够便宜，可以在数据到达监控系统时一次性在线进行，但事后异常检测的临时性异常检测成本太高，无法进行交互式处理。 通过我们在当今行业中看到的监控数据大小，以及您应该“衡量所有移动的一切”的态度，通常就是这种情况。 多TB异常检测分析通常速度慢得令人无法接受，并且需要的资源比现有资源多。 同样，我们将此置于我们大多数人正在使用典型的开源工具和方法进行监控的环境中。

## 2.5 结论

虽然很容易对异常检测中的成功案例感到兴奋，但大多数时候其他人的技术不会直接转换到您的系统和数据。 这就是为什么你必须自己学习什么是有效的，什么适合在某些情况下使用而不是在其他情况下使用等等。

我们的建议将构成本书其余部分的讨论框架，一般来说，您可能应该在数据到达时“在线”使用异常检测。 存储结果，但在大多数情况下不要提醒他们。 请记住，地图不是领土：指标不是系统，异常不是危机，三西格玛不太可能，等等。