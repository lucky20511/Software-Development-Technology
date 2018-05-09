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



