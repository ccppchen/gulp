# What is gulp?
　　`gulp` 是前端开发过程中一种基于流的代码构建工具，是自动化项目的构建利器；她不仅能对网站资源进行优化，而且在开发过程中很多重复的任务能够使用正确的工具自动完成；使用她，不仅可以很愉快的编写代码，而且大大提高我们的工作效率。


　　`gulp`是基于`Nodejs`的自动任务运行器， 她能自动化地完成` javascript、coffee、sass、less、html/image、css` 等文件的测试、检查、合并、压缩、格式化、浏览器自动刷新、部署文件生成，并监听文件在改动后重复指定的这些步骤。在实现上，她借鉴了Unix操作系统的管道（pipe）思想，前一级的输出，直接变成后一级的输入，使得在操作上非常简单。
　　
![gulp-plugin](http://ccppchen.github.io/images/gulp_plugin.png)

# 安装
首先确保你已经正确安装了nodejs环境。然后以全局方式安装gulp：
```
npm install -g gulp
```

全局安装gulp后，还需要在每个要使用gulp的项目中都单独安装一次。把目录切换到你的项目文件夹中，然后在命令行中执行：
```sh
npm install gulp
```

如果想在安装的时候把gulp写进项目package.json文件的依赖中，则可以加上--save-dev：
```sh
npm install --save-dev gulp
```
**这样就完成了gulp的安装，接下来就可以在项目中应用gulp了。继续学习 **

## gulpfile.js

```js
'use strict';
var gulp        = require('gulp');
var plugins     = require('gulp-load-plugins')();
var pkg         = require("./package.json");
var browserSync = require('browser-sync').create();
var jshint      = require('gulp-jshint');
var png         = require('imagemin-pngquant');
var clean       = require('gulp-clean');


var yeoman = {
  app: "app",
  dist: "dist"
}

var banner = 
"/** \n\
* jQuery WeUI V" + pkg.version + " \n\
* By 野草\n\
* http://ccppchen.github.io/jquery-weui/\n \
*/\n";

// 监测文件改动并自动刷新
gulp.task('server', ['compass'], function(){
  browserSync.init({
       server: {
           baseDir: yeoman.app
       },
       port: 8000
   });
});

gulp.task('watch', function(){
  gulp.watch("sass/**/*.scss", ['compass']);
  gulp.watch([yeoman.app+'/*.html', yeoman.app+'/styles/**/*.css', yeoman.app+'/images/**/*']).on('change', browserSync.reload);
});

// 编译sass
gulp.task('compass', function() {
  return gulp.src("sass/**/*.scss")
    .pipe(plugins.compass({
      config_file: './config.rb',
      css: yeoman.app+'/styles',
      sass: 'sass',
      image: yeoman.app+'/images',
      style: 'expanded',
      comments: true
    }))
    .pipe(plugins.autoprefixer({
      browsers: [ '> 1%', 'Last 5 versions' ]
    }));
});

gulp.task('compass-pro', function() {
  return gulp.src("sass/**/*.scss")
    .pipe(plugins.compass({
      css: yeoman.dist+'/styles',
      sass: 'sass',
      image: yeoman.app+'/images',
      style: 'compressed',
      comments: true
    }))
    .pipe(plugins.autoprefixer({
      browsers: [ '> 1%', 'Last 5 versions' ]
    }))
    .pipe(plugins.header(banner))
    .pipe(gulp.dest(yeoman.dist+'/styles'));
});

// js合并压缩混淆
gulp.task('js', function(){
  gulp.src([
      "app/scripts/app.js"
    ])
    .pipe(jshint())
    .pipe(jshint.reporter())
    .pipe(plugins.concat({ path: 'main.js'}))
    .pipe(plugins.uglify())
    .pipe(plugins.extReplace('.min.js'))
    .pipe(plugins.header(banner))
    .pipe(gulp.dest(yeoman.dist+'/scripts'))
});

// html压缩
gulp.task('html', function () {
    gulp.src(yeoman.app+'/**/*.html')
    .pipe(plugins.htmlmin({
      collapseWhitespace: true
    }))
    .pipe(gulp.dest(yeoman.dist));
});

// image压缩
gulp.task('images', function () {
    gulp.src(yeoman.app+'/images/**/*')
    .pipe(plugins.imagemin({
        progressive: true,
        use: [png()]
    }))
    .pipe(gulp.dest(yeoman.dist+'/images'));
});

// 删除dist
gulp.task('clean', function(){
  gulp.src('./'+yeoman.dist)
  .pipe(clean({force: true}));
});


gulp.task('default', ['compass', 'watch', 'server']);
gulp.task('build', ['clean', 'compass-pro', 'js', 'html', 'images', 'server']);



```
# gulp的使用

1. 建立gulpfile.js文件

`gulp`也需要一个文件作为它的主文件，在`gulp`中这个文件叫做`gulpfile.js`。新建一个文件名为gulpfile.js的文件，然后放到你的项目目录中。之后要做的事情就是在`gulpfile.js`文件中定义我们的任务了。下面是一个最简单的`gulpfile.js`文件内容示例，它定义了一个默认的任务。

```js
var gulp = require('gulp');
gulp.task('default',function(){
    console.log('hello world');
});
```
2.运行`gulp`任务 

要运行gulp任务，只需切换到存放gulpfile.js文件的目录(windows平台请使用cmd或者Power Shell等命令行工具)，然后在命令行中执行gulp命令就行了，gulp后面可以加上要执行的任务名，例如`gulp task1`，如果没有指定任务名，则会执行任务名为default的默认任务。

# gulp API
1. gulp.src(['src1','src2'])  获取到想要处理的文件流
2. gulp.task('server',function(){ ... } ) 或者gulp.task('server', ['task1','task2']) 注册任务
3. gulp.dest('dest') 把流中的内容写入到文件中
4. gulp.watch('src', ['task1','task2']) 实时监测文件变动
5. gulp.run('task1','task2','task3')  运行任务，**并行运行**
6. gulp.pipe(minify()) 导入到`gulp`的插件中

# js文件压缩
`gulp-uglify`插件用来压缩js文件。
```js
var gulp = require('gulp'),
    uglify = require("gulp-uglify");
 
gulp.task('minify-js', function () {
    gulp.src('src/*.js')          // 要压缩的js文件
    .pipe(uglify())              //使用uglify进行压缩
    .pipe(gulp.dest('dist/js')); //压缩后的路径
});
```
# css文件压缩
`gulp-minify-css`插件用来压缩css文件。
```js
var gulp = require('gulp'),
    minifyCss = require("gulp-minify-css");
 
gulp.task('minify-css', function () {
    gulp.src('src/*.css') // 要压缩的css文件
    .pipe(minifyCss())    //压缩css
    .pipe(gulp.dest('dist/css'));
});
```
# html文件压缩
`gulp-minify-html`插件用来压缩html文件。
```js
var gulp = require('gulp'),
    minifyHtml = require("gulp-minify-html");
 
gulp.task('minify-html', function () {
    gulp.src('src/*.html') // 要压缩的html文件
    .pipe(minifyHtml())    //压缩
    .pipe(gulp.dest('dist/html'));
});
```
# js代码检查
`gulp-jshint`插件，用来检查js代码。
```js
var gulp = require('gulp'),
    jshint = require("gulp-jshint");
 
gulp.task('jsLint', function () {
    gulp.src('src/*.js')
    .pipe(jshint())
    .pipe(jshint.reporter()); // 输出检查结果
});
```
# js代码压缩混淆
`gulp-uglify`
```js
gulp.task('uglify', ["js"], function() {
  return gulp.src(['./dist/js/*.js', '!./dist/js/*.min.js'])
    .pipe(uglify({
      preserveComments: "license"
    }))
    .pipe(ext_replace('.min.js'))
    .pipe(gulp.dest('./dist/js'));
});
```
# 文件合并
`gulp-concat`插件，用来把多个文件合并为一个文件,我们可以用它来合并js或css文件等。
```js
var gulp = require('gulp'),
    concat = require("gulp-concat");
    
gulp.task('concat', function () {
    gulp.src('src/*.js')     //要合并的文件
    .pipe(concat('all.js'))  // 合并匹配到的js文件并命名为 "all.js"
    .pipe(gulp.dest('dist/js'));
});
```

# 图片压缩
`gulp-imagemin`插件来压缩jpg、png、gif等图片。
```js
var gulp = require('gulp');
var imagemin = require('gulp-imagemin');
var pngquant = require('imagemin-pngquant'); //png图片压缩插件

gulp.task('default', function () {
    gulp.src('src/images/*')
    .pipe(imagemin({
        progressive: true,
        use: [pngquant()] //使用pngquant来压缩png图片
    }))
    .pipe(gulp.dest('dist'));
});
```
# 自动刷新
`gulp-livereload`插件，当代码变化时，它可以帮我们自动刷新页面。
```js
var gulp = require('gulp'),
    less = require('gulp-less'),
    livereload = require('gulp-livereload');
    
gulp.task('less', function() {
  gulp.src('less/*.less')
    .pipe(less())
    .pipe(gulp.dest('css'))
    .pipe(livereload());
});
gulp.task('watch', function() {
  livereload.listen(); //要在这里调用listen()方法
  gulp.watch('less/*.less', ['less']);
});
```
# 启服务
`gulp-connect`这个插件是启一个web服务器
```js
gulp.task('server', function () {
  connect.server();
});
```
# 重命名
`gulp-rename`插件，给文件重新命名
```js
gulp.task('minifyjs', function() {
    return gulp.src('src/*.js')
        .pipe(concat('main.js'))                  //合并所有js到main.js
        .pipe(gulp.dest('minified/js'))           //输出main.js到文件夹
        .pipe(rename({suffix: '.min'}))           //rename压缩后的文件名
        .pipe(uglify())                           //压缩
        .pipe(gulp.dest('minified/js'));          //输出
});
```
# 改变文件的扩展名
`gulp-ext-replace`这个插件可以改变文件扩展名
```js
gulp.task('change', function() {
  gulp.src('styles/*.css')
      .pipe(ext_replace('.min.css'))
      .pipe(gulp.dest('dist'))
});
```

# 删除
`del`插件，删除文件
```js
gulp.task('clean', function(cb) {
    del(['minified/css', 'minified/js'], cb)
});
```
# 自动加载gulp插件
`gulp-load-plugins`这个插件能自动帮你加载`package.json`文件里的gulp插件

package.json
```json
{
  "devDependencies": {
    "gulp": "~3.6.0",
    "gulp-rename": "~1.2.0",
    "gulp-ruby-sass": "~0.4.3",
    "gulp-load-plugins": "~0.5.1"
  }
}
```

gulpfile.js

```js
var gulp = require('gulp');
//加载gulp-load-plugins插件，并马上运行它
var plugins = require('gulp-load-plugins')();

plugins.rename  
plugins.rubySass
```

# 文件头部注释
`gulp-header` 这个插件是给文件头部加上自定义注释内容的

```js
var header = require('gulp-header');
gulp.task('less', function () {
  return gulp.src(['./src/less/jquery-weui.less'])
  .pipe(less())
  .pipe(autoprefixer())
  .pipe(header(banner)) //加上头部注释
  .pipe(gulp.dest('./dist/css/'));
});
```

# 自动加上css3浏览器前缀
`gulp-autoprefixer`这个插件会自动加上css3浏览器前缀
```js
var header = require('gulp-header');
gulp.task('less', function () {
  return gulp.src(['./src/less/jquery-weui.less'])
  .pipe(less())
  .pipe(autoprefixer()) //自动加上浏览器前缀
  .pipe(header(banner))
  .pipe(gulp.dest('./dist/css/'));
});
```
# sass
`gulp-sass`是自动编译sass的插件
```js
var gulp        = require('gulp');
var browserSync = require('browser-sync').create();
var sass        = require('gulp-sass');

// Static Server + watching scss/html files
gulp.task('serve', ['sass'], function() {

    browserSync.init({
        server: "./app"
    });

    gulp.watch("app/scss/*.scss", ['sass']);
    gulp.watch("app/*.html").on('change', browserSync.reload);
});

// Compile sass into CSS & auto-inject into browsers
gulp.task('sass', function() {
    return gulp.src("app/scss/*.scss")
        .pipe(sass())
        .pipe(gulp.dest("app/css"))
        .pipe(browserSync.stream());
});

gulp.task('default', ['serve']);
```

# compass
`gulp-compass`
```js
var compass = require('gulp-compass');
 
gulp.task('compass', function() {
  gulp.src('./src/*.scss')
    .pipe(compass({
      config_file: './config.rb',
      css: 'stylesheets',
      sass: 'sass'
    }))
    .pipe(gulp.dest('app/assets/temp'));
});
```
# 浏览器自动刷新
`browser-sync` 浏览器自动刷新
```js
var gulp        = require('gulp');
var browserSync = require('browser-sync').create();
var reload      = browserSync.reload;

// Save a reference to the `reload` method

// Watch scss AND html files, doing different things with each.
gulp.task('serve', function () {

    // Serve files from the root of this project
    browserSync.init({
        server: {
            baseDir: "./"
        }
    });

    gulp.watch("*.html").on("change", reload);
});
```

```sh
npm install --save-dev gulp gulp-clean-css gulp-minify-html gulp-jshint gulp-uglify gulp-concat gulp-imagemin gulp-rename gulp-load-plugins gulp-header gulp-autoprefixer gulp-compass browser-sync
```
