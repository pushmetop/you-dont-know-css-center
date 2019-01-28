# 你所不知道的 CSS 居中
 
## 前言

很多的小伙伴在，学习 `PHP` 的时候最早面对的问题之一就是 `require` 、 `include` 和 `require_once` 、`include_once` 的相爱相杀。

在了解了它们相爱相杀的故事后，往往就开始使用起了框架。框架固然是干活的好工具，但是你知道你平时 `new` 一个新类的时候，发生了什么吗？有想过为什么我们 `遵循规范` 就会自动的帮我们做好一切的加载吗？ 让我们一切来探索发现其中的奥秘。

<!--more-->

## 时间线

### 蒸汽时代

在 `PHP` 代码的顶部你是不是经常看到这样的代码。

```php
require 'lionis.php';
require 'is.php';
require 'cool.php';
```

如果只是引入几个 `PHP` 脚本，那还可以接受。那引入成千上万个脚本的时候，爆炸是在所难免的。如果对一个脚本改了个名字，还需要对引入改脚本的每个脚本改名，能不爆炸吗？连打出这段话都怎么绕。

### 电气时代

在 `PHP` 电气时代，开始出现了 `__autoload` 和 `spl_autoload_register` 函数注册自定义的自动加载策略。

通俗的来说，`__autoload` 和 `spl_autoload_register` 是一个 `杀手组织`，他们会去雇佣 `各国杀手` (`函数`)。当我们想搞定某个人的时候（`new`），只需要提供名字(`类名`)，剩下的 `杀手` 会帮我们搞定的。

##### __autoload
PHP 5 开始提供这个函数 [传送门](http://php.net/manual/zh/function.autoload.php)。当你使用的 `类` 找不到的时候，它把类名当成参数扔进这个函数。

```
<?php
// Lionis.php
class Lionis {
    public function __construct() {
        echo '欧耶耶, 我就是 Lionis';
    }
}

```

```
<?php
// index.php
function __autoload($classname) {
    $filename = './' . $classname . '.php';
    require_once $filename;
}

$lionis = new Lionis();
```

输出
```
欧耶耶, 我就是 Lionis
```

##### spl_autoload_register

如果我们 `项目` 很大很老又或者你是一个 `爱折腾` 的少先队员，需要引入的东西有不一样的规范，这时候如果都放在 `__autoload` 函数里，这个函数马上就会膨胀的。而且 `__autoload` 是全局唯一的，如果被人占用了，可能会导致错误。（欲使一个人灭亡，必将先使其膨胀。）

PHP 5.1.2 开始提供这个函数 [传送门](http://php.net/manual/zh/function.spl-autoload-register.php)，注册给定的函数作为 `__autoload` 的实现。所以，我们看一些框架或插件在自己使用的时候，为了兼容可能会出现 `function_exists(spl_autoload_register)`。

```
<?php
function lionisIsCoolFind($classname) {
    require './' . $classname . '.php';
}

// 函数
spl_autoload_register('lionisIsCoolFind');

// 匿名函数
spl_autoload_register(function($require) {
    require './' . $classname . '.php';
});

// 类中的函数
spl_autoload_register(array('Lionis', 'loadClass'));
```

欧耶，这下我们可以写很多不同的自动加载函数了。

### 信息时代

`师傅小心，前面有妖气！` 。如果我们每个人都自己实现一套自动加载的方法，每个PHP `组件`和 `框架`都使用独特的自动加载器，而且每个框架使用不同的逻辑加载PHP类、接口和性状。

那当我们使用一些第三方框架的时候，还需要去弄清楚引导文件中的 `自动加载器`，那样是有多和 `时间` 过不去呢。 `PHP-FIG` 认识到了这个问题了，推荐使用 `PSR-4` 规范，来促进组件之间的 `互操作性`，这样我们就可以使用一个自动加载器了。

##### PSR-4 规范

利用命名空间的前缀和文件系统中的目录对应起来。

映射关系为
```
namespace    => filePath
\Lionis\Cool => cool
```

带有命名空间的类
```php
<?php
// 该文件为 cool/Real.php
namespace \Lionis\Cool;

class Real {
}
```

创建一个对象
```
<?php
// 该文件为 index.php

$lionis = new \Lionis\Cool\Real;

```

这个时候，按照 `PSR-4` 的规范，自动加载器应该去加载 `cool/` 目录下的 `Real.php`。

`不对！`那这样不是还要自己去实现 `自动加载器` 嘛，不然怎么 `无中生有` 出现 `自动加载器` 呢？难道官方 `内置` 了?

你 `out` 了吧，我们可以使用依赖管理器 `composer` 来生成 `PSR-4` 自动加载器。你可能会疑问，那我的旧项目没有遵循 `PSR-4` 规范怎么办？嘿嘿，让我们来探索发现一下 `composer` 是怎么解决这个问题的吧。

## Composer

哦吼吼，我们这次的重点在与探究自动加载，所以 `composer` 的安装和使用等，就不去讨论了。

`composer` 自动加载设置了 4种 `加载方式`：

* PSR-0
* PSR-4
* classmap
* files

### PSR-0

要求命名空间和目录层层对应，且可以使用 `_` 作为路径分隔符，但是这会导致目录结果变得过深。

在 `composer` 执行 `install` 等操作时，`composer` 会把文件中的配置存储在 `vendor/composer/autoload_psr0.php`文件中的返回数组中。

例如：定义了Very\\Good=>vendor\\Lionis\\IsReal\\Cool，在调用 use Very\\Good\\Love\\SomeClass，`PSR-0` 加载的实际目录为 vendor/Lionis/IsReal/Cool/Very/Good/Love/SomeClass.php。

对吧，这简直深得吓人，所以 `PSR-0` 被官方废除了。但是一些主流的框架已经实现了 `PSR-0`，为了向下兼容还是要实现 `PSR-0`。

composer.json配置：
```
"autoload": {
    "psr-0": {
        "Very\\Good": "vendor\Lionis\IsReal\Cool"
    }
}
```

### PSR-4

`PSR-4` 是现在比较推荐的方法，用于替代 `PSR-0`。
与 `PSR-0` 不同的是，取消掉了 `_` 作为分隔符和目录结构。

在 `composer` 执行 `install` 等操作时，`composer` 会把文件中的配置存储在 `vendor/composer/autoload_psr4.php`文件中的返回数组中。

例如：定义了Very\\Good=>vendor\\Lionis\\IsReal\\Cool，在调用 use Very\\Good\
Love\\SomeClass，`PSR-4` 加载的实际目录为 vendor/Lionis/IsReal/Cool/Love/SomeClass.php。

composer.json配置：
```
"autoload": {
    "psr-4": {
        "Very\\Good": "vendor\Lionis\IsReal\Cool"
    }
}
```

### classmap

`classmap` 通过配置指定的目录和文件，在 `composer` 执行 `install` 等操作时，`composer` 会去扫描对应的目录下以`.php`结尾的文件中的 `class`，并存储在 `vendor/composer/autoload_classmap.php`文件中的返回数组中。

composer.json配置：

```
"autoload": {
    "classmap": [
        "Lionis/",
        "Xiaoer/"
    ]
}
```

如果 Lionis 下有一个叫 VeryCool的文件，那么在`vendor/composer/autoload_classmap.php` 中会生成。

```
<?php

// autoload_classmap.php @generated by Composer

$vendorDir = dirname(dirname(__FILE__));
$baseDir = dirname($vendorDir);

return array(
    'VeryCool' => $baseDir . '/Lionis/VeryCool.php',
    // 其他的映射
);
```

### files

`files` 就是直接简单粗暴的加载文件。在 `composer` 执行 `install` 等操作时，`composer` 会把文件中的配置存储在 `vendor/composer/autoload_static.php`文件中的生成一个 `$files` 数组。

composer.json 配置：
```
"autoload": {
    "files": ["Lionis/Very/Cool.php"]
}
```

### 小结

`composer` 通过使用 `composer.json`，用 `json` 的格式来指定我们需要`自动加载`的`规则`。我们只要在入口文件引入 `vendor/autoload.php` 就能很方便的便能使用 `自动加载`。

如果你对 `composer` 实现 `自动加载` 的原理感兴趣，可以顺着 `vendor` 中的 `autoload.php` 去看看源码。

## 总结

从 `石器时代` 到 `信息时代`，`PHP` 经历了很多试验和改变后正在变得越来越好。当然，许多优秀的框架让我们开发速度更快，需要理解的一些知识点也随之被隐藏起来，让我们更加专注于实现逻辑。但是，我们有的时候还是要尝试的去理解他们工作的原理，来提升我们自己。像我老师说过的，所不定一下子踩到狗屎运了呢。

## 更多

[细说 PHP 类库自动加载](https://github.com/qinjx/adv_php_book/blob/master/class_autoload.md)

## 打赏&联系

如果您感觉有收获，欢迎给我打赏，以激励我输出更多的优质内容。

![打赏&联系](https://raw.githubusercontent.com/pushmetop/resource/master/donate/donate.png)

> 本文原稿来自 [PushMetop](https://pushmetop.github.io)
