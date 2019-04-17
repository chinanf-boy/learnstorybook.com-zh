---
title: "Construct a screen"
tocTitle: "Screens"
description: "Construct a screen out of components"
commit: b62db62
---

# 构建一个屏幕

我们专注于自下而上构建UI;从小做起并增加复杂性。这样做使我们能够孤立地开发每个组件，找出其数据需求，并在Storybook中使用它。所有这些都无需站起服务器或构建屏幕！

在本章中，我们通过组合屏幕中的组件并在Storybook中开发该屏幕来继续提高复杂性。

## 嵌套容器组件

由于我们的应用程序非常简单，我们将构建的屏幕非常简单，只需简单地包装`TaskList`容器组件（通过Vuex提供自己的数据）在某些布局中并拉出顶层`error`商店外的字段（我们假设如果连接到我们的服务器有问题，我们将设置该字段）。让我们创造一个表现形式`PureInboxScreen.vue`在你的`src/components/`夹：

```html
<template>
  <div>
    <div class="page lists-show" v-if="error">
      <div class="wrapper-message">
        <span class="icon-face-sad" />
        <div class="title-message">Oh no!</div>
        <div class="subtitle-message">Something went wrong</div>
      </div>
    </div>
    <div class="page lists-show" v-else>
      <nav>
        <h1 class="title-page">
          <span class="title-wrapper">Taskbox</span>
        </h1>
      </nav>
      <task-list/>
    </div>
  </div>
</template>

<script>
import TaskList from "./TaskList.vue";

export default {
  name: "pure-inbox-screen",
  props: {
    error: {
      type: Boolean,
      default: false
    }
  },
  components: {
    TaskList
  }
};
</script>
```

然后，我们可以创建一个容器，再次抓取数据`PureInboxScreen`在`src/components/InboxScreen.vue`：

```html
<template>
  <div>
    <pure-inbox-screen :error="error"/>
  </div>
</template>

<script>
import PureInboxScreen from "./PureInboxScreen";
import { mapState } from "vuex";

export default {
  name: "inbox-screen",
  components: {
    PureInboxScreen
  },
  computed: {
    ...mapState(["error"])
  }
};
</script>
```

我们也改变了`App`用于渲染的组件`InboxScreen`（最终我们会使用路由器来选择正确的屏幕，但我们不用担心这一点）：

```html
<template>
  <div id="app">
    <inbox-screen/>
  </div>
</template>

<script>
import store from "./store";
import InboxScreen from "./components/InboxScreen.vue";
import "../src/index.css";

export default {
  name: "app",
  store,
  components: {
    InboxScreen
  }
};
</script>
```

然而，事情变得有趣的是在故事书中渲染故事。

正如我们之前看到的那样`TaskList`组件是一个**容器**这使得`PureTaskList`表现部分。根据定义，容器组件不能简单地单独呈现;他们希望通过一些上下文或连接到服务。这意味着要在Storybook中呈现容器，我们必须模拟（即提供假装版本）它所需的上下文或服务。

放置时`TaskList`进入故事书，我们能够通过简单地渲染这个问题来回避这个问题`PureTaskList`并避免容器。我们会做类似的事情并渲染`PureInboxScreen`在故事书中也是。

但是，对于`PureInboxScreen`我们有一个问题，因为虽然`PureInboxScreen`本身就是表现，它的孩子，`TaskList`， 不是。从某种意义上说`PureInboxScreen`被“容器”污染了。所以，当我们设置我们的故事`src/components/PureInboxScreen.stories.js`：

```javascript
import { storiesOf } from '@storybook/vue';
import PureInboxScreen from './PureInboxScreen';

storiesOf('PureInboxScreen', module)
  .add('default', () => {
    return {
      components: { PureInboxScreen },
      template: `<pure-inbox-screen/>`,
    };
  })
  .add('error', () => {
    return {
      components: { PureInboxScreen },
      template: `<pure-inbox-screen :error="true"/>`,
    };
  });
```

我们看到了虽然`error`故事工作得很好，我们有一个问题`default`故事，因为`TaskList`没有要连接的Vuex商店。（在尝试测试时，您也会遇到类似的问题`PureInboxScreen`用单元测试）。

![Broken inbox](/broken-inboxscreen-vue.png)

避免此问题的一种方法是永远不要在应用程序中的任何位置呈现容器组件，除非在最高级别，而是将所有数据要求传递到组件层次结构中。

但是，开发人员**将**不可避免地需要在组件层次结构中进一步渲染容器。如果我们想在Storybook中渲染大部分或全部应用程序（我们这样做！），我们需要一个解决此问题的方法。

<div class="aside">
As an aside, passing data down the hierarchy is a legitimate approach, especially when using <a href="http://graphql.org/">GraphQL</a>. It’s how we have built <a href="https://www.chromaticqa.com">Chromatic</a> alongside 670+ stories.
</div>

## 为故事提供背景

好消息是很容易提供Vuex商店`PureInboxScreen`在一个故事！我们可以在故事文件中创建一个新商店，并将其作为故事的上下文传递：

```javascript
import { action } from '@storybook/addon-actions';
import { storiesOf } from '@storybook/vue';
import Vue from 'vue';
import Vuex from 'vuex';

import { defaultTaskList } from './PureTaskList.stories';
import PureInboxScreen from './PureInboxScreen.vue';

Vue.use(Vuex);
export const store = new Vuex.Store({
  state: {
    tasks: defaultTaskList,
  },
  actions: {
    pinTask(context, id) {
      action('pinTask')(id);
    },
    archiveTask(context, id) {
      action('archiveTask')(id);
    },
  },
});

storiesOf('PureInboxScreen', module)
  .add('default', () => {
    return {
      components: { PureInboxScreen },
      template: `<pure-inbox-screen/>`,
      store,
    };
  })
  .add('error', () => {
    return {
      components: { PureInboxScreen },
      template: `<pure-inbox-screen :error="true"/>`,
    };
  });
```

存在类似的方法来为其他数据库提供模拟的上下文，例如[Apollo](https://www.npmjs.com/package/apollo-storybook-decorator)，[Relay](https://github.com/orta/react-storybooks-relay-container)和别的。

在Storybook中循环浏览状态可以轻松测试我们是否已正确完成此操作：

<video autoPlay muted playsInline loop >

  <source
    src="/finished-inboxscreen-states.mp4"
    type="video/mp4"
  />
</video>

## 组件驱动开发

我们从底部开始`Task`，然后进展到`TaskList`，现在我们在这里使用全屏UI。我们的`InboxScreen`容纳嵌套的容器组件并包括随附的故事。

<video autoPlay muted playsInline loop style="width:480px; height:auto; margin: 0 auto;">
  <source
    src="/component-driven-development-optimized.mp4"
    type="video/mp4"
  />
</video>

[**Component-Driven Development**](https://blog.hichroma.com/component-driven-development-ce1109d56c8e)允许您在向上移动组件层次结构时逐渐扩展复杂性。其中的好处包括更集中的开发过程以及所有可能的UI排列的覆盖范围。简而言之，CDD可以帮助您构建更高质量和更复杂的用户界面。

我们还没有完成 - 在构建UI时，工作不会结束。我们还需要确保它随着时间的推移保持持久。
