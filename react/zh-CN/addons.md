---
title: "插件"
tocTitle: "插件"
description: "了解如何使用流行的示例，集成与使用插件"
commit: "dac373a"
---

# 插件

故事书，拥有强大的系统[插件](https://storybook.js.org/addons/introduction/)您可以使用它来增强团队中每个人的开发人员体验。如果您一直在线性地学习本教程，那么到目前为止我们已经引用了多个插件，并且您已经在[测试章节](/react/en/test/)。

<div class="aside">
<strong>Looking for a list of potential addons?</strong>
<br/>
😍 You can seen the list of officially-supported and strongly-supported community addons <a href="https://storybook.js.org/addons/addon-gallery/">here</a>.
</div>

我们可以永远写下为所有特定用例配置和使用插件。现在，让我们努力整合Storybook生态系统中最受欢迎的插件之一：[旋钮](https://github.com/storybooks/storybook/tree/master/addons/knobs)。

## 设置旋钮

旋钮是设计师和开发人员在受控环境中试验和玩组件而不需要编码的绝佳资源！您基本上提供动态定义的字段，用户可以使用这些字段操作传递给您的故事中的组件的道具。以下是我们要实施的内容......

<video autoPlay muted playsInline loop>
  <source
    src="/addon-knobs-demo.mp4"
    type="video/mp4"
  />
</video>

### 安装

首先，我们需要安装所有必需的依赖项。

```bash
yarn add @storybook/addon-knobs
```

在你的注册旋钮`.storybook/addons.js`文件。

```javascript
import '@storybook/addon-actions/register';
import '@storybook/addon-knobs/register';
import '@storybook/addon-links/register';
```

<div class="aside">
<strong>📝 Addon registration order matters!</strong>
<br/>
The order you list these addons will dictate the order in which they appear as tabs on your addon panel (for those that appear there).
</div>

而已！是时候在故事中使用它了。

### 用法

让我们使用对象旋钮类型`Task`零件。

首先，导入`withKnobs`装饰者和`object`旋钮类型为`Task.stories.js`：

```javascript
import React from 'react';
import { storiesOf } from '@storybook/react';
import { action } from '@storybook/addon-actions';
import { withKnobs, object } from '@storybook/addon-knobs/react';
```

接下来，在故事中`Task`通过`withKnobs`作为参数`addDecorator()`功能：

```javascript
storiesOf('Task', module)
  .addDecorator(withKnobs)
  .add(/*...*/);
```

最后，整合`object`“默认”故事中的旋钮类型：

```javascript
storiesOf('Task', module)
  .addDecorator(withKnobs)
  .add('default', () => {
    return <Task task={object('task', { ...task })} {...actions} />;
  })
  .add('pinned', () => <Task task={{ ...task, state: 'TASK_PINNED' }} {...actions} />)
  .add('archived', () => <Task task={{ ...task, state: 'TASK_ARCHIVED' }} {...actions} />);
```

现在，新的“旋钮”选项卡应显示在底部窗格中“操作记录器”选项卡旁边。

如记录[这里](https://github.com/storybooks/storybook/tree/master/addons/knobs#object)，`object`knob类型接受标签和默认对象作为参数。标签是常量，显示在插件面板中文本字段的左侧。您传递的对象将表示为可编辑的JSON blob。只要您提交有效的JSON，您的组件就会根据传递给对象的数据进行调整！

## 插件演变你的故事书的范围

你的故事书实例不仅是一个很棒的例子[CDD环境](https://blog.hichroma.com/component-driven-development-ce1109d56c8e)，但现在我们提供了一个互动的文档来源。PropTypes很棒，但是一个设计师或一个对组件代码完全陌生的人将能够通过实现旋钮插件的Storybook非常快速地找出它的行为。

## 使用旋钮查找边缘案例

此外，通过轻松访问编辑传递数据到组件，QA工程师或预防性UI工程师现在可以将组件推送到极限！举个例子，会发生什么`Task`如果我们的列表项有一个*MASSIVE*串？

![Oh no! The far right content is cut-off!](/addon-knobs-demo-edge-case.png)😥

由于能够快速地尝试对组件的不同输入，我们可以相对轻松地找到并修复这些问题！让我们通过添加样式来解决溢出问题`Task.js`：

```javascript
// This is the input for our task title. In practice we would probably update the styles for this element
// but for this tutorial, let's fix the problem with an inline style:
<input
  type="text"
  value={title}
  readOnly={true}
  placeholder="Input title"
  style={{ textOverflow: 'ellipsis' }}
/>
```

![That's better.](/addon-knobs-demo-edge-case-resolved.png)👍

## 添加新故事以避免回归

当然，我们总是可以通过在旋钮中输入相同的输入来重现此问题，但最好为此输入编写固定的故事。这将增加您的回归测试，并清楚地概述组件对团队其他成员的限制。

让我们在Task.stories.js中为长文本案例添加一个故事：

```javascript
const longTitle = `This task's name is absurdly large. In fact, I think if I keep going I might end up with content overflow. What will happen? The star that represents a pinned task could have text overlapping. The text could cut-off abruptly when it reaches the star. I hope not`;

storiesOf('Task', module)
  .add('default', () => <Task task={task} {...actions} />)
  .add('pinned', () => <Task task={{ ...task, state: 'TASK_PINNED' }} {...actions} />)
  .add('archived', () => <Task task={{ ...task, state: 'TASK_ARCHIVED' }} {...actions} />)
  .add('long title', () => <Task task={{ ...task, title: longTitle }} {...actions} />);
```

现在我们已经添加了故事，只要我们想要处理它，我们就可以轻松地重现这个边缘情况：

![Here it is in Storybook.](/addon-knobs-demo-edge-case-in-storybook.png)

如果我们使用[视觉回归测试](/react/en/test/)如果我们打破我们的椭圆机解决方案，我们也会收到通知。这些不起眼的边缘案件总是容易被遗忘！

### 合并更改

不要忘记将您的更改与git合并！

## 与团队共享插件

旋钮是让非开发人员玩你的组件和故事的好方法。但是，他们可能很难在本地机器上运行故事书。这就是为什么将您的故事书部署到在线位置可能真的很有帮助。在下一章中，我们将做到这一点！
