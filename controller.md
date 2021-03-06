# 控制器

一个继承`\Swoft\Web\Controller`的类既是控制器，控制器有两种注解自动注册和手动注册两种方式。建议使用注解自动注册，方便简洁，维护简单。多次注册相同的路由前者会被后者覆盖。

## 注解自动注册

注解自动注册常用到三个注解 `@AutoController()`、`@Inject()`、`@RequestMapping()`

> **@AutoController**
>
> 已经使用@AutoController，不能再使用@Bean注解。
>
> @AutoController注解不需要指定bean名称，统一类为bean名称
>
> @AutoController\(\)默认自动解析controller前缀，并且使用驼峰格式。
>
> @AutoController\(prefix="/demo2"\)或@AutoController\("/demo2"\)功能一样，两种使用方式。
>
> **@Inject**
>
> 使用和之前的一样
>
> **@RequestMapping**
>
> @RequestMapping\(route="/index2"\)或@RequestMapping\("/index2"\)功能一样两种方式使用，这种默认是支持get和post方式@RequestMapping\(route="/index2", method=RequestMethod::GET\)注册支持的方法
>
> 不使用@RequestMapping或RequestMapping\(\)功能一样，都是默认解析action方法，以驼峰格式，注册路由。

```php
 /**
 * 控制器demo
 *
 * @AutoController(prefix="/demo2")
 */
class DemoController extends Controller
{
    /**
     * 注入逻辑层
     *
     * @Inject()
     * @var IndexLogic
     */
    private $logic;

    /**
     * 定义一个route,支持get和post方式，处理uri=/demo2/index
     *
     * @RequestMapping(route="index", method={RequestMethod::GET, RequestMethod::POST})
     */
    public function actionIndex()
    {
        // 获取所有GET参数
        $get = $this->request()->query();
        // 获取name参数默认值defaultName
        $getName = $this->request()->query('name', 'defaultName');
        // 获取所有POST参数
        $post = $this->request()->post();
        // 获取name参数默认值defaultName
        $postName = $this->request()->post('name', 'defaultName');
        // 获取所有参，包括GET或POST
        $inputs = $this->request()->input();
        // 获取name参数默认值defaultName
        $inputName = $this->request()->input('name', 'defaultName');
        // 返回一个数组，由请求发起端通过 Header[Accept] 决定返回的数据格式
        return compact('get', 'getName', 'post', 'postName', 'inputs', 'inputName');
    }

    /**
     * 定义一个route,支持get,以"/"开头的定义，直接是根路径，处理uri=/index2
     *
     * @RequestMapping(route="/index2", method=RequestMethod::GET)
     */
    public function actionIndex2()
    {
        // 重定向一个URI
        return $this->response()->redirect('/login/index');
    }

    /**
     * 没有使用注解，自动解析注入，默认支持get和post,处理uri=/demo2/index3
     */
    public function actionIndex3()
    {
        return true;
    }
}
```

## 手动注册

手动注册常用@Bean()、@Inject()注解，手动注册还要多一步就是在`app/routes.php`里面注册自己的路由规则。

> 手动注册@Bean\(\)只能这样缺省方式。并且不能使用@AutoController注解

```php
/**
 * 控制器demo
 *
 * @Bean()
 */
class DemoController extends Controller
{
    /**
     * 注入逻辑层
     *
     * @Inject()
     * @var IndexLogic
     */
    private $logic;

    /**
     * uri=/demo2/index
     */
    public function actionIndex()
    {
        // 获取所有GET参数
        $get = $this->request()->query();
        // 获取name参数默认值defaultName
        $getName = $this->request()->query('name', 'defaultName');
        // 获取所有POST参数
        $post = $this->request()->post();
        // 获取name参数默认值defaultName
        $postName = $this->request()->post('name', 'defaultName');
        // 获取所有参，包括GET或POST
        $inputs = $this->request()->input();
        // 获取name参数默认值defaultName
        $inputName = $this->request()->input('name', 'defaultName');
        // 返回一个数组，由请求发起端通过 Header[Accpet] 决定返回的数据格式
        return compact('get', 'getName', 'post', 'postName', 'inputs', 'inputName');
    }
}
```

routes.php手动注册路由

```php
$router->map(['get', 'post'], '/demo2/index', 'app\controllers\DemoController@index');
$router->get('/index2', 'app\controllers\DemoController@index2');
$router->get('/demo2/index3', 'app\controllers\DemoController@index3');
```

## Controller 返回的数据类型

我们不建议 Action 指定返回的格式类型，而是根据客户端请求时的 Header 里面的 Accept 决定，比如 Accept 为 `application/json`，我们则应该返回 Json 格式，为 `text/html` 则应该返回 View 视图，为 `text/plain` 则应该返回一个 raw body 。不用担心这部分，我们已经为您实现了，您只需要在 Action 内返回以下类型即可。

- 数组
- 字符串
- 布尔类型
- 数字 int, float ...
- 实现 `\Swoft\Contract\Arrayable` 的对象

目前我们仅支持了 View, Json, Raw 三种格式，后续我们会增加更多的格式。

在 Controller 内抛出异常将由 ExceptionHandler 捕获并进行处理，包括我们建议返回给客户端一个 4xx, 5xx 的状态码时，也是抛出一个相对应的异常，然后由 ExceptionHandler 捕获并统一进行处理


