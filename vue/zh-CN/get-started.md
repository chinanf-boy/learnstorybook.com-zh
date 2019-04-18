---
title: "Get started"
tocTitle: "Get started"
description: "Setup Vue Storybook in your development environment"
commit: d1c4858
---

# 开始吧

故事书在开发模式下与您的应用程序一起运行。它可以帮助您构建与应用程序的业务逻辑和上下文隔离的UI组件。本期“学习故事书”适用于Vue;其他版本的存在[React](/react/en/get-started)和[Angular](/angular/en/get-started)。

![Storybook and your app](/storybook-relationship.jpg)

## 设置Vue故事书

我们需要按照几个步骤在您的环境中设置构建过程。首先，我们想要使用[Vue CLI](https://cli.vuejs.org)设置我们的构建系统，并启用[Storybook](https://storybook.js.org/)和[Jest](https://facebook.github.io/jest/)在我们创建的应用中测试。我们运行以下命令：

```bash
# Create our application, using a preset that contains jest:
npx -p @vue/cli vue create --preset hichroma/vue-preset-learnstorybook taskbox
cd taskbox

# Add Storybook:
npx -p @storybook/cli sb init
```

我们可以快速检查应用程序的各种环境是否正常工作：

```bash
# Run the test runner (Jest) in a terminal:
yarn test:unit

# Start the component explorer on port 6006:
yarn run storybook

# Run the frontend app proper on port 8080:
yarn serve
```

<div class="aside">
  NOTE: If <code>yarn test:unit</code> throws an error, you may not have <a href="https://yarnpkg.com/lang/en/docs/install/">yarn installed</a> or you may need to install <code>watchman</code> as advised in <a href="https://github.com/facebook/create-react-app/issues/871#issuecomment-252297884">this issue</a>.
</div>

我们的三种前端应用程序模式：自动化测试（Jest），组件开发（Storybook）和应用程序本身。

![3 modalities](/app-three-modalities-vue.png)

根据您正在处理的应用程序的哪个部分，您可能希望同时运行其中一个或多个。由于我们目前的重点是创建单个UI组件，因此我们将坚持运行Storybook。

## 重用CSS

Taskbox重用了GraphQL和React Tutorial中的设计元素[example app](https://blog.hichroma.com/graphql-react-tutorial-part-1-6-d0691af25858)，所以我们不需要在本教程中编写CSS。我们只需将LESS编译为单个CSS文件并将其包含在我们的应用程序中。复制和粘贴[this compiled CSS](https://github.com/hichroma/learnstorybook-code/blob/master/src/index.css)成`src/index.css`然后通过编辑将CSS导入应用程序`<style>`标签`src/App.vue`它看起来像：

```html
<style>
@import './index.css';
</style>
```

![Taskbox UI](/ss-browserchrome-taskbox-learnstorybook.png)

<div class="aside">
If you want to modify the styling, the source LESS files are provided in the GitHub repo.
</div>

## 添加资产

我们还需要添加字体和图标[directories](https://github.com/hichroma/learnstorybook-code/tree/master/public)到了`public/`夹。

我们还需要更新我们的故事书脚本来服务`public`目录（在`package.json`）：

```json
{
  "scripts": {
    "storybook": "start-storybook -p 6006 -s public"
  }
}
```

添加样式和资产后，应用程序将呈现一些奇怪的。没关系。我们现在还没有开发应用程序。我们开始构建我们的第一个组件！
