### 为什么

小程序不支持css预处理语言sass, less, stylus，于是想办法通过编译scss 达到目的，由于不用配置服务那么 webpack 不是特别好用，基于流的 gulp 显然更合适。

### 怎么做

- 全局安装 node

- 全局安装 gulp

- 小程序目录中初始化 npm

```javascript
npm i gulp -g
npm init -y
npm i gulp gulp-sass gulp-rename gulp-postcss postcss-font-base64 stream-combiner2 -D
```

- gulp 读取 gulpfile.js 的配置 所以先新建 gulpfile.js。

```javascript
const src = './pages'
const gulp = require('gulp')
const watch = require('gulp-watch')
const sass = require('gulp-sass')
const rename = require('gulp-rename')
const postcss = require('gulp-postcss')
const base64 = require('postcss-font-base64')
const combiner = require('stream-combiner2')

// 定义一个任务
gulp.task('scss', () => {
  var combined = combiner.obj([
    gulp.src(`${src}/**/*.scss`),
    sass().on('error', sass.logError),
    postcss([base64()]),
    rename((path) => (path.extname = '.wxss')),
    gulp.dest(src)
  ])

  combined.on('error', async (error) => {
    await console.log(error)
  })
})

// 监听scss文件，自动执行编译任务
gulp.task('watch_scss', () => {
  watch(`${src}/**/*.scss`, gulp.series('scss'))
})

// 默认的任务 运行时直接用 gulp 即可
gulp.task('default', gulp.series('watch_scss'))
```

原来使用的是 gulp 自带的 watch ,但是在运行时只执行了一次 watch 操作，后面安装了 gulp-watch 代替 gulp.watch 

> gulp.watch()无法监听到新增加的文件，这样一来，我们每次增加文件时都要执行gulp命令来重启服务。这并不是我们希望的结果。可以引入gulp-watch模块解决这个问题。

**目前只在本地开发环境时尝试过，算是个预研测试，还没测试打包到线上会不会有问题，仅供参考**