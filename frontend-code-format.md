# 前端代码格式化检查及项目应用
## ESLint
### 介绍
> Find and fix problems in your JavaScript code
ESLint可以检查JS/TS文件中格式及语法错误，并修复它们。
### 安装
1. 安装
> npm install eslint --save-dev
> #or
> yarn add eslint --dev

2. 生成配置文件
> npx eslint --init
> #or
> yarn run eslint --init

3. 配置
[配置项列表](https://eslint.org/docs/rules/)

### 使用
通过[命令行](https://cn.eslint.org/docs/user-guide/command-line-interface)使用eslint。
> eslint file1.js file2.js // 检查某些文件
> #or
> eslint "src/**" // 检查某个路径下的文件

下面列出一写常用的指令，其他指令可以去上面链接里看，也可以用
> eslint -h

查看所有选项。
> --ext .js .tsx // 申明检测的文件后缀
> --init // 生成默认配置文件
> --fix // 自动修正异常（不保存）
> --fix-dry-run // 自动修正（保存文件）
> --cache // 只检查修改了的文件
> --debug // 输出debug信息


### 相关链接
[官网](https://eslint.org/)
[GitHub](https://github.com/eslint/eslint)

## Prettier
### 介绍
#### 什么是Prettier
* Opinionated的代码格式器
* 支持多种语言(JS,JSX,TS,JSON,Less, SCSS, GraphQL, markdown...)
* 可以与多种IDE集成(VScode, Sublime, Atom, Vim等)
* 有一些可配置项
> Opinionated，固执己见的。其实可以理解为强行做了一些规定，无法改变。也正是因为这样，才会说has few options，只提供一小部分可配置项。
#### 为什么要使用Prettier
统一代码格式。
### 安装
> npm install --save-dev prettier
> #or
> yarn add --dev prettier
#### 配置文件
配置文件支持JSON/JS/YAML等格式。
.prettierrc.json:
```json
{
  "trailingComma": "es5",
  "tabWidth": 4,
}
```
.prettierrc.js or prettier.config.js
```javascript
module.exports = {
  trailingComma: "es5",
  tabWidth: 4,
  semi: false,
};
```
.prettierrc.yml
```yml
trailingComma: "es5"
tabWidth: 4
```
### 使用
> prettier [options] [file/dir/glob ...]
例如，格式化当前路径下所有文件
> prettier --write .
#### 常用命令
> --check -c 检查文件格式
> --debug-check 输出检查过程中的日志，担心格式化过程中影响到代码逻辑的时候使用，来确认有没有影响到代码
> --find-config-path 输出config文件的路径
> --config 使用某个配置 prettier --config config_file --write target_file
> --ignore-path 忽略配置文件的路径
> --write 修正检测到的不正确的地方
#### 结合linter使用
这里以ESLint为例。
> npm install --save-dev eslint-config-prettier
然后再eslint的config文件里，如.eslintrc.json
```json
{
  "extends": [
    "prettier",
    "...other config",
  ]
}
```
### 相关链接
[官网](https://prettier.io/)
[eslint-config-prettier](https://github.com/prettier/eslint-config-prettier)

## Stylelint
> A mighty, modern linter that helps you avoid errors and enforce conventions in your styles.

### 安装
```shell
npm install --save-dev stylelint stylelint-config-standard
```
### 使用
> stylelint [options] [dir/file]
> --fix 自动修复
> --color log颜色
> --config 设置配置

### 相关链接
[官网](https://stylelint.io/)

## 项目使用及自用配置
在项目中综合eslint + Prettier + stylelint + husky + lint-stage实现代码格式化。这里以React+TS为例，对于Vue只需要替换掉React相关的库
安装prettier,eslint及相关plugin
```shell
npm install eslint --save-dev
npm install eslint-plugin-react --save-dev
npm install --save-dev eslint-config-prettier
npm install eslint-plugin-import --save-dev
npm install eslint-plugin-promise --save-dev
npm install eslint-plugin-react-hooks --save-dev
npm install --save-dev @typescript-eslint/eslint-plugin
npm install --save-dev @typescript-eslint/parser
npm install --save-dev eslint-plugin-prettier
npm install --save-dev --save-exact prettier
```
.eslintrc.js
```javascript
module.exports = {
  env: {
    browser: true,
    es6: true,
    jest: false,
  },
  extends: [
    'eslint:recommended', // 启用推荐规则
    'plugin:react/recommended', // 启用React的推荐规则
    'plugin:@typescript-eslint/eslint-recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:promise/recommended',
    // 关闭eslint配置中与prettier冲突的格式化规则
    'prettier',
    'prettier/react',
    'prettier/@typescript-eslint',
  ],
  globals: {
    Atomics: 'readonly',
    SharedArrayBuffer: 'readonly',
  },
  // ts的eslint
  parser: '@typescript-eslint/parser',
  // parserOptions: { // 关联tsconfig文件，但lint时间明显增加，不建议关联
  //   ecmaFeatures: {
  //     jsx: true
  //   },
  //   project: "./tsconfig.json",
  // },
  plugins: [
    'react',
    'prettier',
    'import',
    '@typescript-eslint',
    'react-hooks',
  ],
  settings: {
    // eslint-plugin-react的配置 详情https://www.npmjs.com/package/eslint-plugin-react
    'react': {
      'createClass': 'createReactClass',
      'pragma': 'React',
      'version': 'detect',
    },
  },
  rules: {
    /**
     * eslint-react规则（包含tslint规则和react相关规则）
     * 注释中以‘CUSTOM’结尾的规则为可自定义规则（具体项目可自行调整）
     * 其余注释各业务方请不要随意配置，以规范为准
     * 以@typescript-eslint开头的规则由@typescript-eslint插件提供，用于检测ts
     *
     */
    // ============命名=============
    'camelcase': 'always',
    // '@typescript-eslint/camelcase': [2, { 'properties': 'always' }], // 强制使用驼峰命名  --CUSTOM
    // '@typescript-eslint/class-name-casing': [2, { 'allowUnderscorePrefix': false }], // class和interface采用帕斯卡命名(不允许下划线)
    '@typescript-eslint/interface-name-prefix': 0, // interface命名必须以I开头

    // ============空格 && 缩进=============
    '@typescript-eslint/indent': [2, 2, { // 缩进
      'FunctionDeclaration': {
        'body': 1,
        'parameters': 2,
      },
      'SwitchCase': 1,
    }],
    'eol-last': [1, 'always'], // 要求或禁止文件末尾存在空行
    'func-call-spacing': [2, 'never'], // 要求或禁止在函数标识符和其调用之间有空格
    'template-tag-spacing': [2, 'always'], // 要求或禁止在模板标记和它们的字面量之间有空格
    'spaced-comment': [2, 'always', { // 要求或禁止在注释前有空白
      'line': {
        'markers': ['/'],
        'exceptions': ['-', '+'],
      },
      'block': {
        'markers': ['!'],
        'exceptions': ['*'],
        'balanced': true,
      },
    }],
    'space-infix-ops': 2, // 要求中缀操作符周围有空格
    'comma-spacing': [2, { 'after': true }], // 强制在逗号前后使用一致的空格
    'no-trailing-spaces': 2, // 禁用行尾空格
    'space-before-function-paren': [2, 'never'], // 要求或禁止函数圆括号之前有一个空格
    'no-multi-spaces': 2, // 禁止使用多个空格
    'object-curly-spacing': [2, 'always'], // 对象大括号旁必须有空格
    '@typescript-eslint/type-annotation-spacing': [2, { // 声明类型时必须无空格
      'overrides': {
        'colon': {
          'before': false,
          'after': false,
        },
        'arrow': {
          'before': true,
          'after': true,
        },
      },
    }],
    'no-unexpected-multiline': 2, // 禁止不期待的多行写法
    'operator-linebreak': [2, 'after', { 'overrides': { '?': 'before', ':': 'before' } }], // 过长需换行时操作符的位置  --CUSTOM

    // ============符号相关=============
    'comma-style': [2, 'last'], // 逗号规则
    'comma-dangle': [2, { // 行末尾必须有逗号
      'functions': 'always-multiline',
      'arrays': 'always-multiline',
      'objects': 'always-multiline',
      'imports': 'always-multiline',
      'exports': 'always-multiline',
    }],
    'semi-style': [2, 'last'], // 强制分号的位置
    'semi': [2, 'always'], // 语句必须分号结尾
    'jsx-quotes': [2, 'prefer-double'], // JSX元素中的字符串必须使用双引号
    'quotes': [2, 'single'], // 字符串必须使用单引号
    '@typescript-eslint/member-delimiter-style': [2, { // interface, type内的成员末尾必须使用统一符号（逗号）
      'multiline': {
        'delimiter': 'semi',
        'requireLast': true,
      },
      'singleline': {
        'delimiter': 'semi',
        'requireLast': false,
      },
    }],
    'eqeqeq': [2, 'smart'], // 强制使用三等，除了对比null/undefined  --CUSTOM
    'no-extra-parens': 0, // 禁止不必要的括号 (as any写法会被误判)

    // ============箭头函数相关=============
    'arrow-parens': 2, // 要求箭头函数的参数使用圆括号
    'no-confusing-arrow': [2, { 'allowParens': true }], // 禁止在可能与比较操作符相混淆的地方使用箭头函数
    'arrow-spacing': [2, { 'before': true, 'after': true }],
    'arrow-body-style': [2, 'as-needed'], // 要求箭头函数体使用大括号

    // ============其他=============
    'prefer-const': 2, // 要求使用 const 声明那些声明后不再被修改的变量
    'one-var-declaration-per-line': 2, // 禁止一次性定义多个变量
    'key-spacing': [2, { 'afterColon': true }], // object的key的“:”之后至少有一个空格
    'import/order': 2, // import引入按照一定顺序
    'import/no-default-export': 2, // 不允许defalut export  --CUSTOM
    'no-inner-declarations': 1, // 禁止在嵌套的块中出现变量声明或 function 声明  --CUSTOM
    'require-atomic-updates': 1, // 禁止由于 await 或 yield的使用而可能导致出现竞态条件的赋值  --CUSTOM
    'no-case-declarations': 1, // 不允许在 case 子句中使用词法声明  --CUSTOM
    'promise/no-return-wrap': 1, // 避免在不需要时将值包在Promise.resolve或Promise.reject中  --CUSTOM
    'prefer-rest-params': 0, // 要求使用剩余参数而不是 arguments  --CUSTOM
    'prefer-template': 1, // 要求使用模板字面量而非字符串连接  --CUSTOM
    '@typescript-eslint/consistent-type-assertions': 1, // 强制规范类型定义的方式  --CUSTOM
    '@typescript-eslint/no-this-alias': 1, // 禁止对this使用别名  --CUSTOM
    '@typescript-eslint/no-namespace': 1, // 禁止使用自定义TypeScript模块和名称空间  --CUSTOM
    '@typescript-eslint/explicit-module-boundary-types': 'off',
    '@typescript-eslint/no-unused-vars': ['error', { // 禁止使用未使用的变量  --CUSTOM
      'vars': 'all',
      'args': 'none',
      'ignoreRestSiblings': true,
    }],
    'no-undef': 0, // 禁用未声明的变量，除非它们在 /*global */ 注释中被提到 （原因：全局变量较常用，定义在global.d.ts中即可）
    'no-constant-condition': 0, // 禁止在条件中使用常量表达式
    'prefer-spread': 0, // 要求使用扩展运算符而非 .apply()
    'no-useless-escape': 0, // 禁用不必要的转义字符 (意义不大)
    'dot-notation': 0, // object操作要求使用点号 (意义不大)
    'no-unused-vars': 0, // 禁止出现未使用过的变量（与typescript规则重复）
    'promise/always-return': 0, // promise.then必须return
    'promise/no-callback-in-promise': 0, // promise.then中禁止使用回调函数
    '@typescript-eslint/no-var-requires': 0, // 禁止var foo = require("foo"）用import代替
    '@typescript-eslint/explicit-function-return-type': 0, // 对返回类型不明确的函数必须声明类型
    '@typescript-eslint/no-non-null-assertion': 0, // 禁止使用!的非null断言后缀运算符
    '@typescript-eslint/no-use-before-define': 0, // 在定义变量和函数之前禁止使用
    '@typescript-eslint/no-explicit-any': 0, // 禁止使用any类型
    '@typescript-eslint/no-angle-bracket-type-assertion': 0, // 禁止使用尖括号范型
    '@typescript-eslint/no-inferrable-types': 0, // 不允许对初始化为数字，字符串或布尔值的变量或参数进行显式类型声明

    // ============React规则=============
    'react/default-props-match-prop-types': 1, // 有默认值的属性必须在propTypes中指定  --CUSTOM
    'react/no-array-index-key': 1, // 不要使用数组索引作为key，尽可能使用ID
    'react/no-multi-comp': 1, // 一个文件只能存在一个组件 --CUSTOM
    'react/no-unused-prop-types': 1, // 禁止未使用的prop参数
    'react/prefer-es6-class': 1, // 强制使用es6 extend方法创建组件
    'react/require-default-props': 1, // 非require的propTypes必须制定默认值
    'react/self-closing-comp': 1, // 没有children的组件和html必须使用自闭和标签
    'react/sort-comp': 1, // 对组件的方法排序
    'react/sort-prop-types': 1, // 对prop排序
    'react/style-prop-object': 1, // 组件参数如果是style，value必须是object
    'react/jsx-boolean-value': 1, // 属性值为true的时候，省略值只写属性名
    'react/jsx-closing-tag-location': 1, // 强制开始标签闭合标签位置
    'react/jsx-equals-spacing': 1, // 属性赋值不允许有空格
    'react/jsx-first-prop-new-line': 1, // 只有一个属性情况下单行
    'react/jsx-key': 2, // 强制遍历出来的jsx加key
    'react/jsx-max-props-per-line': [1, { 'maximum': 1 }], // 每行最多几个属性
    'react/jsx-no-comment-textnodes': 1, // 检查jsx注释
    'react/jsx-pascal-case': 1, // 检查jsx标签名规范
    'react/jsx-wrap-multilines': [1,
      {
        'declaration': 'parens-new-line',
        'assignment': 'parens-new-line',
        'return': 'parens-new-line',
        'arrow': 'ignore',
        'condition': 'ignore',
        'logical': 'ignore',
        'prop': 'ignore',
      },
    ],

    'react-hooks/rules-of-hooks': 2, // 检查 Hook 的规则
    'react-hooks/exhaustive-deps': 0, // 检查 Effect 的依赖（autofix时会自动添加依赖，不安全，故关掉） --CUSTOM
  },
  overrides: [ // 为.js文件设置覆盖规则
    {
      'files': [ './**/*.js' ],
      'excludedFiles': '*.spec.js',
      'rules': {
        'no-var': 0,
      },
    },
  ],
};
```
prettier.config.js
```javascript
module.exports = {
  tabWidth: 2, // 一个tab代表几个空格数，默认为2个
  useTabs: false, // 是否使用tab进行缩进，默认为false，表示用空格进行缩减
  singleQuote: true, // 字符串是否使用单引号，默认为false，使用双引号
  semi: true, // 行位是否使用分号，默认为true
  trailingComma: 'all', // 是否使用尾逗号，有三个可选值"<none|es5|all>"
  bracketSpacing: true, // 对象大括号直接是否有空格，默认为true，效果：{ foo: bar }
  parser: 'babel', // 代码的解析引擎。
  arrowParens: 'always',  // 箭头函数参数使用圆括号 (str) => { ... }
  endOfLine: 'lf',
};
```
安装stylelint
```shell
npm install --save-dev stylelint stylelint-config-standard stylelint-config-prettier
```
.stylelintrc.json
```json
{
  "extends": [
    "stylelint-config-standard",
    "stylelint-config-prettier"
  ],
  "rules": {
    "declaration-colon-space-after": "always",
    "at-rule-empty-line-before": [ "always", {
      "except": [
        "after-same-name",
        "inside-block",
        "blockless-after-same-name-blockless",
        "first-nested"
      ],
      "ignore": ["after-comment"]
    } ],
    "at-rule-no-unknown": null,
    "max-nesting-depth": [4, {
      "ignore": [
        "blockless-at-rules",
        "pseudo-classes"
      ]
    }],
    "color-hex-case": null,
    "function-parentheses-newline-inside": "never-multi-line",
    "selector-pseudo-element-colon-notation": "double",
    "selector-type-case": "lower",
    "no-descending-specificity": null,
    "selector-pseudo-class-no-unknown": [true, {
      "ignorePseudoClasses": [ "global"]
    }]
  },
  "ignoreFiles": [
    "node_modules/**",
    "src/style/fonts/**",
    "**/*.json",
    "**/*.svg"
  ]
}
```
安装husky和lint-staged
```shell
npm i husky --save-dev
npm i lint-stage --save-dev
```
package.json
```json
{
  "husky": {
    "hooks": {
      "pre-commit": "lint-staged" // 只检测当前提交的内容
    }
  },
  "lint-staged": {
    "src/**/*.{ts,tsx,js}": "eslint --ext=jsx,ts,tsx src --fix",
    "src/**/*.(css|less)": [
      "stylelint"
    ]
  },"
}
```
vscode设置prettier,打开setting.json(可以全局也可以当前工作空间),保存自动格式化。
setting.json
```json
{
  "editor.codeActionsOnSave": {
      "source.fixAll.eslint": true,
      "source.fixAll.stylelint": true,
  },
  "eslint.format.enable": true,
  "eslint.validate": [
      "javascript",
      "javascriptreact",
      "typescript",
      "typescriptreact",
  ],
  "[javascript]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescriptreact]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "[typescript]": {
      "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
}
```
