## Feign在同步，异步下设置请求头的问题

很多的微服务项目都是用Feign来实现远程调用，也就是发起一起HTTP请求通信。

很多系统的权限的校验的东西都是通过请求头的里面的参数来实现的，转发请求的时候自然也需要将head转发到目标服务。

## 同步

一般同步调用时是没什么问题的。

通过实现Feign的RequestInterceptor接口，重写里面的apply方法，为RequestTemplate添加请求头信息。

```java
public class FeignRequestInterceptor implements RequestInterceptor {
    //这里也是从ThreadLocal拿到数据，但是我不知道框架是在什么时候放进去的
     ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        //juint时为空
        if (attributes != null){
            HttpServletRequest request = attributes.getRequest();
            Enumeration<String> headerNames = request.getHeaderNames();
            Set<String> header = new HashSet<>();
            if (headerNames != null){
                while (headerNames.hasMoreElements()){
                    String name = headerNames.nextElement();
                    header.add(name);
                    String values = request.getHeader(name);
                    requestTemplate.header(name,values);
                }
            }
        }
}
```

## 异步

有一个场景，需要提前返回给前端，然后异步调用远程服务。

这个时候如果调用提前返回，此时子线程在刚刚上面写好的拦截器就无法获取到请求头了。

不管我怎么赋值，他就是赋值的时候值还在，用的时候就不在了。我想可能是随着方法被执行完毕被垃圾回收了。

**方法一,通过ThreadLoacl：**

>既然无法直接通过获取HttpServletRequest来获取Header，那么可以稍微改造一下，在原来的基础上添加一个拦截器。
>所有的请求过来的时候，在拦截器中将Header先取出来，然后设置到本线程私有的Map中。
>
>原来的apply方法在提交请求的时候再通过ThreadLocal提供的remove方法，清除掉。
>
>只要把对该Map的操作封装一个工具类，工具类中实现get/set方法即可。
>
>其实这种方式就是换了一个地方保存请求头，因此实用性与便捷性都还可以。



```java
 //异步的时候需要
        ServletRequestAttributes servletRequestAttributes = ThreadLocalUtil.attributesThreadLocal.get();
        if (servletRequestAttributes != null){
            ServletRequestAttributes requestAttributes = servletRequestAttributes;
            HttpServletRequest request = requestAttributes.getRequest();
            Enumeration<String> headerNames = request.getHeaderNames();
            Set<String> header = new HashSet<>();
            if (headerNames != null){
                while (headerNames.hasMoreElements()){
                    String name = headerNames.nextElement();
                    header.add(name);
                    String values = request.getHeader(name);
                    requestTemplate.header(name,values);
                }
            }
            //用完记得删除
            ThreadLocalUtil.attributesThreadLocal.remove();
        }
```

