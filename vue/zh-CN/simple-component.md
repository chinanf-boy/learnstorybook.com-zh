---
title: 'Build a simple component'
tocTitle: 'Simple component'
description: 'Build a simple component in isolation'
commit: b2274bd
---

# 构建简单组件

我们将按照[Component-Driven Development](https://blog.hichroma.com/component-driven-development-ce1109d56c8e)（CDD）方法。这是一个从“自下而上”构建 uis 的过程，从组件开始，到屏幕结束。CDD 帮助您在构建 UI 时扩展所面临的复杂性。

## 任务

![Task component in three states](/task-states-learnstorybook.png)

`Task`是我们应用程序的核心组件。根据任务的具体状态，每个任务的显示方式略有不同。我们会显示一个选中（或未选中）的复选框，一些关于任务的信息，以及一个“固定”按钮，允许我们在列表中上下移动任务。把这些放在一起，我们需要这些道具：

- `title`–描述任务的字符串
- `state`-任务当前在哪个列表中，是否已签出？

当我们开始建造`Task`首先，我们编写与上述不同类型的任务草图相对应的测试状态。然后我们使用故事书使用模拟数据隔离地构建组件。我们将“视觉测试”组件的外观，并在我们进行的过程中给出每个状态。

这个过程类似于[Test-driven development](https://en.wikipedia.org/wiki/Test-driven_development)（TDD）我们可以打电话给[Visual TDD](https://blog.hichroma.com/visual-test-driven-development-aec1c98bed87)“。

## 现在设置

首先，让我们创建任务组件及其附带的故事文件：`src/components/Task.vue`和`src/components/Task.stories.js`.

我们将从`Task`，只需输入我们知道需要的属性和您可以对任务执行的两个操作（在列表之间移动）：

```html
<template>
  <div class="list-item">
    <input type="text" :readonly="true" :value="this.task.title" />
  </div>
</template>

<script>
  export default {
    name: 'task',
    props: {
      task: {
        type: Object,
        required: true
      }
    }
  };
</script>
```

上面，我们为`Task`基于 todos 应用程序的现有 HTML 结构。

下面我们在故事文件中构建任务的三个测试状态：

```javascript
import {storiesOf} from '@storybook/vue';
import {action} from '@storybook/addon-actions';

import Task from './Task';

export const task = {
  id: '1',
  title: 'Test Task',
  state: 'TASK_INBOX',
  updatedAt: new Date(2018, 0, 1, 9, 0)
};

export const methods = {
  onPinTask: action('onPinTask'),
  onArchiveTask: action('onArchiveTask')
};

storiesOf('Task', module)
  .add('default', () => {
    return {
      components: {Task},
      template: `<task :task="task" @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
      data: () => ({task}),
      methods
    };
  })
  .add('pinned', () => {
    return {
      components: {Task},
      template: `<task :task="task" @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
      data: () => ({task: {...task, state: 'TASK_PINNED'}}),
      methods
    };
  })
  .add('archived', () => {
    return {
      components: {Task},
      template: `<task :task="task" @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
      data: () => ({task: {...task, state: 'TASK_ARCHIVED'}}),
      methods
    };
  });
```

故事书中有两个基本的组织层次。组件及其子故事。把每一个故事看作是一个组成部分的排列。您可以根据需要为每个组件提供任意多的故事。

- **成分**
  - 故事
  - 故事
  - 故事

为了启动故事书，我们首先调用`storiesOf()`函数来注册组件。我们为组件添加了一个显示名称——这个名称出现在故事书应用程序的侧边栏上。

`action()`允许我们创建出现在**行动**单击时故事书用户界面的面板。因此，当我们构建一个 pin 按钮时，我们将能够在测试 UI 中确定按钮单击是否成功。

由于我们需要将相同的操作集传递给组件的所有排列，因此可以方便地将它们组合成一个`methods`变量和用途每次都将它们传递到我们的故事定义中（它们通过`methods`财产）。

另一个好处是捆绑`methods`组件需要的是`export`它们并在重用此组件的组件的故事中使用它们，稍后我们将看到。

为了定义我们的故事，我们称之为`add()`每个测试状态生成一个故事。动作故事是一个函数，它返回一组定义故事的属性——在本例中是`template`故事的字符串`components`，`data`和`methods`该模板将使用。

创建故事时，我们使用基本任务（`task`）以构建组件期望的任务形状。这通常是根据真实数据的样子建模的。再一次，`export`-使用这个形状将使我们能够在以后的故事中重用它，如我们将看到的。

<div class="aside">
<a href="https://storybook.js.org/addons/introduction/#2-native-addons"><b>Actions</b></a> help you verify interactions when building UI components in isolation. Oftentimes you won't have access to the functions and state you have in context of the app. Use <code>action()</code> to stub them in.
</div>

## 配置

我们还必须对故事书配置设置进行一个小的更改。（`.storybook/config.js`）所以它注意到我们`.stories.js`文件并使用我们的 CSS 文件。默认情况下，故事书在`/stories`目录；本教程使用的命名方案与`.spec.js`Vue CLI 支持的用于自动测试的命名方案。

```javascript
import {configure} from '@storybook/vue';

import '../src/index.css';

const req = require.context('../src', true, /.stories.js$/);
function loadStories() {
  req.keys().forEach(filename => req(filename));
}

configure(loadStories, module);
```

完成此操作后，重新启动 Storybook 服务器将生成三种任务状态的测试用例：

<video autoPlay muted playsInline controls >
  <source
    src="/inprogress-task-states.mp4"
    type="video/mp4"
  />
</video>

## 建设国家

现在我们有了故事书设置、导入的样式和构建的测试用例，我们可以快速开始实现组件的 HTML 以匹配设计的工作。

组件目前仍然是基本的。首先编写实现设计而不需要太多细节的代码：

```html
<template>
  <div :class="taskClass">
    <label class="checkbox">
      <input
        type="checkbox"
        :checked="isChecked"
        :disabled="true"
        name="checked"
      />
      <span class="checkbox-custom" @click="$emit('archiveTask', task.id)" />
    </label>
    <div class="title">
      <input
        type="text"
        :readonly="true"
        :value="this.task.title"
        placeholder="Input title"
      />
    </div>
    <div class="actions">
      <a @click="$emit('pinTask', task.id)" v-if="!isChecked">
        <span class="icon-star" />
      </a>
    </div>
  </div>
</template>

<script>
  export default {
    name: 'task',
    props: {
      task: {
        type: Object,
        required: true
      }
    },
    computed: {
      taskClass() {
        return `list-item ${this.task.state}`;
      },
      isChecked() {
        return this.task.state === 'TASK_ARCHIVED';
      }
    }
  };
</script>
```

上面的附加标记与我们先前导入的 CSS 结合在一起，将生成以下 UI：

<video autoPlay muted playsInline loop>
  <source
    src="/finished-task-states.mp4"
    type="video/mp4"
  />
</video>

## 组件已构建！

我们现在成功地构建了一个组件，不需要服务器或运行整个前端应用程序。下一步是以类似的方式逐个构建剩余的任务框组件。

如您所见，孤立地开始构建组件既简单又快速。我们可以期望用更少的 bug 和更多的修饰来生成一个更高质量的 UI，因为它可以挖掘并测试所有可能的状态。

## 自动化测试

故事书为我们提供了一种在构建过程中可视化测试应用程序的好方法。“故事”将有助于确保我们不会在继续开发应用程序的过程中以视觉方式破坏我们的任务。然而，在这个阶段，这是一个完全手工的过程，必须有人努力点击每个测试状态，确保它表现良好，没有错误或警告。我们不能自动完成吗？

### 快照测试

快照测试是指为给定的输入记录组件的“已知良好”输出，然后在将来输出更改时标记组件的实践。这是对故事书的补充，因为故事书是查看组件新版本和可视化更改的快速方法。

<div class="aside">
Make sure your components render data that doesn't change, so that your snapshot tests won't fail each time. Watch out for things like dates or randomly generated values.
</div>

与[Storyshots addon](https://github.com/storybooks/storybook/tree/master/addons/storyshots)将为每个故事创建快照测试。通过添加对包的开发依赖项来使用它：

```bash
yarn add --dev @storybook/addon-storyshots jest-vue-preprocessor babel-plugin-require-context-hook
```

然后创建一个`tests/unit/storybook.spec.js`包含以下内容的文件：

```javascript
import registerRequireContextHook from 'babel-plugin-require-context-hook/register';
import initStoryshots from '@storybook/addon-storyshots';

registerRequireContextHook();
initStoryshots();
```

我们需要在我们的`jest.config.js`：

```js
  transformIgnorePatterns: ["/node_modules/(?!(@storybook/.*\\.vue$))"],
```

最后，我们需要对`babel.config.js`：

```js
module.exports = api => ({
  presets: ['@vue/app'],
  ...(api.env('test') && {plugins: ['require-context-hook']})
});
```

一旦完成以上操作，我们就可以运行`yarn test:unit`见以下输出：

![Task test runner](/task-testrunner.png)

现在我们为每个`Task`故事。如果我们改变`Task`，将提示我们验证更改。
