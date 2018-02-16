---
title: 从零开始学Laravel
tags:
  - PHP
  - Laravel
categories: PHP学习心得&备忘
abbrlink: 19653b91
date: 2018-02-13 16:00:00
---

# Laravel5.5的安装
安装好WAMP环境后，在wampmanager.ini文件中将PHP版本更改为php7，查看php版本可以通过phpinfo()函数查看。
,下载好后解压至Apache工作目录下。启动wamp服务后，正常情况下访问localhost/laravel/public就能显示Laravel的欢迎界面。

附上Laravel一键安装包
[下载地址](http://laravelacademy.org/resources-download)

# PhpStrom安装
编程还是离不开JB全家桶的~这步没有省略主要是因为jb全家桶更新3.4后大量激活方法和激活服务器集体失效。这里使用了[ilanyu](http://blog.lanyus.com/)大佬提供的[本地反向代理激活方法](https://github.com/ilanyu/ReverseProxy)。不过有条件还是要支持正版~
<!-- more -->
# Laravel的路由
Laravel5.5版本中把路由的routes文件拿出来单独建立了一个routes文件。这里感觉和django的urls.py文件很类似，也是起到了Controller的作用，值得一提的是必须制定http请求类型
## 路由选项
```php
// 基础路由
Route::get('/', function () {
    return view('welcome');
});

Route::post('test', function(){
    return 'hello test';
});

//多请求路由
Route::match(['get', 'post'], 'test2', function(){
    return 'hello test2';
});

Route::any('test3', function (){
    return 'hello test3';
});
```
## 路由参数
这里虽然写起来比django的要复杂，但是个人感觉比urls.py中的逻辑要清楚。
```php
//路由参数
Route::get('user/{id}', function ($id){
    return 'id '. $id;
});

Route::get('user/{name?}', function ($name = null){
    return 'name '. $name;
});

Route::get('user/{id}/{name?}', function ($id, $name){
    return 'id '. $id. ' '. 'name '. $name; 
})->where(['id' => '[0-9]+', 'name' => '[A-Za-z]+']);
```

## 路由命名
命名的好处是可以直接通过命名之后的路由进行重定向，带参数的路由可以指定初始值，相当于给这个url一个名字,可以直接使用route()生成对应的url。
```php
$app->get('user/{id}/profile', ['as' => 'profile', function ($id) {
    //
}]);

$url = route('profile', ['id' => 1]);
```
生成重定向
```php
return redirect()->route('profile',1);
```
这样就会重定向到 user/1/profile;

## 路由群组
这个相比django就比较代码上繁琐了，但是逻辑还是很好的
```php
//此处为前缀群组
Route::group(['prefix' => 'member'], function(){
	Route::get('hello', function(){
		return 'hello';
	});

	Route::get('world', function(){
		return 'world';
		});
});
```
这个时候想显示hello时就不能直接访问hello，要访问member/hello 才行了