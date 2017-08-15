---
uuid: a26ff6c0-3a54-11e7-b496-b5e2f51b1564
title: php的yield和生成器
date: 2017-05-17 00:28:03
tags: php
---

相比较迭代器，**生成器**提供了一种更容易的方法来实现简单的对象迭代，性能开销和复杂性都大大降低。

一个生成器函数看起来像一个普通的函数，不同的是普通函数返回一个值，而一个生成可以yield生成许多它所需要的值，并且每一次的生成返回值只是暂停当前的执行状态，当下次调用生成器函数时，PHP会从上次暂停的状态继续执行下去。

<!--more-->
我们在使用生成器的时候可以像关联数组那样指定一个键名对应生成的值。如下生成一个键值对与定义一个关联数组相似。

```php
function xrange($start, $limit, $step = 1) {
    for ($i = $start, $j = 0; $i <= $limit; $i += $step, $j++) {
        // 给予键值
        yield $j => $i;
    }
}

$xrange = xrange(1, 10, 2);
foreach ($xrange as $key => $value) {
    echo $key . ' => ' . $value . "\n";
}

```

实际上生成器函数返回的是一个Generator对象，这个对象不能通过new实例化，并且实现了Iterator接口。

```php
Generator implements Iterator {
    public mixed current(void)
    public mixed key(void)
    public void next(void)
    public void rewind(void)
    // 向生成器传入一个值
    public mixed send(mixed $value)
    public void throw(Exception $exception)
    public bool valid(void)
    // 序列化回调
    public void __wakeup(void)
}
```

可以看到出了实现Iterator的接口之外Generator还添加了send方法，用来向生成器传入一个值，并且当做yield表达式的结果，然后继续执行生成器，直到遇到下一个yield后会再次停住。

```php
function printer() {
    while(true) {
        echo 'receive: ' . yield . "\n";
    }
}

$printer = printer();
$printer->send('Hello');
$printer->send('world');
```

以上的例子会输出：
```shell
receive: Hello
receive: world
```

在上面的例子中，经过第一个send()方法，yield表达式的值变为Hello，之后执行echo语句，输出第一条结果receive: Hello，输出完毕后继续执行到第二个yield处，只不过当前的语句没有执行到底，不会执行输出。如果将例子改改就能够看出来yield的继续执行到哪里。

```php
function printer() {
    $i = 1;
    while(true) {
        echo 'this is the yield ' . $i . "\n";
        echo 'receive: ' . yield . "\n";
        $i++;
    }
}

$printer = printer();
$printer->send('Hello');
$printer->send('world');

```

这次的输出便会变为:

```shell
this is the yield 1
receive: hello
this is the yield 2
receive: world
this is the yield 3
```

这边可以清楚的看出send之后的继续执行到第二个yield处，之前的代码照常执行。

我们再对例子进行适当的修改：

```php
function printer() {
    $i = 1;
    while(true) {
        echo 'this is the yield ' . (yield $i) . "\n";
        $i++;
    }
}

$printer = printer();
var_dump($printer->send('first'));
var_dump($printer->send('second'));
```

执行一下会发现结果为:

```shell
this is the yield first
int(2)
this is the yield second
int(3)
```

让我们来看一下，是不是发现了问题，跑出来的结果不是从1开始的而是从2开始，这是为啥嘞，我们来分析一下：
在此之前我们先来跑另外一段代码：

```php
function printer() {
    $i = 1;
    while(true) {
        echo 'this is the yield ' . (yield $i) . "\n";
        $i++;
    }
}

$printer = printer();
var_dump($printer->current());
var_dump($printer->send('first'));
var_dump($printer->send('second'));
```

这个时候我们会发现执行的结果变成了:

```shell
int(1)
this is the yield first
int(2)
this is the yield second
int(3)
```

可以看到在第一次调用生成器函数的时候，生成器已经执行到了第一个yield表达式处，所以在$printer->send('first')之前，生成器便已经yield 1出来了，只是没有对这个生成的值进行接收处理，在send()了之后，echo语句便会紧接着完整的执行，执行完毕继续执行$i++，下次循环便是var_dump(2)。

至此，我们看到了yield不仅能够返回数据而且还可以接收数据，而且两者可以同时进行，此时yield便成了数据双向传输的工具，这就为了实现协程提供了可能性。



参考[在PHP中使用协程实现多任务调度](http://www.laruence.com/2015/05/28/3038.html)
加深理解
