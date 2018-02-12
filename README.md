# 搭建三端统一开发环境

## 初始换react native 项目
 - 使用[react native 官方文档](https://facebook.github.io/react-native/docs/getting-started.html)搭建react naitve环境。使用Building Projects with Native Code环境
 - 初始化 react native 项目。`react-native init rnWeb`
 - `cd rnWeb`

## 添加react-native-web 依赖包
 - 命令如下
 
```bash
yarn add react react-dom react-native-web
yarn add --dev babel-plugin-react-native-web

## 搭建web打包环境
``` 
 - 在根目录下添加入口文件index.web.js
 ```js
 //index.web.js
import { AppRegistry } from 'react-native';
import App from './App';

AppRegistry.registerComponent('App', () => App);
AppRegistry.runApplication('App', { rootTag: document.getElementById('app') });
```

 - 根目录下，新建web文件夹，添加打包配置。
 - 新建webpack.config.js
 ```bash
 // web/webpack.config.js
 
 const path = require('path');
 const webpack = require('webpack');
 const HtmlWebpackPlugin = require('html-webpack-plugin');
 
 const appDirectory = path.resolve(__dirname, '../');
 
 // This is needed for webpack to compile JavaScript.
 // Many OSS React Native packages are not compiled to ES5 before being
 // published. If you depend on uncompiled packages they may cause webpack build
 // errors. To fix this webpack can be configured to compile to the necessary
 // `node_module`.
 const babelLoaderConfiguration = {
   test: /\.js$/,
   // Add every directory that needs to be compiled by Babel during the build.
   // include: [
   //   path.resolve(appDirectory, 'index.web.js'),
   //   path.resolve(appDirectory, 'src'),
   //   path.resolve(appDirectory, 'node_modules/react-native-uncompiled')
   // ],
   use: {
     loader: 'babel-loader',
     options: {
       cacheDirectory: true,
       // Babel configuration (or use .babelrc)
       // This aliases 'react-native' to 'react-native-web' and includes only
       // the modules needed by the app.
       plugins: ['react-native-web'],
       // The 'react-native' preset is recommended to match React Native's packager
       presets: ['react-native']
     }
   }
 };
 
 // This is needed for webpack to import static images in JavaScript files.
 const imageLoaderConfiguration = {
   test: /\.(gif|jpe?g|png|svg)$/,
   use: {
     loader: 'url-loader',
     options: {
       name: '[name].[ext]'
     }
   }
 };
 
 module.exports = {
   // your web-specific entry file
   entry: path.resolve(appDirectory, 'index.web.js'),
 
   // configures where the build ends up
   output: {
     filename: 'bundle.web.js',
     path: path.resolve(appDirectory, 'dist')
   },
 
   // ...the rest of your config
 
   module: {
     rules: [
       babelLoaderConfiguration,
       imageLoaderConfiguration
     ]
   },
 
   plugins: [
     // `process.env.NODE_ENV === 'production'` must be `true` for production
     // builds to eliminate development checks and reduce build size. You may
     // wish to include additional optimizations.
     new webpack.DefinePlugin({
       'process.env.NODE_ENV': JSON.stringify(process.env.NODE_ENV || 'development'),
       __DEV__: process.env.NODE_ENV === 'production' || true
     }),
     new HtmlWebpackPlugin({
       title: '志勇的app',
       template: './web/template.ejs',
     }),
   ],
 
   resolve: {
     // If you're working on a multi-platform React Native app, web-specific
     // module implementations should be written in files using the extension
     // `.web.js`.
     extensions: [ '.web.js', '.js' ],
     alias: {
       'react-native': 'react-native-web',
     },
   },
   devServer: {
     port: 3003
   }
 }

 ```
  - 添加模板文件 template.ejs
  ```bash
<!DOCTYPE html>
<html>
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8"/>
    <meta name="viewport"
          content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=no,viewport-fit=cover"/>
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
<div id="app"></div>
</body>
</html>
```
- 添加启动命令 web ，web-prd 到根目录package.json内
```bash
"web": "./node_modules/.bin/webpack-dev-server -d --config ./web/webpack.config.js --inline --hot --colors",
    "web-prd": "./node_modules/.bin/webpack -p --config ./web/webpack.config.js"
```

- 启动： `npm run web`

