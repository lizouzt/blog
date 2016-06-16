---
layout:     post
title:      "Gulp + Webpack + AMD + NG1 + REACT + BLADE的开发、编译、部署环境搭建"
subtitle:   "有点儿麻烦，但是就这样了"
date:       2016-06-15 12:00:00
author:     "Start Bootstrap"
header-img: "img/post-bg-01.jpg"
---

## 综述

虽然FIS干这个比较屌，但是还是选择了攒一套，有点儿简陋，但是慢慢丰富吧。

主要实现这么几个功能：

* 能够支持多模块开发，每个模块基于`AMD`进行依赖管理
* 能够支持模块内的自由打包
* 支持模块之间互访静态资源
* 支持本地调试Blade模板
* 支持自动化部署测试环境以及同步CDN（待开发）

接下来分别介绍`目录结构`，`common模块配置`，`普通模块配置`，`Blade模板和本地Server调试`。


## 目录结构

首先，配置基础开发环境，后续我们称之为`base`，私有repos在[这里](https://github.com/yunlaiwu/base)，这里介绍下是啥样的：

```
.
+-- cache
+-- modules
|   +-- module_a
|   +-- module_b
+-- node_modules
+-- pages
+-- composer.json
+-- config.php
+-- gulpfile.js
+-- helpers.php
+-- index.php
+-- package.json
```

其中`cache`、`pages`、`composer.json`、`config.php`、`helpers.php`、`index.php`是和本地Blade模板调试相关的。

`gulpfile.js`用于配置Gulp的编译脚本，由于我们的amd的config文件是所有模块共用的，所以所有模块共享这一个编译文件（这么干是为了省事儿），当然，针对不同模块有不同的task。

如下是我们的`gulpfile.js`的样子，包含对common和demo模块的编译：

```javascript

var gulp = require('gulp');
var amdOptimize = require('amd-optimize');
var concat = require('gulp-concat');
var sourcemap = require('gulp-sourcemaps');
var rev = require('gulp-rev');
var watch = require('gulp-watch');
var runSequence = require('gulp-run-sequence');
var clean = require('gulp-clean');
var fs = require('fs');
var less = require('gulp-less');
var csso = require('gulp-csso');
var uglify = require('gulp-uglify');
var eventStream = require("event-stream");
var order = require("gulp-order");
var ngAnnotate = require('gulp-ng-annotate');

var requireConfig = {
	baseUrl: __dirname,
	paths: {
		'jQuery': 'node_modules/jquery/dist/jquery',
		'angular': 'node_modules/angular/angular',
		'angular-route': 'node_modules/angular-route/angular-route'
	},
	findNestedDependencies: true,
	shim: {
		'angular': {
			exports: 'angular'
		},
		'jQuery': {
			exports: 'jQuery'
		},
		'angular-route': {
			deps: ['angular']
		}
	}
};

var deepClone = function(obj) {
	return JSON.parse(JSON.stringify(obj));
};

var getSrcPath = function(mod) {
	return 'modules/' + mod + '/public/' + mod;
};

var COMMON_LIB = ['angular', 'jQuery', 'angular-route'];


/********************* for common ****************************/


var MODULE_NAME_COMMON = 'common';

gulp.task('common-clean', function() {
	gulp.src(getSrcPath(MODULE_NAME_COMMON) + '/dist')
		.pipe(clean());
});

function commonBuild() {
	runSequence('common-clean', 'common-package-lib');
}

gulp.task('common-package-lib', function() {

	var pack = 'lib';
	var srcPath = getSrcPath(MODULE_NAME_COMMON);

	return eventStream.merge(
			gulp.src("node_modules/almond/almond.js"),
			gulp.src(srcPath + '/src/ylw.js'),
			gulp.src([
				srcPath + '/src/' + pack + '/**/*.js'
			])
			.pipe(amdOptimize(srcPath + '/src/' + pack + '/main', requireConfig))
			.pipe(concat('index.js'))
		)
		.pipe(order(["**/almond.js", "**/ylw.js", "**/index.js"]))
		.pipe(concat(pack + '.js'))
		.pipe(ngAnnotate())
		.pipe(rev())
		.pipe(gulp.dest(srcPath + '/dist'))
		.pipe(rev.manifest(srcPath + '/dist/manifest.json', {
			base: __dirname + '/' + srcPath + '/dist',
			merge: true
		}))
		.pipe(gulp.dest(srcPath + '/dist'));
});

gulp.task('common', function() {
	watch([getSrcPath(MODULE_NAME_COMMON) + '/src/**'], commonBuild);
});

gulp.task('common-build', commonBuild);


/********************* for demo ****************************/

var MODULE_NAME_DEMO = 'demo';

function demoBuild() {
	runSequence('demo-clean', 'demo-package-main', 'demo-package-main-css');
}
gulp.task('demo-clean', function() {
	gulp.src(getSrcPath(MODULE_NAME_DEMO) + '/dist')
		.pipe(clean());
});

gulp.task('demo-package-main', function() {

	var pack = 'main';
	var srcPath = getSrcPath(MODULE_NAME_DEMO);
	var rConfig = deepClone(requireConfig);
	//very important
	rConfig.exclude = COMMON_LIB;

	return gulp.src([
			srcPath + '/src/' + pack + '/**/*.js'
		])
		.pipe(amdOptimize(srcPath + '/src/' + pack + '/js/app', rConfig))
		.pipe(concat(pack + '.js'))
		.pipe(ngAnnotate())
		.pipe(rev())
		.pipe(gulp.dest(srcPath + '/dist'))
		.pipe(rev.manifest(srcPath + '/dist/manifest.json', {
			base: __dirname + '/' + srcPath + '/dist',
			merge: true
		}))
		.pipe(gulp.dest(srcPath + '/dist'));
});

gulp.task('demo-package-main-css', function() {
	var pack = 'main';
	var srcPath = getSrcPath(MODULE_NAME_DEMO);
	return gulp.src([
			srcPath + '/src/' + pack + '/**/*.css'
		])
		.pipe(less())
		.pipe(csso())
		.pipe(concat(pack + '.css'))
		.pipe(rev())
		.pipe(gulp.dest(srcPath + '/dist'))
		.pipe(rev.manifest(srcPath + '/dist/manifest.json', {
			base: __dirname + '/' + srcPath + '/dist',
			merge: true
		}))
		.pipe(gulp.dest(srcPath + '/dist'));
});

gulp.task('demo', function() {
	watch([getSrcPath(MODULE_NAME_DEMO) + '/src/**'], demoBuild);
});

gulp.task('demo-build', demoBuild);
```

`modules`目录用于托管各个模块，每个模块应该是一个独立的repos，modules内部要添加`.gitignore`防止内部模块被提交：

```
*
!.gitignore
```

下面先来介绍下common模块的配置。

## common模块配置

common模块是一个站点中通用js和css的承载地，它需要包含一个或者多个脚本或者样式的package来供其他模块选择性加载。

我们的工程使用了NG1，所以配置文件将NG和JQuery的相关库都shim出来：

```javascript
var requireConfig = {
	baseUrl: __dirname,
	paths: {
		'jQuery': 'node_modules/jquery/dist/jquery',
		'angular': 'node_modules/angular/angular',
		'angular-route': 'node_modules/angular-route/angular-route'
	},
	findNestedDependencies: true,
	shim: {
		'angular': {
			exports: 'angular'
		},
		'jQuery': {
			exports: 'jQuery'
		},
		'angular-route': {
			deps: ['angular']
		}
	}
};
```

这里注意，我们的baseUrl是基于base的根目录的，每个模块也都基于这个目录补全模块的path。

我们将这些重要的库都放在common的名为lib的package中，方便别的模块使用，如下这样编译出该package：

```javascript
gulp.task('common-package-lib', function() {

	var pack = 'lib';
	var srcPath = getSrcPath(MODULE_NAME_COMMON);

	return eventStream.merge(
			gulp.src("node_modules/almond/almond.js"),
			gulp.src(srcPath + '/src/ylw.js'),
			gulp.src([
				srcPath + '/src/' + pack + '/**/*.js'
			])
			.pipe(amdOptimize(srcPath + '/src/' + pack + '/main', requireConfig))
			.pipe(concat('index.js'))
		)
		.pipe(order(["**/almond.js", "**/ylw.js", "**/index.js"]))
		.pipe(concat(pack + '.js'))
		.pipe(ngAnnotate())
		.pipe(rev())
		.pipe(gulp.dest(srcPath + '/dist'))
		.pipe(rev.manifest(srcPath + '/dist/manifest.json', {
			base: __dirname + '/' + srcPath + '/dist',
			merge: true
		}))
		.pipe(gulp.dest(srcPath + '/dist'));
});
```

通过eventStream来合并不同的文件流，并且用`gulp-order`来管理他们的合并顺序，这里一共有三个文件流：

**almond.js**，这个是用来做AMD的shim的，暂且不表；
**ylw.js**，这里边就只有一个跟模块加载有用的方法：

```javascript
window.YLW = {
	use: function(module, pack, resource, cb) {
		var path = 'modules/' + module + '/public/' + module + '/src/' + pack + '/' + resource;
		return require(path, cb);
	}
};
```

他提供一个全局的use方法，根据模块名，包名，和资源路径（一般是包的入口文件的路径）来加载一个包或者包内的部分资源。任何模块都可以使用这个方法加载其他模块的资源包内的资源，但前提是需要在模板中加载对应的资源包（Todo，动态包加载）。

**AMD文件**，通过AMD依赖管理的一系列文件。

接下来，会合并文件，并且用`gulp-rev`追加版本号并且产生`manifest`文件。

你可能奇怪，为啥manifest里的路径设置的那么奇葩，请参看[github上的讨论](https://github.com/sindresorhus/gulp-rev/issues/83)。

## 普通模块配置

所有的模块的Gulp里的task的名称都需要以该模块的名称开头，比如`common`、`demo`。

每个task都针对一个package，目前package只用来打js和css的包。

如下是一个普通模块的名为main的package的编译task：

```javascript
gulp.task('demo-package-main', function() {

	var pack = 'main';
	var srcPath = getSrcPath(MODULE_NAME_DEMO);
	var rConfig = deepClone(requireConfig);
	//very important
	rConfig.exclude = COMMON_LIB;

	return gulp.src([
			srcPath + '/src/' + pack + '/**/*.js'
		])
		.pipe(amdOptimize(srcPath + '/src/' + pack + '/js/app', rConfig))
		.pipe(concat(pack + '.js'))
		.pipe(ngAnnotate())
		.pipe(rev())
		.pipe(gulp.dest(srcPath + '/dist'))
		.pipe(rev.manifest(srcPath + '/dist/manifest.json', {
			base: __dirname + '/' + srcPath + '/dist',
			merge: true
		}))
		.pipe(gulp.dest(srcPath + '/dist'));
});
```

这里要强调的是，**务必`exclude`掉common中的AMD模块**，如这里用`COMMON_LIB`来指示剔除对NG和JQuery相关代码的包含。

## Blade模板和本地Server调试

针对Blade模板调试，希望有一套不依赖Laravel的本地调试环境。所以使用`philo/laravel-blade`，最新的`3.*`版本支持5以上的框架。

所以，在`composer.json`中配置如下：

```json
{
	"require": {
		"philo/laravel-blade": "3.*"
	}
}
```

如此，通过执行`composer install`，来安装该库，当然，你得先安装composer，详情见[官方文档](https://getcomposer.org/download/)，OSX用户直接执行如下命令即可：

```bash
# setup your develop environment

php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('SHA384', 'composer-setup.php') === '070854512ef404f16bac87071a6db9fd9721da1684cd4589b1196c3faf71b9a2682e2311b36a5079825e155ac7ce150d') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
mv composer.phar /usr/local/bin/composer
```

接下来我们需要一个webserver，好在php5.4以后为我们提供了一个内置server，通过如下命令启动我们的server：

```
php -S localhost:8000 index.php
```

注意，我们这里使用了指定php文件的方式启动内置服务器，而非通过目录方式启动，这是因为我们要灵活控制静态资源的响应，具体可参看[这篇文档](http://php.net/manual/zh/features.commandline.webserver.php)。

我们的`index.php`内容如下所示：

```php
require __DIR__ . '/config.php';
require __DIR__ . '/helpers.php';
require __DIR__ . '/vendor/autoload.php';

if (preg_match('/\.(?:png|jpg|jpeg|gif|js|css)$/', $_SERVER["REQUEST_URI"])) {

	if(preg_match('/modules/', $_SERVER["REQUEST_URI"])) {
		return false;
	}


	$path = substr($_SERVER["REQUEST_URI"], 1);
	$mod = substr($path, 0, strpos($path, '/'));

	header("Location: http://$_SERVER[HTTP_HOST]/modules/$mod/public/$path");
}


function public_path($path) {

	$mod = substr($path, 0, strpos($path, '/'));
	return __DIR__ . '/modules/' . $mod . '/public/' . $path;
}

use Philo\Blade\Blade;

$debugModules = ['demo', 'common'];


//Init Blade renderer
$paths = array();

foreach ($debugModules as $mod) {
	array_push($paths, __DIR__ . "/modules/$mod/resources/views");
}

$renderer = new Blade($paths, __DIR__ . '/cache');

//Load page based on uri
$url = "http://$_SERVER[HTTP_HOST]$_SERVER[REQUEST_URI]";
$path = substr($url, strrpos($url, '/') + 1);
if (empty($path)) {
	$path = "index";
} 

require __DIR__ . '/pages/' . $path . '.php';

//Render page
$page = new Page();
echo $page->show($renderer);
```

`config.php`为我们提供了一些全局配置，所有配置性的东西可以放在这里：

```php
const BASE_URL = 'http://localhost';
```

`helpers.php`文件主要提供模板的工具方法，这里用来加载我们的资源包，内容如下所示：

```php

if (! function_exists('gulp')) {
    function gulp($mod, $resource)
    {
        $manifest = json_decode(file_get_contents(public_path("$mod/dist/manifest.json")), true);

        if (isset($manifest[$resource])) {
            return "$mod/dist/" . $manifest[$resource];
        }
        throw new InvalidArgumentException("File {$mod} {$resource} not defined in rev manifest.");
    }
}

```

非常清晰的可以看出来，他提供一个名为`gulp`的辅助方法，用来根据模块名`$mod`和资源名`$resource`来定位manifest文件并且加载包资源。

除此之外，我们还对静态资源做了处理，将线上的静态资源相对路径替换为本地的，这里进行了重定向。

`public_path`是对我们框架中的同名方法进行重写，好让我们的`gulp`工具方法来定位到正确的manifest文件。

然后，我们告诉webserver，模板文件的存放地址，这里支持传入数组，制定多个模板文件的位置。

最后，我们通过对请求的url分析，来加载对应的页面。

这里，我们的页面存放于`/pages`目录中，这里举个页面的例子：

```php
class Page
{
	function show($renderer)
	{
		//You should mock your data here
		return $renderer->view()->make('demo', ['foo' => 'bar'])->render();
	}
}
```

这里的`make`调用可以指定要加载的template和给定Mock数据，这里很重要，基本上我们的服务器给定的数据都可以在这里模拟出来。

## 大功告成

总结下我们用到的命令：

* 编译某个模块

```shell
gulp {module}-build
```

* watch并编译模块

```shell
gulp {module}
```

* 启动本地测试服务器

```shell
php -S localhost:8000 index.php
```

Todo：环境部署脚本 + 上线脚本



