---
title: "Wire in data"
tocTitle: "Data"
description: "Learn how to wire in data to your UI component"
commit: 28bc240
---

# 连线数据

到目前为止，我们创建了孤立的无状态组件 - 对于Storybook非常有用，但在我们在应用程序中提供一些数据之前最终没有用处。

本教程不关注构建应用程序的细节，因此我们不会在此处深入研究这些细节。但我们将花点时间研究一下与容器组件连接数据的常见模式。

## 容器组件

我们的`TaskList`目前编写的组件是“表现性的”（见[this blog post](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)因为它不会与其自身实现之外的任何内容交谈。要获取数据，我们需要一个“容器”。

这个例子使用[Vuex](https://vuex.vuejs.org)，Vue的默认数据管理库，为我们的应用程序构建一个简单的数据模型。但是，此处使用的模式同样适用于其他数据管理库[Apollo](https://www.apollographql.com/client/)和[MobX](https://mobx.js.org/)。

首先，安装vuex

```bash
yarn add vuex
```

然后我们将构建一个简单的Vuex存储，它响应在一个名为的文件中改变任务状态的操作`src/store.js`（故意保持简单）：

```javascript
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    tasks: [
      { id: '1', title: 'Something', state: 'TASK_INBOX' },
      { id: '2', title: 'Something more', state: 'TASK_INBOX' },
      { id: '3', title: 'Something else', state: 'TASK_INBOX' },
      { id: '4', title: 'Something again', state: 'TASK_INBOX' },
    ],
  },
  mutations: {
    ARCHIVE_TASK(state, id) {
      state.tasks.find(task => task.id === id).state = 'TASK_ARCHIVED';
    },
    PIN_TASK(state, id) {
      state.tasks.find(task => task.id === id).state = 'TASK_PINNED';
    },
  },
  actions: {
    archiveTask({ commit }, id) {
      commit('ARCHIVE_TASK', id);
    },
    pinTask({ commit }, id) {
      commit('PIN_TASK', id);
    },
  },
});
```

在我们的顶级应用程序组件中（`src/App.vue`我们可以轻松地将商店连接到我们的组件层次故障：

```html
<template>
  <div id="app">
    <task-list/>
  </div>
</template>

<script>
import store from "./store";
import TaskList from "./components/TaskList.vue";
import "../src/index.css";

export default {
  name: "app",
  store,
  components: {
    TaskList
  }
};
</script>
```

然后我们会更新我们的`TaskList`从商店中读取数据。首先让我们将现有的表现版本移到文件中`src/components/PureTaskList.vue`（将组件重命名为`pure-task-list`），并用容器包装。

在`src/components/PureTaskList.vue`：

```html
/* This file moved from TaskList.vue */
<template>/* as before */

<script>
import Task from "./Task";
export default {
  name: "pure-task-list",
  ...
}
```

在`src/components/TaskList.vue`：

```html
<template>
  <div>
    <pure-task-list :tasks="tasks"/>
  </div>
</template>

<script>
import PureTaskList from "./PureTaskList";
import { mapState } from "vuex";

export default {
  name: "task-list",
  components: {
    PureTaskList
  },
  computed: {
    ...mapState(["tasks"])
  }
};
</script>
```

保持表现形式的原因`TaskList`分开是因为它更容易测试和隔离。由于它不依赖于商店的存在，因此从测试角度来看处理起来要容易得多。我们重命名`src/components/TaskList.stories.js`成`src/components/PureTaskList.stories.js`，并确保我们的故事使用表现版：

```javascript
import { storiesOf } from '@storybook/vue';
import { task } from './Task.stories';

import PureTaskList from './PureTaskList';
import { methods } from './Task.stories';

export const defaultTaskList = [
  { ...task, id: '1', title: 'Task 1' },
  { ...task, id: '2', title: 'Task 2' },
  { ...task, id: '3', title: 'Task 3' },
  { ...task, id: '4', title: 'Task 4' },
  { ...task, id: '5', title: 'Task 5' },
  { ...task, id: '6', title: 'Task 6' },
];

export const withPinnedTasks = [
  ...defaultTaskList.slice(0, 5),
  { id: '6', title: 'Task 6 (pinned)', state: 'TASK_PINNED' },
];

const paddedList = () => {
  return {
    template: '<div style="padding: 3rem;"><story/></div>',
  };
};

storiesOf('PureTaskList', module)
  .addDecorator(paddedList)
  .add('default', () => ({
    components: { PureTaskList },
    template: `<pure-task-list :tasks="tasks" @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    data: () => ({
      tasks: defaultTaskList,
    }),
    methods,
  }))
  .add('withPinnedTasks', () => ({
    components: { PureTaskList },
    template: `<pure-task-list :tasks="tasks" @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    data: () => ({
      tasks: withPinnedTasks,
    }),
    methods,
  }))
  .add('loading', () => ({
    components: { PureTaskList },
    template: `<pure-task-list loading @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    methods,
  }))
  .add('empty', () => ({
    components: { PureTaskList },
    template: `<pure-task-list  @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    methods,
  }));
```

<video autoPlay muted playsInline loop>
  <source
    src="/finished-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

同样，我们需要使用`PureTaskList`在我们的Jest测试中：

```js
import Vue from 'vue';
import PureTaskList from '../../src/components/PureTaskList.vue';
import { withPinnedTasks } from '../../src/components/PureTaskList.stories';

it('renders pinned tasks at the start of the list', () => {
  const Constructor = Vue.extend(PureTaskList);
  const vm = new Constructor({
    propsData: { tasks: withPinnedTasks },
  }).$mount();
  const lastTaskInput = vm.$el.querySelector('.list-item:nth-child(1).TASK_PINNED');

  // We expect the pinned task to be rendered first, not at the end
  expect(lastTaskInput).not.toBe(null);
});
```
