---
title: 一年后再拿起VScode(1)
toc: true
top: true
categories:
  - 前端
tags:
  - vscode
description: 编辑器vscode的使用，eslint+vetur+prettier美化vue项目
abbrlink: '312e9705'
date: 2020-05-08 03:29:09
---
# VScode的代码美化--vue项目
插件：eslint+vetur+prettier

<!--more-->
## 序言
<font face="楷体" color="">一年前跟学长尝试了一个vue项目，第一次接触编辑器，以前学的C语言和Java原来编译他们那个叫IDE，是集成了编辑器和编译器的，用过ＶS studio，所以第一次拿起vscode就感觉设计哲学很符合自己的心意，但是接下来就让自己脑袋疼起来了，问题就是代码格式化。</font>

## 全局setting.json
```json
{
   //字符大小，可自己调整
  "editor.fontSize": 14,
  //设置tab的站位
  "editor.tabSize": 2,
  //在保存时自动格式化代码
  "editor.codeActionsOnSave": {
    "source.fixAll": true
  },
  // 全局使用保存自动格式化功能，vue项目关闭此项!!!!!!!
  "editor.formatOnSave": true,
  // 使用单引号包含字符串
   "prettier.singleQuote": true,
  // 不添加行尾分号
  "prettier.semi": false,
  //配置终端为cmd便于使用其他命令，如hexo
  "terminal.integrated.shell.windows": "C:\\WINDOWS\\System32\\cmd.exe",
  "git.enableSmartCommit": true,
}
```
值得注意的是，网上搜了很多教程，但大多数都存在的问题就是，他们的设置不是针对vue项目，而eslint、vetur、prettier这些组件的美化格式是冲突的，所以针对不同的项目要在工作区的setting.json进行不同的配置，来处理他们的冲突。
## 工作区的setting.json
```json
  /* 关闭编辑器自带保存格式化功能，此功能会用Vetur进行格式化。*/
  "editor.formatOnSave": false
```
这一点很重要。vetur可以用来识别.vue文件但是他的格式和eslint的格式不一样，默认vscode使用vetur的格式，而如果项目中引入了eslint依赖，并且setting.json中设置了eslint对html、css、js、vue的格式化，这样针对js文件就有了两种格式化，保存时总是报错（令我一度放弃过eslint）。

* **以上的设置是最基本的一些，也是最必要的。当然还有很对配置选项，都可以上官网或者一些（带注释的）教程上找**

## .eslintrc文件
eslint插件用于根据工程目录的.eslintrc.js配置文件在编辑器中显示一些错误提示，后面的自动格式化根据这里的错误提示进行格式化操作。前提是项目安装了eslint依赖。
```js
module.exports = {
  root: true,
  env: {
    node: true
  },
  //这里对eslint和prettier的矛盾做了调整，可查看prettier官网
  extends: ["plugin:vue/essential", "eslint:recommended", "@vue/prettier"],
  parserOptions: {
    parser: "babel-eslint"
  },
  rules: {
    "no-console": process.env.NODE_ENV === "production" ? "warn" : "off",
    "no-debugger": process.env.NODE_ENV === "production" ? "warn" : "off"
  }
};

```
在.eslintrc.js可以看到总体的eslint规则合并了vue、eslint和prettier的一些插件库进行语法分析(eslint针对js和vue，prettier针对js、html和css,这里eslint和prettier的一些冲突已经处理)
eslint很强大，关于其他配置官方给出了文档，vue官方也给出了[标配](https://github.com/vuejs/eslint-config-vue)。
下面我列出某个团队自己的规范,相信能看懂，哈哈：
```js
module.exports = {
  root: true,
  parserOptions: {
    parser: 'babel-eslint',
    sourceType: 'module'
  },
  env: {
    browser: true,
    node: true,
    es6: true,
  },
  extends: ['plugin:vue/recommended', 'eslint:recommended'],

  // add your custom rules here
  //it is base on https://github.com/vuejs/eslint-config-vue
  rules: {
    "vue/max-attributes-per-line": [2, {
      "singleline": 10,
      "multiline": {
        "max": 1,
        "allowFirstLine": false
      }
    }],
    "vue/singleline-html-element-content-newline": "off",
    "vue/multiline-html-element-content-newline":"off",
    "vue/name-property-casing": ["error", "PascalCase"],
    "vue/no-v-html": "off",
    'accessor-pairs': 2,
    'arrow-spacing': [2, {
      'before': true,
      'after': true
    }],
    'block-spacing': [2, 'always'],
    'brace-style': [2, '1tbs', {
      'allowSingleLine': true
    }],
    'camelcase': [0, {
      'properties': 'always'
    }],
    'comma-dangle': [2, 'never'],
    'comma-spacing': [2, {
      'before': false,
      'after': true
    }],
    'comma-style': [2, 'last'],
    'constructor-super': 2,
    'curly': [2, 'multi-line'],
    'dot-location': [2, 'property'],
    'eol-last': 2,
    'eqeqeq': ["error", "always", {"null": "ignore"}],
    'generator-star-spacing': [2, {
      'before': true,
      'after': true
    }],
    'handle-callback-err': [2, '^(err|error)$'],
    'indent': [2, 2, {
      'SwitchCase': 1
    }],
    'jsx-quotes': [2, 'prefer-single'],
    'key-spacing': [2, {
      'beforeColon': false,
      'afterColon': true
    }],
    'keyword-spacing': [2, {
      'before': true,
      'after': true
    }],
    'new-cap': [2, {
      'newIsCap': true,
      'capIsNew': false
    }],
    'new-parens': 2,
    'no-array-constructor': 2,
    'no-caller': 2,
    'no-console': 'off',
    'no-class-assign': 2,
    'no-cond-assign': 2,
    'no-const-assign': 2,
    'no-control-regex': 0,
    'no-delete-var': 2,
    'no-dupe-args': 2,
    'no-dupe-class-members': 2,
    'no-dupe-keys': 2,
    'no-duplicate-case': 2,
    'no-empty-character-class': 2,
    'no-empty-pattern': 2,
    'no-eval': 2,
    'no-ex-assign': 2,
    'no-extend-native': 2,
    'no-extra-bind': 2,
    'no-extra-boolean-cast': 2,
    'no-extra-parens': [2, 'functions'],
    'no-fallthrough': 2,
    'no-floating-decimal': 2,
    'no-func-assign': 2,
    'no-implied-eval': 2,
    'no-inner-declarations': [2, 'functions'],
    'no-invalid-regexp': 2,
    'no-irregular-whitespace': 2,
    'no-iterator': 2,
    'no-label-var': 2,
    'no-labels': [2, {
      'allowLoop': false,
      'allowSwitch': false
    }],
    'no-lone-blocks': 2,
    'no-mixed-spaces-and-tabs': 2,
    'no-multi-spaces': 2,
    'no-multi-str': 2,
    'no-multiple-empty-lines': [2, {
      'max': 1
    }],
    'no-native-reassign': 2,
    'no-negated-in-lhs': 2,
    'no-new-object': 2,
    'no-new-require': 2,
    'no-new-symbol': 2,
    'no-new-wrappers': 2,
    'no-obj-calls': 2,
    'no-octal': 2,
    'no-octal-escape': 2,
    'no-path-concat': 2,
    'no-proto': 2,
    'no-redeclare': 2,
    'no-regex-spaces': 2,
    'no-return-assign': [2, 'except-parens'],
    'no-self-assign': 2,
    'no-self-compare': 2,
    'no-sequences': 2,
    'no-shadow-restricted-names': 2,
    'no-spaced-func': 2,
    'no-sparse-arrays': 2,
    'no-this-before-super': 2,
    'no-throw-literal': 2,
    'no-trailing-spaces': 2,
    'no-undef': 2,
    'no-undef-init': 2,
    'no-unexpected-multiline': 2,
    'no-unmodified-loop-condition': 2,
    'no-unneeded-ternary': [2, {
      'defaultAssignment': false
    }],
    'no-unreachable': 2,
    'no-unsafe-finally': 2,
    'no-unused-vars': [2, {
      'vars': 'all',
      'args': 'none'
    }],
    'no-useless-call': 2,
    'no-useless-computed-key': 2,
    'no-useless-constructor': 2,
    'no-useless-escape': 0,
    'no-whitespace-before-property': 2,
    'no-with': 2,
    'one-var': [2, {
      'initialized': 'never'
    }],
    'operator-linebreak': [2, 'after', {
      'overrides': {
        '?': 'before',
        ':': 'before'
      }
    }],
    'padded-blocks': [2, 'never'],
    'quotes': [2, 'single', {
      'avoidEscape': true,
      'allowTemplateLiterals': true
    }],
    'semi': [2, 'never'],
    'semi-spacing': [2, {
      'before': false,
      'after': true
    }],
    'space-before-blocks': [2, 'always'],
    'space-before-function-paren': [2, 'never'],
    'space-in-parens': [2, 'never'],
    'space-infix-ops': 2,
    'space-unary-ops': [2, {
      'words': true,
      'nonwords': false
    }],
    'spaced-comment': [2, 'always', {
      'markers': ['global', 'globals', 'eslint', 'eslint-disable', '*package', '!', ',']
    }],
    'template-curly-spacing': [2, 'never'],
    'use-isnan': 2,
    'valid-typeof': 2,
    'wrap-iife': [2, 'any'],
    'yield-star-spacing': [2, 'both'],
    'yoda': [2, 'never'],
    'prefer-const': 2,
    'no-debugger': process.env.NODE_ENV === 'production' ? 2 : 0,
    'object-curly-spacing': [2, 'always', {
      objectsInObjects: false
    }],
    'array-bracket-spacing': [2, 'never']
  }
}

```
## .prettierrc文件
在文件根目录下创建.prettierrc对prettier格式化进行自定义规则设置：
```json
{
  /* 使用单引号包含字符串 */
  "singleQuote": true,
  /* 不添加行尾分号 */
  "semi": false,
  /* 在对象属性添加空格 */
  "bracketSpacing": true,
  /* 优化html闭合标签不换行的问题 */
  "htmlWhitespaceSensitivity": "ignore"
}
```
以上就是我的最基本的配置，如有更多关于vscode美化代码的建议，欢迎评论。
