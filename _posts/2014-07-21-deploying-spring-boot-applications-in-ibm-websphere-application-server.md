---
layout: post
title: "Deploying Spring Boot Applications in IBM WebSphere Application Server"
modified: 2014-07-21 15:29:49 -0400
tags: [DevOps,Software Engineering,Technology Operations,Release Management,WebSphere Application Server,WebSphere Liberty Profile]
image:
  feature: abstract-10.jpg
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
  <application id="Gradle___ContentRestServices___ncrs_war__exploded_" location="/Users/Naru/IdeaProjects/ContentRestServices/out/artifacts/ContentRestServices/exploded/ncrs.war" name="Gradle___ContentRestServices___ncrs_war__exploded_" type="war" />
</server>
{% endhighlight %}

Since Spring Boot starter package for web ```spring-boot-starter-web``` uses embedded tomcat by default, I ended up specifying the following in my ```build.gradle``` file as follows:

```
providedRuntime("org.springframework.boot:spring-boot-starter-tomcat")
```

The ```providedRuntime``` directive makes sure that the war plugin moves the embeeded tomcat jar files to ```/WEB-INF/lib-provided``` from ```/WEB-INF/lib``` directory. 

>>>The premise of Spring Boot is to develop and run Spring applications as standalones - extremely good for development teams. 

If you build the war file Without the above directive, the embedded tomcat jars will also be placed along with the other jar dependencies in the ```/WEB-INF/lib``` directory. [Here](https://github.com/spring-projects/spring-boot/issues/1194) is the question I posed to the Spring Boot team to get my application war file to work in a regular Tomcat or WLP environment.

Once the above situation with the application war file was fixed, my application deployed and ran fine within the WLP environment. After successfully getting the application to work in WLP environment, I moved onto deploying it in my test WAS 8.5.5.2 environment. To my surpise, the application did not even deploy properly. I had to perform a bunch of steps to get my REST application to work.

## Deploying on WebSphere Application Server (WAS)  
The key to a successful deployment of Spring Boot applications in WAS is to setup a shared library as shown below:

![Shared Library Setup in WAS]({{ site.url }}/images/WAS-shared-library-setup.png) 

I removed all the jar files under ```/WEB-INF/lib```  from my war file and placed them into the shared library. After setting up the shared library, I changed the class loader order as follows:

![Class Loader Order]({{ site.url }}/images/class-loader-order.png)

Though the application started, I got hit with JPA persistence issues as follows:

{% highlight java %}
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

I could get the Spring Boot with JPA application to work was to downgrade hibernate jars to version ```hibernate-release-4.2.15.Final```.

