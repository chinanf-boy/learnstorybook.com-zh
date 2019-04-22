---
title: '接连数据'
tocTitle: '数据'
description: '了解如何将数据，接连到 UI 组件'
commit: 28bc240
---

# 接连数据

到目前为止，我们创建了孤立的无状态组件 - Storybook 很棒，但作用不大，除非我们在应用程序中，为他们提供一些数据。

本教程不关注构建应用程序的细节，因此我们不会在此处深入研究这些细节。但我们将花点时间研究一下，容器组件接连数据的常见模式。

## 容器组件

我们的`TaskList`目前编写的组件是"外观(presentational)"型的 (见[这篇博文](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)) 因为它不会与其自身实现之外的任何内容交谈。为了获取数据，我们需要一个"容器"。

这个例子使用[Vuex](https://vuex.vuejs.org)，Vue 的默认数据管理库，构建一个简单的数据模型。但是，此处使用的模式，同样适用于其他数据管理库，如[Apollo](https://www.apollographql.com/client/)和[MobX](https://mobx.js.org/)。

首先，安装 vuex

```bash
yarn add vuex
```

然后我们将构建一个简单的 Vuex 存储，它响应在一个名为`src/store.js`的文件中，改变任务状态的操作（故意保持简单）：

```javascript
import Vue from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex);

export default new Vuex.Store({
  state: {
    tasks: [
      {id: '1', title: 'Something', state: 'TASK_INBOX'},
      {id: '2', title: 'Something more', state: 'TASK_INBOX'},
      {id: '3', title: 'Something else', state: 'TASK_INBOX'},
      {id: '4', title: 'Something again', state: 'TASK_INBOX'}
    ]
  },
  mutations: {
    ARCHIVE_TASK(state, id) {
      state.tasks.find(task => task.id === id).state = 'TASK_ARCHIVED';
    },
    PIN_TASK(state, id) {
      state.tasks.find(task => task.id === id).state = 'TASK_PINNED';
    }
  },
  actions: {
    archiveTask({commit}, id) {
      commit('ARCHIVE_TASK', id);
    },
    pinTask({commit}, id) {
      commit('PIN_TASK', id);
    }
  }
});
```

在我们的顶级应用程序组件（`src/App.vue`）中，我们可以轻松地将 store 连接到我们的组件层：

```html
<template>
  <div id="app">
    <task-list />
  </div>
</template>

<script>
  import store from './store';
  import TaskList from './components/TaskList.vue';
  import '../src/index.css';

  export default {
    name: 'app',
    store,
    components: {
      TaskList
    }
  };
</script>
```

然后我们会更新我们的`TaskList`，从 store 中读取数据。首先将我们现有的外观型组件版本，移到文件`src/components/PureTaskList.vue`中（将组件重命名为`pure-task-list`），并用一个容器组件包装。

在`src/components/PureTaskList.vue`：

```html
/* 该文件来自 TaskList.vue */
<template>/* 与之前一样 */

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
    <pure-task-list :tasks="tasks" />
  </div>
</template>

<script>
  import PureTaskList from './PureTaskList';
  import {mapState} from 'vuex';

  export default {
    name: 'task-list',
    components: {
      PureTaskList
    },
    computed: {
      ...mapState(['tasks'])
    }
  };
</script>
```

保留`TaskList`组件的外观型版本的原因，是因为分开它，更容易测试和隔离。由于它不依赖于 store 的存在，因此从测试角度来看，处理起来要容易得多。我们把`src/components/TaskList.stories.js`重命名成`src/components/PureTaskList.stories.js`，并确保我们的故事使用外观型版本：

```javascript
import {storiesOf} from '@storybook/vue';
import {task} from './Task.stories';

import PureTaskList from './PureTaskList';
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

storiesOf('PureTaskList', module)
  .addDecorator(paddedList)
  .add('default', () => ({
    components: {PureTaskList},
    template: `<pure-task-list :tasks="tasks" @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    data: () => ({
      tasks: defaultTaskList
    }),
    methods
  }))
  .add('withPinnedTasks', () => ({
    components: {PureTaskList},
    template: `<pure-task-list :tasks="tasks" @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    data: () => ({
      tasks: withPinnedTasks
    }),
    methods
  }))
  .add('loading', () => ({
    components: {PureTaskList},
    template: `<pure-task-list loading @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    methods
  }))
  .add('empty', () => ({
    components: {PureTaskList},
    template: `<pure-task-list  @archiveTask="onArchiveTask" @pinTask="onPinTask"/>`,
    methods
  }));
```

<video autoPlay muted playsInline loop>
  <source
    src="/finished-tasklist-states.mp4"
    type="video/mp4"
  />
</video>

同样，我们需要在我们的 Jest 测试中，使用`PureTaskList`:

```js
import Vue from 'vue';
import PureTaskList from '../../src/components/PureTaskList.vue';
import {withPinnedTasks} from '../../src/components/PureTaskList.stories';

it('renders pinned tasks at the start of the list', () => {
  const Constructor = Vue.extend(PureTaskList);
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
