# 使用gulp构建一个模板项目

2019/09/22 23:49

<!-- TOC -->

- [目标](#目标)
- [步骤](#步骤)
- [gulp命令](#gulp命令)
- [gulp插件](#gulp插件)
  - [删除dist目录](#删除dist目录)
  - [压缩、混淆](#压缩混淆)
  - [使用es6](#使用es6)
  - [文件重命名](#文件重命名)
  - [合并多文件，输出单一文件](#合并多文件输出单一文件)
  - [处理css浏览器兼容性前缀](#处理css浏览器兼容性前缀)
  - [处理sass，编译压缩](#处理sass编译压缩)
  - [起一个本地服务，开始调接口吧](#起一个本地服务开始调接口吧)
    - [browser-sync](#browser-sync)
    - [gulp-connect + http-proxy-middleware](#gulp-connect--http-proxy-middleware)
    - [browser-sync 与 gulp-connect 区别](#browser-sync-与-gulp-connect-区别)
- [注意](#注意)
  - [return 和 done() 有什么区别](#return-和-done-有什么区别)
  - [为什么热刷新不好使](#为什么热刷新不好使)
  - [做代理解决跨域问题时，遇到404问题](#做代理解决跨域问题时遇到404问题)
  - [为什么引入第三方依赖包如jquery后，源代码找不到$](#为什么引入第三方依赖包如jquery后源代码找不到)

<!-- /TOC -->

## 目标

使用gulp从零构建一个登录页面

## 步骤

1. `npm init -y`
2. `git init` 建立git仓库

   设置 `.gitignore`

   ```js
   node_modules
   .idea
   ```

3. 安装gulp

   ```js
   npm install --global gulp-cli

   npm i -D gulp
   ```

4. 创建一个简单的gulpfile

   ```js

   const { series, src, dest } = require('gulp');

   function jsTask() {
    return src('src/**/*.js')
            .pipe(dest('dist'))
   }

   exports.default = series(jsTask);
   ```

## gulp命令

1. 查看帮助

   `gulp -h`

2. 查看所有任务

   `gulp --tasks`

3. 执行任务

   `gulp [任务名称]`

## gulp插件

### 删除dist目录

`npm i -D del`

```js
const del = require('del');
function cleanTask(done) {
  del.sync('dist')
    done()
}
```

### 压缩、混淆

`npm i -D gulp-uglify`

```js
const uglify = require('gulp-uglify');

 .pipe(uglify())  // 压缩、混淆
```

### 使用es6

`npm i -D @babel/core @babel/preset-env gulp-babel`

```js
const babel = require('gulp-babel');

 .pipe(babel({
            presets: ['@babel/env']
          }))
```

### 文件重命名

`npm i -D gulp-rename`

```js
const rename = require("gulp-rename");

.pipe(rename('zbw.js'))

.pipe(rename(function (path) {
    path.dirname += "/ciao";
    path.basename += "-goodbye";
    path.extname = ".md";
  }))

.pipe(rename({
    dirname: "main/text/ciao",
    basename: "aloha",
    prefix: "bonjour-",
    suffix: "-hola",
    extname: ".md"
  }))
```

### 合并多文件，输出单一文件

`npm i -D gulp-concat`

```js
const concat = require("gulp-concat");

 .pipe(concat('all.js'))
```

### 处理css浏览器兼容性前缀

`npm i -D gulp-autoprefixer`

```js
const autoprefixer = require('gulp-autoprefixer');

 .pipe(autoprefixer({
            cascade: false,
            remove: false // 删除过时的prefixer
          }))

 "browserslist": [
    "last 2 version",
    "> 2%"
  ]
```

### 处理sass，编译压缩

`npm i -D gulp-sass`

```js
const sass = require('gulp-sass');

.pipe(sass({
          outputStyle: 'compressed'
        }))
```

### 起一个本地服务，开始调接口吧

#### browser-sync

`npm i -D browser-sync`

```js
const browserSync = require('browser-sync').create();
const reload = browserSync.reload;

// 在js任务后
.pipe(dest('./dist/js/'))  // 输出到根目录下dist文件夹
.pipe(reload({ stream: true }))

// 在css任务后
.pipe(dest('./dist'))
.pipe(reload({ stream: true }))

function server(done) {
  browserSync.init({
    server: {
      baseDir: './'  // 指定根目录
    }
  })
  done()
}

const buildTask = series([cleanTask, jsTask, cssTask, server, watcherTask])

exports.default = buildTask;
```

#### gulp-connect + http-proxy-middleware

`gulp-connect` 用来起服务，配合 `http-proxy-middleware` 做代理，解决跨域问题

`npm i -D gulp-connect http-proxy-middleware`

```js
const connect = require('gulp-connect');
const proxy = require('http-proxy-middleware');

/**
 * @description 起服务
 */
function serverTask() {
  connect.server({
    root: './dist',
    port: 8888,
    livereload: true,
    middleware: function(connect, opt) {
        return [
            proxy('/api/',  {
                target: 'https://devtk.aibeike.com',
                changeOrigin:true,
                pathRewrite: {  // 覆写路径
                  '^/api/': ''
                }
            })
        ]
    }
  })
}

/**
 * @description 处理html任务
 * @returns
 */
function htmlTask() {
  return src('src/index.html')
          .pipe(dest('dist'))
          .pipe(connect.reload()) // reload解决热加载问题，js和css资源同理
}

// serverTask任务和watcherTask任务要并行处理，否则执行到serverTask服务器启动后，任务就自动关闭了，热加载功能会丢失
const assetsTask = series(cleanTask, htmlTask, jsTask, cssTask, serverTask)

const buildTask = parallel(assetsTask, watcherTask)

exports.default = buildTask;
```

#### browser-sync 与 gulp-connect 区别

`browser-sync` 没有配合使用的代理插件，在解决跨域问题有些鸡肋

`gulp-connect` 配合 `http-proxy-middleware` 使用可以很方便的解决跨域问题

## 注意

### return 和 done() 有什么区别

return 是当返回promise时

done回调方法比较通用，比如热刷新reload时，return就不好使了

### 为什么热刷新不好使

这段代码不可以写成return，只能写成done

### 做代理解决跨域问题时，遇到404问题

遇到404的一种很常见的情况是，在proxy时，使用了 `/api` 作为请求接口的前缀链接，但是真正的接口地址中是不存在 `/api` 这样的路径的，此处需要使用 `pathRewrite` 覆写删除 `/api`。

同样的问题，参考资料：[http-proxy-middleware配合gulp使用时的一些坑](https://ymbo.github.io/2018/01/09/http-proxy-middleware%E9%85%8D%E5%90%88gulp%E4%BD%BF%E7%94%A8%E6%97%B6%E7%9A%84%E4%B8%80%E4%BA%9B%E5%9D%91/)

### 为什么引入第三方依赖包如jquery后，源代码找不到$

处理js资源时，src方法读取js文件的顺序是有要求的。

```js
/**
 * @description 处理js任务
 * @returns
 */
function jsTask() {
  // 读取src文件夹下所有js文件
  // 注意js文件打包引入的先后顺序，先导入lib，再导入应用代码
  return src(['src/lib/*.js', 'src/js/*.js'])
          .pipe(babel({ // 编译es6
            presets: ['@babel/env']
          }))
          .pipe(uglify())  // 压缩、混淆
          .pipe(concat('bundle.js')) // 合并所有文件，输出all.js文件
          // .pipe(rename('bundle.js')) // 重命名为zbw.js
          .pipe(dest('./dist/js/'))  // 输出到根目录下dist文件夹
          .pipe(connect.reload()) // 热加载
}
```
