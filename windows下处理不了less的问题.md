# 记录在windows下Less文件处理异常
遇到了个很傻瓜的问题，等真的找到原因的时候，感觉自己宛若一个智障。
## 场景
因为需要修改Antd的主题，所以用less-loader在webpack里处理Antd的less文件（重写样式变量）。
代码如下,webpack.config.js:
```javascript
module.exports = {
  ...,
  module: {
    ...,
    rules: [
      ...,
      // antd的less解析
      {
        test: /\.less$/,
        include: /node_modules\/antd/,
        use: [
          'style-loader',
          'css-loader',
          {
            loader: 'less-loader',
            options: {
              lessOptions: {
                modifyVars: {
                  'primary-color': '#14783c',
                  'layout-header-background': '#14783c',
                  'layout-sider-background': '#fff',
                },
                javascriptEnabled: true,
              },
            },
          },
        ],
      },
    ],
  }
};
```
这里把antd的less单独拿出来处理，是因为项目中的less都通过css-modules处理了name，单独处理可以避免antd的样式呗css-modules处理后无法正确引入。

## 问题
在mac环境下代码运行正常，而在windows环境下出现了如下报错:
```shell
ERROR in ./node_modules/antd/dist/antd.less 1:0
Module parse failed: Unexpected character '@' (1:0)
You may need an appropriate loader to handle this file type, currently no loaders are configured to process this file. See https://webpack.js.org/concepts#loaders
> @import "../lib/style/index.less";
| @import "../lib/style/components.less";
 @ ./src/root.tsx 10:0-29
 @ ./src/index.tsx 3:0-30 5:50-54

webpack 5.10.0 compiled with 1
```
从错误可以看出是antd的less文件没有被loader处理，一开始本以为是loader执行的先后顺序或者windows环境下less相关库的版本问题。但是一直没有搜到有相关信息，没有定位出具体问题。

## 解决过程
1. 移除项目里.less的文件loader处理，统一使用antd的那部分loader的处理（场景标题下的示意代码，移除include）。目的是确认会不会是因为多个相同test导致loader执行有问题。经测试，antd样式正常，理所应当项目中自己的写的less因为css-modules规则没了，会出现样式错乱之类的问题。
2. 于是，真就以为多个相同test会有问题（但是在官方webpack文档并没有找到想说明）想尝试通过修改test、include、exclude等方式区分开这2个规则。尝试无果。
3. test是可以传一个function的，于是我就直接用function里写log来确定到底有没有执行到node_modules/antd里
```javascript
{
  test: (file) => {
    if (/node_modules\/antd/.test(file)) {
      console.log(file);
    }
    return /.less/.test(file);
  }
}
```
惊讶的发现，压根没有log。我直接人傻了，然后就把所有file都log然后cmd+f搜索node_modules/antd。
4. 居然没有node_modules/antd，我就怀疑是include的正则写错了，排除掉了antd路径。移除后发现，还是没有。我直接人傻了，然后log所有包含node_modules和antd的file，发现是有的。就复制路径出来进行正则测试，一直false。鬼使神差的，我注意到了/和\的问题。经过测试，的确是这个引起的，处理过后windows和mac处理正常。

## 原因
原因很简单，在windows环境下路径分隔符是\，而在mac下是/。所以在webpack中的include的正则无法匹配到antd的路径，因此antd的less就没有被loader处理到。所以只需要在正则里同事匹配\和/就可以解决问题。
```javascript
{
  include: /node_modules\/antd/, // windows里路径是\ mac路径分隔分是/
}
```
其实很多路径正则里好像都有看到过这种写法，就是没注意到。
**经过这个问题，在之后任何使用正则匹配的地方都应该要考虑windows和mac、linux环境的兼容问题**