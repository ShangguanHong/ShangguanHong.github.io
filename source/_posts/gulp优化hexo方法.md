---
title: gulp优化hexo方法
copyright: false
date: 2019-08-18 00:45:17
categories: Hexo学习笔记
tags:
- Hexo
---

本文转载自：[gulp优化hexo方法](https://www.cnblogs.com/Mayfly-nymph/p/10623234.html)



`gulp` 通过对站点使用的静态资源进行压缩，来优化网站的访问速度。 

首先安装 `gulp` 以及所需要的模块：

```shell
npm install gulp -g
```

```shell
npm install gulp-htmlclean gulp-htmlmin gulp-minify-css gulp-uglify gulp-imagemin --save
```

然后在博客的根目录下创建 **gulpfile.js** 文件写入代码：

```js
var gulp = require('gulp');
var minifycss = require('gulp-minify-css');
var uglify = require('gulp-uglify');
var htmlmin = require('gulp-htmlmin');
var htmlclean = require('gulp-htmlclean');
var imagemin = require('gulp-imagemin');

// 压缩html
gulp.task('minify-html', function() {
    return gulp.src('./public/**/*.html')
        .pipe(htmlclean())
        .pipe(htmlmin({
            removeComments: true,
            minifyJS: true,
            minifyCSS: true,
            minifyURLs: true,
        }))
        .pipe(gulp.dest('./public'))
});
// 压缩css
gulp.task('minify-css', function() {
    return gulp.src('./public/**/*.css')
        .pipe(minifycss({
            compatibility: 'ie8'
        }))
        .pipe(gulp.dest('./public'));
});
// 压缩js
gulp.task('minify-js', function() {
    return gulp.src('./public/js/**/*.js')
        .pipe(uglify())
        .pipe(gulp.dest('./public'));
});
// 压缩图片
gulp.task('minify-images', function() {
    return gulp.src('./public/images/**/*.*')
        .pipe(imagemin(
        [imagemin.gifsicle({'optimizationLevel': 3}),
        imagemin.jpegtran({'progressive': true}),
        imagemin.optipng({'optimizationLevel': 7}),
        imagemin.svgo()],
        {'verbose': true}))
        .pipe(gulp.dest('./public/images'))
});
// 默认任务
gulp.task('default', gulp.parallel(
    'minify-html','minify-css','minify-js','minify-images'
));
```

最后执行命令，上传 GitHub

```shell
hexo clean && hexo g && gulp && hexo d
```

