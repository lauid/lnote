---
uuid: f192d6e0-3a4b-11e7-b593-1fcd04ea44f8
title: php迭代器和yield
date: 2017-05-16 23:25:50
tags: php
---


PHP提供了统一的迭代器接口：
```php
Iterator extends Traversable
{
	//返回当前元素
	abstract public mixed current(void)
	//返回当前元素的键
	abstract public scalar key(void)
	//向下移动到下一个元素
	abstract public void next(void)
	//返回到迭代器的第一个元素
	abstract public void remind(void)
	//检查当前位置是否有效
	abstract public boolean valid(void)
}
```

<!-- more -->

通过**Iterator接口**, 我们可以自行实现,来观察迭代器的调用顺序.
```php
class MyIterator implements Iterator
{
	private $position = 0;
	private $arr = array(
		'first', 'second', 'third',
	);

	public function __construct()
	{
		$this->_position = 0;
	}

	public function rewind()
	{
		var_dump(__METHOD__);
		$this->_position = 0;
	}

	public function current()
	{
		var_dump(__METHOD__);
		return $this->arr[ $this->_position ];
	}

	public function key()
	{
		var_dump(__METHOD__);
		return $this->position;
	}

	public function next()
	{
		var_dump(__METHOD__);
		++$this->_position;
	}

	public function valid()
	{
		var_dump(__METHOD__);
		return isset( $this->arr[ $this->_position ] );
	}

	$it = new MyIterator();
	foreach( $it as $key => $value )
	{
		var_dump( $key , $value );
		echo PHP_EOL;
	}
}
```

php运行结果
```shell
string(18) "MyIterator::rewind"
string(17) "MyIterator::valid"
string(19) "MyIterator::current"
string(15) "MyIterator::key"
int(0)
string(5) "first"

string(16) "MyIterator::next"
string(17) "MyIterator::valid"
string(19) "MyIterator::current"
string(15) "MyIterator::key"
int(1)
string(6) "second"

string(16) "MyIterator::next"
string(17) "MyIterator::valid"
string(19) "MyIterator::current"
string(15) "MyIterator::key"
int(2)
string(5) "third"

string(16) "MyIterator::next"
string(17) "MyIterator::valid"
```

通过这个例子能够清楚的看到了foreach循环中调用的顺序。从例子也能看出通过迭代器能够将一个普通的对象转化为一个可被遍历的对象。这在有些时候，能够将一个普通的UsersInfo对象转化为一个可以遍历的对象，那么就不需要通过UsersInfo::getAllUser()获取一个数组然后遍历数组，而且还可以在对象中对数据进行预处理。
