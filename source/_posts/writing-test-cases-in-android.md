---
title: 【译】Android 写测试用例的注意事项
date: 2017-01-18 15:53:55
tags:
category: Android
---

> * 原文地址：[The Do’s and Don’ts of Writing Test cases in Android.](https://blog.mindorks.com/the-dos-and-don-ts-of-writing-test-cases-in-android-70f1b5dab3e1#.sjelh11mm)
* 原文作者：[Anshul Jain](https://blog.mindorks.com/@anshuljain?source=post_header_lockup)
* 译文出自：[掘金翻译计划](https://github.com/xitu/gold-miner)
* 译者：[Zhiw](https://github.com/Zhiw)
* 校对者：[Draftbk](https://github.com/draftbk)，[rockzhai](https://github.com/rockzhai)


# Android 写测试用例的注意事项

在本文中，我将根据我的经验来尝试解释写测试用例的最佳实践。我会在本文中使用 Espresso 代码，但是这些实践都可以应用在单元测试和 UI 测试上。为了更好的解释，我以一个新闻应用作为例子。

> 下面提到的应用的功能和条件纯属虚构，仅仅是为了解释这个最佳实践，并且与 Google Play 中上架或者已下架的任何应用不相似。:P

**该新闻应用有以下界面**

- **语言选择** — 当用户第一次打开应用时，他必须选择至少一种语言。选择一种或多种语言后，用户的选择将会保存到 shared preferences 中，然后用户将会直接跳转到新闻列表界面。

- **新闻列表** — 当用户打开新闻列表界面，请求和语言参数将会发送到服务器，然后响应结果将会显示到 recycler view （id 为 *news_list* ）上。如果 shared preference 没有该语言或者服务器没有给出成功的响应，错误界面将会对用户可见并且 recycler view 将会消失。新闻列表界面有一个按钮，如果用户只选择了一种语言，按钮显示为 “Change your Language”，如果用户选择了多种语言，则显示为 “Change your Languages”。（我向上帝发誓，这是一个虚构的应用）

- **新闻详情** — 顾名思义，当用户点击任何新闻列表条目的时候，该应用将被启动。

应用的功能都已经足够了。让我们来看看为新闻列表界面写的测试用例。这是我最开始写的代码。

![Markdown](http://i1.piimg.com/1949/d1520ac5242054b3.png)

#### 仔细确定测试用例的目的

在第一个测试用例 *testClickOnAnyNewsItem()* 中，如果服务器没有发送成功的响应，这个测试用例将会失败，因为 recycler view 的 visibility 是 GONE。但这并不是测试用例的目的。**这个测试用例要通过或者失败，最低的要求是 recycler view 要显示**，如果因为某些原因没有显示，那么这个测试用例不应该视为**失败**。该测试的正确代码应该像这样。

![Markdown](http://i1.piimg.com/1949/8e950c3072136967.png)

#### 测试用例本身应该是完整的

当我开始测试时，我总是按照以下顺序测试界面

- 语言选择

- 新闻列表

- 新闻详情

因为我首先测试了语言选择界面，所以在测试新闻列表界面之前，一种语言已经被设置好了。但是当我先测试新闻列表界面时，该测试就失败了。失败的原因很简单 — 语言没有选择，因此 recycler view 也就不显示。**因此，测试用例的执行顺序不应该影响到测试结果**。所以，在运行测试用例之前，语言选项应该被保存到 shared preferences 中，这样的话，该测试用例就独立于语言选择界面的测试了。

![Markdown](http://i1.piimg.com/1949/7d54085d16277ea1.png)

#### 避免测试用例中的条件编码

在第二个测试用例 *testChangeLanguageFeature()* 中，我们获得用户选择的语言种类数，然后基于此，我们写了一个 if-else 的条件来测试。但是 if-else 条件应该写在你的实际代码中，而不是写在测试代码里。每一个条件都应该单独测试。因此，我们应该写如下的两个测试用例，而不是一个。

![Markdown](http://i1.piimg.com/1949/ed55274b0f7f2185.png)

#### 测试用例应该独立于外部因素

在大多数应用中，我们都要与外部代理如网络和数据库进行交互。测试用例可以在执行过程中调用一个请求发送到服务器，响应可能成功或者失败。但是如果服务器的响应失败了，那么这个测试用例不应该视为失败。我们这样考虑 — 如果测试用例失败了，然后我们在我们的客户端代码中进行更改，以便测试代码可以正常工作。但是在这种情况下，我们是否需要对客户端代码进行任何更改？— **不**。

但是你也不应该完全避免测试网络请求和响应。由于服务器是外部代理，所以可能存在一种情况，当它发送一些错误响应时，这可能会导致应用崩溃。因此，你写的测试用例应该覆盖服务器所有可能的响应，甚至是那种服务器永远不会发送的响应。以此方法，所有的代码都将被覆盖，然后确保应用能优雅的处理所有的响应并且不崩溃。

> 以正确的方式编写测试用例与编写用于测试的代码一样重要。

感谢阅读本文。我希望这将帮助你写出更好的测试用例。你可以在 [LinkedIn](http://www.linkedin.com/in/anshul-jain-b7082573) 上联系我。你也可以在 [这里](https://medium.com/@anshuljain) 查看我的其他 medium 文章。

***了解更多关于编程方面的内容，关注 [***Mindorks***](https://blog.mindorks.com) ，当我们有新文章发布的时候，你就能收到通知。***
