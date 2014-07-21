---
layout: post
title: "Deploying Spring Boot Applications in IBM WebSphere Application Server"
modified: 2014-07-20 20:23:08 -0400
tags: [DevOps,Software Engineering,Technology Operations,Release Management, WebSphere Application Server]
image:
  feature: abstract-11.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---
Though the Spring Projects make our lives easier on a daily basis, as a development lead, I do face technical difficulties in bridging the gap between development and operations. One such difficulty was to get one of my applications developed using a wide variety of Spring technnologies - Spring Data JPA, Spring Data Rest, Spring Hateoas, and Spring Data Redis - to deploy on WebSphere Application Server. I was excited to use Spring Boot 1.1.4 and Spring IO Platform 1.0.1 to manage my development and version management. 

I enjoyed my dev time every day until it was time to deploy the application on IBM WebSphere Application Server (WAS). Expecting the Operations team to troubleshoot the deployment problems may not even be possible given the human resource availability/constraints on that team. Therefore, it becomes an additional development story within a Sprint (in Agile Scrum parlance) for the core dev team to execute.  

## Deploying on WebSphere Liberty Profile (WLP)
My initial idea was to get the application up and running within WLP. You can download the runtime [here](https://developer.ibm.com/wasdev/downloads/liberty-profile-using-non-eclipse-environments/).

After a little bit of struggle, I got WLP working within my IntelliJ IDE as shown below:

![WLP in IntelliJ]({{ site.url }}/images/WLP-in-IntelliJ.png) 

and the ```server.xml``` should look somewhat like the following:

{% highlight xml%}
<?xml version="1.0" encoding="UTF-8"?>
<server description="new server">
  <!-- Enable features -->
  <featureManager>
    <feature>localConnector-1.0</feature>
    <feature>jsp-2.2</feature>
  </featureManager>
  <!-- To access this server from a remote client add a host attribute to the following element, e.g. host="*" -->
  <httpEndpoint httpPort="9080" httpsPort="9443" id="defaultHttpEndpoint" />
  <applicationMonitor updateTrigger="mbean" />
  <jspEngine jdkSourceLevel="16" />
  <application id="Gradle___NetStarContentRestServices___ncrs_war__exploded_" location="/Users/Naru/IdeaProjects/NetStarContentRestServices/out/artifacts/NetStarContentRestServices/exploded/ncrs.war" name="Gradle___NetStarContentRestServices___ncrs_war__exploded_" type="war" />
</server>
{% endhighlight %}

Since Spring Boot starter package for web ```spring-boot-starter-web``` uses embedded tomcat by default, I ended up specifying the following in my ```build.gradle``` file as follows:

{% highlight gradle %}
providedRuntime("org.springframework.boot:spring-boot-starter-tomcat")
{% endhighlight %}

The ```providedRuntime``` directive makes sure that the war plugin moves the embeeded tomcat jar files to ```/WEB-INF/lib-provided``` from ```/WEB-INF/lib``` directory. 

>The premise of Spring Boot is to develop and run Spring applications as standalones - extremely good for development teams. 

If you build the war file Without the above directive, the embedded tomcat jars will also be placed along with the other jar dependencies in the ```/WEB-INF/lib``` directory. [Here](https://github.com/spring-projects/spring-boot/issues/1194) is the question I posed to the Spring Boot team to get my application war file to work in a regular Tomcat or WLP environment.

Once the above situation with the application war file was fixed, my application deployed and ran fine within the WLP environment. After successfully getting the application to work in WLP environment, I moved onto deploying it in my test WAS 8.5.5.2 environment. To my surpise, the application did not even deploy properly. I had to perform a bunch of steps to get my REST application to work.

## Deploying on WebSphere Application Server (WAS)  
The key to a successful deployment of Spring Boot applications in WAS is to setup a shared library as shown below:

[Shared Library Setup in WAS]({{ site.url }}/images/WAS-shared-library-setup.png) 

I removed all the jar files under ```/WEB-INF/lib```  from my war file and placed them into the shared library. After setting up the shared library, I changed the class loader order as follows:

[Class Loader Order]({{ site.url }}/images/class-loader-order.png)

Though the application started, I got hit with JPA persistence issues as follows:

{% highlight log %}
[7/20/14 11:30:21:544 EDT] 0000005c SystemOut     O 
[7/20/14 11:30:21:544 EDT] 0000005c SystemOut     O   .   ____          _            __ _ _
[7/20/14 11:30:21:544 EDT] 0000005c SystemOut     O  /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
[7/20/14 11:30:21:544 EDT] 0000005c SystemOut     O ( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
[7/20/14 11:30:21:544 EDT] 0000005c SystemOut     O  \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
[7/20/14 11:30:21:544 EDT] 0000005c SystemOut     O   '  |____| .__|_| |_|_| |_\__, | / / / /
[7/20/14 11:30:21:544 EDT] 0000005c SystemOut     O  =========|_|==============|___/=/_/_/_/
[7/20/14 11:30:21:544 EDT] 0000005c SystemOut     O  :: Spring Boot ::        (v1.1.4.RELEASE)
[7/20/14 11:30:21:544 EDT] 0000005c SystemOut     O 
[7/20/14 11:30:25:404 EDT] 0000005c SystemOut     O 2014-07-20 11:30:25,388 ERROR         org.springframework.boot.SpringApplication: 338 - Application startup failed
org.springframework.context.ApplicationContextException: Unable to start embedded container; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration$DispatcherServletConfiguration': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: private org.springframework.boot.autoconfigure.web.ServerProperties org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration$DispatcherServletConfiguration.server; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'serverProperties': Could not bind properties; nested exception is javax.validation.ValidationException: HV000041: Call to TraversableResolver.isReachable() threw an exception.
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.onRefresh(EmbeddedWebApplicationContext.java:135) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.context.support.AbstractApplicationContext.refresh(AbstractApplicationContext.java:476) ~[spring-context-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.refresh(EmbeddedWebApplicationContext.java:120) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.SpringApplication.refresh(SpringApplication.java:691) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.SpringApplication.run(SpringApplication.java:320) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.builder.SpringApplicationBuilder.run(SpringApplicationBuilder.java:142) [spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.context.web.SpringBootServletInitializer.createRootApplicationContext(SpringBootServletInitializer.java:89) [spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.context.web.SpringBootServletInitializer.onStartup(SpringBootServletInitializer.java:51) [spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.web.SpringServletContainerInitializer.onStartup(SpringServletContainerInitializer.java:175) [spring-web-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at com.ibm.ws.webcontainer.webapp.WebAppImpl.initializeServletContainerInitializers(WebAppImpl.java:613) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.webapp.WebAppImpl.initialize(WebAppImpl.java:409) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.webapp.WebGroupImpl.addWebApplication(WebGroupImpl.java:88) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.VirtualHostImpl.addWebApplication(VirtualHostImpl.java:169) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.WSWebContainer.addWebApp(WSWebContainer.java:749) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.WSWebContainer.addWebApplication(WSWebContainer.java:634) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.component.WebContainerImpl.install(WebContainerImpl.java:426) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.component.WebContainerImpl.start(WebContainerImpl.java:718) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.runtime.component.ApplicationMgrImpl.start(ApplicationMgrImpl.java:1177) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.DeployedApplicationImpl.fireDeployedObjectStart(DeployedApplicationImpl.java:1370) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.DeployedModuleImpl.start(DeployedModuleImpl.java:639) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.DeployedApplicationImpl.start(DeployedApplicationImpl.java:968) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.ApplicationMgrImpl.startApplication(ApplicationMgrImpl.java:776) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.ApplicationMgrImpl.startApplicationDynamically(ApplicationMgrImpl.java:1379) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.ApplicationMgrImpl.start(ApplicationMgrImpl.java:2189) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.CompositionUnitMgrImpl.start(CompositionUnitMgrImpl.java:446) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.CompositionUnitImpl.start(CompositionUnitImpl.java:123) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.CompositionUnitMgrImpl.start(CompositionUnitMgrImpl.java:389) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.CompositionUnitMgrImpl.access$500(CompositionUnitMgrImpl.java:117) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.CompositionUnitMgrImpl$1.run(CompositionUnitMgrImpl.java:664) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.security.auth.ContextManagerImpl.runAs(ContextManagerImpl.java:5384) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.security.auth.ContextManagerImpl.runAsSystem(ContextManagerImpl.java:5600) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.security.core.SecurityContext.runAsSystem(SecurityContext.java:255) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.CompositionUnitMgrImpl.startCompositionUnit(CompositionUnitMgrImpl.java:678) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.CompositionUnitMgrImpl.startCompositionUnit(CompositionUnitMgrImpl.java:622) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.runtime.component.ApplicationMgrImpl.startApplication(ApplicationMgrImpl.java:1269) [com.ibm.ws.runtime.jar:na]
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.7.0]
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:94) ~[na:1.7.0]
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:55) ~[na:1.7.0]
    at java.lang.reflect.Method.invoke(Method.java:619) ~[na:2.6 (04-09-2014)]
    at sun.reflect.misc.Trampoline.invoke(MethodUtil.java:87) [na:1.7.0]
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method) ~[na:1.7.0]
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:94) ~[na:1.7.0]
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:55) ~[na:1.7.0]
    at java.lang.reflect.Method.invoke(Method.java:619) ~[na:2.6 (04-09-2014)]
    at sun.reflect.misc.MethodUtil.invoke(MethodUtil.java:291) [na:1.7.0]
    at javax.management.modelmbean.RequiredModelMBean$4.run(RequiredModelMBean.java:1260) [na:1.7.0]
    at java.security.AccessController.doPrivileged(AccessController.java:300) [na:1.7.0]
    at java.security.ProtectionDomain$1.doIntersectionPrivilege(ProtectionDomain.java:87) [na:1.7.0]
    at javax.management.modelmbean.RequiredModelMBean.invokeMethod(RequiredModelMBean.java:1254) [na:1.7.0]
    at javax.management.modelmbean.RequiredModelMBean.invoke(RequiredModelMBean.java:1092) [na:1.7.0]
    at com.sun.jmx.interceptor.DefaultMBeanServerInterceptor.invoke(DefaultMBeanServerInterceptor.java:831) [na:1.7.0]
    at com.sun.jmx.mbeanserver.JmxMBeanServer.invoke(JmxMBeanServer.java:813) [na:1.7.0]
    at com.ibm.ws.management.AdminServiceImpl$1.run(AdminServiceImpl.java:1335) [com.ibm.ws.admin.core.jar:na]
    at com.ibm.ws.security.util.AccessController.doPrivileged(AccessController.java:118) [bootstrap.jar:WAS855.SERV1 [cf021412.02]]
    at com.ibm.ws.management.AdminServiceImpl.invoke(AdminServiceImpl.java:1228) [com.ibm.ws.admin.core.jar:na]
    at com.ibm.ws.management.commands.AdminServiceCommands$InvokeCmd.execute(AdminServiceCommands.java:251) [com.ibm.ws.admin.core.jar:na]
    at com.ibm.ws.console.core.mbean.MBeanHelper.invoke(MBeanHelper.java:241) [wsccore-core_module.jar:na]
    at com.ibm.ws.console.appdeployment.ApplicationDeploymentCollectionAction.execute(ApplicationDeploymentCollectionAction.java:578) [appmanagement-appmanagement_module.jar:na]
    at org.apache.struts.action.RequestProcessor.processActionPerform(Unknown Source) [struts.jar:na]
    at org.apache.struts.action.RequestProcessor.process(Unknown Source) [struts.jar:na]
    at org.apache.struts.action.ActionServlet.process(Unknown Source) [struts.jar:na]
    at org.apache.struts.action.ActionServlet.doPost(Unknown Source) [struts.jar:na]
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:595) [javax.j2ee.servlet.jar:na]
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:668) [javax.j2ee.servlet.jar:na]
    at com.ibm.ws.webcontainer.servlet.ServletWrapper.service(ServletWrapper.java:1230) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.servlet.ServletWrapper.handleRequest(ServletWrapper.java:779) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.servlet.ServletWrapper.handleRequest(ServletWrapper.java:478) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.servlet.ServletWrapperImpl.handleRequest(ServletWrapperImpl.java:178) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.filter.WebAppFilterChain.invokeTarget(WebAppFilterChain.java:136) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.filter.WebAppFilterChain.doFilter(WebAppFilterChain.java:79) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.filter.WebAppFilterManager.doFilter(WebAppFilterManager.java:960) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.filter.WebAppFilterManager.invokeFilters(WebAppFilterManager.java:1064) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.webapp.WebAppRequestDispatcher.dispatch(WebAppRequestDispatcher.java:1385) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.webapp.WebAppRequestDispatcher.forward(WebAppRequestDispatcher.java:194) [com.ibm.ws.webcontainer.jar:na]
    at org.apache.struts.action.RequestProcessor.doForward(Unknown Source) [struts.jar:na]
    at org.apache.struts.tiles.TilesRequestProcessor.doForward(Unknown Source) [struts.jar:na]
    at org.apache.struts.action.RequestProcessor.processForwardConfig(Unknown Source) [struts.jar:na]
    at org.apache.struts.tiles.TilesRequestProcessor.processForwardConfig(Unknown Source) [struts.jar:na]
    at org.apache.struts.action.RequestProcessor.process(Unknown Source) [struts.jar:na]
    at org.apache.struts.action.ActionServlet.process(Unknown Source) [struts.jar:na]
    at org.apache.struts.action.ActionServlet.doPost(Unknown Source) [struts.jar:na]
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:595) [javax.j2ee.servlet.jar:na]
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:668) [javax.j2ee.servlet.jar:na]
    at com.ibm.ws.webcontainer.servlet.ServletWrapper.service(ServletWrapper.java:1230) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.servlet.ServletWrapper.handleRequest(ServletWrapper.java:779) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.servlet.ServletWrapper.handleRequest(ServletWrapper.java:478) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.servlet.ServletWrapperImpl.handleRequest(ServletWrapperImpl.java:178) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.filter.WebAppFilterChain.invokeTarget(WebAppFilterChain.java:136) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.filter.WebAppFilterChain.doFilter(WebAppFilterChain.java:97) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.console.core.servlet.WSCUrlFilter.setUpCommandAssistance(WSCUrlFilter.java:955) [isclite.jar:na]
    at com.ibm.ws.console.core.servlet.WSCUrlFilter.continueStoringTaskState(WSCUrlFilter.java:504) [isclite.jar:na]
    at com.ibm.ws.console.core.servlet.WSCUrlFilter.doFilter(WSCUrlFilter.java:325) [isclite.jar:na]
    at com.ibm.ws.webcontainer.filter.FilterInstanceWrapper.doFilter(FilterInstanceWrapper.java:195) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.filter.WebAppFilterChain.doFilter(WebAppFilterChain.java:91) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.filter.WebAppFilterManager.doFilter(WebAppFilterManager.java:960) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.filter.WebAppFilterManager.invokeFilters(WebAppFilterManager.java:1064) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.servlet.CacheServletWrapper.handleRequest(CacheServletWrapper.java:87) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.WebContainer.handleRequest(WebContainer.java:914) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.WSWebContainer.handleRequest(WSWebContainer.java:1662) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.webcontainer.channel.WCChannelLink.ready(WCChannelLink.java:200) [com.ibm.ws.webcontainer.jar:na]
    at com.ibm.ws.http.channel.inbound.impl.HttpInboundLink.handleDiscrimination(HttpInboundLink.java:459) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.http.channel.inbound.impl.HttpInboundLink.handleNewRequest(HttpInboundLink.java:526) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.http.channel.inbound.impl.HttpInboundLink.processRequest(HttpInboundLink.java:312) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.http.channel.inbound.impl.HttpICLReadCallback.complete(HttpICLReadCallback.java:88) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.tcp.channel.impl.AioReadCompletionListener.futureCompleted(AioReadCompletionListener.java:175) [com.ibm.ws.runtime.jar:na]
    at com.ibm.io.async.AbstractAsyncFuture.invokeCallback(AbstractAsyncFuture.java:217) [com.ibm.ws.runtime.jar:na]
    at com.ibm.io.async.AsyncChannelFuture.fireCompletionActions(AsyncChannelFuture.java:161) [com.ibm.ws.runtime.jar:na]
    at com.ibm.io.async.AsyncFuture.completed(AsyncFuture.java:138) [com.ibm.ws.runtime.jar:na]
    at com.ibm.io.async.ResultHandler.complete(ResultHandler.java:204) [com.ibm.ws.runtime.jar:na]
    at com.ibm.io.async.ResultHandler.runEventProcessingLoop(ResultHandler.java:775) [com.ibm.ws.runtime.jar:na]
    at com.ibm.io.async.ResultHandler$2.run(ResultHandler.java:905) [com.ibm.ws.runtime.jar:na]
    at com.ibm.ws.util.ThreadPool$Worker.run(ThreadPool.java:1864) [com.ibm.ws.runtime.jar:na]
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration$DispatcherServletConfiguration': Injection of autowired dependencies failed; nested exception is org.springframework.beans.factory.BeanCreationException: Could not autowire field: private org.springframework.boot.autoconfigure.web.ServerProperties org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration$DispatcherServletConfiguration.server; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'serverProperties': Could not bind properties; nested exception is javax.validation.ValidationException: HV000041: Call to TraversableResolver.isReachable() threw an exception.
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:292) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1185) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:537) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:302) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:298) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:193) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.ConstructorResolver.instantiateUsingFactoryMethod(ConstructorResolver.java:370) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.instantiateUsingFactoryMethod(AbstractAutowireCapableBeanFactory.java:1094) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBeanInstance(AbstractAutowireCapableBeanFactory.java:989) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:504) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:302) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:298) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:198) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.getOrderedBeansOfType(EmbeddedWebApplicationContext.java:367) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.getServletContextInitializerBeans(EmbeddedWebApplicationContext.java:233) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext$1.onStartup(EmbeddedWebApplicationContext.java:213) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.createEmbeddedServletContainer(EmbeddedWebApplicationContext.java:164) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.context.embedded.EmbeddedWebApplicationContext.onRefresh(EmbeddedWebApplicationContext.java:132) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    ... 111 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Could not autowire field: private org.springframework.boot.autoconfigure.web.ServerProperties org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration$DispatcherServletConfiguration.server; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'serverProperties': Could not bind properties; nested exception is javax.validation.ValidationException: HV000041: Call to TraversableResolver.isReachable() threw an exception.
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:508) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:87) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessPropertyValues(AutowiredAnnotationBeanPostProcessor.java:289) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    ... 132 common frames omitted
Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'serverProperties': Could not bind properties; nested exception is javax.validation.ValidationException: HV000041: Call to TraversableResolver.isReachable() threw an exception.
    at org.springframework.boot.context.properties.ConfigurationPropertiesBindingPostProcessor.postProcessBeforeInitialization(ConfigurationPropertiesBindingPostProcessor.java:290) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.context.properties.ConfigurationPropertiesBindingPostProcessor.postProcessBeforeInitialization(ConfigurationPropertiesBindingPostProcessor.java:242) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInitialization(AbstractAutowireCapableBeanFactory.java:407) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.initializeBean(AbstractAutowireCapableBeanFactory.java:1545) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.doCreateBean(AbstractAutowireCapableBeanFactory.java:539) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.createBean(AbstractAutowireCapableBeanFactory.java:475) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory$1.getObject(AbstractBeanFactory.java:302) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.DefaultSingletonBeanRegistry.getSingleton(DefaultSingletonBeanRegistry.java:228) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.doGetBean(AbstractBeanFactory.java:298) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.AbstractBeanFactory.getBean(AbstractBeanFactory.java:193) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.findAutowireCandidates(DefaultListableBeanFactory.java:1017) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:960) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:858) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:480) ~[spring-beans-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    ... 134 common frames omitted
Caused by: javax.validation.ValidationException: HV000041: Call to TraversableResolver.isReachable() threw an exception.
    at org.hibernate.validator.internal.engine.ValidatorImpl.isReachable(ValidatorImpl.java:1316) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.hibernate.validator.internal.engine.ValidatorImpl.isValidationRequired(ValidatorImpl.java:1292) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.hibernate.validator.internal.engine.ValidatorImpl.validateConstraint(ValidatorImpl.java:475) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.hibernate.validator.internal.engine.ValidatorImpl.validateConstraintsForDefaultGroup(ValidatorImpl.java:424) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.hibernate.validator.internal.engine.ValidatorImpl.validateConstraintsForCurrentGroup(ValidatorImpl.java:388) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.hibernate.validator.internal.engine.ValidatorImpl.validateInContext(ValidatorImpl.java:340) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.hibernate.validator.internal.engine.ValidatorImpl.validate(ValidatorImpl.java:158) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.springframework.validation.beanvalidation.SpringValidatorAdapter.validate(SpringValidatorAdapter.java:92) ~[spring-context-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.validation.DataBinder.validate(DataBinder.java:746) ~[spring-context-4.0.6.RELEASE.jar:4.0.6.RELEASE]
    at org.springframework.boot.bind.PropertiesConfigurationFactory.validate(PropertiesConfigurationFactory.java:283) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.bind.PropertiesConfigurationFactory.doBindPropertiesToTarget(PropertiesConfigurationFactory.java:278) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.bind.PropertiesConfigurationFactory.bindPropertiesToTarget(PropertiesConfigurationFactory.java:225) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    at org.springframework.boot.context.properties.ConfigurationPropertiesBindingPostProcessor.postProcessBeforeInitialization(ConfigurationPropertiesBindingPostProcessor.java:287) ~[spring-boot-1.1.4.RELEASE.jar:1.1.4.RELEASE]
    ... 147 common frames omitted
Caused by: java.lang.ClassCastException: com.ibm.websphere.persistence.PersistenceProviderImpl incompatible with javax.persistence.spi.PersistenceProvider
    at javax.persistence.Persistence$1.isLoaded(Persistence.java:92) ~[hibernate-jpa-2.0-api-1.0.1.Final.jar:1.0.1.Final]
    at org.hibernate.validator.internal.engine.resolver.JPATraversableResolver.isReachable(JPATraversableResolver.java:56) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.hibernate.validator.internal.engine.resolver.DefaultTraversableResolver.isReachable(DefaultTraversableResolver.java:130) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.hibernate.validator.internal.engine.resolver.SingleThreadCachedTraversableResolver.isReachable(SingleThreadCachedTraversableResolver.java:46) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    at org.hibernate.validator.internal.engine.ValidatorImpl.isReachable(ValidatorImpl.java:1307) ~[hibernate-validator-5.0.3.Final.jar:5.0.3.Final]
    ... 159 common frames omitted
{% endhighlight %}

```java.lang.ClassCastException: com.ibm.websphere.persistence.PersistenceProviderImpl 
```
The exception is due to the JPA 2.0 jars that are shipped with WAS 8.5.5.2. Unfortunately, I couldn't find a way to fix the issue by changing out the default java persistence API as shown below:

![Persistence Provider]({{ site.url }}/images/alternate-persistence-provider-in-WAS.png) 

Since Spring Boot 1.1.4 and starter package ```spring-boot-starter-data-jpa``` contains hibernate JPA 2.1 support, it will collide with the JPA 2.0 jars that ship with WAS 8.5.5.2. There is no quick way to make WAS load your jar files through class loader order ```PARENT_LAST```. 

The only way I could get the Spring Boot with JPA application was to downgrade hibernate jars to version ```hibernate-release-4.2.15.Final``` - especially, ```hibernate-entitymanager-4.2.15.Final.jar``` and ```hibernate-core-4.2.15.Final```.



