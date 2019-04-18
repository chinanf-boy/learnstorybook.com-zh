---
title: "Testing"
description: "Learn the ways to test UI components"
commit: 8bf107e
---

# 测试UI组件

没有测试，没有故事书教程是完整的。测试对于创建高质量的UI至关重要。在模块化系统中，微小的调整可能会导致重大的回归。到目前为止，我们遇到了三种类型的测试

-   **视觉测试**依赖开发人员手动查看组件以验证其正确性。它们帮助我们在构建时检查组件的外观。
-   **快照测试**使用Storyshots捕获组件的渲染标记。它们可以帮助我们及时了解导致渲染错误和警告的标记更改。
-   **单元测试**使用Jest验证在给定固定输入的情况下组件的输出保持不变。它们非常适合测试组件的功能质量。

## “但它看起来不错吗？”

不幸的是，单独的上述测试方法不足以防止UI错误。用户界面很难测试，因为设计是主观的，细致入微的。可视化测试过于手动，快照测试在用于UI时会触发太多误报，而像素级单元测试的价值很低。完整的故事书测试策略还包括视觉回归测试。

## 故事书的视觉回归测试

视觉回归测试旨在捕捉外观的变化。他们通过捕获每个故事的屏幕截图并将它们提交到表面更改进行比较来工作。这非常适合验证布局，颜色，大小和对比度等图形元素。

<video autoPlay muted playsInline loop style="width:480px; margin: 0 auto;">
  <source
    src="/visual-regression-testing.mp4"
    type="video/mp4"
  />
</video>

故事书是视觉回归测试的绝佳工具，因为每个故事本质上都是一个测试规范。每次我们编写或更新故事时，我们都会免费获得规格！

有许多用于视觉回归测试的工具。对于专业团队，我们建议[**Chromatic**](https://www.chromaticqa.com/)，由Storybook维护者制作的插件，在云中运行测试。

## 设置视觉回归测试

Chromatic是一个无障碍的故事书插件，用于在云中进行视觉回归测试和审查。由于它是付费服务（免费试用），因此可能不适合所有人。但是，Chromatic是生产视觉测试工作流程的一个有益的例子，我们将免费试用。我们来看一下。

### 发起Git

首先，您要在本地目录中为项目设置Git。Chromatic使用Git历史来跟踪您的UI组件。

```bash
$ git init
```

接下来将文件添加到第一次提交。

```bash
$ git add .
```

现在提交文件。

```bash
$ git commit -m "taskbox UI"
```

### 获得色彩

将包添加为依赖项。

```bash
yarn add --dev storybook-chromatic
```

导入Chromatic在你的`.storybook/config.js`文件。

```javascript
import { configure } from '@storybook/vue';
import 'storybook-chromatic';

import '../src/index.css';

const req = require.context('../src', true, /.stories.js$/);
function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

然后[login to Chromatic](https://bit.ly/2Is93Ez)使用您的GitHub帐户（Chromatic仅要求轻量级权限）。创建名为“taskbox”的项目并复制您的唯一项目`app-code`。

<video autoPlay muted playsInline loop style="width:520px; margin: 0 auto;">
  <source
    src="/chromatic-setup-learnstorybook.mp4"
    type="video/mp4"
  />
</video>

在命令行中运行test命令以设置Storybook的可视化回归测试。不要忘记添加您的唯一应用代码来代替`<app-code>`。

```bash
./node_modules/.bin/chromatic test --app-code=<app-code>
```

<div class="aside">
<code>--do-not-start</code> is an option that tells Chromatic not to start Storybook. Use this if you already have Storybook running. If not omit <code>--do-not-start</code>.
</div>

第一次测试完成后，我们会为每个故事提供测试基准。换句话说，每个故事的屏幕截图都被称为“好”。这些故事的未来变化将与基线进行比较。

![Chromatic baselines](/chromatic-baselines.png)

## 捕获UI更改

视觉回归测试依赖于将新呈现的UI代码的图像与基线图像进行比较。如果捕获到UI更改，则会收到通知。通过调整背景来了解它是如何工作的`Task`零件：

![code change](/chromatic-change-to-task-component.png)

这会为项目生成新的背景颜色。

![task background change](/chromatic-task-change.png)

使用之前的test命令运行另一个Chromatic测试。

```bash
./node_modules/.bin/chromatic test --app-code=<app-code>
```

点击您将看到更改的网络用户界面链接。

![UI changes in Chromatic](/chromatic-catch-changes.png)

有很多变化！组件层次结构在哪里`Task`是个孩子的`TaskList`和`Inbox`意味着一个小小的调整滚雪球成为主要的回归。这种情况恰恰是开发人员除了其他测试方法之外还需要视觉回归测试的原因。

![UI minor tweaks major regressions](/minor-major-regressions.gif)

## 查看更改

视觉回归测试确保组件不会意外更改。但是，您仍然需要确定更改是否是有意的。

如果有意更改，则需要更新基线，以便将来的测试与故事的最新版本进行比较。如果更改是无意的，则需要修复。

<video autoPlay muted playsInline loop style="width:480px; margin: 0 auto;">
  <source
    src="/website-workflow-review-merge-optimized.mp4"
    type="video/mp4"
  />
</video>

由于现代应用程序是由组件构建的，因此我们在组件级别进行测试非常重要。这样做有助于我们找出变化的根本原因，即组件，而不是对变化的症状，屏幕和复合组件做出反应。

## 合并更改

当我们完成审核后，我们已准备好自信地合并UI更改 - 知道更新不会意外地引入错误。如果你喜欢新的`papayawhip`后台然后接受更改，如果没有恢复到以前的状态。

![Changes ready to be merged](/chromatic-review-finished.png)

故事书可以帮助你**建立**组件;测试可以帮助你**保持**他们。本教程介绍了四种类型的UI测试，包括可视化，快照，单元和可视化回归测试。您可以通过将它们添加到CI脚本来自动执行最后三个。这有助于您运送组件而无需担心偷渡漏洞。整个工作流程如下所示。

![Visual regression testing workflow](/cdd-review-workflow.png)
