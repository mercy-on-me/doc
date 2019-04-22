# gulp
==**Gulp**== javascript 构建工具

## 1. 安装

```properties
安装全局 gulp : sudo npm install gulp -g
```

## 2. 创建项目
- 假如项目根路径是 : gulptest

```properties
第一步, 进入项目根据经 : cd gulptest

第二步, 初始化项目 : gulp init   <!--会自动生成 package.json 文件-->

第三步, 在项目内安装 gulp : npm install gulp --save-dev  <!--安装完成后会自动创建 node-modules 文件夹,里边有 gulp 文件夹,是创建任务时必须的-->
    --save-dev : 在 package.json 中添加 Dev 依赖项 gulp

```

### 3. 文件夹结构

```properties
|- app/
      |- css/
      |- fonts/
      |- images/ 
      |- index.html
      |- js/ 
      |- scss/
  |- dist/
  |- gulpfile.js
  |- node_modules/
  |- package.json
```

## 任务

```properties
var gulp = require('gulp")  <!--告诉node 去 node-modules 文件夹中找 gulp 文件夹,将它分配给定义的 gulp-->

gulp.task('task-name', function(){
    ...任务内容
});

任务写完后调用任务 : gulp task-name
```


```properties
gulp.task('task-name', function () {
  return gulp.src('source-files') // Get source files with gulp.src
    .pipe(aGulpPlugin()) // Sends it through a gulp plugin
    .pipe(gulp.dest('destination')) // Outputs the file in the destination folder
})

解析 : 一个真正的任务需要两个额外的gulp方法 - gulp.src和gulp.dest。
    gulp.src告诉Gulp任务用于任务的文件，
    gulp.dest告诉Gulp完成任务后输出文件的位置。

```
