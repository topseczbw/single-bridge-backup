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
  - [browser-sync](#browser-sync)
- [注意](#注意)
  - [return 或者 done()](#return-或者-done)
  - [为什么热刷新不好使](#为什么热刷新不好使)

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

### browser-sync

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

## 注意

### return 或者 done()

return 是当返回promise时
done比较通用  比如热刷新reload时，return就不好使了

### 为什么热刷新不好使

这段代码不可以写成return，只能写成done
