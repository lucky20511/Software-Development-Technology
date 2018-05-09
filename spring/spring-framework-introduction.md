# IOC container

[https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/beans.html](https://docs.spring.io/spring/docs/3.2.x/spring-framework-reference/html/beans.html)

### Difference between BeanFactory and ApplicationContext

**Bean Factory**

* Bean instantiation/wiring

**Application Context**

* Bean instantiation/wiring
* Automatic BeanPostProcessor registration &lt;context:annotation-config/&gt;
* Automatic BeanFactoryPostProcessor registration
* Convenient MessageSource access \(for i18n\)
* ApplicationEvent publication

### How to instantiate prototype bean each time?

Spring provides method injection to solve the above problem. It is a another kind of injection used for dynamically overridding a class and its **abstract** methods to create instances every time the bean is injected. 







