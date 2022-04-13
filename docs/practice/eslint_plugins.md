# 🔌 ESLint x Plugins

上一篇在配置 ESLint x TypeScript 时，我们发现了 Parser 和 Plugin 的规律，下面来聊聊一些常见的 Parser 和 Plugin。

## ESLint x Babel

在第一篇文章里就说了，可以通过 `env` 来设定 ESLint 的默认 ECMAScript parser 的版本，所以 ESLint 其实是自带有 parser 的，但是它只支持 [最新版的 ECMAScript 标准](https://github.com/eslint/eslint/blob/a675c89573836adaf108a932696b061946abf1e6/README.md#what-about-experimental-features "ESLint parser 支持最新的 ECMAScript 标准")。

而 JavaScript 依然在不断发展，时不时又出一些新的 API 或者提案，对于要尝鲜的开发者，ESLint 的 Parser 就解析不动 `.js` 了，因此，我们需要 [@babel/eslint-parser](https://www.npmjs.com/package/@babel/eslint-parser)。

```sh
npm i -D @babel/core @babel/eslint-parser
```

然后添加 `.babelrc.js`，在里面写项目的 babel 规则，这里按你自己需求来就好了，我就不写啦。

然后在 `overrides` 里新增对 `.js` 的处理：

```js
module.exports = {
  env: {
    // 支持浏览器环境
    browser: true,
    // 识别 CommonJS
    node: true,
    // 识别 ES 的代码，使用 ECMAScript 2021 自动设置 ecmaVersion parser 为 12，
    es2021: true,
  },
  extends: [
    "eslint:recommended", // eslint 自己的推荐规则，最佳实践最小集
    "plugin:prettier/recommended", // 禁用 eslint 关于代码的风格的规则，使用 prettier 的风格
  ],
  overrides: [
    // 处理 JS 文件
    {
      files: ["**/*.{js,jsx}"], // 只处理 js 和 jsx 文件
      parser: "@babel/eslint-parser", // 使用 babel 来解析 js 文件
      parserOptions: {
        sourceType: "module", // 支持 import/export
        allowImportExportEverywhere: false,
        ecmaFeatures: {
          globalReturn: false,
        },
      },
    },
    // 处理 TS 文件
    {
      files: ["**/*.{ts,tsx}"], // 只处理 ts 和 js 文件
      parser: "@typescript-eslint/parser", // 能看懂 TypeScript
      parserOptions: {
        project: ["./tsconfig.json"], // 告诉 eslint：tsconfig 在哪
      },
      extends: [
        // typescript-eslint 的推荐规则，只是这些最佳规则都是针对 TS 的
        "plugin:@typescript-eslint/recommended",
        // tsconfig.json 里 Type Checking 的推荐规则
        "plugin:@typescript-eslint/recommended-requiring-type-checking",
      ],
      plugins: [
        // 使用 typescript x eslint 的插件
        "@typescript-eslint",
      ],
    },
  ],
};

```

## ESLint x React

解析 React 文件主要有两大难题：
* 不识别 `import React from 'react'`，会报 `React is not used`
* 不识别 `.jsx`, `.tsx` 文件

如果你的项目是 React + TypeScript，那么要在 `tsconfig.json` 里添加对 JSX 的支持：

```json
{
  ...
  "jsx": "react"
}
```

然后在 `overrides` 里也添加对 `.js` 和 `.jsx` 的解析：

```js
module.exports = {
  overrides: [
    {
      files: ["**/*.{jsx,js,ts,tsx}"],
      ...ESLint x TypeScript 配置
    }
  ]
}
```

因为 ESLint 这里会 TypeScript 的 parser 结合 `tsconfig.json` 来解析 `.js` 和 `.jsx`，所以 ESLint 是能看懂 `.jsx` 的内容的。

那如果项目是 React + JavaScript 呢？或者我就要分开处理 TypeScript 和 JavaScript 呢？我们难道要用 React Parser 么？**No！我们需要的是 ESLint x React 的插件！个人理解 JSX 更属于 JavaScript 的一种特性，而不是语法类型，所以要用 Plugin。**

遵循刚刚说的规律，我们安装 [eslint-plugin-react](https://www.npmjs.com/package/eslint-plugin-react)：

```sh
npm i D eslint-plugin-react
```

最后只需继承它即可，注意这里的 `extends` 大部分时候都可以是 `plugin` 的缩写版本：

```js
"extends": [
  "eslint:recommended",
  "plugin:react/recommended"
]
```

## ESLint x Vue

Vue 和 React 同理，它只需要一个 [eslint-plugin-vue](https://eslint.vuejs.org/)：

```sh
npm i -D eslint-plugin-vue
```

我们依然可以在 `overrides` 中新增一条只针对 `.vue` 文件的配置：

```json
overrides: [
  // 处理 vue 文件
  {
    files: ["**/*.vue"], // 只处理 vue 文件
    extends: ["plugin:vue/vue3-recommended"], // 使用 vue3 的推荐规则
  }
]
```

这样就可以对所有 `.vue` 文件执行 `eslint '**/*.vue' --fix` 了。

## `extends` 和 `plugins` 的区别

不知道你有没有在刚刚配置 eslint 的时候，有两个配置项一直很让人迷惑：`extends` 以及 `plugins`。

举个例子，我们要配置 eslint x typescript，可以看到官网有这样的配置：

```js
module.exports = {
  root: true,
  parser: '@typescript-eslint/parser',
  plugins: [
    '@typescript-eslint',
  ],
  extends: [
    'eslint:recommended',
    'plugin:@typescript-eslint/recommended',
  ],
};
```

神奇的是，当你去掉 `plugins` 之后发现 `eslint` 依然可以正常工作。更神奇的是，只要你写了 `extends`，那么连 `parser` 也可以不用加，要知道没有指定 `parser` 选项，eslint 可看不懂你的 TypeScript 文件。

所以说，到底是 `plugins` 加上了 TypeScript 的能力还是 `extends` 加上了 TypeScript 的规则呢？真让人头大，直到终于有一天受不了了，翻找了一下网上的资料发现了[这个帖子](https://stackoverflow.com/questions/61528185/eslint-extends-vs-plugins-v2020)。

先来说结论吧：**每个 `plugins` 只是开启了这个插件，而 `extends` 则会继承别人写好的一份 `.eslintrc` 的配置，这份配置不仅仅包括了 `rules` 还有 `parser`，`plugins` 之类的东西。**

所以回到问题，为什么在继承了 `plugin:@typescript-eslint/recommended` 之后就可以不写 `plugins` 和 `parser` 呢？因为别人已经把配置都放在 `recommended` 这份配置表里了，这样对使用的人来说，就可以少写很多配置项了。

也就是说，下面两份配置是等价的：

```js
module.exports = {
  parser: "@typescript-eslint/parser",
  parserOptions: { sourceType: "module" },
  plugins: ["@typescript-eslint"],
  extends: [],
  rules: {
    "@typescript-eslint/explicit-function-return-type": [
      "error",
      {
        allowExpressions: true
      }
    ]
  }
}
```

以及

```js
module.exports = {
  plugins: [],
  extends: ["plugin:@typescript-eslint/recommended"],
  rules: {
    "@typescript-eslint/explicit-function-return-type": [
      "error",
      {
        allowExpressions: true
      }
    ]
  }
}
```

对于第一份配置：
* 需要手动添加 `parser`, `parserOptions`, `plugins`
* 只开启了 `@typescript-eslint/explicit-function-return-type` 一个规则

对于第二份配置：
* `plugin:@typescript-eslint/recommended` 自动添加了 `parser`, `parserOptions`, `plugins`
* 一些推荐的 TypeScript ESLint 规则也自动加上了
* 只对 `@typescript-eslint/explicit-function-return-type` 这个规则进行自定义配置

## 考考你

看到这，你的 `.eslintrc.js` 应该已经写了不少代码了，当然相信你也能慢慢找到 `eslint`, `plugin`, `config`, `prettier`, `parser` 这些关键词之间排列组合的一些规律了。

不妨来考考你：
* 我要用 XXX 公司的 ESLint 规则集，应该找哪个名字的 NPM 包？
* 我要解析 YYY 语法，应该搜哪个 NPM 包呢？
* 要屏蔽 XXLint 工具和 Prettier 冲突的规则，应该用哪个 NPM 包？
* 要让处理 `.xxx` 后缀的文件，应该用到哪些 NPM 包呢？

如果你对上面的问题都能做到心中有数，那配置 [StyleLint](https://stylelint.io/user-guide/configure/) 就再简单不过了。实际上无论是哪个 xxlint 他们的 NPM 命名规律都是非常类似的。话不多出，马上开始学习下一章吧~
