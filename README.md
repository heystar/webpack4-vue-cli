# webpack4-vue-cli

> Vue-CLI 升级到Webpack 4 Vue.js project

## Build Setup

``` bash
# install dependencies
npm i

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run e2e tests
npm run e2e

# run all tests
npm test
```

# 升级指南 Upgrade Guide
注意：你需要一个已有的vue-cli安装项目，若没有请先执行
```
npm vue vue-cli -g
```
```
vue init webpack <my-project>
```
安装过程中会出现`Failed at the chromedriver@2.41.0 install script 'node install.js`错误，执行：
```
npm install chromedriver --chromedriver_cdnurl=http://cdn.npm.taobao.org/dist/chromedriver
```
如果设置过淘宝镜像`npm config set registry https://registry.npm.taobao.org` 请改回原npm地址`npm config set registry http://registry.npmjs.org` 这是因为需要执行`npm-check`命令
# step 1 packages update and install
```
npm i npm-check -g
```
```
cd <path>/<my-project>
```
```
npm-check -u
```
```
npm i webpack-cli -D
```
# step 2 set webpack mode
修改`webpack.dev.conf.js`
```
const devWebpackConfig = merge(baseWebpackConfig, {
  // 修改部分开始
  mode: 'development',
  // 修改部分结束
  module: {
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap, usePostCSS: true })
  },
  ...
```
修改`webpack.prod.conf.js`
```
const webpackConfig = merge(baseWebpackConfig, {
  // 修改部分开始
  mode: 'production',
  // 修改部分结束
  module: {
    rules: utils.styleLoaders({
  ...
```
# step 3 set vue-loader
修改`webpack.dev.conf.js`
```
const HtmlWebpackPlugin = require('html-webpack-plugin')
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
const portfinder = require('portfinder')
// 修改部分开始
const { VueLoaderPlugin } = require('vue-loader')
// 修改部分结束
...
```

```
...
plugins: [
    // 修改部分开始
    new VueLoaderPlugin(),
    // 修改部分结束
    new webpack.DefinePlugin({
      'process.env': require('../config/dev.env')
    }),
...
```
修改`webpack.prod.conf.js`
```
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')
// 修改部分开始
const { VueLoaderPlugin } = require('vue-loader')
// 修改部分结束
...
```
```
...
plugins: [
    // 修改部分开始
    new VueLoaderPlugin(),
    // 修改部分结束
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    new webpack.DefinePlugin({
      'process.env': env
    }),
...
```
# step 4 set split chunk
修改`webpack.prod.conf.js`，添加`optimization`
```
...
output: {
    path: config.build.assetsRoot,
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  // 加入部分代码开始
  optimization: {
    minimizer: true,
    splitChunks: {
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name: 'vendor',
          chunks: 'all'
        },
        manifest: {
          name: 'manifest',
          minChunks: Infinity
        },
      }
    },
  },
  // 加入部分代码结束
  plugins: [
    new VueLoaderPlugin(),
...
```
删除`pulgins`中`optimize.CommonsChunkPlugin`的代码
```
...
 // keep module.id stable when vendor modules does not change
    new webpack.HashedModuleIdsPlugin(),
    // enable scope hoisting
    new webpack.optimize.ModuleConcatenationPlugin(),
    // 删除部分开始
    // split vendor js into its own file
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks (module) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      minChunks: Infinity
    }),
    // This instance extracts shared chunks from code splitted chunks and bundles them
    // in a separate chunk, similar to the vendor chunk
    // see: https://webpack.js.org/plugins/commons-chunk-plugin/#extra-async-commons-chunk
    new webpack.optimize.CommonsChunkPlugin({
      name: 'app',
      async: 'vendor-async',
      children: true,
      minChunks: 3
    }),
    // 删除部分结束
    // copy custom static assets
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
...
```
# step 5 use mini-css-extract-plugin replace extract-text-webpack-plugin
```
npm i mini-css-extract-plugin -D
```
```
npm un extract-text-webpack-plugin -D
```
修改`build/utils.js`
```
const ExtractTextPlugin = require('extract-text-webpack-plugin')
```
替换为：
```
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
```
```
...
// Extract CSS when that option is specified
    // (which is the case during production build)
    if (options.extract) {
      return ExtractTextPlugin.extract({
        use: loaders,
        fallback: 'vue-style-loader'
      })
    } else {
      return ['vue-style-loader'].concat(loaders)
    }
...
```
修改为：
```
...
// Extract CSS when that option is specified
    // (which is the case during production build)
    if (options.extract) {
      return [MiniCssExtractPlugin.loader].concat(loaders)
    } else {
      return ['vue-style-loader'].concat(loaders)
    }
...
```
修改`webpack.prod.conf.js`
```
const ExtractTextPlugin = require('extract-text-webpack-plugin')
```
替换为：
```
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
```
`pulgins`中下面这段配置
```
// extract css into its own file
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css'),
      // Setting the following option to `false` will not extract CSS from codesplit chunks.
      // Their CSS will instead be inserted dynamically with style-loader when the codesplit chunk has been loaded by webpack.
      // It's currently set to `true` because we are seeing that sourcemaps are included in the codesplit bundle as well when it's `false`,
      // increasing file size: https://github.com/vuejs-templates/webpack/issues/1110
      allChunks: true,
    }),
```
替换为：
```
new MiniCssExtractPlugin({
   filename: utils.assetsPath('css/[name].[contenthash:12].css'),
   allChunks: true,
 }),
```

# step 5 run
```
npm run dev
```
`dev`环境能正常运行
```
npm run build
```
`build`环境出现eslint、OptimizeCssAssetsWebpackPlugin插件错误
降级解决`eslint`错误:
```
npm i eslint@4.19.1 -D
```
删除`optimize-css-assets-webpack-plugin`插件
```
npm un optimize-css-assets-webpack-plugin -D
```
修改`webpack.prod.conf.js`
删除引用：
```
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
```
删除`pulgins`中配置：
```
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    new OptimizeCSSPlugin({
      cssProcessorOptions: config.build.productionSourceMap
        ? { safe: true, map: { inline: false } }
        : { safe: true }
    }),
```
Webpack4不再需要`UglifyJsPlugin`插件,删除`uglifyjs-webpack-plugin`插件
```
npm un uglifyjs-webpack-plugin -D
```
删除引用：
```
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')
```
删除`pulgins`中配置：
```
    new UglifyJsPlugin({
      uglifyOptions: {
        compress: {
          warnings: false
        }
      },
      sourceMap: config.build.productionSourceMap,
      parallel: true
    }),
```
