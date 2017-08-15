---
title: number bit
date: 2017-01-04 22:54:04
tags: php
---

```php
public function test()
{
	$advId = 1458013447;
	$id = 5;
	echo $total = $advId << 32 | $id;
	echo PHP_EOL;

	echo $result = $total & 0xFFFFFFFF;
	echo PHP_EOL;
	echo $result = $total >> 32 & 0xFFFFFFFF;
	echo PHP_EOL;
}
```
