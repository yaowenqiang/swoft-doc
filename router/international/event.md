## 事件机制

事件机制是通过观察者模式实现的，主要用于解决，当一个事件发生后，需要执行一连串更新操作。传统的编程方式，就是在事件的代码之后直接加入处理逻辑，当增加更新操作的逻辑后，代码会变得难以维护。一个完整的事件由事件源、事件监听器注册表、发布事件。

## 定义监听器

首先继承 `\Swoft\Event\IApplicationListener` 接口，并且实现事件处理函数 `onApplicationEvent`，其次使用 `@Listener()` 注解，实现监听器和事件的绑定。如下 demo ，定义每次请求前触发的事件监听器。

>  @Listener\(event="eventName"\) 或 @Listener\("eventName"\) 含义一样，都是绑定一个监听器到事件。
>
> 同一个事件名称可以绑定多个事件监听器
>
> 监听器和事件绑定是启动的时候，通过扫描注解自动绑定，无需手动绑定
>
> 事件目前都是同步，后续会实现异步事件。

```php
/**
 * 请求前
 *
 * @Listener(Event::BEFORE_REQUEST)
 */
class BeforeRequestListener implements IApplicationListener
{
    /**
     * 事件回调
     *
     * @param ApplicationEvent|null $event      事件对象
     * @param array                 ...$params  事件附加信息
     */
    public function onApplicationEvent(ApplicationEvent $event = null, ...$params)
    {
        // header获取日志ID和spanid请求跨度ID
        $logid = RequestContext::getRequest()->getHeader('logid', uniqid());
        $spanid = RequestContext::getRequest()->getHeader('spanid', 0);
        $uri = RequestContext::getRequest()->getRequestUri();

        $contextData = [
            'logid'       => $logid,
            'spanid'      => $spanid,
            'uri'         => $uri,
            'requestTime' => microtime(true),
        ];
        RequestContext::setContextData($contextData);
    }
}
```

## 发布事件

发布事件可以传递一个事件对象，也可以不传到事件对象，使用可变参数传递数据。具体使用哪种看实际情况选择。

### 事件对象

继承 `\Swoft\Event\ApplicationEvent` 父类，实现相应逻辑，即可实现一个事件对象的定义。父类 `$handled` 属性，含义是，是否停止后续监听器执行，默认是 `false` ，并且构造函数中，还可以传递事件源对象。

```php
class EventObj extends \Swoft\Event\ApplicationEvent
{
    // ...
}
```

```php
// 触发事件，未传递任何信息
App::trigger(Event::BEFORE_REQUEST);

// 触发事件，通过可变参数传递数据
App::trigger(Event::BEFORE_REQUEST,null, $data, $data2);

// 触发事件，通过事件对象传递信息
App::trigger(Event::BEFORE_REQUEST, new EventObj(new Object()));
```



