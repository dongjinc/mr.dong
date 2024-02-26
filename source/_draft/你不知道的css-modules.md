---
title: 你不知道的css-modules
layout: post
comments: true
published: false
---

问题
1.Cannot find module './scss/style/name.css' or its corresponding type declarations.ts(2307)

首先是ts-loader, 抛出错误, 其中 TypeScript 不知道有其他文件.ts，.tsx, .css文件, 所以如果导入的文件后缀未知，它会抛出错误。
需要全局在global.d.ts声明, 比如:
declare module '*.css';
declare module '*.scss';

2.TypeScript error Property '**' does not exist on type 'DetailedHTMLProps<……> 

 namespace React{
        interface HTMLAttributes<T> extends AriaAttributes, DOMAttributes<T>{
            // 扩展HTML标签的属性类型
            // 如果需要使用自定义属性的化需要
            path?: string | undefined | { [key: string]: any};
            styleName?: string
        }
    }

```js
    module.exports = {
        module: { // 
            rules: [
                {
                    test: /\.css$/i,
                    loader: "css-loader",
                    options: {
                        importLoaders: 0, // https://github.com/webpack-contrib/css-loader/issues/228#issuecomment-312885975 解答 我认为这个可以通过 在css文件中引用 @import './xxx.scss' 就能体会它的作用
                        modules: {
                            auto: true, // true- 启用 CSS 模块或可互操作的 CSS 格式，为所有满足条件的文件设置选项值或为所有满足条件的modules.mode文件设置选项值local/\.module(s)?\.\w+$/i.test(filename) modules.mode icss/\.icss\.\w+$/i.test(filename), 如果开启auto, 默认仅对 module(s), icss 做modules处理文件
                            namedExport: true, // 启用名为 export 的 ES 模块
                            exportLocalsConvention: "asIs", // 默认情况下，导出的 JSON 键镜像类名（即asIs值）。修改导出变量的是否驼峰化
                        },
                    },
                },
            ],
        },
    };
    import { fooBaz, bar } from "./styles.css";
    console.log(fooBaz, bar);
```

```js
exportLocalsConvention xx
'asIs'	string	类名将按原样导出。
'camelCase'	string	类名将被骆驼化，原始类名不会从本地人中删除
'camelCaseOnly'	string	类名将被驼峰化，原来的类名将从本地人中删除
'dashes'	string	只有类名中的破折号会被驼峰化
'dashesOnly'	string	类名中的破折号将被驼峰化，原始类名将从本地人中删除

module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: "css-loader",
        options: {
          esModule: true,
          modules: {
            namedExport: true, 
          },
        },
      },
    ],
  },
};
```

```js

[name] the basename of the resource
[folder] the folder the resource relative to the compiler.context option or modules.localIdentContext option.
[path] the path of the resource relative to the compiler.context option or modules.localIdentContext option.
[file] - filename and path.
[ext] - extension with leading ..
[hash] - the hash of the string, generated based on localIdentHashSalt, localIdentHashFunction, localIdentHashDigest, localIdentHashDigestLength, localIdentContext, resourcePath and exportName
[<hashFunction>:hash:<hashDigest>:<hashDigestLength>] - hash with hash settings.
[local] - original class.


module.exports = {
  module: {
    rules: [
      {
        test: /\.css$/i,
        loader: "css-loader",
        options: {
          esModule: true,
          modules: {
           localIdentName: "[path][name]__[local]--[hash:base64:5]", // 允许配置生成的本地标识名称。
          },
        },
      },
    ],
  },
};
```




## css-loader
对于production构建，建议从您的包中提取 CSS，以便以后能够使用并行加载 CSS/JS 资源。这可以通过使用mini-css-extract-plugin来实现，因为它会创建单独的 css 文件。对于developmentmode （包括webpack-dev-server），您可以使用style-loader，因为它使用多个 <style></style> 将 CSS 注入 DOM 并且运行速度更快。

note: 不要style-loader和mini-css-extract-plugin一起使用。

## react-css-modules


## babel-plugin-react-css-modules


babel-plugin-react-css-modules 和 css-loader 要一起配置
前者对应解析tsx后者解析css、scss、less等文件
前者可配置在.babelrc 或者 webpack配置中