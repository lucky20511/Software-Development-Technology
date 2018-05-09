# Autowired vs getBean

By default, a Spring context will pay no attention to_@Autowired_annotations. In order to process them, the context needs to have a_AutowiredAnnotationBeanPostProcessor_bean registered in the context.

```
<context:annotation-config/>
```

registers one of these for you \(along with a few others\), so you do need it \(unless you register_AutowiredAnnotationBeanPostProcessor_yourself, which is perfectly valid\).

If you don't like having_@Autowired_in your code, then you can explicitly inject properties in the XML using , which just moves the clutter from one place to another.

If your context is extremely simple, then you can use implicit autowiring, as described[here](http://static.springsource.org/spring/docs/3.0.x/spring-framework-reference/html/beans.html#beans-factory-autowire). Essentially, this tells Spring to autowire automatically by property name or type. This required very little configuration, but it very quickly gets out of control - it's automatic nature means it's hard to control, and gives you very little flexibility.

In general_@Autowired_really is the best option.

With application context, you are injecting the bean by yourself.

Annotation wiring isn’t turned on in the Spring container by default. So, before you can use annotation-based autowiring, you'll need to enable it in your Spring configuration. The simplest way to do that is with the

```
<context:annotation-config>
  <beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:context="http://www.springframework.org/schema/context"
  xsi:schemaLocation="http://www.springframework.org/schema/beans
  http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
  http://www.springframework.org/schema/context
  http://www.springframework.org/schema/context/spring-context-3.0.xsd">
<context:annotation-config />
...
</beans>
```

Hope this helps.



