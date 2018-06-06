---
title: codis3.2遍历指定slot所有key
date: 2018-04-26 09:37:24
tags: codis
categories: php
---

codis3.2不支持KEYS\* 命令，而官方有个SLOTSSCAN命令，可以替代。

php Demo代码

<!--more-->

```php

$host = '127.0.0.1';
$port = 4379;
$redis = new Redis();
$redis->connect( $host, $port );
#####SLOTSSCAN slotnum cursor [COUNT count]
$slotId = 579 
$cursor = 0;
$count = 20;
$i = 0;
do
{
	list( $cursor, $keys ) = $redis->rawCommand( 'SLOTSSCAN', $slotId, $cursor, 'COUNT', $count );
	printf( 'cursor:%s, keys:%s' . PHP_EOL, $cursor, var_export( $keys, true ) );
}while( $cursor > 0 );

```

PS.集群所有proxy和server必须在codis3版本以上才支持
