---
layout: post
title: Const VS Define
category: blog
description: PHP中Const和Define的区别
---

### 简介

我们有两种方式定义常量，分别是const和define():

    const FOO = 'BAR';
    define('FOO', 'BAR');

它们的主要区别在于const在编译时定义常量，而define在运行时定义。

### 区别

1.const不能用于有条件的定义，如果要定义全局常量，必须定义在最外层。

    if (...) {
        const FOO = 'BAR';    // invalid 

    } 

    if (...) {
        define('FOO', 'BAR'); // valid 

    }
    //define一个常见应用是检查常量是否定义
    if (!defined('FOO')) {
        define('FOO', 'BAR');

    }

2.const只接受静态标量数据，而define可以接受任何表达式，但从php5.6起const也可接受常量表达式。

    const BIT_5 = 1 << 5;    // valid since PHP 5.6, invalid previously 
    define('BIT_5', 1 << 5); // always valid

3.const只接受常量名字，而define接受任何表达式作为名字。

    for ($i = 0; $i < 32; ++$i) {
        define('BIT_' . $i, 1 << $i); 
    }

4.const总是大小写敏感的，而define可以通过设置第三个参数为true定义大小写不敏感的常量。

    define('FOO', 'BAR', true);
    echo FOO; // BAR 
    echo foo; // BAR

5.const在当前命名空间定义常量，而define必须传递带命名空间的全名。

    namespace A\B\C; 
    // To define the constant A\B\C\FOO: 
    const FOO = 'BAR'; 
    define('A\B\C\FOO', 'BAR');

6.从php5.6起const可以定义数组常量，但define不支持，到php7const和define都将支持数组常量。

    const FOO = [1, 2, 3];    // valid in PHP 5.6 
    define('FOO', [1, 2, 3]); // invalid in PHP 5.6, valid in PHP 7.0

7.const是语言结构且在编译时定义，因而比define更快（至少两倍）。使用define定义大量常量会较慢，人们甚至发明了[`hidef`](http://pecl.php.net/package/hidef)和[`apc_load_constants`](http://php.net/apc_load_constants)来解决这个问题。

    hidef: Allow definition of user defined constants in simple ini files, which are then processed like internal constants, without any of the usual performance penalties.
    apc_load_constants: Loads a set of constants from the cache.

8.const可以用于定义类常量和接口常量，而define不能这么用。

    class Foo {
        const BAR = 2; // valid 

    } 
    // but 
    class Baz {
        define('QUX', 2); // invalid 

    }

### 总结
尽量用const.
