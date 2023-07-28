# Writing Technical Articles

### 论文结构

- 论文的典型结构：

  - 摘要，通常不超过100-150字;

  - 简介（简要说明）：简介问题，概述解决方案，要明确说明为什么问题很重要或有趣。

  - 相关工作：引用过去 2-3 年左右前人研究的相关内容。

  - 论文其余部分的大纲：“论文的其余部分组织如下，在第 2 节中，我们介绍..第 3 节介绍 ...最后，我们将在第5节中描述未来的工作。“

  - 主体

    - 问题
    - 方法， 体系结构
    - 结果

    主体部分应该包含足够的动机，至少有一个示例场景，最好是两个，带有说明性数字，然后是 一个清晰的通用问题陈述模型，即功能， 特别强调“新”功能。

    *实现*：包含实际实现细节，要提到实现语言、平台、位置、依赖关系其他软件包和最低资源使用量。

    *评价*：它在实践中是如何运作的？提供真实或模拟的性能指标、最终用户研究、提及外部技术采用者等。

  - 总结和展望今后的工作

  - 确认

  - 参考书目

  - 附录

首先编写方法和结果部分，它们相辅相成。然后再写问题部分，再是结论，再是引言。然后写引言，因为它覆盖了结论的内容。最后写摘要。最后的最后，论文起一个标题。

### 标题

- 避免使用除最容易理解的缩写之外的所有缩写。

- 避免使用“新颖”、“绩效评估”和“架构”等常用词 ，因为几乎每篇论文都会进行性能评估。

  使用形容词来描述作品的独特特征， 例如，可靠、可扩展、高性能、健壮、低复杂性或 低成本。

- 如果需要论文标题的灵感，可以[查阅](http://www.cs.ucsd.edu/users/braghava/systems-topic-generator.html)*自动系统研究主题或论文标题生成器*。

### 摘要

- 摘要**不得**包含参考文献。
- 避免在摘要中使用“在本文中”。
- 避免抽象的一般动机。不必证明互联网的重要性或解释什么是QoS。
- 不仅要突出问题，还要突出主要结果。
- 由于摘要将被搜索引擎使用，确保识别作品的术语可在此处找到。特别是名字开发的任何协议或系统以及一般领域（“质量服务“、”协议验证“、”服务创建环境”） 应包含在摘要中。
- 避免方程和数学。 

### 引言

- 避免使用常用和陈词滥调的短语，例如“XYZ 的最新进展”或 任何暗示互联网发展的东西。

- 确保引言让读者知道这篇论文的内容，不仅是了解你的一般研究领域有多重要。 

- 引言必须表明问题、然后概述方法和/或贡献（甚至可能是对结果的一般描述。 

- 在引言中重复摘要是浪费空间。

- 示例不良介绍：

  > Here at the institute for computer research, me and my colleagues have created the SUPERGP system and have applied it to several toy problems. We had previously fumbled with earlier versions of SUPERGPSYSTEM for a while. This system allows the programmer to easily try lots of parameters, and problems, but incorporates a special constraint system for parameter settings and LISP S-expression parenthesis counting.
  >
  > The search space of GP is large and many things we are thinking about putting into the supergpsystem will make this space much more colorful.

- 优良引言示例，来自埃里克·西格尔的课堂：

  > - Many new domains for genetic programming require evolved programs to be executed for longer amounts of time. For example, it is beneficial to give evolved programs direct access to low-level data arrays, as in some approaches to signal processing \cite{teller5}, and protein segment classification \cite{handley,koza6}. This type of system automatically performs more problem-specific engineering than a system that accesses highly preprocessed data. However, evolved programs may require more time to execute, since they are solving a harder task.
  >
  >   - **Previous or obvious approach**:
  >
  >     (Note that you can also have a related work section that gives more details about previous work.)) One way to control the execution time of evolved programs is to impose an absolute time limit. However, this is too constraining if some test cases require more processing time than others. To use computation time efficiently, evolved programs must take extra time when it is necessary to perform well, but also spend less time whenever possible.
  >
  >   - **Approach/solution/contribution**:
  >
  >     The first sentence of a paragraph like this should say what the contribution is. Also gloss the results.In this chapter, we introduce a method that gives evolved programs the incentive to strategically allocate computation time among fitness cases. Specifically, with an *aggregate computation time ceiling* imposed over a series of fitness cases, evolved programs dynamically choose when to stop processing each fitness case. We present experiments that show that programs evolved using this form of fitness take less time per test case on average, with minimal damage to domain performance. We also discuss the implications of such a time constraint, as well as its differences from other approaches to {\it multiobjective problems}. The dynamic use of resources other than computation time, e.g., memory or fuel, may also result from placing an aggregate limit over a series of fitness cases.
  >
  >   - **Overview:**
  >
  >     The following section surveys related work in both optimizing the execution time of evolved programs and evolution over Turing-complete representations. Next we introduce the game Tetris as a test problem. This is followed by a description of the aggregate computation time ceiling, and its application to Tetris in particular. We then present experimental results, discuss other current efforts with Tetris, and end with conclusions and future work.

### 参考书目

- 避免在参考书目中使用*et al.，*除非列表很长。

- 互联网草稿必须标明“正在进行中”。

- 书籍引用包括出版年份，但没有 ISBN 编号。

- 现在可以接受包含URL，但包含指向在IEEE和ACM出版物上发表的论文的作者网页需要聊率版权情况。避免使用长网址，最好指向常规网址，因为不太可能改变。

- 在名字和姓氏之间留一个空格，即“J. P. Doe”， 不是“J.P.Doe”。

- 对于会议论文，必须标明会议地点、月份和完整的会议名称，而不仅仅是一些缩写。

- 检查互联网草稿是否已作为 RFC 发布，或者是否有 较新的版本。

  