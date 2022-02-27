---
title: 前端规范自动化检查指南
date: 2022-02-28 00:22:35
tags:
description: 前端规范要如何在到代码提交时实现自动化检查呢
---
# 前端规范自动化检查指南

文件夹参考 项目下 ./codeReview 需先安装 husky 与 shelljs

## 使用说明

1. 文件夹 ./codeReview 集成了对项目提交内容中 packages 下的 js/jsx 文件的国际化/埋点自动检查

2. 在项目文件夹下使用 bash/cmd/PowerShell 进行 commit 提交，即可对项目进行国际化/埋点检查，并在未通过时抛出异常并生成检查报告

   如在 E:/e-front 中进行检查，未通过时会在 E:/ 抛出检查报告

3. 当前文件夹代码是通过 husky 中的 pre-commit 钩子触发，故文件夹迁移时，需在 package.json 添加以下字段

```json
  "scripts": {
    "code-review": "node ./codeReview/index"
  },
  "husky": {
    "hooks": {
      "pre-commit": "node ./codeReview/index && npm run lint-staged"
    }
  },
```

## 代码思路

### 整体思路

通过 husky 中的 pre-commit 钩子，在代码提交时触发文件夹代码

```json
  "scripts": {
    "code-review": "node ./codeReview/index"
  },
  "husky": {
    "hooks": {
      "pre-commit": "node ./codeReview/index && npm run lint-staged"
    }
  },
```

通过 shelljs 执行 git 命令，通过回调拿到缓存区所有文件名

```js
const shell = require('shelljs');

if (!shell.which('git')) {
  shell.echo('Sorry, this script requires git');
  shell.exit(1);
}

const shellargs = shell.exec('git diff --staged --diff-filter=ACMR --name-only -z');
const output = shellargs.output;

// let output = 'packages/project-one/src/pages/Management/components/EditDetail.jsx\x00packages/project-one/src/pages/Management/index.jsx\x00';
//'packages\\project-one\\src\\pages\\LessonsLearnedCheck\\index.js',

let _files = output.split('\x00');
_files.pop();
```

效验文件列表后，调用 check.js 检查埋点与国际化

```js
const { checkOneFile } = require('./check.js');

const checkResult = [];
let cnt = 0;

for (const f of _files) {
  const r = checkOneFile(f);
  if (r.length) {
    cnt++;
    checkResult.push(`File：${f}`, `${r.join('\r\n')}\r\n`);
  }
}
```

如检查不通过，保存检查日志并中止提交

```js
if (checkResult.length) {
  // isPass = false;
  console.log('>', '\x1B[31m报警文件数：\x1B[0m', `\x1B[31m${cnt}\x1B[0m\n`);
  console.log(`\x1B[31m${checkResult.join('\n')}\x1B[0m`);

  const checkResultText = `检查日期：${today}\r\n报警文件数：${cnt}\r\n\r\n${checkResult.join(
    '\r\n'
  )}`;

  const outfileName = `./../${name} ${today} 检查报告.txt`; // 保存到项目外

  fs.writeFileSync(outfileName, checkResultText);

  console.log('>', '报告已保存：', outfileName);
  console.log('>', '\x1B[31m前端规范检查不通过\x1B[0m');
  process.exit(1);
}

console.log('>', '\x1B[32m前端规范检查通过\x1B[0m');
process.exit(0);
```

### 文件效验

读取文件内容，忽略注释后，通过正则匹配对埋点和国际化进行检查

<!-- ```js
const fs = require('fs');

const checkOneFile = (filePath, outputSimple) => {
  let cont = fs.readFileSync(filePath, {});
  if (!cont) {
    return [];
  }

  cont = cont.toString();
  cont = _removeJsComments(cont);
  if (!cont) {
    return [];
  }

  const allRes = [];

  // _checkOneFileI18n(filePath, cont) || { pass: true };

  // _checkOneFileMd(filePath, cont) || { pass: true };

  return allRes;
};

exports.checkOneFile = checkOneFile;
```

国际化：检查中文字符是否做了国际化适配，没有 .d( \*\* ) 包裹的中文都认为是不通过

```js
const _checkOneFileI18n = (filePath, fileCont) => {
  if (!fileCont) {
    return { pass: true };
  }

  const lines = fileCont.split(/\n/);
  if (!lines.length) {
    return { pass: true };
  }

  // 中文检查
  const reg1 = new RegExp(/[\u4e00-\u9fa5]+[ ]*[^()<>"'`]*[ ]*[\u4e00-\u9fa5]+/g);

  // 逐行检查
  const _checkI18n = (str) => {
    const kws = str.match(reg1);
    if (!kws) {
      return [];
    }

    const arr = [];
    for (let w of kws) {
      w = w.replace(/\*/g, '\\*').replace(/\+/g, '\\+');
      let r;

      // ...注释忽略 ...console 忽略 ...埋点忽略

      // 没有 .d( ** ) 包裹的中文都认为是不通过
      r = new RegExp(`\.d\\([\\S\\r\\n ]*${w}[\\S\\r\\n ]*\\)`);
      if (!r.test(str)) {
        arr.push(w);
      }
    }
    return arr;
  };

  let res = [];

  // ...遍历每一行

  if (res.length) {
    return { pass: false, data: res };
  }
  return { pass: true };
};
```

埋点：对指定文件：tableDS.js 和 pages\/\*\/indexed.js 进行检查, 包含 TrackPage 或 TrackHook 判定通过

```js
const _checkOneFileMd = (filePath, fileCont) => {
  // 检查 tableDS.js 和 pages/*/index.js
  const reg1 = new RegExp(
    `.*(stores\\\\tableDS\.js|stores\/tableDS\.js|pages\\\\[a-zA-Z0-9_]+\\\\index\.js|pages\/[a-zA-Z0-9_]+\/index\.js)`
  );

  const reg2 = new RegExp(`(TrackPage\\(.*\\)|TrackHook\\(.*\\))`);

  if (reg1.test(filePath)) {
    const chkCont = reg2.test(fileCont);
    if (!chkCont) {
      return { pass: false, data: filePath };
    }
  }

  return { pass: true };
};
``` -->

## 实现效果

```log
Chengjy03@VDI-B-00054 MINGW64 /d/Project/one (feature-codeReview)
$ git commit -m "updte"
husky > pre-commit (node v14.15.0)

> 国际化规范指引：
> 埋点规范指引  ：

D:\Project\One


> 提交文件数： 0
> 检查文件数： 0
> 前端规范检查通过
```

或者

```log
D:\Project\One
packages/project-one/src/pages/Management/index.jsx

> 提交文件数： 1
> 检查文件数： 1
_checkOneFileMd packages/project-one/src/pages/Management/index.jsx
> 报警文件数： 1

File：packages/project-one/src/pages/Management/index.jsx
Error：未做国际化：测试
Error：未做埋点跟踪

> 报告已保存： ./../One 2022-01-27 检查报告.txt
> 前端规范检查不通过
husky > pre-commit hook failed (add --no-verify to bypass)
```
