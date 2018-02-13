---
title: PHP学习小知识点归纳
tags:
  - PHP
categories: PHP学习心得&备忘
abbrlink: ea2f965e
date: 2018-02-13 12:28:00
---

** Heredoc结构形式长字符串 **
首先使用定界符表示字符串（<<<），接着在“<<<“之后提供一个标识符GOD，然后是字符串，最后以提供的这个标识符结束字符串。
```php
<?php 
$string1 = <<<GOD
我有一只小毛驴，我从来也不骑。
有一天我心血来潮，骑着去赶集。
我手里拿着小皮鞭，我心里正得意。
不知怎么哗啦啦啦啦，我摔了一身泥.
GOD;

echo $string1;
?>
```

** 单双引号区别 **
单引号串和双引号串在PHP中的处理是不相同的。双引号串中的内容可以被解释而且替换，而单引号串中的内容总被认为是普通字符。
```php
$str='hello';
echo "str is $str"; //运行结果: str is hello
echo 'str is $str'; //运行结果: str is $str
```
<!-- more -->
** 资源类型 **
这里和python比较类似，不过还是留档一下。
```php
<?php 
//首先采用“fopen”函数打开文件，得到返回值的就是资源类型。
$file_handle = fopen("/data/webroot/resource/php/f.txt","r");
if ($file_handle){
    while (!feof($file_handle)) { //判断是否到最后一行
        $line = fgets($file_handle); //读取一行文本
        echo $line; //输出一行文本
        echo "<br />"; //换行
    }
}
fclose($file_handle);//关闭文件
?>
```

** 空类型 **
php空类型是NULL且对大小不敏感，python中为None对大小写敏感。

** 常量 **
php中有常量这个概念！这点比没有常量概念的python好多了啊。
```php
<?php
define("PI",3.14);
$r=3;
echo "面积为:".(PI*$r*$r)."<br />";
echo "周长为:".(2*PI*$r)."<br />";
?>
```

判断常量是否被定义
```php
//bool defined(string constants_name)

<?php 
define("PI1",3.14);
$p = "PI1";
$is1 =defined($p);
$is2 = defined("PI2");
var_dump($is1);   // true
var_dump($is2);   // false
?>
```

** 赋值运算符 **
类似c语言的取址，“&”：引用赋值，意味着两个变量都指向同一个数据。它将使两个变量共享一块内存，如果这个内存存储的数据变了，那么两个变量的值都会发生变化。
```php
$c = &$a;
```

** 运算符 **
```php
var_dump($a === $b); //全等
var_dump($a <> $b);  //不等 返回bool
var_dump($a !== $b); //非全等（类型+数据）

$b = $a >= 60 ? "及格": "不及格"; // 三元运算符
```

** 连接运算符 **
和其他语言不一样，php使用"."来连接字符串
```php
	$a = "张先生";
	$tip = $a.",欢迎您在慕课网学习PHP！";
	
    $b = "东边日出西边雨";	
    $b .= ",道是无晴却有晴";
    
	$c = "东边日出西边雨";	
    $c = $c.",道是无晴却有晴";
```

** 错误控制运算符 **
PHP中提供了一个错误控制运算符“@”，对于一些可能会在运行过程中出错的表达式时，我们不希望出错的时候给客户显示错误信息，这样对用户不友好。于是，可以将@放置在一个PHP表达式之前，该表达式可能产生的任何错误信息都被忽略掉；

如果激活了track_error（这个玩意在php.ini中设置）特性，表达式所产生的任何错误信息都被存放在变量$php_errormsg中，此变量在每次出错时都会被覆盖，所以如果想用它的话必须尽早检查。

需要注意的是：错误控制前缀“@”不会屏蔽解析错误的信息，不能把它放在函数或类的定义之前，也不能用于条件结构例如if和foreach等。
```php
<?php  
 $conn = @mysql_connect("localhost","username","password");
 echo "出错了，错误原因是：".$php_errormsg;
?>
```

** foreach循环 **
只取值，不取下标

```php
<?php
 foreach (数组 as 值){
//执行的任务
}
?>
```

同时取下标和值 
```php
<?php
foreach (数组 as 下标 => 值){
 //执行的任务
}
?>
```