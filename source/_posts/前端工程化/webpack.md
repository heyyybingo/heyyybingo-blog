# 常见配置项

- mode
- entry
- output
- module
- plugins
- optimization

# 优化

## 代码压缩

### js

- terser-webpack-plugin https://webpack.docschina.org/plugins/terser-webpack-plugin/
- uglifyjs-webpack-plugin

### css

- mini-css-extract-plugin

### html

- html-webpack-plugin

```js
new htmlWebpackPlugin({
   title: "京东商城",
   template: "./index.html",
   filename: "index.html",
   minify: {
     // 压缩HTML⽂件
     removeComments: true, // 移除HTML中的注释
     collapseWhitespace: true, // 删除空⽩符与换⾏符
     minifyCSS: true // 压缩内联css
  }
 }),
```

### js tree shaking

```js
optimization: {
  usedExports: true; // 哪些导出的模块被使⽤了，再做打包
}
```

```js
//package.json
"sideEffects":false //正常对所有模块进⾏tree shaking , 仅⽣产模式有效，需要配合usedExports
```

## 代码分割

```js
 optimization: {
   splitChunks: {
      chunks: "all", // 所有的 chunks 代码公共的部分分离出来成为⼀个单独的⽂件
    },
 },
```

## 加速构建

### 配置 loader 时，使用 test、include、exclude 三个配置项来缩⼩ loader 的处理范围

```js
rules: [
      {
        test: /\.css$/,
        include: path.resolve(__dirname, "./src"),
        use: ["style-loader", "css-loader"],
      },
      {
        test: /\.less$/,
        include: path.resolve(__dirname, "./src"),
        use: [
          // "style-loader",
          MiniCssExtractPlugin.loader,
          {
            loader: "css-loader",
            options: {
              //css modules 开启
              modules: true,
            },
          },
          {
            loader: "postcss-loader",
          },
          "less-loader",
        ],
      },
      {
        test: /\.(png|jpe?g|gif)$/,
        include: path.resolve(__dirname, "./src"),
        use: {
          loader: "url-loader",
          options: {
            name: "[name]_[hash:6].[ext]",
            outputPath: "images/",
            //推荐使用url-loader 因为url-loader支持limit
            //推荐小体积的图片资源转成base64
            limit: 12 * 1024, //单位是字节 1024=1kb
          },
        },
      },
      {
        test: /\.js$/,
        include: path.resolve(__dirname, "./src"),
        exclude: /node_modules/,
        use: {
          loader: "babel-loader",
        },
      },
    ],

```

### resolve.alias

配置通过别名来将原导⼊路径映射成⼀个新的导⼊路径。拿 react 为例，我们引⼊的 react 库，⼀般存在两套代码：

```js
 resolve: {
    //查找第三方优化
    modules: [path.resolve(__dirname, "./node_modules")],
    alias: {
      "@": path.join(__dirname, "./src"),
      react: path.resolve(__dirname, "./node_modules/react/umd/react.production.min.js"),
      "react-dom": path.resolve(__dirname, "./node_modules/react-dom/umd/react-dom.production.min.js")
    },
  },

```

### resolve.extensions

在导⼊语句没带⽂件后缀时，webpack 会⾃动带上后缀后，去尝试查找⽂件是否存在。

- 后缀列表应尽量小。
- 导⼊语句尽量的带上后缀。

```js
resolve: {
  extensions: [".js", ".json", ".jsx", ".ts"];
}
```

### 使⽤ externals 优化 cdn 静态资源

```js
module.exports = {
  //...
  externals: {
    jquery: "jQuery",
  },
};
```
