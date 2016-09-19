## gulp-handlebars-precompile

提取html文件中的handerbars模板文件，预编译到指定目录，并且可以在html文件中建立script标签

## Installation

```bash
npm install gulp -g
npm install gulp-handlebars-precompile --save-dev
```

## Usage

### 基本用法 ###

文件目录结构如下：

```
├── gulpfile.js              # gulpfile文件
├── src/                     # 源文件目录
	└── index.html        	 # index.html
    └── tpl/           		 # 模板目录
        └── product.hbs  		 # 模板文件
└── dev/                     # 编译目录
```

html文件，代码如下

```html
	<!--hbs "src/tpl/product.hbs"-->
```

在根目录创建gulpfile.js，代码如下

```js
var gulp = require('gulp');
var precompile = require('gulp-handlebars-precompile');

gulp.task('dev_tpl', function(){
    return gulp.src('src/*.html')
       .pipe(precompile({
              reg: /<!\-\-hbs\s+"([^"]+)"\-\->/g,
              dest: "dev/tpl/"
        }))
       .pipe(gulp.dest('dev/'))
});

```

执行

```bash
	gulp dev_tpl
```

task结束后会在dev/tpl/建立以html文件名建立的文件夹，里面有预编译的js文件,即：

```
└── dev/                     # 编译目录
	└── tpl/
		└── index/
			└── product.js
	└── index.html        	 # index.html
```

### 其他用法： ###

#### 设置baseSrc ####

通过设置baseSrc,设置模板的根目录

html文件：

```html
	<!--hbs "product.hbs"-->
```

gulpfile文件:

```js
var gulp = require('gulp');
var precompile = require('gulp-handlebars-precompile');

gulp.task('dev_tpl', function(){
    return gulp.src('src/*.html')
       .pipe(precompile({
              reg: /<!\-\-hbs\s+"([^"]+)"\-\->/g,
              baseSrc: "src/tpl/",
              dest: "dev/tpl/"
        }))
       .pipe(gulp.dest('dev/'))
});
```

效果与基本用法的效果相同

#### 在html文件中建立script链接 ####

设置scriptSrc, 会在html文件中自动建立script标签指向设置的地址

html文件：

```html
	<!--hbs "product.hbs"-->
```

gulpfile文件:

```js
var gulp = require('gulp');
var precompile = require('gulp-handlebars-precompile');

gulp.task('dev_tpl', function(){
    return gulp.src('src/*.html')
       .pipe(precompile({
              reg: /<!\-\-hbs\s+"([^"]+)"\-\->/g,
              baseSrc: "src/tpl/",
              dest: "dev/tpl/",
              scriptSrc: 'tpl/'
        }))
       .pipe(gulp.dest('dev/'))
});
```

执行

```bash
	gulp dev_tpl
```

在dev/index.html中

原来的

```html
	<!--hbs "product.hbs"-->
```

会被替换为

```html
	<script src="tpl/index/product.js"></script>
```
#### 配合gulp-inline-source实现js内联效果 ####

html文件：

```html
	<!--hbs "product.hbs"-->
```

gulpfile文件:

```js
var gulp = require('gulp');
var precompile = require('gulp-handlebars-precompile');

gulp.task('dev_tpl', function(){
    return gulp.src('src/*.html')
       .pipe(precompile({
              reg: /<!\-\-hbs\s+"([^"]+)"\-\->/g,
              baseSrc: "src/tpl/",
              dest: "dev/tpl/",
              scriptSrc: 'tpl/',
              inline: true
        }))
       .pipe(gulp.dest('dev/'))
});
```

执行

```bash
	gulp dev_tpl
```

在dev/index.html中

原来的

```html
	<!--hbs "product.hbs"-->
```

会被替换为

```html
	<script src="tpl/index/product.js" inline></script>
```

然后可以用gulp-inline-source实现内联效果

#### 设置命名空间 ####

默认的命名空间为template，设置namespace修改命名空间

html文件：

```html
	<!--hbs "product.hbs"-->
```

gulpfile文件:

```js
var gulp = require('gulp');
var precompile = require('gulp-handlebars-precompile');

gulp.task('dev_tpl', function(){
    return gulp.src('src/*.html')
       .pipe(precompile({
              reg: /<!\-\-hbs\s+"([^"]+)"\-\->/g,
              baseSrc: "src/tpl/",
              dest: "dev/tpl/",
              scriptSrc: 'tpl/',
              inline: true,
              namespace: 'kevin'
        }))
       .pipe(gulp.dest('dev/'))
});
```

tpl.hbs文件为：
```html
{{#each data}}
	<li href="{{itemLink}}">
	    {{test}}
	</li>
{{/each}}
```

会被编译为：

```js
this["kevin"] = this["kevin"] || {};
this["kevin"]["product"] = Handlebars.template({"1":function(depth0,helpers,partials,data) {
    var helper, alias1=helpers.helperMissing, alias2="function", alias3=this.escapeExpression;

  return "<li href=\""
    + alias3(((helper = (helper = helpers.itemLink || (depth0 != null ? depth0.itemLink : depth0)) != null ? helper : alias1),(typeof helper === alias2 ? helper.call(depth0,{"name":"itemLink","hash":{},"data":data}) : helper)))
    + "\">\n    "
    + alias3(((helper = (helper = helpers.test || (depth0 != null ? depth0.test : depth0)) != null ? helper : alias1),(typeof helper === alias2 ? helper.call(depth0,{"name":"test","hash":{},"data":data}) : helper)))
    + "\n</li>\n";
},"compiler":[6,">= 2.0.0-beta.1"],"main":function(depth0,helpers,partials,data) {
    var stack1;

  return ((stack1 = helpers.each.call(depth0,(depth0 != null ? depth0.data : depth0),{"name":"each","hash":{},"fn":this.program(1, data, 0),"inverse":this.noop,"data":data})) != null ? stack1 : "");
},"useData":true});
```

#### 如何使用预编译的模板 ####

js中使用预编译的模板

```js
var html = kevin.product(data);
```