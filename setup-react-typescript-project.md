# React + Typescrpit + Webpack搭建项目
相关技术栈:
> React + ReactRouter + ReduxToolKit + ReduxSaga  // react全家桶
> Typescript  // 类型校验
> Webpack // 编译打包
> Less  // css库
> ESlint + StyleLint + Prettier // 格式、语法规范 这里就不讲了

**所有相关依赖都是直接安装的最新的（2021.2.20），不同的版本会有一些不同的用法。最后我会贴出package.json**
## 项目结构
```
.
├── README.md
├── config  // 配置文件
│   ├── default.config.json
│   ├── development.config.json
│   └── production.config.json
├── global.d.ts // ts全局声明
├── mock  // mock
│   └── index.js
├── package-lock.json
├── package.json
├── prettier.config.js  // prettier配置
├── src
│   ├── common  // 公共内容
│   │   ├── assets  // 图片、字体等静态资源
│   │   ├── styles.css  // 公共样式
│   │   └── type.ts
│   ├── components  // 公共组件
│   ├── fetch // 接口相关
│   ├── index.ejs // 入口html模板
│   ├── index.tsx // 入口文件
│   ├── lib // 外部依赖
│   ├── pages // 页面
│   ├── redux // redux
│   ├── root.less // 根节点样式
│   ├── root.tsx  // 根节点
│   ├── routes  // 路由
│   └── utils // 工具库
├── tsconfig.json // ts配置
├── webpack  // webpack配置
│   ├── webpack.common.js
│   ├── webpack.dev.js
│   └── webpack.prod.js
├── .eslintrc.js  // eslint配置
├── .stylelintrc.json  // stylelint配置
├── .gitignore
└── webpack.config.js // webpack配置入口
```
## 依赖安装
### React、Typescript、Webpack安装
```shell
# React相关
npm i react react-dom react-router-dom react-redux @reduxjs/toolkit redux-saga
# ts
npm i typescript --save-dev
# webpack
npm i webpack webpack-cli --save-dev
```
### Webpack插件安装
```shell
npm i progress-bar-webpack-plugin friendly-errors-webpack-plugin clean-webpack-plugin html-webpack-plugin tsconfig-paths-webpack-plugin webpack-merge --save-dev
npm i compression-webpack-plugin --save-dev
npm i webpack-bundle-analyzer --save-dev
npm i webpack-dev-server --save-dev
```
### 安装types
```shell
npm i @types/react @types/react-dom @types/react-redux @types/react-router-dom @typescript-eslint/eslint-plugin @typescript-eslint/parser --save-dev
```
### 安装babel
```shell
# babel核心、babel-react、babel-typescript
npm i @babel/core @babel/preset-env @babel/preset-react @babel/preset-typescript --save-dev
npm i babel-plugin-module-resolver babel-plugin-import --save-dev
# 会用到的一些babel plugin
npm i @babel/plugin-proposal-class-properties --save-dev
npm i @babel/plugin-proposal-decorators --save-dev
npm i @babel/plugin-proposal-nullish-coalescing-operator --save-dev
npm i @babel/plugin-proposal-optional-chaining --save-dev
npm i @babel/plugin-syntax-dynamic-import --save-dev
npm i @babel/plugin-transform-runtime --save-dev
```
### 安装loader
```shell
npm i babel-loader css-loader file-loader style-loader url-loader --save-dev
```
### 安装less
```shell
npm i less less-loader --save-dev
```
### 安装postcss
```shell
npm i postcss-loader postcss-import-sync2 postcss-less postcss-preset-env postcss-pxtorem --save-dev
npm i autoprefixer --save-dev
```
### lint相关
```shell
# eslint
npm i eslint eslint-plugin-import eslint-plugin-prettier eslint-plugin-promise eslint-plugin-react eslint-plugin-react-hooks eslint-config-prettier @typescript-eslint/eslint-plugin @typescript-eslint/parser --save-dev
# stylelint
npm i stylelint stylelint-config-prettier stylelint-config-standard --save-dev
# prettier
npm i prettier --save-dev --save-exact
```
### 其它
```shell
# 设置node变量，可以用来区分执行环境
npm i cross-env --save-dev
# 根据需要自行调整 这里列一我比较常用的
npm i axios classnames dayjs
```
## Webpack配置
webpack.config.js
```javascript
const devConfig = require('./webpack/webpack.dev');
const prodConfig = require('./webpack/webpack.prod');
// 根据环境加载不同配置
module.exports = process.env.NODE_ENV === 'production' ? prodConfig : devConfig;
```
webpack.common.js
```javascript
const path = require('path');
const ProgressBarPlugin = require('progress-bar-webpack-plugin');
const FriendlyErrorsWebpackPlugin = require('friendly-errors-webpack-plugin');
const { CleanWebpackPlugin } = require('clean-webpack-plugin');
const TsconfigPathsPlugin = require('tsconfig-paths-webpack-plugin');
const webpack = require('webpack');
const autoprefixer = require('autoprefixer');
const publicPath = process.env.NODE_ENV === 'production' ? '' : '/';
const SCOPE_NAME = '[path][name]__[local]';
module.exports = {
  entry: path.resolve(__dirname, '../src/index.tsx'),
  target: 'web',
  resolve: {
    modules: [path.resolve(__dirname, '../'), 'node_modules'],
    alias: {},
    extensions: ['.tsx', '.ts', '.js', '.less', '.css'],
    symlinks: false,
    cacheWithContext: false,
    plugins: [
      new TsconfigPathsPlugin({
        configFile: path.resolve(__dirname, '../tsconfig.json'),
      }),
    ],
    fallback: {
      util: false,
    },
  },
  module: {
    rules: [
      {
        test: /\.(ts|tsx|js|jsx)?$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'babel-loader',
            options: {
              sourceType: 'unambiguous',
              presets: [
                '@babel/preset-env',
                '@babel/preset-react',
                '@babel/preset-typescript',
              ],
              plugins: [
                [
                  'module-resolver',
                  {
                    extensions: ['.js', '.jsx', '.ts', '.tsx', '.less', '.css'],
                    alias: {},
                  },
                ],
                ['@babel/plugin-transform-runtime'],
                ['@babel/plugin-proposal-class-properties', { loose: true }],
                [
                  '@babel/plugin-proposal-decorators', // 支持装饰器
                  {
                    legacy: true,
                  },
                ],
                '@babel/plugin-syntax-dynamic-import', // 动态导入
                '@babel/plugin-proposal-optional-chaining', // 这两个是用来处理 a && a.b => a.?b的 避免多层对象写的过于复杂
                '@babel/plugin-proposal-nullish-coalescing-operator',
              ],
            },
          },
        ],
      },
      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader',
          'postcss-loader',
          // 最新的（5.0.0）postcss需要单独写config文件，下面这种写法会报错，可以使用(v3.0.0)
          // {
          //   loader: 'postcss-loader',
          //   options: {
          //     plugins: () => [autoprefixer()],  // 浏览器兼容性前缀补全
          //   },
          // },
        ],
      },
      {
        test: /\.less$/,
        exclude: /node_modules/,
        use: [
          'style-loader',
          {
            loader: 'css-loader',
            options: {
              modules: {
                localIdentName: SCOPE_NAME,
              },
              importLoaders: 3,
              sourceMap: false,
            },
          },
          'postcss-loader',
          // 最新的（5.0.0）postcss需要单独写config文件，下面这种写法会报错，可以使用(v3.0.0)
          // {
          //   loader: 'postcss-loader',
          //   options: {
          //     plugins: [autoprefixer()],
          //   },
          // },
          {
            loader: 'less-loader',
          },
        ],
      },
      {
        test: /\.(jpe?g|png|gif|svg)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'url-loader',
            options: {
              esModule: false, // 不然src会变成[object%20Module]
              emitFile: true,
              limit: 3 * 1024,
              name: 'images/[name]__[hash:5].[ext]',
              publicPath: publicPath,
            },
          },
        ],
      },
      {
        test: /\.(woff|woff2|eot|ttf|mp3|mp4)$/,
        exclude: /node_modules/,
        use: [
          {
            loader: 'file-loader',
            options: {
              name: 'assets/[name]__[hash:5].[ext]',
              publicPath: publicPath,
            },
          },
        ],
      },
    ],
  },
  plugins: [
    new ProgressBarPlugin(), // 进度条
    new FriendlyErrorsWebpackPlugin(),  // 报错信息
    new CleanWebpackPlugin({  // 清除上次编译的内容
      verbose: true, // Write logs to console.
      dry: false,
    }),
  ],
};
```
webpack.dev.js
```javascript
const path = require('path');
const { merge } = require('webpack-merge');
const HTMLWebpackPlugin = require('html-webpack-plugin');
const common = require('./webpack.common');
// 根据环境做一些不同的配置
// const env = process.env.NODE_ENV;
// const config = require(`../config/${env}.config.json`);
module.exports = merge(common, {
  mode: 'development',
  output: {
    publicPath: '/',
    path: path.resolve(__dirname, '../dist'),
  },
  devServer: {
    contentBase: path.join(__dirname, '../dist'),
    compress: true,
    host: '0.0.0.0',
    port: 3000,
    hot: true,
    proxy: {
      // 代理
      // '/': {
      //   target: config.backend,
      // },
    },
  },
  devtool: 'eval-cheap-module-source-map',
  plugins: [
    new HTMLWebpackPlugin({
      cache: false,
      filename: 'index.html',
      template: path.resolve(__dirname, '../src/index.ejs'),
      favicon: path.resolve(__dirname, '../src/common/assets/favicon.ico'),
    }),
  ],
});

```
webpack.prod.js
```javascript
const path = require('path');
const { merge } = require('webpack-merge');
const HTMLWebpackPlugin = require('html-webpack-plugin');
const TerserPlugin = require('terser-webpack-plugin');
const { BundleAnalyzerPlugin } = require('webpack-bundle-analyzer');
const CompressionPlugin = require('compression-webpack-plugin');
const common = require('./webpack.common');

const vendors = [
  'react',
  'react-dom',
  'react-router-dom',
  '@reduxjs/toolkit',
  'react-redux',
  'axios',
  'redux-saga',
];
module.exports = merge(common, {
  entry: {
    vendor: vendors,
    index: {
      import: path.resolve(__dirname, '../src/index.tsx'),
      dependOn: 'vendor',
    },
  },
  mode: 'production',
  output: {
    filename: 'js/[contenthash].js',
    chunkFilename: 'chunk/[chunkhash].js',
    path: path.resolve(__dirname, '../dist'),
  },
  plugins: [
    // 针对html文件压缩 聊胜于无
    new HTMLWebpackPlugin({
      cache: false,
      filename: 'index.html',
      template: path.resolve(__dirname, '../src/index.ejs'),
      minify: {
        collapseWhitespace: true, // 折叠空白区域
        preserveLineBreaks: false,
        minifyCSS: true, // 压缩文内css
        minifyJS: true, // 压缩文内js
        removeComments: true, // 移除注释
      },
    }),
    // 启动gzip
    new CompressionPlugin({
      test: /.js$/, // 还可以扩展其他文件类型
    }),
    // 打包分析 如果需要的时候移除注释就好了
    // new BundleAnalyzerPlugin(),
  ],
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
    minimize: true,
    minimizer: [
      new TerserPlugin(),
    ],
  },
});

```
postcss.config.js
```javascript
// 如果使用的是最新的postcss
// 在webpack里直接写options 或者 postcssOptions好像不行（也可能是我写错了）还没有仔细研究 下面是官方写法
module.exports = {
  plugins: [
    require('autoprefixer'),
  ],
};

```
到这里就完成了基本的webpack配置。还可以根据项目需要如移动端可以补充px2rem之类的插件；需要antd可以针对antd做一些vender、或者主题less处理；针对编译过程、打包结果进行优化等等。这里就不扩展了，挖个坑，以后再单独写一篇webpack学习笔记。
大家有什么好的webpack方案也可以安利一下哇。
## TypeScript
可以直接通过tsc --init 生成一个tsconfig.json。再根据需要自己调整。
tsconfig.json的部分内容
```json
{
  ...其他的一些配置
  "files": [
    "./global.d.ts" // 申明一个全局的.d.ts
  ],
}
```
./global.d.ts
```typescript
// 这边申明了less文件 之后使用import styles from 'xxx.less'就不会有ts报错了
declare module '*.less' {
  const content:{[className:string]:string};
  export default content;
}
// 如果使用的第三方库没有ts，可以自己在这边申明

```

## 代码
### 入口
这里也可以直接使用html或者其他模板语言
index.ejs
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>demo</title>
</head>
<body>
  <div id="root"></div>
</body>
</html>
```
index.tsx
```javascript
import React from 'react';
import * as ReactDOM from 'react-dom';
import { Root } from './root';
ReactDOM.render(
  <Root/>,
  document.getElementById('root'),
);

```
我习惯把入口和root（React代码结构的根节点）分开来写。
root.tsx
```javascript
import React from 'react';
import { Provider } from 'react-redux';
import { HashRouter } from 'react-router-dom';  // router模式 自行选择 这里拿hash做例子
import { getStore } from './redux/root-store';  // redux获取store实例的方法
import './common/styles.css'; // 公共css 一般用来写清楚浏览器样式之类的内容
import { Routes } from './routes';  // router
/**
redux store的实例，这里导出是方便一些不在context里的组件或者一些方法使用redux里的数据
注意，这些不在context里的组件是不会因为store的变化重新渲染的
*/
export const store = getStore();
export const Root:React.FC = React.memo(function Root() {
  // 启用react的严格模式 具体可以去react的官网了解一下
  return (
    <React.StrictMode>
      <Provider store={store}>
        <HashRouter>
          <Routes/>
        </HashRouter>
      </Provider>
    </React.StrictMode>
  );
});

```
### redux
**这里想提一嘴，忘记在官网还是哪里看到的，如果你不确定是否需要用redux，就不要用。如果复杂度不高的项目（比较直接的就是不需要多组件共享数据之类的情况），redux只会让代码变复杂，没什么帮助。如果要使用，也一定要想清楚那些数据需要redux进行管理。维护过一个啥啥啥都放在redux里的项目，简直是噩梦。**
redux这里使用了ReduxToolKit，可以去官网了解一下，比起手写redux能少些不少代码，而且对ts支持也好很多。
异步中间件使用redux-saga，不了解的同学也可以去官网了解一下。可以根据需要自己选择中间件，比如redux-thunk之类的。
root-store.ts
```javascript
import { configureStore } from '@reduxjs/toolkit';
import createSagaMiddleware from 'redux-saga';
import { rootReducer } from './root-reducer';
import { rootSaga } from './root-saga';
export function getStore() {
  const sagaMiddleWare = createSagaMiddleware();
  const store = configureStore({
    reducer: rootReducer,
    middleware: [sagaMiddleWare], // 可以使用多个中间件，往数组里补充对应的实例就行了
  });
  sagaMiddleWare.run(rootSaga);
  return store; // 这里返回store的实例
}

```
root-reducer.ts
```javascript
import { combineReducers } from '@reduxjs/toolkit';
import { userSlice } from './user'; // user相关内容下面会写
// 这个好像没啥说的，方法名很明确了，就是组合reducer
export const rootReducer = combineReducers({
  user: userSlice.reducer,
});
// 这里返回了reducer的return type，便于其他地方使用
export type RootState = Readonly<ReturnType<typeof rootReducer>>;

```
root-saga.ts
```javascript
import { all, spawn, call } from 'redux-saga/effects';
import { userSaga } from './user/saga'; // user相关内容下面会写
// saga
export function* rootSaga() {
  const sagas = [
    userSaga,
  ];
  try {
    yield all(
      sagas.map((saga) => spawn(function* () {
        while(true) {
          yield call(saga);
          break;
        }
      })),
    );
  } catch (err) {
    console.log('root saga error', err);
  }
};

```
再提供一个简单的demo，也就是上面user的内容
user.ts
```javascript
import { createAction, createSlice, PayloadAction } from '@reduxjs/toolkit';
import { UserInfo, UserState } from './type'; // 定义一些ts 就不说了
// 初始state
const initialState:UserState = {
    userInfo: {
      name: '',
      avatarUrl: '',
    },
    token: '',
  };
const REDUCER_NAME_SPACE = 'user';  // space name
export const userSlice = createSlice({
  name: REDUCER_NAME_SPACE,
  initialState,
  // 简单写几个reducer
  reducers: {
    // 重置state，比如退出登录之类的时候执行
    initUserState(state:UserState):void {
      state = initialState;
    },
    setUserState(state:UserState, { payload }:PayloadAction<Partial<UserState>>):void {
      state = {
        ...state,
        ...payload,
      };
    },
  },
});

export const { initUserState, setUserState, setUserInfo } = userSlice.actions;
// 异步action 这里拿登录做例子
export const login = createAction<{username:string;password:string}>(`${REDUCER_NAME_SPACE}/login`);

```
/user/saga.ts
```javascript
import message from 'antd/lib/message';
import { take, call, fork } from 'redux-saga/effects';  // saga相关知识就先不扩展了
import { fetchLogin } from '../../fetch/http';
import { login } from './index';

// saga的一个核心就是generator 不了解的同学可以去看看es6里相关内容
function* watchLogin() {
  while(true) {
    try {
      // 获取action的payload
      const { payload } = yield take(login);
      const { username, password } = payload;
      // 执行请求
      const res = yield call(fetchLogin, username, password);
      // 一些业务逻辑
      if (res.data) {
        history.push('/home');
      } else {
        // toast、tip之类的
      }
    } catch (err) {
      console.log('login error', err);
      alert(err.message);
    }
  }
}

export function* userSaga() {
  yield fork(watchLogin);
}

```
### router
申明route
index.tsx
```javascript
import React, { Suspense } from 'react';
import { RouteProps, Route, Switch } from 'react-router-dom';
const routes:RouteProps[] = [
  {
    path: '/login',
    exact: true,
    // 按需加载，申明chunkName是方便看，比如打包的时候可以直接使用chunkName（会有缓存问题，还是用chunkhash比较好，具体还是看业务场景）
    // 使用import的话只能用export default哦
    component: React.lazy(() => import(/* webpackChunkName:"LoginPage" */ '../pages/login')),
  },
  {
    path: '/',
    component: React.lazy(() => import(/* webpackChunkName:"HomePage" */ '../pages/index')),
  },
];
export const Routes = () => (
  // 可以做一个loading状态
  <Suspense fallback={<div>loading</div>} >
    <Switch>
      {
        routes.map((val, key) => (
          <Route
            key={`route_${key}`}
            {...val}/>
        ))
      }
    </Switch>
  </Suspense>
);

```
### 页面
这里简单些个index
pages/index/index.tsx
```javascript
import React from 'react';
import styles from './index.less';  // 我这边直接使用的className，如果要用styleName的话，需要加react-css-module相关loader和插件
const IndexPage:React.FC = () => (
  <div className={styles.container}>这里是首页</div>
);
export default React.memo(IndexPage);

```
## 运行
到这里项目基本上就搭好啦，接下来就运行啦。
package.json
```json
{
  "scripts": {
    "start": "cross-env NODE_ENV=development webpack serve",
    "build": "cross-env NODE_ENV=production webpack"
  }
}
```
在package.json里增加上面的内容，然后执行npm run start就可以啦。
最后贴出package.json，如果运行异常请确认依赖版本。
**不同版本用法会有很大区别，如：**
React 16开始支持hooks等特性；webpack4, 5配置上也会有一些改动；等等，如果跑不起来可以根据版本做一些调整。
```json
{
  "name": "demo",
  "version": "1.0.0",
  "description": "基于React + typescript + webpack搭建的项目",
  "main": "index.js",
  "scripts": {
    "start": "cross-env NODE_ENV=development webpack serve",
    "build": "cross-env NODE_ENV=production webpack"
  },
  "keywords": [
    "React",
    "Typescript"
  ],
  "author": "Ego Tao",
  "license": "ISC",
  "dependencies": {
    "@reduxjs/toolkit": "^1.5.0",
    "axios": "^0.21.1",
    "classnames": "^2.2.6",
    "dayjs": "^1.10.4",
    "react": "^17.0.1",
    "react-dom": "^17.0.1",
    "react-redux": "^7.2.2",
    "react-router-dom": "^5.2.0",
    "redux-saga": "^1.1.3"
  },
  "devDependencies": {
    "@babel/core": "^7.12.17",
    "@babel/plugin-proposal-class-properties": "^7.12.13",
    "@babel/plugin-proposal-decorators": "^7.12.13",
    "@babel/plugin-proposal-nullish-coalescing-operator": "^7.12.13",
    "@babel/plugin-proposal-optional-chaining": "^7.12.17",
    "@babel/plugin-syntax-dynamic-import": "^7.8.3",
    "@babel/plugin-transform-runtime": "^7.12.17",
    "@babel/preset-env": "^7.12.17",
    "@babel/preset-react": "^7.12.13",
    "@babel/preset-typescript": "^7.12.17",
    "@types/react": "^17.0.2",
    "@types/react-dom": "^17.0.1",
    "@types/react-redux": "^7.1.16",
    "@types/react-router-dom": "^5.1.7",
    "@typescript-eslint/eslint-plugin": "^4.15.1",
    "@typescript-eslint/parser": "^4.15.1",
    "babel-loader": "^8.2.2",
    "babel-plugin-import": "^1.13.3",
    "babel-plugin-module-resolver": "^4.1.0",
    "clean-webpack-plugin": "^3.0.0",
    "compression-webpack-plugin": "^7.1.2",
    "css-loader": "^5.0.2",
    "eslint": "^7.20.0",
    "eslint-config-prettier": "^7.2.0",
    "eslint-plugin-import": "^2.22.1",
    "eslint-plugin-prettier": "^3.3.1",
    "eslint-plugin-promise": "^4.3.1",
    "eslint-plugin-react": "^7.22.0",
    "eslint-plugin-react-hooks": "^4.2.0",
    "file-loader": "^6.2.0",
    "friendly-errors-webpack-plugin": "^1.7.0",
    "html-webpack-plugin": "^5.1.0",
    "less": "^4.1.1",
    "less-loader": "^8.0.0",
    "postcss-import-sync2": "^1.1.0",
    "postcss-less": "^4.0.0",
    "postcss-loader": "^5.0.0",
    "postcss-preset-env": "^6.7.0",
    "postcss-pxtorem": "^5.1.1",
    "progress-bar-webpack-plugin": "^2.1.0",
    "style-loader": "^2.0.0",
    "stylelint": "^13.10.0",
    "stylelint-config-prettier": "^8.0.2",
    "stylelint-config-standard": "^20.0.0",
    "tsconfig-paths-webpack-plugin": "^3.3.0",
    "typescript": "^4.1.5",
    "url-loader": "^4.1.1",
    "webpack": "^5.23.0",
    "webpack-cli": "^4.5.0",
    "webpack-dev-server": "^3.11.2",
    "webpack-merge": "^5.7.3"
  }
}

```
## 总结
搭建过程中，还是踩了几个坑。
1. 依赖缺失。忘了装less，没有报缺失less而是报了个比较奇怪的错（忘了截图）。然后去google了半天，都说是缺失文件。我找了半天没找到是啥原因。然后就直接去报错的地方打log，发现是.less文件有问题。回去移除掉用了less的地方，果然就好了。然后才发现是少了less。直接裂开。
2. 依赖版本变化，版本变化导致一些用法变化。之前项目postcss用的3.0.0，这次装的5.0.0。导致loader报错，去github看了官方用法才解决。
这个项目只是最最基础的React+TypeScript+Less+Webpack