# 缓存

缓存目前只支持 Redis, Redis 使用有两种方式直接调用和延迟收包调用。

## Redis 配置

在 `config/beans/redis.php` 配置 Redis 信息。

```php
return [

    // ...
    
    // redis连接池配置
    "redisPool" => [
        'class'           => \Swoft\Pool\RedisPool::class,
        "uri"             => '127.0.0.1:6379,127.0.0.1:6379',// useProvider为false时，从这里识别配置
        "maxIdel"         => 6,// 最大空闲连接数
        "maxActive"       => 10,// 最大活跃连接数
        "maxWait"         => 20,// 最大的等待连接数
        "timeout"         => 200,// 引用properties.php配置值
        "balancer"        => '${randomBalancer}',// 连接创建负载
        "serviceName"     => 'user',// 服务名称，对应连接池的名称格式必须为xxxPool/xxxBreaker
        "useProvider"     => false,
        'serviceprovider' => '${consulProvider}' // useProvider为true使用，用于发现服务
    ],

    // ...

];
```

## Redis 使用

RedisClient 的方法和 PHP Redis 扩展方法是一一致的。唯一不一样的是提供直接调用和延迟收包调用\(用于并发\)

```php
// 直接调用
RedisClient::set('name', 'redis client stelin', 180);
$name = RedisClient::get('name');
RedisClient::get($name);

// 延迟收包调用
$ret = RedisClient::deferCall('get', ['name']);
$ret2 = RedisClient::deferCall('get', ['name']);

$result = $ret->getResult();
$result2 = $ret2->getResult();

$data = [
    'redis' => $name,
    'defer' => $result,
    'defer2' => $result2,
];
```



