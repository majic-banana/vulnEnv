



## 背景

- 请求转发可以转发到动态或静态资源，如果转发到静态资源，会直接将其内容写入响应流。
- 请求转发是内部行为可以访问WEB-INF目录下的内容
- 请求转发的同样也会进行路径匹配，且匹配规则和web.xml中的配置一致





web.xml

```xml
<servlet>
          <servlet-name>spring</servlet-name>
          <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
          <init-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:springmvc.xml</param-value>
          </init-param>
          <load-on-startup>1</load-on-startup>
      </servlet>
      <servlet-mapping>
          <servlet-name>spring</servlet-name>
          <url-pattern>/</url-pattern>  

      </servlet-mapping>

```

> `/`这个路径不会过拦截 xxx.jsp路径   `/*`才会



## **InternalResourceViewResolver**



**springmvc.xml**

![image-20241103154438754](E:\vulEnv\envConfig\images\image-20241103154438754.png)

访问路径 ： `login`

controller能够进入，能够正常输出：

![image-20241103154323558](E:\vulEnv\envConfig\images\image-20241103154323558.png)



**其核心渲染逻辑**

![image-20241103154138437](E:\vulEnv\envConfig\images\image-20241103154138437.png)

> 可见，它本身只是将Model中参数（key-value)，添加到request，然后转发给jsp文件，让tomcat来渲染jsp （jsp的渲染是由servlet容器来负责的）
>
> 
>
> **InternalResourceViewResolver自身不提供任何渲染逻辑**







 **关于include**:[【Servlet】关于RequestDispatcher的原理（forward 和 include的区别）_requestdispatcher.include-CSDN博客](https://blog.csdn.net/gaoshan12345678910/article/details/82183820)

### 物理地址是html

#### 没有配置`<mvc:resource>`：

1.**不启用include**



> requestURI == dispacherPath 

![image-20241103145258165](E:\vulEnv\envConfig\images\image-20241103145258165.png)果然



因为路径匹配 DispatcherServlet的路径`/`所以，再一次进入到DispatcherServlet ，

![image-20241103171303481](E:\vulEnv\envConfig\images\image-20241103171303481.png)

但是进入doDispach时无法找到handler

![image-20241103172823146](E:\vulEnv\envConfig\images\image-20241103172823146.png)其结果当然就是 404：

![image-20241103173057437](E:\vulEnv\envConfig\images\image-20241103173057437.png)



2.**启用include**

![image-20241103191143035](E:\vulEnv\envConfig\images\image-20241103191143035.png)

> 图中表达式表明这是include请求

在毫无疑问再次进入DispatcherSevlet，但这次请求是 include请求，还是无法匹配到handler,所以相当于

null.include(null)

最终结果：

![image-20241103190519325](E:\vulEnv\envConfig\images\image-20241103190519325.png)

#### 配置了`<mvc:resource>`

1. **启用include**



可以访问到对应的资源（可以看到doservice中有专门处理incude请求的逻辑，这使得在匹配handler时能够匹配到，并且也的确将静态资源写入响应中了，但是会出现中文乱码的情况，可能需要某些配置才能解决。

![image-20241103185254349](E:\vulEnv\envConfig\images\image-20241103185254349.png)

![](E:\vulEnv\envConfig\images\image-20241103185847594.png)

2.**不启用include**:   没有中文乱码

![image-20241103192014409](E:\vulEnv\envConfig\images\image-20241103192014409.png)

### 物理地址为jsp（非include)

![image-20241103153934642](C:\Users\王晨旭\AppData\Roaming\Typora\typora-user-images\image-20241103153934642.png)

> 在访问/login的调试中改变了其值，

结果：

#### ![image-20241103155048394](C:\Users\王晨旭\AppData\Roaming\Typora\typora-user-images\image-20241103155048394.png)

根据上述内容，很容以分析出原因：.jsp文件不匹配 `/`路径，所以，直接请求(已经将Model中的数据放入request)转发到了jsp文件，交由tomcat来对其解析



- 如果配置了`<mvc:resource>`则在寻找Handler时就不为空，那毫无疑问，有专门的handler,将该静态资源写入response



### 结论

InternalResourceViewResolver只能渲染jsp（实际是tomcat渲染）,其他文件一律是静态资源,需要配置才能访问

**如果想要访问这些静态资源**，

1. 设置`<mvc:resource>`标签
2. **或者**配置DefaultServlet让该请求不经过DispatcherServlet。



## **Themeleaf**

==不需要配置`<mvc:resource>`,也不用配置DefaultServlet。==

可以猜到，它直接渲染指定的静态文件。其渲染逻辑由自己实现，不会像InternalResourceViewResolver那样请求转发到jsp，所以必须在模版文件使用**Themeleaf**相关的**命名空间**和**表达式**（不使用就直接原样输出）。





### 配置

> 以spring5为例

在 Spring MVC 中使用 XML 配置 Thymeleaf 视图解析器的步骤如下：

1. **添加依赖**：确保在 `pom.xml` 中添加 Thymeleaf 的依赖。

   ```xml
   <dependency>
       <groupId>org.thymeleaf</groupId>
       <artifactId>thymeleaf-spring5</artifactId>
       <version>3.0.12.RELEASE</version>
   </dependency>
   ```

2. **配置 XML 文件**：在 `spring-servlet.xml`（或你的配置文件）中添加以下配置：

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xmlns:context="http://www.springframework.org/schema/context"
          xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
   
       <!--组件扫描-->
       <context:component-scan base-package="com.powernode.springmvc.controller"/>

       <!--视图解析器-->
       <bean id="thymeleafViewResolver" class="org.thymeleaf.spring6.view.ThymeleafViewResolver">
           <property name="characterEncoding" value="UTF-8"/>
           <property name="order" value="1"/>
           <property name="templateEngine">
               <bean class="org.thymeleaf.spring6.SpringTemplateEngine">
                   <property name="templateResolver">
                    <bean class="org.thymeleaf.spring6.templateresolver.SpringResourceTemplateResolver">
                           <property name="prefix" value="/WEB-INF/thymeleaf/"/>
                           <property name="suffix" value=".html"/>
                           <property name="templateMode" value="HTML"/>
                           <property name="characterEncoding" value="UTF-8"/>
                    </bean>
                   </property>
               </bean>
           </property>
       </bean>
   
</beans>
   ```
   


