中文 / [English](README.en.md)
## 关于
neatlogic-web是neatlogic平台前端代码，所有子模块的前端代码都在neatlogic-web中。

neatlogic-web基于vue2.x，UI框架在iView4.x源码基础上经过调整。

## 构建依赖
vue/cli 5.x,webpack 5.x
先全局安装vue/cli最新版
```
npm install -g @vue/cli
```

## 初始化依赖
```
npm install
```

## 本地开发环境运行项目
```
npm run serve
```

## 打包项目代码
```
npm run build
```

## 更新某个依赖
```
npm update 依赖包名字

eg:npm update neatlogic-ui
```
## 自动纠正文件里不符合eslint的语法
```
npm run lint
```

## 设置自动eslintrc
https://www.cnblogs.com/weilai-info/p/10988829.html


## 设置vscode的个人配置
为了统一代码风格，多人开发时不会出现格式错乱，进行二开时请使用相同的代码风格配置。
#### 安装插件
EsLint、vetur、Prettier - Code formatter、i18nhelper(自动进行i18n键值替换和多语言翻译)

#### 设置>命令面板>输入“setting”>选择“首选项：打开设置”>打开 settings.json
```
{
   "gitlens.views.repositories.files.layout": "list",
    "git.confirmSync": false,
    "gitlens.advanced.similarityThreshold": null,
    "editor.tabSize": 2,
    "vetur.validation.script": true,
    "vetur.validation.style": true,
    "vetur.format.defaultFormatter.html": "js-beautify-html",
    "editor.formatOnSave": false,
    "vetur.format.defaultFormatterOptions": {
      "js-beautify-html": {
        "wrap_attributes": "auto",
        "indent_size": 2,
        "indent_char": " ",
        "indent_with_tabs": false,
        "eol": "\n",
        "end_with_newline": true,
        "preserve_newlines": true,
        "max-preserve-newlines": 3,
        "indent-inner-html": true,
        "wrap_line_length": 120,
        "editorconfig": true
      }
    },
    "html.format.maxPreserveNewLines": 3,
    "html.format.enable": false,
    "[vue]": {
      "editor.defaultFormatter": "dbaeumer.vscode-eslint"
    },
    "[less]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "gitlens.advanced.blame.customArguments": [],
    "[javascript]": {
      "editor.defaultFormatter": "vscode.typescript-language-features"
    },
    "[css]": {
      "editor.defaultFormatter": "HookyQR.beautify"
    },
    "[html]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "javascript.updateImportsOnFileMove.enabled": "always",
  
      "[json]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
    },
    "[git-commit]": {
      "editor.rulers": [
        72
      ],
      "workbench.editor.restoreViewState": false
    },
    "gitlens.advanced.messages": {},  
    "gitlens.gitCommands.closeOnFocusOut": true,
    "workbench.startupEditor": "newUntitledFile",
    "npm.enableRunFromFolder": true,
    "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true
    },
    "editor.autoClosingBrackets":"always",
    "security.workspace.trust.untrustedFiles": "newWindow",
    "workbench.editorAssociations": {
      "*.html": "default"
    },
    "editor.unicodeHighlight.ambiguousCharacters": false,
    "diffEditor.ignoreTrimWhitespace": false,
    "eslint.alwaysShowStatus": true,
    "eslint.format.enable": true,
    "extensions.ignoreRecommendations": true,
    "npm.keybindingsChangedWarningShown": true,
    "settingsSync.ignoredSettings": [],
    "window.zoomLevel": 1,
    "typescript.disableAutomaticTypeAcquisition": true,
    "editor.suggest.snippetsPreventQuickSuggestions": false,
    "eslint.codeActionsOnSave.rules": null
}

```