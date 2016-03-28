---
title: gulp插件解析 
publish: false
categories: [前端开发]
keywords: 小书匠,markdown
time: 2015-03-28 
---


### babel
现在我们将它分为两个单独的包： babel-cli和babel-core。
```cmd
npm install --global babel-cli
npm install --save-dev babel-core
```
如果你想要在命令行使用Babel，你可以安装babel-cli，或者你需要在一个Node项目中使用Babel，你可以使用babel-core。

**增加插件和预设值**
你需要根据你需要完成的任务来单独安装相应的插件。
```cmd
npm install --save-dev babel-preset-es2015
```
在.babelrc中配置：
```
{
    "presets": ["es2015"]
}
```

> 参考: [cnodejs.org][1]  &&  [github.com][2]

### BrowserSync

对于修改或简单实例的使用，命令行用法可以说是相当有帮助的。全局安装Browsersync后，你可以这样运行它：
```cmd
$ browser-sync start 
```
命令行常用的API:
> --files 文件路径看
--server 运行本地服务器（使用您的CWD作为Web根）
--host  指定主机名使用
--port  指定要使用的端口
#### Browsersync API
在 2.0.0 版本之前 
我们允许直接引用Browsersync模块后即可使用：
```cmd
// 引用 browserSync 模块
var browserSync = require("browser-sync");

// 启动服务器
browserSync({server: "./app"});

// 调用reload方法 
browserSync.reload("core.css");
```
在2.0.0+版本（推荐） 
虽然上述方式依然支持，但现在我们推荐以下方式代替。调用 .create() 意味着你得到一个唯一的实例并允许您创建多个服务器或代理。
```
// require 加载 browser-sync 模块
var bs = require("browser-sync").create();

// .init 启动服务器
bs.init({
    server: "./app"
});

// 现在请BS，而不是方法
// 主Browsersync模块出口
bs.reload("*.html");
```
常用的API:
> .create() 创建一个唯一的实例并允许您创建多个服务器或代理
.init( config, cb ) 启动一个服务器
.reload( arg )  导致浏览器刷新，要么注入文件，实时更新改动
.stream( opts ) 返回一个变换流，并且可以充当一次或多个文件

**options (选项配置**
```js
browserSync({
   notify: false, //浏览器中的通知。
   port: 9000, //使用（而不是一个自动检测到Browsersync）特定端口 
   server: {
     baseDir: ['.tmp', 'app'], //多个基目录
     routes: { //值是文件夹要提供的（相对于当前的工作目录）
       '/bower_components': 'bower_components'
     }
   }
}
```

**Browsersync + Gulp.js**
```
var gulp        = require('gulp');
var browserSync = require('browser-sync').create();
var sass        = require('gulp-sass');
var reload      = browserSync.reload;

// 静态服务器 + 监听 scss/html 文件
gulp.task('serve', ['sass'], function() {

    browserSync.init({
        server: "./app"
    });

    gulp.watch("app/scss/*.scss", ['sass']);
    gulp.watch("app/*.html").on('change', reload);
});

// scss编译后的css将注入到浏览器里实现更新
gulp.task('sass', function() {
    return gulp.src("app/scss/*.scss")
        .pipe(sass())
        .pipe(gulp.dest("app/css"))
        .pipe(reload({stream: true}));
});

gulp.task('default', ['serve']);
```

> 参考 [browsersync][3]
### Ruby SASS && SASS

gulp-ruby-sass基于ruby和sass，编译Sass文件为 CSS文件。 特点：比gulp-sass，但是更稳定，功能更丰富。
```
var gulp = require('gulp');
var sass = require('gulp-ruby-sass');

gulp.task('default', function () {
    return gulp.src('src/scss/app.scss')
        .pipe(sass({sourcemap: true, sourcemapPath: '../scss'}))
        .pipe(gulp.dest('dist/css'));
});
//sourcemap可用：需要Sass>=3.3.0 和 sourcemapPath选项。
//sourcemapPath：因为gulp-ruby-sass 不知道你的CSS目录 或者你服务安装目录，你不得不给一些额外的信息来帮助浏览器找到sourcemaps.
```
gulp-sass是调用node-sass,有node.js环境就够了。
```
gulp.task('sass', function() {
    return gulp.src(src.scss)
        .pipe(sass())
        .pipe(gulp.dest(src.css))
        .pipe(reload({stream: true}));
});
```
通过查看依赖关系以及部分源码可以发现，gulp-ruby-sass和gulp-sass的区别就在于编译器的不同和编译过程不同。

: gulp-ruby-sass是调用sass，所以需要ruby环境，需要生成临时目录和临时文件
: gulp-sass是调用node-sass,有node.js环境就够了，编译过程不需要临时目录和文件，直接通过buffer内容转换。
搞定这个问题后，就可以愉快地使用了，我又能继续学习css了。
> 参考：https://segmentfault.com/a/1190000003112509

**错误提示**
Syntax error: Invalid GBK character “\xE5″
sass文件编译时候使用ruby环境，在xp环境中没有任何问题，但是在windows7环境下无论是界面化的koala工具还是命令行模式的都会出现以下错误。

找到ruby的安装目录，里面也有sass模块，如这个路径：
C:\Ruby\lib\ruby\gems\1.9.1\gems\sass-3.3.14\lib\sass
在这个文件里面engine.rb，添加一行代码（同方法1）
Encoding.default_external = Encoding.find(‘utf-8′)
放在所有的require XXXX 之后即可。
> 参考：http://www.tuicool.com/articles/f2YVRv

### Browserify
> 曾经，有一个 gulp-browserify 摆在我面前，后来它被加入了黑名单。

社区为 gulp 开发了各种各样的插件，但 gulp 生态系统是有逼格的，不是来者不拒。对于一些不符合 gulp 理念的插件，被毫不留情地加入了黑名单，gulp-browserify 便是其中之一。
**直接使用Browserify**
要点：Stream 转换
vinyl-source-stream + vinyl-buffer
vinyl-source-stream: 将常规流转换为包含 Stream 的 vinyl 对象；
vinyl-buffer: 将 vinyl 对象内容中的 Stream 转换为 Buffer。
```
var browserify = require('browserify');  
var gulp = require('gulp');  
var uglify = require('gulp-uglify');  
var source = require('vinyl-source-stream');  
var buffer = require('vinyl-buffer');

gulp.task('browserify', function() {  
  return browserify('./src/js/app.js')
    .bundle()
    .pipe(source('bundle.js')) // gives streaming vinyl file object
    .pipe(buffer()) // <----- convert from streaming to buffered vinyl file object
    .pipe(uglify()) // now gulp-uglify works 
    .pipe(gulp.dest('./dist/js'));
});
```
把常规流转转成 vinyl 对象流是在 gulp 中直接使用 browserify 的关键点，**理解了这点，所有问题都迎刃而解了，其他操作也是在此基础上进行的。

### Gulp 高级技巧
**gulp 预期输入的是 Vinyl 文件对象，直接输入读取流当然不行。**
使用 gulp 过程中，偶尔会遇到 Streaming not supported 这样的错误。这通常是因为常规流与 vinyl 文件对象流有差异、gulp 插件使用了只支持 buffer （不支持 stream）的库。比如，不能把 Node 常规流直接传递给 gulp 及其插件。
> 译者注：Gulp 管道中的「流」不是操作 Strings 和 Buffers 的常规 Node.js 流，而是启用 object mode 的流。Gulp 中的流发送的是 vinyl 文件对象。

**Vinyl 文件对象**

Gulp 使用 vinyl-fs ，并从 vinyl-fs 继承了 gulp.src() 和 gulp.dest() 方法。而 vinyl-fs 使用了 vinyl 文件对象，即 「虚拟文件格式」。要在 gulp 及其插件中使用常规读取流，需要把读取流转换为 vinyl 文件对象先。
vinyl-source-stream 便是一个转换工具：

```
var source = require('vinyl-source-stream');  
var marked = require('gulp-marked');

fs.createReadStream('*.md')  
  .pipe(source())
  .pipe(marked())
  .pipe(gulp.dest('dist/'));
```
> Vinyl 文件对象的内容可以是以下三种形式：Stream Buffer null

gulp.src() 传输的流是把内容转换成 buffer 的 vinyl 文件对象，因此 gulp 中得到的不是数据块（chunk），而是包含 buffer 的虚拟文件。Vinyl 文件格式有一个 contents 属性来存放文件内容（buffer 或者 stream 或者 null）。

**Gulp 默认使用 buffer**
buffer优点：基于完整的源内容做转换
缺点：对于大文件处理效率低下，因为文件发射回对象流之前必须完整读取
因此 gulp 选择默认使用内容转换成 buffer 的 vinyl 对象流，以方便处理。

>参考： *https://csspod.com/advanced-tips-for-using-gulp-js/*


### 其他插件
gulp-load-plugins *自动加载gulp开头的插件*
gulp-useref gulp-if gulp-htmlmin *html模板&压缩*
gulp-babel gulp-eslint  gulp-uglify *js依赖&检查&压缩*
gulp-sass gulp-autoprefixer gulp-cssnano  *css转换&后缀&美化*
gulp-cache gulp-imagemin *图片缓存&压缩*
gulp-plumber *错误处理*
gulp-sourcemaps *压缩地图*
gulp-size *统计大小&时间*
del *删除文件*
lodash *工具js*
main-bower-files wiredep *bower依赖*
> 参考：yeoman加载器 generator-webapp


  [1]: http://cnodejs.org/topic/56460e0d89b4b49902e7fbd3
  [2]: https://github.com/thejameskyle/babel-handbook/blob/master/translations/zh-Hans/README.md
  [3]: http://www.browsersync.cn/docs/