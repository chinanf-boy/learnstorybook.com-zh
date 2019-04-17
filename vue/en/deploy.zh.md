---
title: 'Deploy Storybook'
tocTitle: 'Deploy'
description: 'Deploy Storybook online with GitHub and Netlify'
---

# 部署故事书

在本教程中，我们在开发机器上运行了 Storybook。您可能还想与团队分享该故事书，尤其是非技术成员。值得庆幸的是，在线部署 Storybook 很容易。

<div class="aside">
<strong>Did you setup Chromatic testing earlier?</strong>
<br/>
🎉 Your stories are already deployed! Chromatic securely indexes your stories online and tracks them across branches and commits. Skip this chapter and go to the <a href="/vue/en/conclusion">conclusion</a>.
</div>

## 导出为静态应用程序

要部署 Storybook，我们首先需要将其导出为静态 Web 应用程序。这个功能已经内置到 Storybook 中，我们只需要通过添加脚本来激活它`package.json`。

```javascript
{
  "scripts": {
    "build-storybook": "build-storybook -c .storybook -s public -o storybook-static"
  }
}
```

现在当你通过故事书运行时`npm run build-storybook`，它会输出一个静态的故事书`storybook-static`目录。

## 持续部署

每当我们推送代码时，我们都希望共享最新版本的组件。为此，我们需要不断部署 Storybook。我们将依靠 GitHub 和 Netlify 来部署我们的静态站点。我们正在使用 Netlify 免费计划。

### GitHub 上

首先，您要在本地目录中为项目设置 Git。如果您从上一个测试章节开始，请跳转到在 GitHub 上设置存储库。

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

转到 GitHub 并设置存储库[here](https://github.com/new)。将您的仓库命名为“任务箱”。

![GitHub setup](/github-create-taskbox.png)

在新的 repo 设置中，复制 repo 的原始 URL，并使用以下命令将其添加到 git 项目中：

```bash
$ git remote add origin https://github.com/<your username>/taskbox.git
```

最后将存储库推到 Github

```bash
$ git push -u origin master
```

### 网络化

NetLify 有一个内置的持续部署服务，允许我们在不需要配置自己的 CI 的情况下部署故事书。

<div class="aside">
If you use CI at your company, add a deploy script to your config that uploads <code>storybook-static</code> to a static hosting service like S3.
</div>

[Create an account on Netlify](https://app.netlify.com/start)点击“创建站点”。

![Netlify create site](/netlify-create-site.png)

接下来单击 GitHub 按钮将 NetLify 连接到 GitHub。这允许它访问我们的远程任务箱报告。

现在从选项列表中选择任务框 github repo。

![Netlify connect to repo](/netlify-account-picker.png)

通过突出显示要在其 CI 中运行的构建命令以及输出静态站点的目录来配置 NetLify。用于分支选择`master`. 目录是`storybook-static`. 生成命令使用`yarn build-storybook`.

![Netlify settings](/netlify-settings.png)

提交表单以在`master`任务框的分支。

完成后，我们将在 NetLify 上看到一条确认消息，其中包含一个到 TaskBox 在线故事书的链接。如果你在跟踪，你部署的故事书应该是在线的。[like so](https://clever-banach-415c03.netlify.com/).

![Netlify Storybook deploy](/netlify-storybook-deploy.png)

我们完成了你故事书的持续部署！现在我们可以通过链接与队友分享我们的故事。

这有助于作为标准应用程序开发过程的一部分进行可视审查，或者只是为了展示工作。
