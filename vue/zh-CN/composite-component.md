---
title: 'Assemble a composite component'
tocTitle: 'Composite component'
description: 'Assemble a composite component out of simpler components'
commit: c72f06f
---

# 组装复合组件

最后一章我们构建了第一个组件；本章扩展了我们学习构建 TaskList 的任务列表。让我们将组件组合在一起，看看在引入更多复杂性时会发生什么。

## 任务列表

Taskbox 通过将其置于默认任务之上来强调固定任务。这产生了两种变体`TaskList`您需要为以下内容创建故事：默认项目以及默认和固定项目。

![default and pinned tasks](/tasklist-states-1.png)

以来`Task`我们可以异步发送数据**也**在没有连接的情况下需要加载状态进行渲染。此外，当没有任务时，需要空状态。

![empty and loading tasks](/tasklist-states-2.png)

## 现在设置

复合组件与其包含的基本组件没有太大区别。创建一个`TaskList`组件和随附的故事文件：`src/components/TaskList.vue`和`src/components/TaskList.stories.js`。

从粗略的实现开始`TaskList`。你需要导入`Task`来自早期的组件，并将属性和操作作为输入传递。

```html
<template>
  <div>
    <div class="list-items" v-if="loading">loading</div>
    <div class="list-items" v-if="noTasks && !this.loading">empty</div>
    <div class="list-items" v-if="showTasks">
      <task
        v-for="(task, index) in tasks"
        :key="index"
        :task="task"
        @archiveTask="$emit('archiveTask', $event)"
        @pinTask="$emit('pinTask', $event)"
      />
    </div>
  </div>
</template>

<script>
  import Task from './Task';
  export default {
    name: 'task-list',
    props: {
      loading: {
        type: Boolean,
        default: false
      },
      tasks: {
        type: Array,
        default() {
          return [];
        }
      }
    },
    components: {
      Task
    },
    computed: {
      noTasks() {
        return this.tasks.length === 0;
      },
      showTasks() {
        return !this.loading && !this.noTasks;
      }
    }
  };
</script>
```

接下来创建`Tasklist`故事文件中的测试状态。

```javascript
import {storiesOf} from '@storybook/vue';
import {task} from './Task.stories';

import TaskList from './TaskList';
import {methods} from './Task.stories';

export const defaultTaskList = [
  {...task, id: '1', title: 'Task 1'},
  {...task, id: '2', title: 'Task 2'},
  {...task, id: '3', title: 'Task 3'},
  {...task, id: '4', title: 'Task 4'},
  {...task, id: '5', title: 'Task 5'},
  {...task, id: '6', title: 'Task 6'}
];

export const withPinnedTasks = [
  ...defaultTaskList.slice(0, 5),
  {id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED'}
];

const paddedList = () => {
  return {
    template: '<div style="padding: 3rem;"><story/></div>'
  };
};

storiesOf('TaskList', module)
  .addDecorator(paddedList)
  .add('default', () => ({
    components: {TaskList},
    template: `<task-list :tasks="tasks" @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    data: () => ({
      tasks: defaultTaskList
    }),
    methods
  }))
  .add('withPinnedTasks', () => ({
    components: {TaskList},
    template: `<task-list :tasks="tasks" @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    data: () => ({
      tasks: withPinnedTasks
    }),
    methods
  }))
  .add('loading', () => ({
    components: {TaskList},
    template: `<task-list loading @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    methods
  }))
  .add('empty', () => ({
    components: {TaskList},
    template: `<task-list  @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    methods
  }));
```

`addDecorator()`允许我们为每个任务的渲染添加一些“上下文”。在这种情况下，我们在列表周围添加填充，以便更容易进行可视化验证。

<div class="aside">
<a href="https://storybook.js.org/addons/introduction/#1-decorators"><b>Decorators</b></a> are a way to provide arbitrary wrappers to stories. In this case we’re using a decorator to add styling. They can also be used to add other context to components, as we'll see later.
</div>

`task`提供一个的形状`Task`我们创建和导出的`Task.stories.js`文件。同样的，`methods`定义 a 的动作（模拟回调）`Task`组件期望，其中`TaskList`也需要。

现在查看故事书的新内容`TaskList`故事。

<video autoPlay muted playsInline loop>
  <source
    src="/inprogress-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

## 建立国家

我们的组件仍然很粗糙，但现在我们已经了解了要努力的故事。你可能会想到`.list-items`包装过于简单化。你是对的 - 在大多数情况下，我们不会只是添加一个包装器来创建一个新组件。但是**真实的复杂性**的`TaskList`组件在边缘情况下显示`withPinnedTasks`，`loading`，和`empty`。

```html
<template>
  <div>
    <div v-if="loading">
      <div class="loading-item" v-for="(n, index) in 5" :key="index">
        <span class="glow-checkbox" />
        <span class="glow-text">
          <span>Loading</span> <span>cool</span> <span>state</span>
        </span>
      </div>
    </div>
    <div class="list-items" v-if="noTasks && !this.loading">
      <div class="wrapper-message">
        <span class="icon-check" />
        <div class="title-message">You have no tasks</div>
        <div class="subtitle-message">Sit back and relax</div>
      </div>
    </div>
    <div class="list-items" v-if="showTasks">
      <task
        v-for="(task, index) in tasksInOrder"
        :key="index"
        :task="task"
        @archiveTask="$emit('archiveTask', $event)"
        @pinTask="$emit('pinTask', $event)"
      />
    </div>
  </div>
</template>

<script>
  import Task from './Task';
  export default {
    name: 'task-list',
    props: {
      loading: {
        type: Boolean,
        default: false
      },
      tasks: {
        type: Array,
        default() {
          return [];
        }
      }
    },
    components: {
      Task
    },
    computed: {
      noTasks() {
        return this.tasks.length === 0;
      },
      showTasks() {
        return !this.loading && !this.noTasks;
      },
      tasksInOrder() {
        return [
          ...this.tasks.filter(t => t.state === 'TASK_PINNED'),
          ...this.tasks.filter(t => t.state !== 'TASK_PINNED')
        ];
      }
    }
  };
</script>
```

添加的标记会产生以下 UI：

<video autoPlay muted playsInline loop>
  <source
    src="/finished-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

请注意列表中固定项的位置。我们希望固定项目在列表顶部呈现，以使其成为我们用户的优先级。

## 自动化测试

在上一章中，我们学习了如何使用 Storyshots 快照测试故事。同`Task`测试之外没有太多的复杂性，它可以提供。以来`TaskList`增加了另一层复杂性，我们想要验证某些输入以适合自动测试的方式产生某些输出。为此，我们将使用创建单元测试[Jest](https://facebook.github.io/jest/)再加上测试渲染器等[Enzyme](http://airbnb.io/enzyme/)。

![Jest logo](/logo-jest.png)

### 用 Jest 进行单元测试

故事书故事与手动可视化测试和快照测试（见上文）相结合，可以避免 UI 错误。如果故事涵盖了各种各样的组件用例，并且我们使用的工具可以确保人员检查故事的任何变化，那么错误的可能性就大大降低。

然而，有时候魔鬼就是细节。需要一个明确有关这些细节的测试框架。这让我们进行了单元测试。

在我们的例子中，我们想要我们的`TaskList`渲染任何固定的任务**之前**在它中传递的未固定任务`tasks`支柱。虽然我们有一个故事（`withPinnedTasks`）测试这个确切的场景;对于人类评论者来说，如果是组件，则可能是模棱两可的**停止**订购这样的任务，这是一个错误。它肯定不会尖叫**“错误！”**随便的眼睛。

因此，为了避免这个问题，我们可以使用 Jest 将故事呈现给 DOM 并运行一些 DOM 查询代码来验证输出的显着特征。

创建一个名为的测试文件`tests/unit/TaskList.spec.js`。在这里，我们将构建我们的测试，对输出进行断言。

```javascript
import Vue from 'vue';
import TaskList from '../../src/components/TaskList.vue';
import {withPinnedTasks} from '../../src/components/TaskList.stories';

it('renders pinned tasks at the start of the list', () => {
  const Constructor = Vue.extend(TaskList);
  const vm = new Constructor({
    propsData: {tasks: withPinnedTasks}
  }).$mount();
  const lastTaskInput = vm.$el.querySelector(
    '.list-item:nth-child(1).TASK_PINNED'
  );

  // We expect the pinned task to be rendered first, not at the end
  expect(lastTaskInput).not.toBe(null);
});
```

![TaskList test runner](/tasklist-testrunner.png)

请注意，我们已经能够重用`withPinnedTasks`故事和单元测试中的任务列表;通过这种方式，我们可以继续以越来越多的方式利用现有资源（代表组件的有趣配置的示例）。

另请注意，此测试非常脆弱。随着项目的成熟，以及项目的确切实施，这可能是可能的`Task`更改 - 可能使用不同的类名 - 测试将失败，需要更新。这不一定是一个问题，而是使用 UI 的单元测试自由地小心的指示。它们不容易维护。而是依靠视觉，快照和视觉回归（参见[testing chapter](/test/)）尽可能测试。
