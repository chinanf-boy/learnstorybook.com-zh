---
title: '组装，一个复合组件'
tocTitle: '复合组件'
description: '使用更简单的组件，组装一个复合组件'
commit: c72f06f
---

# 组装一个复合组件

上一章我们构建了第一个组件; 本章扩展学习，构建 TaskList ，一个任务列表。 让我们将组件，组合在一起，看看在引入更多复杂性时，会发生什么。

## 任务列表

Taskbox 通过将，选中任务，置于默认任务之上，来强调选中任务。这产生了`TaskList`的两种变体，您需要为以下内容创建故事: 默认项，以及默认与选中项。

![default and pinned tasks](/tasklist-states-1.png)

`Task`可以异步发送数据，我们 **也**需要在没有连接的情况下，具有加载的渲染 _右图_。 此外，没有任务时，要有空的状态 _左图_。

![empty and loading tasks](/tasklist-states-2.png)

## 现在设置

复合组件，与其包含的基本组件没有太大区别。创建一个`TaskList`组件和对应的故事文件: `src/components/TaskList.vue`和`src/components/TaskList.stories.js`。

让我们开始粗略实现`TaskList`。你需要导入早期的`Task`组件，并将 属性和行为，作为输入传递。

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

接下来，在故事文件中，创建`Tasklist`的测试状态。

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

`addDecorator()`允许我们为，每个任务的渲染，添加一些"上下文"。 在这种情况下，我们在列表，周围添加 _padding_，以便更容易进行视觉测试。

<div class="aside">
<a href="https://storybook.js.org/addons/introduction/#1-decorators"><b>Decorators-装饰器</b></a> 是一种为 stories 提供任意包装的方法。 在这种情况下，我们使用装饰器来添加样式。还可以用来，添加其他上下文到组件中，我们后面会看到。
</div>

`task`提供一个`Task`的形状，来自我们创建和导出的`Task.stories.js`文件。 同样的，`methods`定义`Task`组件期望的行为 (模拟回调)，而这也是`TaskList`所需要。

现在查看 Storybook 的 新`TaskList`故事。

<video autoPlay muted playsInline loop>
  <source
    src="/inprogress-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

## 建立状态

我们的组件，仍然很粗糙，但现在我们对要努力的故事，有了想法。 你可能会想到`.list-items`包装，过于简单化。 你是对的 - 在大多数情况下，我们不会只是添加一个包装器，来创建一个新的组件。 但是`TaskList`组件**真正的复杂** ，是在边缘情况`withPinnedTasks`，`loading`，和`empty`的显示情况。

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

添加的标记语言，会产生以下 UI:

<video autoPlay muted playsInline loop>
  <source
    src="/finished-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

请注意列表中， 选中项的位置。 我们希望选中项，在列表顶部呈现，以使其成为我们用户的优先事项。

## 自动化测试

在上一章中，我们学习了如何使用 Storyshots ，进行快照测试 故事。 `Task`测试没有太多的复杂性，已然够用了。 而`TaskList`增加了另一层复杂性，我们希望，用自动测试的方式，验证某些输入，并产生某些输出。 为此，我们将使用[jest-笑话](https://facebook.github.io/jest/)，再加上测试渲染器(例如)[Enzyme](http://airbnb.io/enzyme/)，创建单元测试。

![Jest logo](/logo-jest.png)

### 用 Jest 进行单元测试

Storybook stories 与手动视觉测试，和快照测试 (见上) 相结合，可以避免 UI 错误。 如果故事涵盖了，各种各样的组件用例，并且我们使用的工具，可以让人检查，确保故事的任何变化，那么错误的可能性，就大大降低。

然而，有时候，魔鬼是在细节中的。一个明确这些细节的测试框架是需要的。这也给我们带来了单元测试。

在我们的例子中，我们希望我们的`TaskList`，在传递不选中的任务之前，渲染所有的选中项。 虽然我们有一个故事 (`withPinnedTasks`) ，测试这个确切的场景; 但是如果组件停止对这样的任务进行排序，那么就人类看着来说，这可能是不明确的，_因为只看到表面与行为_， 这是一个 bug。 但它不会尖叫：**"错误!"**，直怼眼睛。

因此，为了避免这个问题，我们可以使用 Jest 将故事呈现给`DOM`，并运行一些`DOM`查询代码，来验证输出的显着特征。

创建一个名为`src/components/TaskList.spec.js`的测试文件。 在这里，我们将构建我们的测试，并对输出进行断言。

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

  // 我们期望先，渲染选中的任务，而不是最后
  expect(lastTaskInput).not.toBe(null);
});
```

![TaskList test runner](/tasklist-testrunner.png)

请注意，我们已经能够重用`withPinnedTasks`故事，和单元测试中的任务列表；通过这种方式，我们可以继续，以越来越多的方式，利用现有资源 (一个组件的有趣配置示例) 。

另请注意，此测试非常脆弱。随着项目的成熟，以及项目的确切实现，这都可能是`Task`的更改 - 可能使用不同的类名，或`textarea`而不是一个`input`- 测试将失败，需要更新。 这不一定是一个问题，但 UI 的单元测试，要小心谨慎。它们不容易维护，替而代之的是，依靠视觉，快照和视觉回归 (参见[测试章节](/test/)) 的 Storybook 测试。
