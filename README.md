# 先来认识一下 Spring Cloud Commons
接口(如ServiceRegistry/DiscoveryClient/LoadBalancerClient)
注解(如!EnableDiscoveryClient/@LoadBalanced)为主,
少量代码实现(如 RandomLoadBalancer).
以及对 Spring 容器(Context) 的扩展(如 NamedContextFactory, bootstrap 配置文件的加载, 容器重启, 容器跟随配置文件刷新等等)
打包好的 starter.

# Spring Cloud Loadbalancer
# 0.一堆类仅为添加一个拦截器
LoadBalancerRequestFactory: 一个工厂, 包装一个为请求对象 HttpRequest 加料的回调 LoadBalancerRequest

LoadBalancerClient: 用于根据 serviceId 选取一个 ServiceInstance, 执行从 LoadBalancerRequestFactory 获得的那个回调

LoadBalancerInterceptor: restTemplate 的拦截器, 拦截后调用 LoadBalancerClient 修改 HttpRequest 对象(主要是 url), 且传入调用 LoadBalancerRequestFactory 生成的回调给 LoadBalancerClient

RestTemplateCustomizer: 为 restTemplate 加上一个拦截器(也可以干点别的, 默认就这一个用处)

SmartInitializingSingleton: 调用 RestTemplateCustomizer 为容器中所有加了 @LoadBalanced 的 RestTemplate 加上一个拦截器

# 1.获取对象的工厂, 以 Spring 容器作为载体管理对象.
NamedContextFactory
继承 DisposableBean, 用于类销毁时执行点东西(指创建的好多个子容器)
继承 ApplicationContextAware, 用于将子容器和当前容器关联起来(所以 Spring 树形扩展这个设计真不错)
泛型 C extends NamedContextFactory.Specification, 无它, 就是个 POJO, 存个 name 和对应的配置 class, 用于初始化容器的(会被注册进去, 然后解析里面的注解啥的...)
此类作用就是管理一大堆(取决于你微服务拆分的程度)子容器, 获取其他代码需要的类型对象


ReactiveLoadBalancer.Factory
定义了获取 ReactiveLoadBalancer 的接口以及与其相关的扩展


LoadBalancerClientFactory
继承 NamedContextFactory, 构造参数指定了几个属性值
实现了 ReactiveLoadBalancer.Factory 的接口, 即提供获取 ReactiveLoadBalancer 的方法.
泛型具体为 LoadBalancerClientSpecification, 还是个POJO


# 2.包含算法逻辑的负载均衡策略的类
Response
server 的封装类, 一般持有一个 ServiceInstance 对象, 如 DefaultResponse

Publisher
响应式编程的东西, 可获取 Response<T> 对象, 一般为 Response<ServiceInstance>

ReactiveLoadBalancer.Factory
定义了获取 ReactiveLoadBalancer 的接口以及与其相关的扩展

ReactiveLoadBalancer
定义了 choose 方法, 即如何选取一个 ServiceInstance, 如轮播, 随机...

ReactorLoadBalancer
定义了 choose 方法的另一形式, 仅返回值不同, 为 Mono<Response<T>> 是 Publisher<Response<T>> 的子类, 返回值为抽象类.

ReactorServiceInstanceLoadBalancer
继承 ReactorLoadBalancer
仅仅作为一个标记类, 无新接口
	

