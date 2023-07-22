---
title: 配置React开发格式化工具指南与避坑
date: 2021-04-26 20:00:00
tags: 工程化
categories: 工程化
---

# 配置React开发格式化工具指南与避坑

前言： _开发项目之前，定下良好的开发规范，能帮助我们规范流程，避免因为偷懒等因素，造成后期项目难以维护，commit log难以阅读等问题。本次借助项目机会，利用格式化工具`prettier`，对项目的格式化，`commitlint`对`git commit`进行约束。便于后期的维护，方便其他人阅读代码。中间踩很多坑，希望能给大家带来帮助_

注：本次项目基于create-react-app创建的react项目

## 配置`prettier`格式化工具

1. react不是为我们提供了`eslint`吗？

答：`eslint`只能进行语法检查，只能帮我们检查语法错误。`prettier`时代码格式化工具，能保证我们代码的格式，即使其他人阅读你的代码，也看到的时和你一样风格的代码。

2. 为什么使用?
   - 按保存键时，代码就被格式化了
   - 代码评审时无须争论代码样式
   - 节省时间和精力

**如果你按照`prettier`进行安装使用，其中有几个坑需要注意，请往下阅读。**

#### 简单配置

1. 安装

   `npm install --save-dev --save-exact prettier`

2. `echo {}> .prettierrc.json` 

   **如果你使用的是`windows`电脑并且使用`vscode`终端，按照官网在你的项目中使用这条命令创建`.prettierrc.json`文件，创建过程不会出现任何的错误，但是在你下面操作的过程中，如使用`npx prettier --write .`命令时会报错，并且你很难定位。原因是，创建的过程中，会创建出一个格式不是`utd8`格式的`json`文件，取决于你的系统和电脑。因为报错信息没有精准定位，导致很难排查。你可以手动创建该文件或者将该文件的格式修改为`utf8`**

3. 创建`.prettierignore`文件，用来忽略那些文件或者文件夹不需要进行格式化

4. 进行格式化`npx prettier --write .`

   使用上面这条命令可以将你选择的文件或者文件夹内的文件进行格式化。如下图所示，格式化之前和格式化之后的对比。

   - 格式化之前的代码片段

   ![格式化之前](/img/格式化/prettier.png)

   格式化之前代码没有换行，所以超出了vscode的显示区，或许很多同学都会配置vscode的最大显示长度。但是因为每个人配置不一样，为了能让一套代码，在所有人的编辑器里显示效果一样。所以使用prettier

   

   - 格式化之后的代码片段

     ![](/img/格式化/prettier_after.png)

5. 每次格式化时，都需要敲``npx prettier --write`这条命令，略显繁琐。我们想要更只能的方式。这里有两种方案，一种是使用`prettier`支持的编辑器插件，如`vscode`的`Prettier - Code formatter`，只需要在`vscode`中安装，然后就可以使用快捷键`Ctrl + Shift + P`来对文件进行格式化。但是我们觉得这种方式还是不够智能，所以重点介绍下面的方法，使用自动化工具帮助我们进行格式化。



#### 配置自动化工具

##### 前置知识

1. `husky` 是一个为 git 客户端增加 hook 的工具。安装后，它会自动在仓库中的 `.git/` 目录下增加相应的钩子；比如 `pre-commit` 钩子就会在你执行 `git commit` 的触发。那么我们可以在 `pre-commit` 中实现一些比如 lint 检查、单元测试、代码格式化等操作。

2. `lint-staged`，一个仅仅过滤出 Git 代码暂存区文件(被 git add 的文件)的工具；我们如果对整个项目的代码做一个检查，可能耗时很长，如果是老项目，要对之前的代码做一个代码规范检查并修改的话，可能导致项目改动很大。

   `lint-staged`，对团队项目和开源项目来说，是一个很好的工具，它是对个人要提交的代码的一个规范和约束。

##### 配置

1. `npx mrm lint-staged`

   [mrm](https://github.com/sapegin/mrm) 是一个自动化工具，它将根据 `package.json` 依赖项中的代码质量工具来安装和配置 husky 和 lint-staged，因此请确保在此之前安装并配置所有代码质量工具，如 Prettier 和 ESlint

   **注： 有可能会安装失败，因为`mrm`需要单独安装，请使用`npm install mrm -D`**

2. 配置`package.json`文件

   `create-react-app`创建的项目，默认为我们提供了`eslint`进行代码检查，为了避免和`prettier`冲突。我们用`prettier`覆盖一部分的`eslint`配置。如下

   ```json
   "eslintConfig": {
       "extends": [
         "react-app",
         "react-app/jest",
         "prettier"
       ]
     }
   ```

	在`package.json`添加下面的配置
	
	```json​	
	  "husky": {
	    "hooks": {
	      "pre-commit": "lint-staged"
	    }
	  },
	  "lint-staged": {
	    "*.{js,css,md,ts,tsx}": "prettier --write"
	  }
	```
	
	添加完毕，当我们使用`git commit`的时候，就会去调用`husky`中定义的`hooks`钩子，这个钩子就会去找`lint-staged`，`lint-staged`中使用了最开始手动格式化得到命令`prettier --write`。到这里，这一套自动化的工具，就会帮助我们自动格式化我们`commit`的代码。



## `commitlint`

_这个工具在我们在每一次commit时，检查我们的commit messages是否符合规范。它为我们提供了一套规则，我们必须按照它提供的规范来进行commit messages的填写。这样做的好处时，我们以后的commit messages非常的适合阅读。通过阅读，能够清楚的知道我们本次commit进行了什么的修改或者操作。_

### 安装配置

1. install

   `npm install -g @commitlint/cli @commitlint/config-conventional`

2. Configure

   `echo "module.exports = {extends: ['@commitlint/config-conventional']}" > commitlint.config.js`

3. 配置规则，当我们使用commit的时候，会帮我们去检验git commit 的messages时候符合规范。

   ```json
   "husky": {
       "hooks": {
         "pre-commit": "lint-staged",
         "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
       }
     }
   ```

#### git commit 格式

```Git
git commit -m <type>[optional scope]: <description>
```

##### 常用的type类别

type ：用于表明我们这次提交的改动类型

- build：主要目的是修改项目构建系统(例如 glup，webpack，rollup 的配置等)的提交

- ci：主要目的是修改项目继续集成流程(例如 Travis，Jenkins，GitLab CI，Circle等)的提交

- docs：文档更新

- feat：新增功能

- fix：bug 修复

- perf：性能优化

- refactor：重构代码(既没有新增功能，也没有修复 bug)

- style：不影响程序逻辑的代码修改(修改空白字符，补全缺失的分号等)

- test：新增测试用例或是更新现有测试

- revert：回滚某个更早之前的提交

- chore：不属于以上类型的其他类型(日常事务)

optional scope：一个可选的修改范围。用于标识此次提交主要涉及到代码中哪个模块。

description：描述此次提交的内容信息

例子

```git
git commit -m "test: test commitlint"
```

以上就是配置`prettier`进行自动化代码格式化和自动化检测，规范git commit 的commitlint的配置使用。还有更多高级功能等待发现。