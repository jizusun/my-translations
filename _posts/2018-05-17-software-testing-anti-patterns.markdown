---
layout: post
title:  "软件测试的反模式（译）"
categories: translations
---

* 英文原文：
    * [Software Testing Anti-patterns](http://blog.codepipes.com/testing/software-testing-antipatterns.html) ([Markdown version](https://github.com/kkapelon/kkapelon.github.io/blob/master/_posts/2018-04-21-software-testing-antipatterns.md))
* 译者： [孙继祖](https://www.linkedin.com/in/jizusun/)、 [黄秋荣](https://github.com/smallsnack)
* 延伸阅读
    - [Discussion thread on Hacker News](https://news.ycombinator.com/item?id=16894927)
    - [Introducing the Software Testing Cupcake (Anti-Pattern)](https://www.thoughtworks.com/insights/blog/introducing-software-testing-cupcake-anti-pattern) 及 [中文翻译](https://insights.thoughtworks.cn/introducing-software-testing-cupcake-anti-pattern/)
    - [Common System and Software Testing Pitfalls: How to Prevent and Mitigate Them: Descriptions, Symptoms, Consequences, Causes, and Recommendations (SEI Series in Software Engineering)](https://www.amazon.com/dp/0133748553)
    - [Perspectives On Agile Software Testing](https://info.thoughtworks.com/ebook-agile-software-testing) and [its book review](https://www.infoq.com/articles/agile-software-testing) 及 [书评中文翻译](http://www.infoq.com/cn/articles/agile-software-testing)
    - [JavaScript测试驱动开发 - 第 1 章　自动化测试让你重获自由](http://www.ituring.com.cn/book/tupubarticle/20075)
* 欢迎提出修订，请访问：[https://github.com/jizusun/jizusun.github.io/edit/master/_posts/2018-05-17-software-testing-anti-patterns.markdown](https://github.com/jizusun/jizusun.github.io/edit/master/_posts/2018-05-17-software-testing-anti-patterns.markdown)

# Table of Content
{:.no_toc}

- TOC
{:toc}


### Introduction 介绍
{: #introduction}

There are several articles out there that talk about testing anti-patterns in the software development process. Most of them however deal with the low level details of the programming code, and almost always they focus on a specific technology or programming language.

有很多其他的文章也在讨论软件开发过程中的测试反模式。但大多数是在谈底层的代码实现，而且几乎都是专注于某种特定的技术或者编程语言。

In this article I wanted to take a step back and catalog some high-level testing anti-patterns that are technology agnostic. Hopefully you will recognize some of these patterns regardless of your favorite programming language.

在这篇文章中，我想退后一步，编录几种高层的、与技术无关的测试反模式。希望你能认识到这些模式，不管你喜欢的是何种编程语言。

### Terminology 术语
{: #terminology}

Unfortunately, testing terminology has not reached a common consensus yet. If you ask 100 developers what is the difference between an integration test, a component test and an end-to-end test you might get 100 different answers. For the purposes of this article I will focus on the definition of the test pyramid as presented below.

不幸的是，测试术语还未达成共识，如果你问 100 个程序员，集成测试、组件测试、端到端测试的区别，你也许会得到 100 种不同的答案。基于本文的目的，我将集中讨论下面这幅测试金字塔中的定义。

![测试金字塔](../assets/2018-05-17-software-testing-anti-patterns/41332131-29fece64-6f0f-11e8-926e-1edbd9526424.png)

If you have never encountered the testing pyramid before, I would urge you to become familiar with it first before going on. Some good starting points are:

如果你以前从未遇到过测试金字塔，我强烈建议你在读下去之前先去熟悉它。下面一些文章可以作为切入点：

*   [The forgotten layer of the test automation pyramid 测试自动化金字塔中的遗忘层](https://www.mountaingoatsoftware.com/blog/the-forgotten-layer-of-the-test-automation-pyramid) (Mike Cohn 2009)
*   [The Test Pyramid 测试金字塔](https://martinfowler.com/bliki/TestPyramid.html) (Martin Fowler 2012)
*   [Google Testing blog 谷歌测试博客](https://testing.googleblog.com/2015/04/just-say-no-to-more-end-to-end-tests.html) (Google 2015)
*   [The Practical Test Pyramid 实用测试金字塔](https://martinfowler.com/articles/practical-test-pyramid.html) (Ham Vocke 2018)

The testing pyramid deserves a whole discussion on its own, especially on the topic of the amount of tests needed for each category. For the current article I am just referencing the pyramid in order to define the two lowest test categories. Notice that in this article User Interface Tests (the top part of the pyramid) are __not__ mentioned (mainly for brevity reasons and because UI tests come with their own specific anti-patterns).

测试金字塔本身就值得去深入讨论，尤其是在每个类别需要多少测试量这个主题上。在本文中，我只是引用测试金字塔，以定义最底层两个测试类别。请注意，在本文中，用户界面测试（测试金字塔的最上层）__没有__ 被提及（主要是出于简洁的目的，也是因为 UI 测试有自己特定的反模式）。

Therefore the two major test categories mentioned as __unit__ and __integration__ tests from now on are:

因此从现在起，被提到的两个主要的测试类别是单元测试和集成测试。

| Tests             | Focus on            | Require                    | Speed     | Complexity | Setup needed |
|-------------------|---------------------|----------------------------|-----------|------------|--------------|
| Unit tests        | a class/method      | the source code            | very fast | low        | No           |
| Integration tests | a component/service | part of the running system | slow      | medium     | Yes          |

| 测试               | 关注点              | 要求                        | 速度       | 复杂度     | 是否需要设置  |
|--------------------|--------------------|-----------------------------|-----------|------------|--------------|
| 单元测试            | 一个类、方法        | 源代码                       | 非常快    | 低         | 不需要        |
| 集成测试            | 一个组件、服务      | 系统的一部分                  | 慢        | 中等        | 需要         |

**Unit tests** are the category of tests that have wider acceptance regarding the naming and what they mean. They are the tests that accompany the source code and have direct access to it. Usually they are executed with an [xUnit framework](https://en.wikipedia.org/wiki/XUnit) or similar library. These tests work directly on the source code and have full view of everything. A single class/method/function is tested (or whatever is the smallest possible working unit for that particular business feature) and anything else is mocked/stubbed.

**单元测试**是对其命名和含义有更广泛接受的测试类别。这些测试伴随着源代码并且可以直接访问它。通常使用 [xUnit框架](https://en.wikipedia.org/wiki/XUnit) 或者类似的库执行它们。这些测试直接作用于源代码上，并且对所有的东西都有全面的了解。被测对象是一个类、方法、函数（或者为某特殊业务功能而设计的最小工作单元），而其他任何东西都是被模拟（mock）或被替代（stub）的。

**Integration tests** (also called service tests, or even component tests) focus on a whole component. A component can be a set of classes/methods/functions, a module, a subsystem or even the application itself. They examine the component by passing input data and examinining the output data it produces. Usually some kind of deployment/bootstrap/setup is required first. External systems can be mocked completely, replaced (e.g. using an in-memory database instead of a real one), or the real external dependency might be used depending on the business case. Compared to unit tests they may require more specialized tools either for preparing the test environment, or for interacting/verifying it.

**集成测试**（也被称作服务测试，或是组件测试）专注于整个组件。一个组件，可以是一组类、方法、函数、一个模块、一个子系统甚至是应用本身。它们通过传递输入数据并检查其生成的输出数据来检查组件。通常需要先进行某种部署、引导、安装。外部系统可以完全被模拟，替代（例如使用内存数据库而不是真实数据库），或者实际的外部依赖可能会根据业务情况而使用。

The second category suffers from a blurry definition and most naming controversies regarding testing start here. The “scope” for integration tests is also highly controversial and especially the nature of access to the application ([black](https://en.wikipedia.org/wiki/Black-box_testing) or [white](https://en.wikipedia.org/wiki/White-box_testing) box testing and whether [mocking](https://en.wikipedia.org/wiki/Mock_object) is allowed or not).
第二类受模糊定义的影响, 关于测试的命名争议从这里开始。集成测试的“范围”也是备受争议的。尤其是访问程序的性质（[黑](https://en.wikipedia.org/wiki/Black-box_testing)或是[白](https://en.wikipedia.org/wiki/White-box_testing)盒测试，并且是否允许[模拟](https://en.wikipedia.org/wiki/Mock_object)）

As a basic rule of thumb if
作为一个基本的测试原则，如果

*   a test uses a database  测试中使用了数据库
*   a test uses the network to call another component/application 测试中使用了网络调用另外一个组件或者应用
*   a test uses an external system (e.g. a queue or a mail server) 测试中使用了一个外部系统（例如，一个队列或者邮件服务器）
*   a test reads/writes files or performs other I/O 测试中读写文件或者执行了其他 I/O 操作
*   a test does not rely on the source code but instead it uses the deployed binary of the app 测试不依赖于源代码，而是使用应用程序的部署二进制文件

…then it is an integration test and not a unit test.

那它就是一个集成测试而不是单元测试。

With the naming out of the way, we can dive into the list. The order of anti-patterns roughly follows their appearance in the wild. Frequent problems are gathered in the top positions.


### Software Testing Anti-Pattern List 软件测试反模式的清单
{: #software-testing-anti-pattern-list}

1.  [Having unit tests without integration tests 有单元测试但没有集成测试](#anti-pattern-1---having-unit-tests-without-integration-tests)
2.  [Having integration tests without unit tests 有集成测试但没有单元测试](#anti-pattern-2---having-integration-tests-without-unit-tests)
3.  [Having the wrong kind of tests 使用了错误种类的测试](#anti-pattern-3---having-the-wrong-kind-of-tests)
4.  [Testing the wrong functionality 测试了错误的功能](#anti-pattern-4---testing-the-wrong-functionality)
5.  [Testing internal implementation 测试了内部实现](#anti-pattern-5---testing-internal-implementation)
6.  [Paying excessive attention to test coverage 过度关注测试覆盖](#anti-pattern-6---paying-excessive-attention-to-test-coverage)
7.  [Having flaky or slow tests 使用了有问题的或者慢的测试 ](#anti-pattern-7---having-flaky-or-slow-tests)
8.  [Running tests manually 手工运行测试](#anti-pattern-8---running-tests-manually)
9.  [Treating test code as a second class citizen 把测试当作二等公民](#anti-pattern-9---treating-test-code-as-a-second-class-citizen)
10.  [Not converting production bugs to tests 没有把生产环境的 bug 转化为测试](#anti-pattern-10---not-converting-production-bugs-to-tests)
11.  [Treating TDD as a religion 将 TDD 看作教派](#anti-pattern-11---treating-tdd-as-a-religion)
12.  [Writing tests without reading documentation first 写测试之前不先读文档](#anti-pattern-12---writing-tests-without-reading-documentation-first)
13.  [Giving testing a bad reputation out of ignorance 由于无知而诋毁测试](#anti-pattern-13---giving-testing-a-bad-reputation-out-of-ignorance)

### Anti-Pattern 1 - Having unit tests without integration tests 反模式1 - 有单元测试但没有集成测试
{: #anti-pattern-1---having-unit-tests-without-integration-tests}

This problem is a classic one with small to medium companies. The application that is being developed in the company has only unit tests (the base of the pyramid) and nothing else. Usually lack of integration tests is caused by any of the following issues:

这个问题对于中小型企业而言比较典型。公司里正在开发的软件，除了单元测试（金字塔的底层）就没有别的了。通常缺少集成测试是如下原因导致的：

1.  The company has no senior developers. The team has only junior developers fresh out of college who have only seen unit tests 公司里没有资深的开发人员。团队里只有刚毕业的初级程序员，只了解过单元测试。
2.  Integration tests existed at one point but were abandoned because they caused more trouble than their worth. Unit tests were much more easy to maintain and so they prevailed. 集成测试曾经存在过，但被舍弃了，因为它们造成的麻烦比它们的价值更多。单元测试更容易维护 ，因此它们占了上风。
3.  The running environment of the application is very “challenging” to setup. Features are “tested” in production. 应用运行环境的搭建非常困难。功能都是在生产环境中“测试”的。

I cannot really say anything about the first issue. Every effective team should have at least some kind of mentor/champion that can show good practices to the other members. The second issue is covered in detail in anti-patterns [5](#anti-pattern-5---testing-internal-implementation), [7](#anti-pattern-7---having-flaky-or-slow-tests) and [8](#anti-pattern-8---running-tests-manually).

对于第一个问题我没什么可说的。每个有效的团队都至少有某种意义上的“导师”，来向其他成员展示好的实践。第二个问题在反模式 [5](#anti-pattern-5---testing-internal-implementation), [7](#anti-pattern-7---having-flaky-or-slow-tests) 和 [8](#anti-pattern-8---running-tests-manually) 里有详细的阐述。

This brings us to the last issue - difficulty in setting up a test environment. Now don’t get me wrong, there are indeed some applications that are __really__ hard to test. Once I had to work with a set of REST applications that actually required special hardware on their host machine. This hardware existed only in production, making integration tests very challenging. But this is a corner case.

我们再看最后一个问题 —— 很难搭建测试环境。这里不要误解我的意思，有些应用程序确实 __非常__ 难以 测试。我曾经要处理一些 REST 应用程序，它们需要在主机上装一种特殊的硬件。这种硬件只存在于生产环境中，使得集成测试非常困难。但这是一个极端的情况。

For the run-of-the-mill web or back-end application that the typical company creates, setting up a test environment should be a non-issue. With the appearance of Virtual Machines and lately Containers this is more true than ever. Basically if you are trying to test an application that is hard to setup, you need to fix the setup process first before dealing with the tests themselves.

对于多数公司创建的一般的 Web 程序或后端应用，搭建测试环境应该不是什么问题。随着虚拟机（Virtual Machines）和最近的容器（Containers)的出现，这比以前容易得多。通常，如果你尝试测试一个很难搭建环境的应用，你应该在处理测试之前先解决搭建环境的流程问题。

But why are integration tests essential in the first place?

但是为什么集成测试在一开始就是必不可少的呢？

The truth here is that there are some types of issues that __only__ integration tests can detect. The canonical example is everything that has to do with database operations. Database transactions, database triggers and any stored procedures can only be examined with integration tests that touch them. Any connections to other modules either developed by you or external teams need integration tests (a.k.a. contract tests). Any tests that need to verify performance, are integration tests by definition. Here is a summary on why we need integration tests:

事实是，有些问题（issues）_只有_ 集成测试才能探测到。经典的例子是，所有与数据库操作相关的事情。数据库事务、数据库触发器、和任何存储过程都只能使用集成测试来验证。任何与你或外部团队开发的其他模块的连接都需要集成测试（又称契约测试，contract tests) 。根据定义，任何需要验证性能的测试都是集成测试。以下是我们为什么需要集成测试。

| Type of issue                           | Detected by Unit tests | Detected by Integration tests |
|-----------------------------------------|------------------------|-------------------------------|
| Basic business logic                    | yes                    | yes                           |
| Component integration problems          | no                     | yes                           |
| Transactions                            | no                     | yes                           |
| Database triggers/procedures            | no                     | yes                           |
| Wrong Contracts with other modules/APIs | no                     | yes                           |
| Wrong Contracts with other systems      | no                     | yes                           |
| Performance/Timeouts                    | no                     | yes                           |
| Deadlocks/Livelocks                     | maybe                  | yes                           |
| Cross-cutting Security Concerns         | no                     | yes                           |

| 问题类型                                 | 能否被单元测试探测到       | 能否被集成测试检测到             |
|-----------------------------------------|------------------------|-------------------------------|
| 基本的业务逻辑                            | 能                       | 能                           |
| 组件集成问题                              | 不能                     | 能                           |
| 事务(Transactions)                      | 不能                     | 能                           |
| 数据库触发器/过程                         | 不能                     | 能                           |
| 跟其他模块的错误契约                       | 不能                     | 能                           |
| 与其他系统的错误契约                       | 不能                     | 能                           |
| 性能/超时                                | 不能                     | 能                           |
| 死锁/活锁                                | 有可能                   | 能                           |
| 交叉安全问题                              | 不能                     | 能                 

Basically any cross-cutting concern of your application will require integration tests. With the recent microservice craze integration tests become even more important as you now have contracts between your own services. If those services are developed by other teams, you need an automatic way to verify that interface contracts are not broken. This can only be covered with integration tests.

基本上，你的应用的任何交叉问题（cross-cutting concern）都需要集成测试。随着最近兴起的微服务热潮，集成测试变得更加重要，因为你现在已经在自己的服务之间建立了契约。如果这些服务是其他团队开发的，你需要一个自动化的方式来验证接口的契约没有坏掉。这只能通过集成测试来解决。

To sum up, unless you are creating something extremely isolated (e.g. a command line linux utility), you really **need** integration tests to catch issues not caught by unit tests.

总而言之，除非你要建立非常独立的东西（比如 Linux 命令行工具），否则你非常 **需要** 集成测试来捕捉单元测试发现不了的问题。

### Anti-Pattern 2 - Having integration tests without unit tests 反模式2 - 有集成测试但没有单元测试
{: #anti-pattern-2---having-integration-tests-without-unit-tests}

This is the inverse of the previous anti-pattern. This anti-pattern is more common in large companies and large enterprise projects. Almost always the history behind this anti-pattern involves developers who believe that unit tests have no real value and only integration tests can catch regressions. There is a large majority of experienced developers who consider unit tests a waste of time. Usually if you probe them with questions, you will discover that at some point in the past, upper management had forced them to increase code coverage (See [anti-pattern 6](#anti-pattern-6---paying-excessive-attention-to-test-coverage)) forcing them to write trivial unit tests.

这是上一个反模式的对立面。这种反模式在大型公司和大型企业项目中更为常见。这种反模式背后的历史几乎总是涉及开发人员，他们认为单元测试没有实际价值，只有集成测试才能获得回归。很多经验丰富的开发人员认为单元测试浪费时间。通常，如果你用一些问题进行试探，你会发现过去的某个时刻，高层管理人员强迫他们增加代码覆盖率（参见[反模式 6](#anti-pattern-6---paying-excessive-attention-to-test-coverage)），并迫使他们写一些琐碎的单元测试。

It is true that in theory you __could__ have only integration tests in a software project. But in practice this would become very expensive to test (both in developer time and in build time). We saw in the table of the previous section that integration tests can also find business logic errors after each run, and so they could “replace” unit tests in that manner. But is this strategy viable in the long run?

确实，理论上你 __可以__ 对一个软件项目只写集成测试。但实际上，这使得测试更加昂贵（无论是开发时间还是构建时间）。我们在上一节的表格里看到，集成测试也可以在每次运行之后发现业务逻辑错误，并且他们可以用这样的方式“替换”单元测试。但是这种策略在长远来看是否可行？

#### Integration tests are complex 集成测试很复杂

Let’s look at an example. Assume that you have a service with the following 4 methods/classes/functions.

让我们看一个例子。假设你有一个服务，包含4个方法（或称为类、函数）。

![Cyclomatic complexity for 4 modules](../assets/2018-05-17-software-testing-anti-patterns/39740330-bac15b38-52c7-11e8-831e-90c1e2b36456.png)

The number on each module denotes its [cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity) or in other words the separate code paths this module can take.

每个模块上的数字表示其圈复杂度（[cyclomatic complexity](https://en.wikipedia.org/wiki/Cyclomatic_complexity)），换句话说，就是这个模块所包含的独立代码路径。

Mary “by the book” Developer wants to write unit tests for this service (because she understands that unit tests __do__ have value). How many tests does she need to write in order to get full coverage of all possible scenarios?

Mary 是一个“书呆子”程序员，她想为这个服务编写单元测试（因为她知道单元测试 __确实__ 很有价值）。她需要写多少测试，才能覆盖所有可能的场景？

It should be obvious that one can write 2 + 5 + 3 + 2 = 12 isolated unit tests that cover fully the **business logic** of these modules. Remember that this number is just for a single service, and the application Mary is working on, has multiple services.

很显然，可以编写 2 + 5 + 3 + 2 = 12 个独立的单元测试，来完全覆盖这些模块的 **业务逻辑**。注意，这个数量仅仅是为了写一个服务的测试。并且 Mary 所开发的应用程序有多个服务。

Joe “Grumpy” developer on the other hand does not believe in the value of unit tests. He thinks that unit tests are a waste of time and he decides to write only integration tests for this module. How many integration tests should he write? He starts looking at all the possible paths a request can take in that service.

另一方面，Joe 是个 “脾气暴躁” 的程序员，他不相信单元测试的价值。他认为单元测试是浪费时间，并决定为这个模块只编写集成测试。他应该写多少集成测试呢？他开始查看一个请求能在这个服务中走出的所有可能路径。

![Examining code paths in a service](../assets/2018-05-17-software-testing-anti-patterns/39740332-bd592f10-52c7-11e8-8d91-6dcca63b7c59.png)

Again it should be obvious that all possible scenarios of codepaths are 2 * 5 * 3 * 2 = 60. Does that mean that Joe will actually write 60 integration tests? Of course not! He will try and cheat. He will try to select a subset of integration tests that feel “representative”. This “representative” subset of tests will give him enough coverage with the minimum amount of effort.

同样显而易见的是，代码路径的所有可能场景都是 2 * 5 * 3  2 = 60。这是否意味着 Joe 需要写 60 个集成测试？当然不！他会试着偷懒。他会尝试选择集成测试的一个看似“有代表性的”子集。这个“有代表性的”测试子集将以最小的努力获得足够的覆盖率。

This sounds easy enough in theory, but can quickly become problematic. The reality is that these 60 code paths are not created equally. Some of them are corner cases. For example if we look at module C we see that is has 3 different code paths. One of them is a very special case, that can only be recreated if C gets a special input from component B, which is itself a corner case and can only be obtained by a special input from component A. This means that this particular scenario might require a very complex setup in order to select the inputs that will trigger the special condition on the component C.

这在理论上听起来很容易，但很快就会引发问题。实际情况是，这60个代码路径不是一样重要的。有些是边界情况。比如，如果我们看模块 C，能发现它有 3 个不同的代码路径。其中一个是很特殊的情况，只有在 C 从 B 得到特殊的输入值时才能重现，而这个特殊值也是一个很特殊的情况，只有在 B 从 A 得到特殊输入值时才能获得。

Mary on the other hand, can just recreate the corner case with a simple unit test, with no added complexity at all.

另一方面，Mary 可以通过一个简单的单元测试就能重现边界情况，完全不会增加任何复杂度。

![Basic unit test](../assets/2018-05-17-software-testing-anti-patterns/39740336-c1b589d2-52c7-11e8-9186-276adf0c32f1.png)

Does that mean that Mary will __only__ write unit tests for this service? After all that will lead her to [anti-pattern 1](#anti-pattern-1---having-unit-tests-without-integration-tests). To avoid this, she will write __both__ unit __and__ integration tests. She will keep all unit tests for the actual business logic and then she will write 1 or 2 integration tests that make sure that the rest of the system works as expected (i.e. the parts that help these modules do their job)

这是否以为着 Mary 将 __只__ 为这个服务写单元测试呢？这将导致她达成[反模式 1](#anti-pattern-1---having-unit-tests-without-integration-tests)。为了避免这种情况，她将保留针对实际业务逻辑所写的所有单元测试，再写 1 到 2 个集成测试，确保系统的其他部分也如期工作（例如，帮助这些模块完成工作的部分）。

The integration tests needed in this system should focus on the rest of the components. The business logic itself can be handled by the unit tests. Mary’s integration tests will focus on testing serialization/deserialization and with the communication to the queue and the database of the system.

这个系统里需要的集成测试应该侧重于其他组件。业务逻辑本身可以被单元测试所涵盖。Mary 的集成测试会侧重于测试序列化/反序列化，以及与队列和数据库的通信。

![correct Integration tests](../assets/2018-05-17-software-testing-anti-patterns/39740343-c5134d30-52c7-11e8-8604-359d88898c66.png)

In the end, the number of integration tests will be much smaller than the number of unit tests (matching the shape of the test pyramid described in the first section of this article).

最后，集成测试的数量应该远小于单元测试的数量（与本文第一部分中描述的测试金字塔的形状相匹配）。

#### Integration tests are slow 集成测试很慢

The second big issue with integration tests apart from their complexity is their speed. Usually an integration test is one order of magnitute slower than a unit test. Unit tests need just the source code of the application and nothing else. They are almost always CPU bound. Integration tests on the other hand can perform I/O with external systems making them much more difficult to run in an effective manner.

除了复杂度之外，集成测试的第二大问题是它们的速度。通常，集成测试比单元测试慢一个数量级。单元测试只依赖应用程序的源代码而不需要其他内容。他们几乎只受到 CPU 的限制。另一方面，集成测试需要与外部系统进行 I/O 操作，使得它们难以更高效地运行。

Just to get an idea on the difference for the running time let’s assume the following numbers.

为了对运行时间的差异有一个直观感受，我们假设以下数字：

*   Each unit test takes 60ms (on average) 每个单元测试需要花 60ms （平均）
*   Each integration test takes 800ms (on average) 每个集成测试需要花 800ms （平均）
*   The application has 40 services like the one shown in the previous section 应用程序有 40 个像上节所述的服务
*   Mary is writing 10 unit tests and 2 integration tests for each service Mary 正在为每个服务编写 10 个单元测试和 2 个集成测试
*   Joe is writing 12 integration tests for each service Joe 正在为每个服务编写 12 个集成测试

Now let’s do the calculations. Notice that I assume that Joe has found the perfect subset of integration tests that give him the same code coverage as Mary (which would not be true in a real application).

现在让我们做个计算。请注意，我假设 Joe 已经找到了完美的集成测试子集，让他达到和 Mary 一样的代码覆盖率（在实际应用程序中不是这样）。

| Time to run            | Having only integration tests (Joe) | Having both Unit and Integration tests (Mary) |
|------------------------|-------------------------------------|-----------------------------------------------|
| Just Unit tests        | N/A                                 | 24 seconds                                    |
| Just Integration tests | 6.4 minutes                         | 64 seconds                                    |
| All tests              | 6.4 minutes                         | 1.4 minutes                                   |


| 运行时间          | 只有集成测试 (Joe)    | 既有单元测试也有集成测试 (Mary) |
|------------------|----------------------|-------------------------------|
| 仅单元测试        | N/A                  | 24 秒                         |
| 仅集成测试        | 6.4 分钟             | 64 秒                          |
| 所有的测试        | 6.4 分钟             | 1.4 分钟                       |

The difference in total running time is enormous. Waiting for 1 minute after each code change is vastly different than waiting for 6 minutes. The 800ms I assumed for each integration test is vastly conservative. I have seen integration test suites where a single test can take several minutes on its own.

总运行时间的差异是巨大的。每次代码更改后等待 1 分钟与等待 6 分钟大不相同。我假设每次集成测试的 800 ms 是非常保守的。我见过有些集成测试集，其中的一个测试就要花费几分钟。

In summary, trying to use __only__ integration tests to cover business logic is a huge time sink. Even if you automate the tests with CI, your feedback loop (time from commit to getting back the test result) will be very long.

总之，尝试 __只__ 使用集成测试来覆盖业务逻辑是巨大的时间漏斗。即使你使用 CI 来自动执行测试，您的反馈循环（从提交到返回测试结果的时间）都将非常长。

#### Integration tests are harder to debug than unit tests 集成测试比单元测试更难调试

The last reason why having only integration tests (without any unit tests) is an anti-pattern is the amount of time spent to debug a failed test. Since an integration test is testing multiple software components (by definition), when it breaks, the failure can come from __any__ of the tested components. Pinpointing the problem can be a hard task depending on the number of components involved.

只有集成测试（而没有单元测试）是反模式的原因之一，是调试一个失败的集成测试要花费大量的时间。因为根据其定义，集成测试是测试多个软件组件，所以当测试出错时，故障可能来自于 _任何_ 被测试到的组件。根据所涉及组件数量的多少，要想确定问题可能是很困难。

When an integration tests fails you need to be able to understand why it failed and how to fix it. The complexity and breadth of integration tests make them extremely difficult to debug. Again, as an example let’s say that your application only has integration tests. The application you are developing is the typical e-shop.

当集成测试失败时，您需要能够理解失败的原因以及如何修复它。集成测试的复杂性和广度使得调试极其困难。现在，再举一个例子，假设你正在开发的应用程序只有集成测试，并且它是一个典型的网上商城。

A developer in your team (or even you) creates a new commit, which triggers the integration tests with the following result:

团队中的开发人员（甚至是您）创建一个新的提交（commit），这会触发集成测试，并产生如下结果：

![breakage of integration tests](../assets/2018-05-17-software-testing-anti-patterns/39740398-011930ce-52c8-11e8-8e7f-cbfeb2d261d7.png)

As a developer you look at the test result and see that the integration test named “Customer buys item” is broken. In the context of an e-shop application this is not very helpful. There are many reasons why this test might be broken.

作为一个开发人员，您可以查看测试结果，并看到一个名为“客户购买物品”的集成测试出错了。在网上商城这样一个情景下，这不是很有帮助。这个测试出错，原因可能很多。

There is no way to know why the test broke without diving into the logs and metrics of the test environment (assuming that they can pinpoint the problem). In several cases (and more complex applications) the only way to truly debug an integration test is to checkout the code, recreate the test environment locally, then run the integration tests and see it fail in the local development environment.

除了深入了解测试环境的日志和指标（假设它们可以查明问题），没有其他办法来弄明白测试出错的原因。在很多情况下（以及更复杂的程序中），唯一真正调试集成测试的办法是，检出（checkout）代码，在本地重新模拟测试环境，然后运行集成测试，并检查它在本地开发环境下有没有出错。

Now imagine that you work with Mary on this application so you have both integration and unit tests. Your team makes some commits, you run all the tests and get the following:

现在想象一下，你和 Mary 一起开发这个应用程序，所以你既有集成测试，也有单元测试。你的团队进行了一些代码提交，你运行了所有的测试，并得到如下结果：

![breakage of both kinds of tests](../assets/2018-05-17-software-testing-anti-patterns/39740401-0594d824-52c8-11e8-81e1-6eba0883547d.png)

Now two tests are broken:

现在有两个测试出错了：

*   “Customer buys item” is broken as before (integration test) “客户购买物品”和之前一样出错了（集成测试）
*   “Special discount test” is also broken (unit test) “特价折扣测试”也出错了（单元测试）

It is now very easy to see the where to start looking for the problem. You can go directly to the source code of the __Discount__ functionality, locate the bug and fix it and in 99% of the cases the integration test will be fixed as well.

现在就很容易知道应该从哪里开始寻找问题。你应该直接进到 _折扣_ 功能的源代码，找到 bug 并修复它，有 99% 的可能集成测试也顺带被修复了。

Having unit tests break _before_ or _with_ integration tests is a much more painless process when you need to locate a bug.

当你需要定位 bug 的时候，在集成测试 _之前_ 或者 _之后_ 让单元测试出错，是一个更加无痛的过程。 

#### Quick summary of why you need unit tests 小结：你为什么需要单元测试

This is the longest section of this article, but I consider it very important. In summary while __in theory__ you could only have integration tests, __in practice__

这是本文最长的一节，但我认为这非常重要。总而言之，__理论上__ 你可以只写集成测试，但 __实践中__

1.  Unit tests are easier to maintain 单元测试更容易维护
2.  Unit tests can easily replicate corner cases and not-so-frequent scenario 单元测试能更好地覆盖边界情况和不常见的场景
3.  Unit tests run much faster than integration tests 集成测试比单元测试运行得快得多
4.  Broken unit tests are easier to fix than broken integration tests 有问题的单元测试比有问题的集成测试更容易修复

If you only have integration tests, you waste developer time and company money. You need **both** unit and integration tests are the same time. They are not mutually exclusive. There are several articles on the internet that advocate using only one type of tests. All these articles are misinformed. Sad but true.

如果你只进行集成测试，则会浪费开发人员的时间和公司资金。你需要 **同时** 进行单元测试和集成测试。它们并不互斥。网上有几篇文章主张只使用一种类型的测试。所有这些文章都是被误导的。很不幸，但确实是这样。

### Anti-Pattern 3 - Having the wrong kind of tests 反模式3 - 采用了错误类型的测试
{: #anti-pattern-3---having-the-wrong-kind-of-tests}

Now that we have seen why we need both kinds of tests (unit __and__ integration), we need to decide on __how many__ tests we need from each category.

现在我们已经看到为什么我们需要这两种测试（ 单元测试 _和_ 集成测试），我们需要决定，每一类我们分别需要的 _多少_ 测试。

There is no hard and fast rule here, it depends on your application. The important point is that you need to spend some time to understand what type of tests add the most value to _your_ application. The test pyramid is only a suggestion on the amount of tests that you should create. It assumes that you are writing a commercial web application, but that is not always the case. Let’s see some examples:

这里没有硬性规定，这取决于你的应用。 重要的是，你需要花一些时间来了解什么类型的测试能为 _你的_ 应用带来最大的价值。 测试金字塔只是你应该创建的测试数量的一个建议。 它假定你正在编写一个商业 Web 应用，但情况并非总是如此。 我们来看一些例子：

#### Example - Linux command line utility 例子 - Linux 命令行工具

Your application is a command line utility. It reads one special format of a file (let’s say a CSV) and exports another format (let’s say JSON) after doing some transformations. The application is self-contained, does not communicate with any other system or use the network. The transformations are complex mathematical processes that are critical for the correct functionality of the application (it should always be correct even if it slow).

你的应用是一个命令行工具。 它读取一种特殊的文件格式（比如 CSV ），并在做一些转换后导出另一种格式（比如 JSON ）。 该应用是独立的，不与任何其他系统通信或使用网络。 转换是复杂的数学过程，对于应用的正确功能至关重要（即使速度缓慢，它也应该始终正确）。

In this contrived example you would need:

在这个编造的例子中，你需要：

*   Lots and lots of unit tests for the mathematical equations. 针对数学方程式的大量单元测试
*   Some integration tests for the CSV reading and JSON writing 针对 CSV 读取和写入的一些集成测试
*   No GUI tests because there is no GUI. 没有 GUI（图形用户界面）测试，因为没有 GUI。

Here is the breakdown of tests for this project:

以下是本项目的测试细目：

![Test pyramid example](../assets/2018-05-17-software-testing-anti-patterns/40963485-4cea4c12-68db-11e8-8964-11074f922edf.png)

Unit tests dominate in this example and the shape is **not** a pyramid.

单元测试在这个例子里占主导，形状并**不是**一个金字塔。

#### Example - Payment Management 例子 - 支付管理

You are adding a new application that will be inserted into an existing big collection of enterprise systems. The application is a payment gateway that processes payment information for an external system. This new application should keep a log of all transactions to an external DB, it should communicate with external payment providers (e.g. Paypal, Stripe, WorldPay) and it should also send payment details to another system that prepares invoices.

你现在要新写一个应用，它被集成到一个已有的大规模的企业系统中。这个应用是一个支付网关，用来为一个外部系统处理支付信息。这个新应用需要保留全量的事务日志到外部的数据库，它应该和外部支付提供商（例如 Paypal、Stripe 或 WorldPay）通信，并应该发送交易详情到准备发票的另一个系统。

In this contrived example you would need

在这个编造的例子中，你需要：

*   Almost no unit tests because there is no business logic  几乎不需要单元测试，因为没有业务逻辑
*   Lots and lots of integration tests for the external communications, the db storage, the invoice system 需要为外部通信、数据库存储和发票系统写大量的集成测试
*   No UI Tests because there is a no UI 没有 UI 测试，因为没有用户界面

Here is the breakdown of tests for this project:

以下是本项目的测试细目：

![Test pyramid example](../assets/2018-05-17-software-testing-anti-patterns/40817728-84bd14f6-6586-11e8-9fd4-5207632d5936.png)

Integrations tests dominate in this example and the shape is **not** a pyramid.

集成测试在这个例子中占主导地位，形状**并不是**一个金字塔。

#### Example - Website creator 例子 - 网站创建器

You are working on this brand new startup that will revolutionize the way people create websites, by offering a one-of-a-kind way to create web applications from within the browser.

你现在正在为一家全新的创业公司工作，该公司通过提供一个独一无二的方式从浏览器里创建 Web 应用，将颠覆人们创建网站的方式，

The application is a graphical designer with a toolbox of all the possible HTML elements that can be added on a web page along with library of premade templates. There is also the ability to get new templates from a marketplace. The website creator works in a very friendly way by allowing you to drag and drop components on the page, resize them, edit their properties and change their colors and appearance.

该应用是一个图形设计器，包含几乎所有可以加入到网页中的 HTML 元素的工具箱，以及预制的模板库，还有从市场获得新模板的能力。这个网站创建器以非常友好的方式工作，并允许你在页面上拖放组件、调整尺寸、编辑属性并更改他们的颜色和外观。

In this contrived example you would need

在这个编造的例子中，你需要

*   Almost no unit tests because there is no business logic 几乎没有单元测试，因为没有业务逻辑
*   Some integration tests for the marketplace 针对市场的一些集成测试
*   Lots and lots of UI tests that make sure the user experience is as advertised 大量的 UI 测试，确保用户体验和广告一致

Here is the breakdown of tests for this project:

以下是本项目的测试细目：

![Test pyramid example](../assets/2018-05-17-software-testing-anti-patterns/40817730-8a73e438-6586-11e8-911e-adb6cb0a9487.png)

UI tests dominate here and the shape is **not** a pyramid.

UI 测试在这里占主导地位，形状**不是**金字塔。

I used some extreme examples to illustrate the point that you need to understand what your application needs and focus only on the tests that give you value. I have personally seen “payment management” applications with no integration tests and “website creator” applications with no UI tests.

我使用了一些极端的例子来说明，你要明白你的应用需要什么，并专注于能带给你价值的测试。我亲眼看到过没有集成测试的“支付管理”应用和没有 UI 测试的 “网站创造器”应用。

There are several articles on the web (I am not going to link them) that talk about a specific amount on integration/unit/UI tests that you need or don’t need. All these articles are based on assumptions that may _not_ be true in your case.

网上有一些文章（我不会贴出来这些链接），这些文章谈论了集成测试/单元测试/UI测试的具体数量。这些文章都是基于假设得出的，并**不**一定适用于你的具体情况。

### Anti-Pattern 4 - Testing the wrong functionality 反模式4 - 测试了错误的功能
{: #anti-pattern-4---testing-the-wrong-functionality}

In the previous sections we have outlined the types and amount of tests you need to have for your application. The next logical step is to explain what functionality you actually need to test.

在上一节中，我们概述了你的应用程序所需要的测试类型和数量。下一个合乎逻辑的步骤是，解释哪些功能才是需要测试的。

In theory, getting 100% code coverage in an application is the ultimate goal. In practice this goal is not only difficult to achieve but also it doesn’t guarantee a bug free application.

理论上，达到 100% 的测试覆盖率是最终的目标。但实际上，这个目标不仅很难实现，还不能保证你的应用程序完全没有 bug。

There are some cases where indeed it is possible to test __all__ functionality of your application. If you start on a green-field project, if you work in a small team that is well behaved and takes into account the effort required for tests, it is perfectly fine to write new tests for all new functionality you add (because the existing code already has tests).

在某些情况下，确实有可能测试到应用程序的 __全部__ 功能。如果你开始于一个绿色的项目（译者注：这里应该指测试全部能跑通过），如果你在一个表现良好的小团队工作，并将测试所需的工作量考虑其中，那么为你所新加的功能写新测试是完全正常的（因为现有的代码已经有测试）。

But not all developers are lucky like this. In most cases you inherit an existing application that has a minimal amount of tests (or even none!). If you are part of a big and established company, working with legacy code is mostly the rule rather than the exception.

但不是所有的开发者都这么幸运。在大多数情况下，你接手的是一个有很少测试（甚至一点测试都没有！）的已有项目。如果你是一个大公司的一员，那么接触遗留旧代码是再正常不过的了。

Ideally you would have enough development time to write tests for both new and existing code for a legacy application. This is a romantic idea that will probably be rejected by the average project manager who is mostly interested on adding new features rather then testing/refactoring. You have to pick your battles and find a fine balance between adding new functionality (as requested by the business) and expanding the existing test suite.

理想情况下，你会有充足的开发时间来为一个旧项目的新代码和现有代码编写测试。这实在是一个浪漫的想法！大多数项目经理都会拒绝这样做，因为他们感兴趣的是添加新功能，而不是测试或重构。你必须选择选择你的战斗，并在添加新功能（根据业务方的要求）和扩展现有测试集直接找到一个好的平衡点。

So what do you test? Where do you focus your efforts? Several times I have seen developers wasting valuable testing time by writing “unit tests” that add little or no value to the overall stability of the application. The canonical example of useless testing is trivial tests that verify the application data model.

所以你要测试什么呢？你应该将精力集中到哪里？有几次，我见到开发者浪费宝贵的时间来写所谓的“单元测试”，但这些测试对应用程序的整体稳定性没有任何价值。无用测试的典型示例是写琐碎的测试来验证应用程序数据模型。

Code coverage is analyzed in detail in its own [anti-pattern](#anti-pattern-6---paying-excessive-attention-to-test-coverage) section. In the present section however we will talk about code “severity” and how it relates to your tests.

代码覆盖率在其自己的 [反模式6](#anti-pattern-6--paying-excessive-attention-to-test-coverage) 一节中被详细地分析过。在本节中，我们将讨论代码“严重性”已经它与测试直接的关系。

If you ask any developer to show you the source code of any application, he/she will probably open an IDE or code repository browser and show you the individual folders.

如果你让任何一个开发人员展示给你任何一个应用程序的源代码，他/她应该会打开 IDE 或代码仓库浏览器，展示给你每个文件夹。

![Source code physical model](../assets/2018-05-17-software-testing-anti-patterns/42945375-d5687082-8b9a-11e8-9a47-550c7bda04c5.png)

This representation is the physical model of the code. It defines the folders in the filesystem that contain the source code. While this hierarchy of folders is great for working with the code itself, unfortunately it doesn’t define the importance of each code folder. A flat list of code folders implies that all code components contained in them are of equal importance.

这种方式所展示的是代码的物理模型，它定义了文件系统中包含源代码的文件夹。尽管这种文件夹层次结构非常适合处理代码本身，但遗憾的是，它没有定义每个代码文件夹的重要性。代码文件夹的平面列表，意味着包含在其中的所有代码组件具有同等的重要性。

This is not true as different code components have a different impact in the overall functionality of the application. As a quick example let’s say that you are writing an eshop application and two bugs appear in production:

然而，不同的代码组件对于应用程序的整体功能却有不同的影响。举个简单的例子，你在编写一个电子商城的应用程序，并在生产系统中发现 两个 bug：

1.  Customers cannot check-out their cart halting all sales 顾客无法结帐，因此所有销售被中断
2.  Customers get wrong recommendations when they browse products. 顾客在浏览商品时得到了错误的推荐

Even though both bugs should be fixed, it is obvious that the first one has higher priority. Therefore if you inherit an eshop application with zero tests, you should write new tests the directly validate the check-out functionality rather than the recommendation engine. Despite the fact that the recommendation engine and the check-out process might exist on sibling folders in the filesystem, their importance is different when it comes to testing.

尽管两个 bug 都得修复，但很明显第一个 bug 具有更高的优先级。因此，如果你接手了一个没有任何测试的电子商城应用程序，你应该写一些新测试来直接验证结帐功能，而非推荐系统。尽管推荐系统和结帐过程可能存在于文件系统的同级目录，但在测试时，它们的重要性是不同的。

To generalize this example, if you work for some time in any medium/large application you will soon need to think about code using a different representation - the mental model.

概括一下这个例子，如果你编写过任何中型或大型应用程序，你很快就需要使用不同的表征，即心智模型，来思考代码。

![Source code mental model 代码心智模型](../assets/2018-05-17-software-testing-anti-patterns/42945417-ee804d60-8b9a-11e8-8e10-666fb5a18e82.png)

I am showing here 3 layers of code, but depending on the size of your application it might have more. These are:

我在这里展示了代码的三个层面，但根据应用程序的大小，它可能包含更多层。这些是：

1.  Critical code - This is the code that breaks often, gets most of new features and has a big impact on application users 关键代码 —— 这些代码容易出问题，提供大多数新功能，并对用户有重大影响。
2.  Core code - This is the code that breaks sometimes, gets few new features and has medium impact on the application users 核心代码 —— 这些代码有时会出问题，提供较少的新功能，并对用户有中等影响。
3.  Other code - This is code that rarely changes, rarely gets new features and has minimal impact on application users. 其他代码 —— 这些代码很少被更改，很少获得新功能，对用户的影响最小。

This mental mode should be your guiding principle whenever you write a new software test. Ask yourself if the functionality you are writing tests for now belongs to the __critical__ or __core__ categories. If yes, then write a software test. If no, then maybe your development time should better be spent elsewhere (e.g. in another bug).

每当你编写新的软件测试时，这个心智模型应该是你的指导原则。问问自己，你正在写测试的功能是否属于 __关键__ 或 __核心__ 类别。如果是，就编写软件测试。如果不是，也许你的开发时间应该更好地花在别处（例如另一个 bug）。

The concept of having code with different severity categories is also great when you need to answer the age old question of how much code coverage is enough for an application. To answer this question you need to either know the severity layers of the application or ask somebody that does. Once you have this information at hand the answer is obvious:

当你要回答这一古老的问题 —— 多少代码覆盖率是足够的，你心里对于代码的重要程度能做到区别对待也是很有用的。要回答这个问题，你要么自己了解应用程序的重要度分层，或者问别人。一旦掌握了这些信息，答案是显而易见的：

Try to write tests that work towards 100% coverage **of critical code**. If you have already done this, then try to write tests that work towards 100% **of core code**. Trying however to get 100% coverage on __total__ code [is not recommended](#anti-pattern-6---paying-excessive-attention-to-test-coverage).

尝试编写测试，达到 100% 的 **关键代码** 覆盖率。如果您已经做到了这一点，那么尝试编写测试以达到 100% 的 **核心代码** 覆盖率。然而，尝试达到 100% 的 __所有代码__ 的覆盖率是 [不推荐的](#anti-pattern-6---paying-excessive-attention-to-test-coverage)。

The important thing to notice here is that the critical code in an application is always a small subset of the overall code. So if in an application critical code is let’s say 20% of the overall code, then getting just 20% overall code coverage is a good first step for reducing bugs in production.

这里需要注意的重要一点是，应用程序中的关键代码始终是总体代码的一小部分。假设一个应用程序的关键代码占总体代码的 20%，那么做到这 20% 的代码被覆盖是减少生产系统中 bug 的至关重要的第一步。

In summary, write unit and integration tests for code that 总之，为以下代码编写单元和集成测试

*   breaks often 经常出问题的
*   changes often 经常改动的
*   is critical to the business 对于业务至关重要的

If you have the time luxury to further expand the test suite, make sure that you understand the diminishing returns before wasting time on tests with little or no value.

如果你有大把大把的时间来扩充测试集，请在浪费时间写那些几乎没有价值的测试之前，确保你明白测试的收益递减性。

### Anti-Pattern 5 - Testing internal implementation 反模式5 - 测试了内部实现
{: #anti-pattern-5---testing-internal-implementation}

More tests are always a good thing. Right?

测试越多越好，不是吗？

Wrong! You also need to make sure that the tests are actually structured in a correct way. Having tests that are written in the wrong manner is bad in two ways.

错！你需要确保测试的结构是正确的。用错误的方式写测试会有两个坏处。

*   They waste precious development time the first time they are written 第一次写测试的时候就浪费了宝贵的开发时间。
*   They waste even more time when they need to be refactored (when a new feature is added) 当测试需要被重构（当新功能被加进来）的时候，会浪费更多的时间。

Strictly speaking, [test code is like any other type of code](#anti-pattern-9---treating-test-code-as-a-second-class-citizen). You will need to refactor it at some point in order to improve it in a gradual way. But if you find yourself routinely changing existing tests just to make them pass when a new feature is added then __your tests are not testing what they should be testing__.

严格地来说，[测试代码和其他类型的代码一样](#anti-pattern-9---treating-test-code-as-a-second-class-citizen)。你需要在某个时候重构它，以便逐步改进它。但是，如果你发现，当新功能加入时，自己经常更改现有的测试来保证原有的测试可以通过，那么 __你的测试并不是在测试那些应该被测试的部分__。

I have seen several companies that started new projects and thinking that they will get it right this time, they started writing a __big__ number of tests to cover the functionality of the application. After a while, a new feature got added and several existing tests needed to change in order to make them pass again. Then another new feature was added and __more__ tests needed to be updated. Soon the amount of effort spent refactoring/fixing the existing tests was actually larger than the time needed to implement the feature itself.

我曾经看到几家公司开始了新的项目，他们觉得这次终于要做正确的事情了，他们开始写 __一大堆__ 测试来覆盖应用程序的功能。过了一段时间，一个新功能被加进来了，部分现有的测试需要被调整，以保证它们依旧通过。然后，又有一个新功能被加进来了，__更多__ 的测试需要被调整。不久，用来重构或修复现有测试的工作量实际上已经大于实现该功能本身所需的时间。

In such situations, several developers just accept defeat. They declare software tests a waste of time and abandon completely the existing test suite in order to focus fully on new features. In some extreme scenarios some changes might even be held back because of the amount of tests that break.

在这种情况下，一些开发人员只能接受失败。他们宣布软件测试浪费时间，并彻底地放弃现有的测试集，从而完全专注于开发新功能。在极端情况下，一些变动甚至可能被延期，因为他们导致大量的测试出错。

The problem here is of course the bad quality of tests. Tests that need to be refactored all the time suffer from tight coupling with the main code. Unfortunately, you need some basic testing experience to understand which tests are written in this “wrong” way.

这里的问题当然是测试的质量差。需要重构的测试总是受制于跟主要代码强耦合。不过，你需要一些基本的测试经验，来理解哪些测试才是用这种“错误”方式编写的。

Having to change a big number of existing tests when a new feature is introduced shows the __symptom__. The actual problem is that tests were instructed to verify internal implementation which is always a recipe for disaster. There are several software testing resources online that attempt to explain this concept, but very few of them show some solid examples.

在引入新功能时不得不修改大量的已有测试，暴露出了 __病症__。实际的问题是，测试被用来验证内部实现，就一定会引发一场灾难。网上有很多软件测试资源试图解释这个概念，但很少能给出可靠的例子。

I promised in the beginning of this article that I will not speak about a particular programming language and I intend to keep that promise. In this section the illustrations show the data structure of your favorite programming language. Think of them as structs/objects/classes that contain fields/values.

我在文章开头就承诺过，我不会谈论某种特定的编程语言，我打算遵守承诺。在这一部分里，插图展示了你最喜爱的编程语言的数据结构。把他们想象成包含字段/值的结构体/对象/类。 

Let’s say that the customer object in an e-shop application is the following:

让我们来假设，一个网上商城的应用程序的`customer` （客户）对象是这样的。

![Tight coupling of tests](../assets/2018-05-17-software-testing-anti-patterns/40220485-dc005e0c-5aab-11e8-9f08-c5020aef4fc5.png)

The customer type has only two values where `0` means “guest user” and `1` means “registered user”. Developers look at the object and write 10 unit tests that verify various cases of guests users and 10 cases of registered user. And when I say “verify” I mean that tests **are looking at this particular field in this particular object**.

客户的 type（种类）只有两个值，`0` 代表“guest user（访客用户）”，`1`代表“registered user（注册用户）”。开发人员看到这个对象，就写了 10 个单元测试来验证访客用户的各种情况。这里，我说的“verify（验证）”指的是 **查看某个对象的某个特定字段**。

Time passes by and business decides that a new customer type with value `2` is needed for affiliates. Developers add 10 more tests that deal with affiliates. Finally another type of user called “premium customer” is added and developers add 10 more tests.

时光飞逝，业务方决定需要为附属用户（affiliates）设定一个值为`2`的新的客户类型。开发人员针对性地又添加了另外10个测试。最后，另外一种叫“高级客户”的用户类型被添加，开发人员又针对性地添加了10个测试。

At this point, we have 40 tests in 4 categories that all look at this particular field. (These numbers are imaginary. This contrived example exists only for demonstration purposes. In a real project you might have 10 interconnected fields within 6 nested objects and 200 tests).

截至目前，我们为 4 个类别写了 40 个测试，都检查的是同一个字段。（这些数字是虚构的，这种编造的例子只是为了演示目的。在真正的项目中，你可能有 10 个相互关联的字段，分别在 6 个嵌套对象中，有 200 个测试）。

![Tight coupling of tests example](../assets/2018-05-17-software-testing-anti-patterns/40220488-df4618e0-5aab-11e8-900f-df054dbe28ee.png)

If you are a seasoned developer you can always imagine what happens next. New requirements come that say:

如果你是一名经验丰富的开发人员，你总能想想接下来会发生什么。有新的需求如下：

1.  For registered users, their email should also be stored 对于注册用户，他们的 email 也应该被存储下来。
2.  For affiliate users, their company should also be stored 对于附属用户，他们的公司也应该被存储下来。
3.  Premium users can now gather reward points. 高级用户现在可以积攒奖励积分。

The customer object now changes as below:

客户对象现在变成了这样:

![Tight coupling of tests broken](../assets/2018-05-17-software-testing-anti-patterns/40220493-e587f1f6-5aab-11e8-9fa4-06d6232e18c6.png)

You now have 4 objects connected with foreign keys and all 40 tests are instantly broken because the field they were checking no longer exists.

您现在有4个对象，通过外键相连接，并且所有40个测试会立即出错，因为它们检查的字段不再存在。

__Of course in this trivial example one could simply keep the existing field to not break backwards compatibility with tests__. In a real application this is not always possible. Sometimes backwards compatibility might essentially mean that you need to keep both old and new code (before/after the new feature) resulting in a huge bloat. Also notice that having to keep old code around just to make unit tests pass is a huge anti-pattern on its own.

__当然，在这个小例子中，可以简单地保留现有的字段，从而不影响对测试的向后兼容__。在实际应用程序中，这并非总是可行的。有时向后兼容可能意味着你需要（在新功能开发之前或者之后）让新代码和旧代码共存，这会导致代码急剧膨胀。还要注意的是，为了使单元测试通过而保留旧代码，是一个巨大的反模式。


In a real application when this happens, developers ask from management some extra time to fix the tests. Project managers then declare that unit testing is a waste of time because they seem to hinder new features. The whole team then abandons the test suite by quickly disabling the failing tests.

在真实的应用程序中，如果这样的事情发生，开发人员会请求管理人员给一些额外的时间来修复测试。然后，项目经理宣称单元测试是浪费时间，因为它们似乎妨碍了新功能的开发。最终，整个团队通过禁用那些失败的测试，放弃了测试集。

The big problem here is not testing, but instead the way the tests were constructed. Instead of testing internal implementation they should instead expected behavior. In our simple example instead of testing directly the internal structure of the customer they should instead check the exact business requirement of each case. Here is how these same tests should be handled instead.

这里的大问题不是测试，而是测试构建的方式。应该测试预期的行为，而不是测试内部实现。在我们的简单示例中，不应该直接测试客户的内部结构，而应该检查每种情况的确切业务需求。下面展示了这些测试应该如何正确构建。

![Tests that test behavior](../assets/2018-05-17-software-testing-anti-patterns/40220498-ee7d35f0-5aab-11e8-924a-54378cb5c0a4.png)

The tests do not really care about the internal structure of the customer object. They only care about its interactions with other objects/methods/functions. The other objects/method/functions should be mocked when needed on a case to case basis. Notice that each type of tests directly maps to a business need rather than a technical implementation (which is always a good practice.)

测试并不关心 `customer` 对象的内部结构。他们只关心它与其他对象、方法或函数的交互。其他的对象、方法或函数应该在需要时根据具体情况进行 mock。请注意，每种类型的测试都直接映射到业务需求，而不是技术实现（这一定是一种好的做法）。

If the internal implementation of the __Customer__ object changes, the verification code of the tests remains the same. The only thing that might change is the setup code for each test, which should be centralized in a single helper function called `createSampleCustomer()` or something similar (more on this in [AntiPattern 9](#anti-pattern-9---treating-test-code-as-a-second-class-citizen))

如果 __Customer__ 对象的内部实现发生更改，则测试的验证（verification）部分保持不变。唯一可能改变的是每个测试的准备（setup）代码，它应该集中在一个名为`createSampleCustomer（）`的辅助函数或类似的东西（更多关于[反模式9](#anti-pattern-9---treating-test-code-as-a-second-class-citizen)）

Of course in theory it is possible for the verified objects themselves to change. In practice it is not realistic for changes to happen at `loginAsGuest()` __and__ `register()` __and__ `showAffiliateSales()` __and__ `getPremiumDiscount()` **at the same time**. In a realistic scenario you would have to refactor 10 tests instead of 40.

当然，从理论上讲，被验证的对象本身可能会发生变化。在实践中，`loginAsGuest()`__和__ `register()` __和__ `showAffiliateSales()` __和__ ` getPremiumDiscount()` **同时发生变化** 是不太可能的。在一个现实的场景中，你应该选择重构10次，而不是40次。


In summary, if you find yourself continuously fixing existing tests as you add new features, it means that your tests are tightly coupled to internal implementation.

总而言之，如果您发现自己在添加新功能时不断修复现有的测试，则意味着您的测试与内部实现密切相关。

### Anti-Pattern 6 - Paying excessive attention to test coverage 反模式6 - 过度关注测试覆盖率
{: #anti-pattern-6---paying-excessive-attention-to-test-coverage}

Code coverage is a favorite metric among software stakeholders. [Endless discussions](https://softwareengineering.stackexchange.com/questions/1380/how-much-code-coverage-is-enough) [have](https://martinfowler.com/bliki/TestCoverage.html) [happened](https://testing.googleblog.com/2010/07/code-coverage-goal-80-and-no-less.html) (and will continue to happen) among developers and project managers on the amount of code coverage a project needs.

代码覆盖率是软件利益相关者们（stakeholders）最喜欢的指标。关于一个项目到底需要怎样的代码覆盖量，开发者和项目经理之间  [曾经](https://martinfowler.com/bliki/TestCoverage.html)  [发生过](https://testing.googleblog.com/2010/07/code-coverage-goal-80-and-no-less.html)  [无休止的争论](https://softwareengineering.stackexchange.com/questions/1380/how-much-code-coverage-is-enough)，而且仍将继续下去。

The reason why everybody likes to talk about code coverage is because it is a metric that is easy to understand and quantify. There are several easily accessible tools that output this metric for most programming languages and test frameworks.

每个人都喜欢谈论代码覆盖的原因是，它是一个易于理解和量化的指标。对于大多数编程语言和测试框架，有几种容易得到的工具可以输出该指标。

_Let me tell you a little secret:_ Code coverage is completely useless as a metric. There is no “correct” code coverage number. This is a trap question. You can have a project with 100% code coverage that still has bugs and problems. The real metrics that you should monitor are the well-known CTM.

_让我告诉你一个小秘密：_ 代码覆盖率是一个毫无用处的指标。没有一个所谓“正确”的代码覆盖数量。这是一个陷阱问题。你可以让一个项目达到 100% 的代码覆盖率，但仍然有 bug 。你应该监控的真正指标是众所周知的 CTM。

#### The Codepipes Testing Metrics (CTM) Codepipes 测试指标（CTM）

Here is their definition if you have never seen them before:

如果你从没见过它们，这是它们的定义。

|  Metric Name  | Description | Ideal value | Usual value | Problematic value |
| -------------      |-------------| -----   |       -----|    -----|
| PDWT | % of Developers writing tests        |   100%      |  20%-70%    |   Anything less than 100% |
| PBCNT | % of bugs that create new tests          |   100%       |  0%-5%    |   Anything less than 100% |
| PTVB | % of tests that verify behavior       |   100%       |  10%   |   Anything less than 100% |
| PTD | % of tests that are deterministic       |   100%       |  50%-80%    |   Anything less than 100% |


| 度量名称  | 描述 | 理想值 | 寻常值 | 问题值 |
| -------------      |-------------| -----   |       -----|    -----|
| PDWT | 写测试的开发者的百分比   |   100%      |  20%-70%    |   任何低于100% |
| PBCNT | bug 能带来新测试的百分比    |   100%   |  0%-5%    |   任何低于100% |
| PTVB | 测试用来验证行为的百分比   |   100%       |  10%   |   任何低于100%  |
| PTD | 测试具有确定性的百分比|   100%       |  50%-80%    |  任何低于100% |


**PDWT** (Percent of Developers who Write Tests) is probably the most important metric of all. There is no point in talking about software testing anti-patterns if you have zero tests in the first place. All developers in the team should write tests. A new feature should be declared __done__ only when it is accompanied by one or more tests.

**PDWT** (写测试的开发者的百分比)可能是最重要的指标。如果没有写过测试，那么谈论软件测试反模式就毫无意义。团队中的所有开发人员都应该写测试。只有伴随一个或更多的测试，一个新功能才能算作 _完成_。

**PBCNT** (Percent of Bugs that Create New tests). Every bug that slips into production is a great excuse for writing a new software test that verifies the respective fix. A bug that appears in production should only appear once. If your project suffers from bugs that appear multiple times in production even after their original “fix”, then your team will really benefit from this metric. More details on this topic in [Antipattern 10](#anti-pattern-10---not-converting-production-bugs-to-tests).

**PBCNT** (bug 能带来新测试的百分比)。每个溜进生产系统的 bug 都是写新测试来验证相应修复的好理由。出现在生产系统的 bug 只应该出现一次。如果你的项目遭受这样的情形，bug 在被修复之后依然在生产系统中反复出现，那么你的团队将真正从这个指标当中受益。关于此主题更多的细节请参考[反模式 10](#anti-pattern-10---not-converting-production-bugs-to-tests)。

**PTVB** (Percent of Tests that Verify Behavior and not implementation). Tightly coupled tests are a huge time sink when the main code is refactored. This topic was already discussed in [Antipattern 5](#anti-pattern-5---testing-internal-implementation).

**PTVB** (测试用来验证行为而非实现的百分比)。当主代码被重构时，紧密耦合的测试会耗费大量的时间。本主题已经在[反模式 5](#anti-pattern-5---testing-internal-implementation)讨论过了。

**PTD** (Percent of Tests that are Deterministic to total tests). Tests should only fail when something is wrong with the business code. Having tests that fail intermittently for no apparent reason is a huge problem that is discussed in [Antipattern 7](#anti-pattern-7---having-flaky-or-slow-tests).

**PTD** （测试具有确定性的百分比）。当业务代码出现问题的时候，测试应该报错。在没有明显原因的情况下，测试间歇性地报错是一个巨大的问题，这在[反模式 7](#anti-pattern-7---having-flaky-or-slow-tests)讨论过。

If after reading about these metrics, you still insist on setting a hard number as a goal for code coverage, I will give you the number **20%**. This number should be used as a rule of thumb and it is based on the [Pareto principle](https://en.wikipedia.org/wiki/Pareto_principle). 20% of your code is causing 80% of your bugs, so if you really want to start writing tests you could do well by starting with that code first. This advice also ties well with [Anti-pattern 4](#anti-pattern-4---testing-the-wrong-functionality) where I suggest that you should write tests for your critical code first.

如果在阅读完这些指标之后，你仍然坚持将代码覆盖率定为硬性指标，我会告诉你这个数字是 **20%**。这个数字应该作为经验法则，它基于[帕累托原则](https://en.wikipedia.org/wiki/Pareto_principle)。20%的代码会导致80%的bug，所以如果你真的想开始写测试，你可以先从这些代码开始开始。这个建议也和[反模式 4](#anti-pattern-4---testing-the-wrong-functionality)紧密相联，我建议你先为你的关键代码编写测试。

Do __not__ try to achieve 100% total code coverage. Achieving 100% code coverage sounds good in theory but almost always is a waste of time:

__不要__ 试图达到 100% 的代码全覆盖。达到 100% 的代码覆盖在理论上听上去不错，但几乎是浪费时间。

*   you have wasted a lost of effort as getting from 80% to 100% is much more difficult than getting from 0% to 20% 你浪费了大量的时间，因为从 80% 提高到 100% 比从 0% 提高到 20% 要困难得多
*   Increasing code coverage has diminishing returns 收益会随代码覆盖率的提高而递减

In any non trivial application there are certain scenarios that needs complex unit tests in order to trigger. The effort required to write these tests will usually be more than the risk involved if these particular scenarios ever fail in production (if ever).

在任何有一定规模的程序中，只有通过复杂的单元测试才能触发某些场景。**编写这些测试所需的努力，通常比这些特殊场景在生产环境上出错所涉及的风险还要多（说的什么鬼!!??**）。

If you have worked with any big application you should know by now that after reaching 70% or 80% code coverage, it is getting very hard to write useful tests for the code that is still untested.

如果你参与过任何大型应用，你就应该明白，当达到 70% 或 80% 的代码覆盖率时，你就很难为那些尚未被测试的代码编写有用的测试。

![Code Coverage Effort](../assets/2018-05-17-software-testing-anti-patterns/41632934-dae114f8-746e-11e8-96a5-3e0bc4d4f8f3.png)

On a similar note, as we already saw in the section for [Antipattern 4](#anti-pattern-4---testing-the-wrong-functionality), there are some code paths that never actually fail in production, and therefore writing tests for them is not recommended. The time spent on getting them covered should be better spent on actual features.

类似的，正如我们在[反模式 4](#anti-pattern-4---testing-the-wrong-functionality)章节所看到的，有些代码路径从未在生产系统上出错，因此并不推荐为它们编写测试。花费在这上面的时间应该更好地用于开发实际的功能。

![Code Coverage Value](../assets/2018-05-17-software-testing-anti-patterns/41632942-e49eb694-746e-11e8-83f0-21f22e87b07d.png)

Projects that need a specific code coverage percentage as a delivery requirement usually force developers to test trivial code in order or write tests that just verify the underlying programming language. This is a huge waste of time and as a developer you have the duty to complain to management who has such unreasonable demands.

将特定的代码覆盖率作为交付要求的项目通常会迫使开发者去依次测试那些无关紧要的代码，或编写测试来验证编程语言本身。这是对时间的巨大浪费，作为一名开发者，你有责任对这种提出无理要求的管理层抗议。

In summary, code coverage is a metric that should **not** be used as a representation for quality of a software project.

总而言之，代码覆盖率这一指标**不应该**用来代表软件项目的质量。

### Anti-Pattern 7 - Having flaky or slow tests 反模式7 - 使用了有问题的或者慢的测试
{: #anti-pattern-7---having-flaky-or-slow-tests}

This particular anti-pattern has [already](https://martinfowler.com/articles/nonDeterminism.html) [been](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html) [documented](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html) [heavily](https://semaphoreci.com/community/tutorials/how-to-deal-with-and-eliminate-flaky-tests) so I am just including it here for completeness.

这种反模式 [已经](https://martinfowler.com/articles/nonDeterminism.html) [被](https://testing.googleblog.com/2016/05/flaky-tests-at-google-and-how-we.html) [大量地](https://testing.googleblog.com/2017/04/where-do-our-flaky-tests-come-from.html) [论述过](https://semaphoreci.com/community/tutorials/how-to-deal-with-and-eliminate-flaky-tests) ，所以我只是为了更完整才在这里提及它。

Since software tests act as an early warning against regressions, they should always work in a reliable way. A failing test should be a cause of concern and the person(s) that triggered the respective build should investigate why the test failed right away.

由于软件测试起到了对回归的预警作用，所以它们应该始终以可靠的方式运行。失败的测试应引起关注，触发相关构建的人员应立即调查测试失败的原因。

This approach can only work with tests that fail in a deterministic manner. A test that sometimes fails and sometimes passes (without any code changes in between) is unreliable and undermines the whole testing suite. The negative effects are two fold

这种方法只能适用于以确定的方式失败的测试。一个时而失败时而通过的测试（中间没有任何代码变化）是不可靠的，会破坏整个测试集。负面效应有两个方面：

*   Developers do not trust tests anymore and soon ignore them
*   Even when non-flaky tests actually fail, it is hard to detect them in a sea of flaky tests

* 开发人员不再信任测试，很快就会忽略它们。
* 即使某些稳定的测试失败了，也很难在不稳定测试（flaky tests）的海洋中发现它们。

A failing test should be easily recognizable by everybody in your team as it changes the status of the whole build. On the other hand if you have flaky tests it is hard to understand if new failures are truly new or they stem from the existing flaky tests.

失败的测试应该很容易被你团队中的每个人识别，因为它改变了整个构建的状态。另一方面，如果你的测试是不可靠的，那么就很难理解新的失败是真正的新失败，还是源于现有的不稳定测试。

![Flaky tests](../assets/2018-05-17-software-testing-anti-patterns/43044734-c70457d6-8dde-11e8-8ee3-44ebf5f6fe14-20200927135836027.png)

Even a small number of flaky tests in enough to destroy the credibility of the rest of test suite. If you have 5 flaky tests for example, run the build and get 3 failures it is not immediately evident if everything is fine (because the failures were coming from the flaky tests) or if you just introduced 3 regressions.

即使是少量的不稳定测试也足以破坏其余测试集的可信度。 例如，如果您有5个不稳定的测试，运行构建并得到了3个失败的测试，那么这很难证明是一切都正常（因为失败来自不稳定的测试），还是引入了3个回归。

A similar problem is having tests that are really really slow. Developers need a quick feedback on the result of each commit (also discussed in the next section) so slow tests will eventually be ignored or even not run at all.

一个类似的问题是有些测试非常非常慢。 开发人员需要得到每次提交结果的快速反馈（这也会在下一节中讨论），因此缓慢的测试最终将被忽略，甚至根本不运行。

In practice flaky and slow tests are almost always integration tests and/or UI tests. As we go up in the testing pyramid, the probabilities of flaky tests are greatly increasing. Tests that deal with browser events are notoriously hard to get right all the time. Flakiness in integration tests can come from many factors but the usual suspect is the test environment and its requirements.

在实践中，不稳定测试和缓慢的测试几乎总是集成测试和/或UI测试。 随着我们沿测试金字塔上升，不稳定测试的概率会大大增加。 众所周知，包含浏览器事件的测试很难做到从不执行出错。 集成测试中的不稳定性，可能来自许多因素，但通常值得怀疑的是测试环境及其需求。

The primary defense against flaky and slow tests is to isolate them in their own test suite (assuming that they are not fixable). You can easily find more abundant resources on how to fix flaky tests for your favorite programming language by searching online so there is no point in me explaining the fixes here.

防止不稳定和缓慢的测试的主要措施，是将它们隔离在它们自己的测试集中（假设它们是不可修复的）。您可以在网上很容易地搜到这些资源，它们讲解了如何修复你最喜爱的编程语言所写出的不可靠测试，因此我在这里就不罗嗦了。

In summary, you should have a reliable test suite (even if it is a subset of the whole test suite) that is rock solid. A test that fails in this suite means that something is really really wrong with the code and any failure means that the code must not be promoted to production.

总之，您应该有一个可靠的测试集（即使它是整个测试集的一个子集），它是坚如磐石的。在这个测试集中，失败的测试意味着代码有一些真正的错误，任何失败都意味着不能将代码提升到生产环境中。

### Anti-Pattern 8 - Running tests manually 反模式8 - 手工运行测试
{: #anti-pattern-8---running-tests-manually}

Depending on your organization you might actually have several types of tests in place. Unit tests, Load tests, User acceptance tests are common categories of test suites that __might__ be executed before the code goes into production.

根据您的组织，您实际上可能有几种类型的测试。单元测试、压力测试、用户验收测试是测试套件的常见类别，这些测试套件**可能**会在代码进入生产环境之前执行。

Ideally all your tests should run automatically without any human intervention. If that is not possible at the very least all tests that deal with correctness of code (i.e. unit and integration tests) **must** run in an automatic manner. This way developers get feedback on the code in the most timely manner. It is very easy to fix a feature when the code is fresh in your mind and you haven’t switched context yet to an unrelated feature.

理想情况下，所有测试都应自动运行，无需任何人工干预。 如果至少无法做到这一点，那么所有处理代码正确性的测试（即单元测试和集成测试）都**必须**以自动化的方式运行。 这样，开发人员就可以最及时地获得对代码的反馈。 当您对代码记忆犹新，并且还没有将上下文切换到不相关的功能时，修复某个功能是非常容易的。

![Test feedback loop tests](../assets/2018-05-17-software-testing-anti-patterns/43044744-fb83ff2a-8dde-11e8-8ad9-508a6c9afb60.png)

In the past the most lengthy step of the software lifecycle was the deployment of the application. With the move into cloud infrastructure where machines can be created on demand (either in the form of VMs or containers) the time to provision a new machine has been reduced to minutes or seconds. This paradigm shift has caught a lot of companies by surprise as they were not ready to handle daily or even hourly deployments. Most of the existing practices were centered around lengthy release cycles. Waiting for a specific time in the release to “pass QA” with manual approval is one of those obsolete practices that is no longer applicable if a company wants to deploy as fast as possible.

在过去，软件生命周期中最耗时的步骤是应用程序的部署。随着迁移到云基础机构，机器可以按需创建(以虚拟机（VM）或容器的形式)，配置新机器的时间已减少到几分钟甚至几秒。这种范式转变让很多公司感到惊讶，因为他们还没有为每天甚至每小时的部署做好准备。大多数现有的实践都围绕着冗长的发布周期。等待发布中的特定时间，以通过手动批准“通过质量检查”则成为一种过时的实践，如果公司希望尽快部署，则这些实践不再适用。

Deploying as fast as possible implies that you trust each deployment. Trusting an automatic deployment requires a high degree of confidence in the code that gets deployed. While there are several ways of getting this confidence, the first line of defense should be your software tests. However, having a test suite that can catch regressions quickly is only half part of the equation. The other half is running the tests automatically (possibly after every commit).

尽快部署意味着您信任每个部署。 信任自动部署需要对部署的代码有高度的信心。 尽管可以通过多种方法获得这种信任，但是第一道防线应该是您的软件测试。 但是，拥有可以快速捕获回归的测试套件只是等式的一半，而另一半则是自动化运行的测试（可能在每次提交之后）。

A lot of companies __think__ that they practice continuous delivery and/or deployment. In reality they don’t. Practicing true CI/CD means that __at any given point in time__ there is a version of the code that is ready to be deployed. This means that the candidate release for deployment is __already__ tested. Therefore having a package version of an application “ready” which has not really “passed QA” is not true CI/CD.

很多公司__认为__他们实践的是持续交付和/或部署。事实上，他们没有。实践真正的CI/CD意味着在任意给定的时间点，都有一个可以部署的代码版本。这意味着部署的候选版本已经测试过的。因此，将没有真正“通过QA”的应用程序打包成“就绪”版本，并不是真正的CI/CD。

Unfortunately, while most companies have correctly realized that deployments should be automated, because using humans for them is error prone and slow, I still see companies where launching the tests is a semi-manual process. And when I say semi-manual I mean that even though the test suite itself might be automated, there are human tasks for house-keeping such as preparing the test environment or cleaning up the test data after the tests have finished. That is an anti-pattern because it is not true automation. **All** aspects of testing should be automated.

不幸的是，尽管大多数公司正确地意识到部署应该是自动化的，因为使用人工进行操作容易出错且速度很慢，但我仍然看到启动测试是半手工过程的公司。 当我说半手动时，我的意思是即使测试套件本身可能是自动化的，也要进行一些内部任务，例如准备测试环境或在测试完成后清理测试数据。 这是一种反模式，因为它不是真正的自动化。 测试的 **所有** 方面都应自动化。

![Automated tests](../assets/2018-05-17-software-testing-anti-patterns/43044756-2400979c-8ddf-11e8-99b4-69f2ce06de8b.png)

Having access to VMs or containers means that it is very easy to create various test environments on demand. Creating a test environment on the fly for an individual pull request should be a standard practice within your organization. This means that each new feature is tested individually on its own. A problematic feature (i.e. that causes tests to fail) should not block the release of the rest of the features that need to be deployed at the same time.

拥有对VM或容器的访问权限意味着可以轻松地按需创建各种测试环境。 为单个拉取请求（Pull Request）动态创建测试环境应该是组织内的标准做法。 这意味着每个新功能（feature）都需要单独测试。 有问题的功能（即导致测试失败）不应阻止需要同时部署的其余功能的发布。

An easy way to understand the level of test automation within a company is to watch the QA/Test people in their daily job. In the ideal case, testers are just creating new tests that are added to an existing test suite. Testers themselves do not run tests manually. The test suite is run by the build server.

了解公司内部测试自动化水平的一个简单方法是观察QA/测试人员的日常工作。在理想情况下，测试人员只是创建新测试，并添加到现有测试套件中。测试人员本身并不手动运行测试。测试套件由构建服务器运行。

In summary, testing should be something that happens all the time behind the scenes by the build server. Developers should learn the result of the test for their individual feature after 5-15 minutes of committing code. Testers should create new tests and refactor existing ones, instead of actually running tests.

总之，测试应该是构建服务器在幕后一直在进行的事情。开发人员应该在提交代码 5-15 分钟后了解他们各自特性的测试结果。测试人员应该创建新的测试并重构现有的测试，而不是实际运行测试。

### Anti-Pattern 9 - Treating test code as a second class citizen 反模式9 - 把测试当作二等公民
{: #anti-pattern-9---treating-test-code-as-a-second-class-citizen}

If you are a seasoned developer, you will spend always some time to structure new code in your mind before implementing it. There are several philosophies regarding code design and some of them are so significant that have their own Wikipedia entry. Some examples are:

如果您是一位经验丰富的开发人员，那么在实现新代码之前，您总是要花一些时间在脑海中构造新代码。关于代码设计有几种哲学，其中一些非常重要，在维基百科上有自己的条目。一些例子：

*   [DRY （不要重复自己）](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
*   [KISS（保持简单愚蠢） ](https://en.wikipedia.org/wiki/KISS_principle)
*   [SOLID（单一功能、开闭原则、里氏替换、接口隔离、依赖反转）](https://en.wikipedia.org/wiki/SOLID_(object-oriented_design))

The first one is arguably the most important one as it forces you to have a single source of truth for code that is reused across multiple features. Depending on your own programming language you may also have access to several other best practices and recommended design patterns. You might even have special guidelines that are specific to your team.

第一个可能是最重要的一个，因为它迫使您拥有单一的真理来源——在多个功能中重复使用。根据您自己的编程语言，您还可以访问其他一些最佳实践和推荐的设计模式。您甚至可能有专门针对您团队的特殊指导方针。

Yet, for some unknown reason several developers do not apply the same principles to the code that holds the software tests. I have seen projects which have well designed feature code, but suffer from tests with huge code duplication, hardcoded variables, copy-paste segments and several other inefficiencies that would be considered inexcusable if found on the main code.

然而，由于某些未知的原因，一些开发人员没有将相同的原则应用到包含软件测试的代码中。我曾见过一些设计良好的功能代码的项目，但在测试中遇到了大量代码重复、硬编码变量、复制-粘贴段，以及其他一些如果在主代码中发现就会被视为不可原谅的低效率问题。

Treating test code as a second class citizen makes no sense, because in the long run all code needs maintenance. Tests will need to be updated and refactored in the future. Their variables and structure will need to change. If you write tests without thinking about their design you are creating additional technical debt that will be added to the one already present in the main code.

把测试代码当作二等公民是没有意义的，因为从长远来看，所有的代码都需要维护。将来需要对测试进行更新和重构。它们的变量和结构需要改变。如果您在编写测试时没有考虑它们的设计，那么您正在增加额外的技术债，这些技术债将被添加到主代码中已经存在的技术债中。

Try to design your tests with the same attention that you give to the feature code. All common refactoring techniques should be used on tests as well. As a starting point

在设计测试时，尽量像关注功能代码一样关注。所有常见的重构技术也应该在测试中使用。作为起点：

*   All test creation code should be centralized. All tests should create test data in the same manner
*   Complex verification segments should be extracted in a common domain specific library
*   Mocks and stubs that are used too many times should not be copied-pasted.
*   Test initialization code should be shared between similar tests.



* 所有测试创建代码都应该集中。所有测试都应该以相同的方式创建测试数据
* 复杂的验证段应该提取到一个公共领域的特定库
* 不应该复制粘贴多次使用的mock和stub。
* 测试初始化代码应该在相似的测试之间共享。

If you employ tools for static analysis, source formatting or code quality then configure them to run on test code, too.

如果您使用了用于静态分析、源代码格式化或代码质量的工具，那么也可以将它们配置为在测试代码上运行。

In summary, design your tests with the same detail that you design the main feature code.

总之，用设计主功能代码时的相同细节来设计测试。

### Anti-Pattern 10 - Not converting production bugs to tests 反模式10 - 没有把生产环境的 bug 转化为测试
{: #anti-pattern-10---not-converting-production-bugs-to-tests}

One of the goals of testing is to catch regressions. As we have seen in [antipattern 4](#anti-pattern-4---testing-the-wrong-functionality), most applications have a “critical” code part where the majority of bugs appear. When you fix a bug you need to make sure that it doesn’t happen again. One of the best ways to enforce this is to write a test for the fix (either unit or integration or both).

测试的目标之一是捕获回归。正如我们在[反模式4](#anti-pattern-4——test -the-wrong function)中看到的，大多数应用程序都有一个“关键”代码部分，其中出现了大多数错误。当你修复一个错误时，你需要确保它不会再次发生。执行这一点的最好方法之一是为针对这次bug修复编写测试(单元、集成或两者兼有)。

Bugs that slip into production are perfect candidates for writing software tests

潜入生产环境的错误，是编写软件测试的完美候选

*   they show a lack of testing in that area as the bug has already passed into production
*   if you write a test for these bugs the test will be very valuable as it guards future releases of the software



* 它们显示了该领域的测试不足，因为bug已经进入了生产阶段
* 如果你为这些 bug 编写了一个测试，这个测试将非常有价值，因为它可以保护软件的未来版本



I am always amazed when I see teams (that otherwise have a sound testing strategy) that don’t write a test for a bug that was found __in production__. They correct the code and fix the bug straight away. For some strange reason a lot of developers assume that writing tests is only valuable when you are adding a new feature only.

当我看到团队（他们有一个健全的测试策略）没有为__生产环境__中发现的bug编写测试时，我总是感到惊讶。他们纠正代码并立即修复错误。由于一些奇怪的原因，许多开发人员认为只有在添加新特性时，编写测试才有价值。

This could not be further from the truth. I would even argue that software tests that stem from actual bugs are more valuable than tests which are added as part of new development. After all you never know how often a new feature will break in production (maybe it belongs to non-critical code that will never break). The respective software test is good to have but its value is questionable.

这与事实相差甚远。我甚至认为，源于实际 bug 的软件测试比在开发新功能时添加的测试更有价值。毕竟，您永远不知道一个新特性在生产中出问题的频率（也许它属于永远不会出错的非关键代码）。所对应的软件测试是好的，但其价值值得怀疑。

On the other hand the software test that you write for a real bug is super valuable. Not only it verifies that your fix is correct, but also ensures that your fix will always be active even if refactorings happen in the same area.

另一方面，为一个真正的 bug 编写的软件测试是非常有价值的。它不仅验证您的修复是正确的，而且还确保即使对这段代码进行重构，您的修复也将始终起作用。 

If you join a legacy project that has no tests this is also the most obvious way to start getting value from software testing. Rather than attempting to guess which code needs tests, you should pay attention to the existing bugs and try to cover them with tests. After a while your tests will have covered the critical part of the code, since by definition all tests have verified things that break often. One of [my suggested metrics](#the-codepipes-testing-metrics-ctm) embodies the recording of this effort.

如果您加入了一个没有测试的遗留项目，这也是开始从软件测试中获得价值的最明显的方法。不要试图猜测哪些代码需要测试，您应该关注现有的bug，并尝试用测试来覆盖它们。一段时间后，您的测试将涵盖代码的关键部分，因为根据定义，所有测试都验证了经常出错的东西。其中一个[我建议的度量标准](#the-codepipes-testing-metrics-ctm)包含了对这项工作的记录。

The only case where it is acceptable to **not** write tests is when bugs that you find in production are unrelated to code and instead stem from the environment itself. A misconfiguration to a load balancer for example is not something that can be solved with a unit test.

唯一可以接受不编写测试的情况是，您在生产中发现的bug与代码无关，而是源于环境本身。例如，负载均衡器的错误配置不能通过单元测试解决。

In summary, if you are unsure on what code you need to test next, look at the bugs that slip into production.

总之，如果您不确定接下来需要测试什么代码，那么请检查一下溜进生产环境的bug。

### Anti-Pattern 11 - Treating TDD as a religion 反模式11 - 将 TDD 看作教派
{: #anti-pattern-11---treating-tdd-as-a-religion}

TDD stands for [Test Driven Development](https://en.wikipedia.org/wiki/Test-driven_development) and like all methodologies before it, it is a good idea on paper until consultants try to convince a company that following TDD blindly is the only way forward. At the time or writing this trend is slowly dying but I decided to mention it here for completeness (as the enterprise world is especially suffering from this anti-pattern).

Broadly speaking when it comes to software tests:

1.  you can write tests __before__ the respective implementation code
2.  you can write tests __at the same time__ as the implementation code
3.  you can write tests __after__ the implementation code
4.  you can write 0 tests (i.e. never) for the implementation code

One of the core tenets of TDD is always following option 1 (writing tests before the implementation code). Writing tests before the code is a good general practice but is certainly not always the __best__ practice.

Writing tests before the implementation code implies that you are certain about your final API, which may or may not be the case. Maybe you have a clear specification document in front of you and thus know the exact signatures of the code methods that need to be implemented. But in other cases you might want to just experiment on something, do a quick spike and work **towards** a solution instead of a solution itself.

For a more practical example, it would be immature for a startup to follow blindly TDD. If you work in a startup company you might write code that will change so fast that TDD will not be a big help. You might even throw away code until you get it “right”. Writing tests __after__ the implementation code, is a perfectly valid strategy in that case.

Writing no tests at all (option 4) is also a valid strategy. As we have seen in [anti-pattern 4](#anti-pattern-4---testing-the-wrong-functionality) there is code that never needs testing. Writing software tests for trivial code because this is the correct way to “do TDD” will get you nowhere.

The obsession of TDD zealots on writing tests first no matter the case, has been a huge detriment to the [mental health of sane developers](https://softwareengineering.stackexchange.com/questions/98485/tdd-negative-experience). This obsession is already documented in various places so hopefully I don’t need to say anything more on the topic (search for “TDD is crap/stupid/dead”).

At this point I would like to admit that several times I have personally implemented code like this:

1.  Implementing the main feature first
2.  Writing the test afterwards
3.  Running the test to see it succeed
4.  Commenting out critical parts of the feature code
5.  Running the test to see it fail
6.  Uncommenting feature code to its original state
7.  Running the test to see it succeed again
8.  Commiting the code

In summary, TDD is a good idea but you don’t have to follow it all the time. If you work in a fortune 500 company, surrounded by business analysts and getting clear specs on what you need to implement, then TDD __might__ be helpful.

On the other hand if you are just playing with a new framework at your house during the weekend and want to understand how it works, then feel free to **not** follow TDD.

### Anti-Pattern 12 - Writing tests without reading documentation first 反模式12 - 写测试之前不先读文档
{: #anti-pattern-12---writing-tests-without-reading-documentation-first}

A professional developer is one who knows the tools of the trade. You might need to spend extra time at the beginning of a project to learn about the technologies you are going to use. Web frameworks are coming out all the time and it always pays off to know all the capabilities that can be employed in order to write effective and concise code.

You should treat software tests with the same respect. Because several developers treat tests as something secondary (see also [Anti-pattern 9](#anti-pattern-9---treating-test-code-as-a-second-class-citizen)) they never sit down to actually learn what their testing framework can do. Copy-pasting testing code from other projects and examples might seem to work at first glance, but this is not the way a professional should behave.

Unfortunately this pattern happens all too often. People are writing several “helper functions” and “utilities” for tests without realizing that their testing framework already offers this function either in a built-in manner or with the help of external modules.

These utilities make the tests hard to understand (especially for junior developers) as they are filled with in-house knowledge that is non transferable to other projects/companies. Several times I have replaced “smart in-house testing solutions” with standard off-the-shelf libraries that do the same thing in a standardized manner.

You should spend some time to learn what your testing framework can do. For example try to find how it can work with:

*   parameterized tests
*   mocks and stubs
*   test setup and teardown
*   test categorization
*   conditional running of tests

If you are also working on the stereotypical web application you should do some minimal research to understand what are the best practices regarding

*   test data creators
*   HTTP client libraries
*   HTTP mock servers
*   mutation/fuzzy testing
*   DB cleanup/rollback
*   load testing and so on

There is no need to re-invent the wheel. The sentence applies to testing code as well. Maybe there are some corner cases where your main application is indeed a snowflake and needs some in-house utility for the core code. But I can bet that your unit and integration tests are not special themselves and thus writing custom testing utilities is a questionable practice.

### Anti-Pattern 13 - Giving testing a bad reputation out of ignorance 反模式13 - 由于无知而诋毁测试
{: #anti-pattern-13---giving-testing-a-bad-reputation-out-of-ignorance}

Even though I mention this as the last anti-pattern, this is the one that forced me to write this article. I am always disappointed when I find people at conferences and meetups who “proudly” proclaim that __all tests are a waste of time__ and that their application works just fine without any testing at all. A more common occurrence is meeting people who are against a specific type of testing (usually either unit or integration) like we have seen in anti-patterns [1](#anti-pattern-1---having-unit-tests-without-integration-tests) or [2](#anti-pattern-2---having-integration-tests-without-unit-tests)

When I find people like this, it is my hobby to probe them with questions and understand their reasons behind hating tests. And always, it boils down to anti-patterns. They previously worked in companies where tests were slow ([Anti pattern 7](#anti-pattern-7---having-flaky-or-slow-tests)), or needed constant refactoring ([Antipattern 5](#anti-pattern-5---testing-internal-implementation)). They have been “burned” by unreasonable requests for 100% code coverage ([Anti-pattern 6](#anti-pattern-6---paying-excessive-attention-to-test-coverage)) or TDD zealots ([Antipattern 11](#anti-pattern-11---treating-tdd-as-a-religion)) that tried to impose to the whole team their own twisted image of what TDD means.

If you are one of those people I truly feel for you. I know how hard it is to work in a company that has bad habits.

Bad experiences of testing in the past should not clutter your judgment when it comes to testing your next greenfield project. Try to look objectively at your team and your project and see if any of the anti-patterns apply to you. If yes, then you are simply testing in the wrong way and no amount of tests will make your application better. Sad but true.

It is one thing for your team to suffer from bad testing habits, and another to mentor junior developers declaring that “testing is a waste of time”. Please don’t do the latter. There are companies out there that don’t suffer from **any** of the anti-patterns mentioned in this article. Try to find them!
