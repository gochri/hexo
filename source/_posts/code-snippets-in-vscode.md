---
title: 基于snippets的vscode插件使用指南
date: 2022-02-28 00:16:19
tags:
description: 来开发一个最简单的vscode-snippet插件吧
---
# 基于snippets的vscode插件使用指南

本文档是关于的 vscode 中集成 code-snippets 拓展的集成指南

## 前言

基本所有主流编辑器都支持 snippets，也就是配置代码片段来提高开发效率，VSCode 也不例外

我们平时使用的 HTML Snippets 等插件，就是 Snippets 规则的集合。

## 安装

拓展-从 visx 安装

或拷贝解压文件到 .vscode/extension 下

或从应用市场直接下载 html snippets 等插件直接使用

## 使用示例

| 触发前缀 |插入到编辑器中的内容  |
| --- | --- |
|csl  | console.log('') |

## 支持的语言

* JavaScript(.js)
* TypeScrit(.ts)
* JavaScript React(.jsx)
* etc
  
## snippets说明

### 创建snippets

在 vscode 中 Ctrl+P 打开命令行 

  
```
>Preferences: Configure User Snippets

新建XXX文件夹的代码片段文件
```
   
即可创建项目级别的代码片段

### snippets规则

规则形式如下

```json
  "Print to console": {
  	"scope": "javascript,typescript,javascriptreact",
  	"prefix": ["log","console"],
  	"body": [
  		"console.log('$1');",
  		"$2"
  	],
  	"description": "Log output to console"
  },
```

1. prefix 是触发的前缀，可以指定多个

2. body 是插入到编辑器中的内容，支持语法参考 body 规则

3. description 是描述

4. scope 是生效的语言，不指定的话就是所有语言都生效

在 jsx 文件下生效需指定为 javascriptreact

#### body 规则

1. 支持通过 $1 $2 指定光标位置，且支持多个 $1 同时编辑

```json
"$1  xxxx $1"
```

2. 可设置默认值，或提供多值选择

```json
"appId: '${1:|appIdA,appIdB|}', pageName: '${2: defaultPageName}"
```

3. 可选取变量

变量参考 [Variables in Snippets](https://code.visualstudio.com/docs/editor/userdefinedsnippets])

```json
"当前文件： $TM_FILENAME",
"当前日期： $CURRENT_YEAR/$CURRENT_MONTH/$CURRENT_DATE"
```

4. 可使用正则处理数据

```json
"${TM_FILENAME/.[a-z]+$/test/i}"
```

相当于

```js
${TM_FILENAME}.replace(/.[a-z]+$/i, 'test')
```

根据这些规则 即可定义新的 code snippets

## vscode插件构建

### 项目构建

首先安装脚手架

```log
npm i yo generator-code --save
```

生成 code-snippets 项目

```log
npx yo code
...
-> New Code Snippets
...
set info default
```

在 package.json 中增加钩子

```json
  "contributes": {
    "snippets": [
      {
        "language": "javascript",
        "path": "./snippets/snippets.code-snippets"
      },
      {
        "language": "javascriptreact",
        "path": "./snippets/snippets.code-snippets"
      },
    ]
  },
```

在 snippets\snippets.code-snippets 中集成 snippets

```json
  "intl": {
    "scope": "javascript,typescript,javascriptreact",
    "prefix": "intl",
    "body": ["intl.get('$1').d('$2')", "$3"],
    "description": "Log output to intl"
  },
```

因为脚手架已经在 .vscode\launch.json 中集成了 Extension 启动方式，在 运行和调试 中可直接运行项目

```json
"configurations": [
        {
            "name": "Extension",
            "type": "extensionHost",
            "request": "launch",
            "args": [
                "--extensionDevelopmentPath=${workspaceFolder}"
            ]
        }
    ]
```

运行通过后，项目已经构建完毕

### 项目发布

在项目中安装打包工具

```log
npm i vsce --save
```

修改 package.json 文件 增加 publisher

```json
  "publisher": "chengjy03",
```

修改项目 Readme.md 文件

最后打包项目

```log
npx vsce package
```

即可生成对应的.vsix 文件

后续可将项目发布至 vscode 插件商店

## 参考教程

[创建 VS Code 扩展：为你的团队提供常用代码片段](https://juejin.cn/post/7030250953215311908)

[创建 VS Code 扩展：插件打包](https://www.jianshu.com/p/535e736c8b97)
