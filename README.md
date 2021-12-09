# 规范git commit

## commitizen + cz-conventional-changelog

commitizen可以通过cz命令，生成一个git commit提交，配合使用cz-conventional-changelog这个适配器，可以生成符合AngularJS规范的**提交说明**

首先在项目中安装commitizen

```
yarn add -D commitizen
```

然后初始化cz-conventional-changelog：

```
npx commitizen init cz-conventional-changelog --save --save-exact
```

初始化命令主要进行了3件事

1. 在项目中安装cz-conventional-changelog依赖
2. 将该依赖保存到package.json的devDependencies字段中
3. 在package.json中新增config.commitizen字段信息，主要用于配制cz工具的适配器路径：

```js
"devDependencies": {
 "cz-conventional-changelog": "^2.1.0"
},
"config": {
  "commitizen": {
    "path": "./node_modules/cz-conventional-changelog"
  }
}
```

接下来可以在package.json中的script字段新增一个命令：

```
"script": {
    "commit": "npx cz"
}
```

接下来我们就可以通过 yarn commit命令来代替git commit进行提交了。

```
➜ yarn commit
yarn run v1.22.17
$ npx cz
cz-cli@4.2.4, @commitlint/cz-commitlint@15.0.0

? 选择一个commit提交类型: (Use arrow keys)
❯ fix:        修补某功能的bug 
  docs:       仅文档新增/改动 
  style:      样式修改 
  refactor:   重构某个功能 
  perf:       性能, 体验优化 
  test:       测试某功能、新增测试用例、更新现有测试 
  build:      修改项目构建系统(例如 webpack，cli 的配置等)的提交 
(Move up and down to reveal more choices)
```

## @commitlint/cli + @commitlint/config-conventional + husky

> @commitlint/cli 是一款校验工具，用来检测提交说明是否符合规范。

> @commitlint/config-conventional 是Angular风格的校验规则。
 
> husky 是一个git钩子，可以用于在提交时对commit的文本进行校验。
 
在项目中新增commitlint.config.js文件并设置校验规则：

```js
module.exports = {
    extends: ['@commitlint/config-conventional']
}
```

以下是对husky 7.X版本的配置

执行命令安装husky：

```
yarn add -D husky

# 初始化 husky 文件夹
npx husky install

# 在.husky文件夹中添加commit-msg钩子
yarn husky add .husky/commit-msg 'npx --no-install commitlint --edit $1'
```

运行完最后一个命令之后，husky会在.husky文件夹下新增一个commit-msg文件，文件内容如下：

```shell
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx --no-install commitlint --edit $1
# 或者 
# npx --no-install commitlint --edit $HUSKY_GIT_PARAMS
```

以上步骤完成之后，当我们提交不规范的commit message时，命令就会被终止。

下面我们来测试一下：

```
➜  learn-commitlint git:(master) ✗ git commit -m "test"
⧗   input: test
✖   subject may not be empty [subject-empty]
✖   type may not be empty [type-empty]

✖   found 2 problems, 0 warnings
ⓘ   Get help: https://github.com/conventional-changelog/commitlint/#what-is-commitlint

husky - commit-msg hook exited with code 1 (error)
➜  learn-commitlint git:(master) ✗ 

```

完美！


## cz-conventional-changelog还是@commitlint/cz-commitlint

在第一节中，我们介绍了cz-conventional-changelog这个适配器，接下来我推荐使用这个@commitlint/cz-commitlint适配器。

这个适配器是以cz-conventional-changelog为灵感，支持更多的可选配置。

安装@commitlint/cz-commitlint

```

yarn add -D @commitlint/cz-commitlint
```

安装完成之后，将package.json文件中的config.commitizen字段改为如下：

```js
// package.json

"config": {
    "commitizen": {
        "path": "@commitlint/cz-commitlint"
    }
}
```


之后我们如果需要对yarn commit中的交互进行汉化，只需要对commitlint.config.js文件进行如下的配置即可：

```js
//commitlint.config.js

module.exports = {
  // 符合angular的校验规则
  extends: ["@commitlint/config-conventional"],
  
  rules: {
    "type-enum": [
      2,
      "always",
      [
        "feature",
        "fix",
        "build",
        "refactor",
        "style",
        "docs",
        "chore",
        "ci",
        "perf",
        "test",
      ],
    ],
    "type-case": [0],
    "type-empty": [0],
    "scope-empty": [0],
    "scope-case": [0],
    "subject-full-stop": [0, "never"],
    "subject-case": [0, "never"],
    "header-max-length": [0, "always", 72],
  },
  prompt: {
    messages: {
      skip: "回车跳过",
      max: "upper %d chars",
      min: "%d chars at least",
      emptyWarning: "内容不能为空",
      upperLimitWarning: "over limit",
      lowerLimitWarning: "below limit",
    },
    questions: {
      type: {
        description: "选择一个commit提交类型",
        enum: {
          feature: {
            description: "新增了什么功能",
            title: "Features",
            emoji: "✨",
          },
          fix: {
            description: "修补某功能的bug",
            title: "Bug Fixes",
            emoji: "🐛",
          },
          docs: {
            description: "仅文档新增/改动",
            title: "Documentation",
            emoji: "📚",
          },
          style: {
            description: "样式修改",
            title: "Styles",
            emoji: "💎",
          },
          refactor: {
            description: "重构某个功能",
            title: "Code Refactoring",
            emoji: "📦",
          },
          perf: {
            description: "性能, 体验优化",
            title: "Performance Improvements",
            emoji: "🚀",
          },
          test: {
            description: "测试某功能、新增测试用例、更新现有测试",
            title: "Tests",
            emoji: "🚨",
          },
          build: {
            description: "修改项目构建系统(例如 webpack，cli 的配置等)的提交",
            title: "Builds",
            emoji: "🛠",
          },
          ci: {
            description: "主要目的是修改项目继续集成流程",
            title: "Continuous Integrations",
            emoji: "⚙️",
          },
          chore: {
            description: "构建过程或辅助工具的变动",
            title: "Chores",
            emoji: "♻️",
          },
        },
      },
      scope: {
        description: "填写改动范围（修改了哪些组件、文件名）",
      },
      subject: {
        description: "大概描述改动内容",
      },
      body: {
        description: "可提供更详细的说明",
      },
      isBreaking: {
        description: "是否为破坏性修改？如接口删除、逻辑迁移、接口参数减少等",
      },
      isIssueAffected: {
        description: "改动修复了哪个问题？",
      },
    },
  },
};

```