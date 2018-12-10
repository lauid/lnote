---
uuid: 34be1d00-fc5e-11e8-8d41-8d1bfee12f25
title: redis原子性
date: 2018-12-10 17:30:17
tags:
---

ps.例

```php
	public function redis()
	{
		$redis = new Redis();
		$redis->connect( '127.0.0.1', 6379 );
		$key = 'testing';
		for( $i = 0; $i < 1000; $i++ )
		{
			$num = (int)$redis->get( $key );
			$num++;
			$redis->set( $key, $num );
			usleep( 1000 );
		}
	}
```

用两个终端执行上面的程序，发现val的结果是小于2000的值，那么可以知道，在程序中执行多个Redis命令并非是原子性的，这也和普通数据库的表现是一样的。
**如果想在上面的程序中实现原子性，可以将get和set改成单命令操作，比如incr，或者使用Redis的事务，或者使用Redis+Lua的方式实现。**


对Redis来说，执行get、set以及eval等API，都是一个一个的任务，这些任务都会由Redis的线程去负责执行，任务要么执行成功，要么执行失败，这就是Redis的命令是原子性的原因。

Redis本身提供的所有API都是原子操作，Redis中的事务其实是要保证批量操作的原子性。

事务是一个原子操作：事务中的命令要么全部被执行，要么全部都不执行。 不支持回滚（roll back）

ps.库存扣减实例
```php
$lua = <<<EOF
    local buyNum = ARGV[1]
    local goodsKey = KEYS[1]  
    local goodsNum = redis.call('get',goodsKey) 
    if goodsNum >= buyNum
        then redis.call('DECRBY',goodsKey,buyNum) 
        return buyNum 
    else 
        return '0'
    end
EOF;
$redis = new Redis();
$redis->connect( '127.0.0.1', 6379 );
$goodsKey = '_goods_';
$buyNum = 1;
$redis->eval( $lua, [ $goodsKey, $buyNum ], 1 );
$redis->evalSha( sha1( $lua ), [ $goodsKey, $buyNum ], 1 );
echo $redis->get( $goodsKey );
```


参考：

[redis事务](http://redisdoc.com/topic/transaction.html)
[redis脚本](http://redisdoc.com/script/eval.html)
