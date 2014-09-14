---
layout: post
title: ""Technology Stack for Web Applications""
modified: 2014-09-14 11:22:59 -0400
tags: ["Information Technology & Management, Spring.io, Continuous Delivery, Continuous Integration, Code Quality Management"]
image:
  feature: abstract-6.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---
Choices abound when it comes to developing websites and to managing them efficiently. This paper outlines a set of web technologies that can be used to build out both the client- and server-side components of J2EE-based websites. In addition to specifying the technologies to be used at various tiers, this paper also discusses best practices in structuring the work products for website deployment using continuous delivery concepts.  

##I.    APPLICATION ARCHITECTURE   
We will stick to the typical web application architecture as shown in the following diagram:

![Application Architecture]({{ site.url }}/images/application-architecture.png)

The following section will outline the prescribed technologies for each one of the layers and partitions. 

###A.  Programming Language
All code elements to use Java 1.7+. 

Build scripts, unit, acceptance, and integration tests can be written using Groovy 2.3+ [1].

###B.   Presentation Layer (Client) 
![Application Architecture]({{ site.url }}/images/presentation-layer.png)

This layer is primarily concerned with user interfaces and ways in which they are presented to the end user. We shall use the following technologies for the above-mentioned packages:

####1)  User Interfaces
HTML5, CSS3, CSS Preprocessors such as Less [2], Bootstrap 3.0+

####2)  Presentation Logic (Client)
Ember.js [3]

###C.  Presentation Layer (Server)
This layer is responsible for producing dynamic content from the layers below and for providing support for navigation.

####1)  Presentation Logic (Server) 
Spring Framework, Spring MVC, Tiles 3, JSTL/Thymeleaf [4]

HTML5-compliant email templates can be created with Thymeleaf templating engine [5].

Spring Framework will provide support for dependency injection for controllers, components, resources, repositories, and services [6]. 

####2)  Presentation Models
Presentation Models shall be POJOs (Plain Old Java Objects) with a little bit of behavior in them to support both client- and server-side presentation logic elements. 

###D.  Service Layer
![Application Architecture]({{ site.url }}/images/service-layer.png)

This layer is responsible for providing support for structuring micro-services in terms of service interfaces and implementations, events that are used as data transfer objects between the presentation and the persistence or data tiers. This layer will also house the domain model converters, when needed, to convert domain models to the needs of the presentation tier. 

####1)  Service Interfaces
Service Interfaces will be straightforward Java interfaces. Service implementations would be straightforward Java classes with annotation embedded to aid in dependency injection. For example, service implementations must have @Service annotation to help with Spring Boot auto-wiring support. 

When there is a need to communicate with external systems for data Spring Integration framework [7] could be used to avoid high coupling and low cohesion between various elements of the software under construction.
  
####2)  Event Types
POJOs to facilitate Command Query Responsibility Segregation (CQRS) pattern [8]. 

####3)  Domain Model Converters
Using Java Generics, a domain model converter takes one or more (collections) persistence domain models and converts them to models that are more appropriate for UI presentation aspects. 

####4)  Content Management
![Application Architecture]({{ site.url }}/images/content-management.png)

#####a)  CMA
Content Management Application (CMA) services would provide the building blocks for creating and publishing content for the websites. CMA would make use of NoSQL document stores such as MongoDB to store and retrieve content using Spring Data MongoDB framework.

#####b)  CDA
Content Delivery Application (CDA) services would provide the building blocks for retrieving content from the NoSQL document store. Spring Data REST and Spring HATEOAS would be primarily employed by this package to deliver JSON output using Jackson Data Bind.

###E.  Data Layer
![Application Architecture]({{ site.url }}/images/data-layer.png)

This layer shall use Spring Data primarily. 

####1)  Domain Models
POJOs annotated with @Entity (javax.persistence.*) when using Spring Data JPA [9] or QueryDSL [10]. When using NoSQL document stores, use Spring Data MongoDB [11].

Components using Spring Data REST [12] would have appropriate “Entity Link” objects for JSON output processing using Jackson Data Bind [13].

###F.  Data Sources
![Application Architecture]({{ site.url }}/images/data-sources.png)

Traditional databases with SQL support and NoSQL document stores could be employed to manage data for the websites. Corporate data such as Customer records and their relationships could be captured using the traditional databases such as Oracle or MySQL DBMS. Website content, that is unstructured in nature, could be stored in the NoSQL document stores such as MongoDB [14].

###G.  External Systems
Systems that are not directly developed to handle the websites would be considered external and therefore be accessed with “Integration Patterns” [15] in mind. Spring Integration framework provides implementations using POJOs for most of the architectural patterns such as endpoint, aggregator, filter, transformer, control bus [7] etc.

###H.  Crosscutting Concerns
This partition is responsible for providing support for all layers in terms of security, tests, and operational management.

![Application Architecture]({{ site.url }}/images/crosscutting-concerns.png)

####1)  Operational Management
Spring Boot [16] will be used to simplify the bootstrapping and development of the websites. All configuration information must be captured using Java Configuration classes annotated with @Configuration and @Profile as opposed to using XML configurations. 

Auditing, trace, health, and metrics of the websites can be customized using the Spring Boot Actuator module [17].

Database Migrations can be accomplished by Flyway database migration tool [18].

####2)  Security
Spring Security [19] to be utilized as a server side framework along with HDIV [20]. Since there is an enterprise solution in this area, a decision in this area will be detailed out at a later time.

####3)  Tests
Unit, Acceptance and Integration tests for the server-side components will be developed using (browser test automation and specification frameworks) Spock [21], Geb [22] in Groovy [1]. 

###I.  Summary
The following picture depicts all layers with interactions:

![Application Architecture]({{ site.url }}/images/tech-stack.png)

##II. SOFTWARE DEVELOPMENT PROCESS
The technology stack for the various areas of the software development process is outlined in the following section.

###A.  Project Management
Basecamp, Pivotal Tracker [23], and MS Project would be used as project management tools.

###B.  Integrated Development Environment (IDE)
IntelliJ IDEA [24] or Spring STS [25] would be used to develop code structures.

###C.  Code Version Management 
Git [26] and GitHub [27] would be used as the version control system and remote private code repository respectively.

###D.  Build & Dependency Management
Gradle [28] would be used as the polyglot build and dependency management tool. Spring IO platform [29]  must be used in the build.gradle file to control the versions of the dependencies of Spring framework packages.

###E.  Code Quality Management aka. Automated Code Reviews
Code quality will be determined in terms of coding standards, overly complex and inefficient code, potential bugs, and bad coding practices. CheckStyle [30], PMD [31], CodeNarc [32] for Groovy [1], FingBugs [33], and JDepend [34] would be used to measure the code quality metrics. JaCoCo [35] would be used to measure code coverage. 

SonarQube [36] would be used to visualize, aggregate, and monitor code quality metrics over a period of time. 

###F.  Artifact Assembly and Publishing
As the need arises, JFrog Artifactory [37] or Sonatype Nexus [38] would be considered.

###G.  Continuous Integration
Jenkins [39] would be used as the continuous integration [40] server to automate the stages in build pipeline as shown below:

![Application Architecture]({{ site.url }}/images/cd-ci-pipeline.png)

User Acceptance Test (UAT) and Production stages would be triggered manually.

##III.    CONCLUSION
This paper discussed a wide range of technologies to develop and manage  websites efficiently. Though the technologies put forth in this paper will fit the application architecture and the development process, it is strongly recommended for the development team to be always on the lookout for better technology elements and incorporate them as it sees fit as the architecture emerges naturally as the website development efforts move forward.

---
[1] “Groovy - Documentation.” [Online]. Available: http://groovy.codehaus.org/Documentation. [Accessed: 06-Sep-2014].

[2] “Getting started | Less.js.” .

[3] “Ember.js - Guides and Tutorials: Ember.js Guides.” [Online]. Available: http://emberjs.com/guides/. [Accessed: 06-Sep-2014].

[4] “Thymeleaf Page Layouts - Thymeleaf: java XML/XHTML/HTML5 template engine.” [Online]. Available: http://www.thymeleaf.org/doc/layouts.html. [Accessed: 06-Sep-2014].

[5] “Rich HTML email in Spring with Thymeleaf - Thymeleaf: java XML/XHTML/HTML5 template engine.” [Online]. Available: http://www.thymeleaf.org/doc/springmail.html. [Accessed: 06-Sep-2014].

[6] “Spring Framework Reference Documentation.” [Online]. Available: http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#new-in-4.0. [Accessed: 06-Sep-2014].

[7] “Spring Integration.” [Online]. Available: http://projects.spring.io/spring-integration/. [Accessed: 03-Sep-2014].

[8] M. Fowler, “CQRS.” [Online]. Available: http://martinfowler.com/bliki/CQRS.html. [Accessed: 03-Sep-2014].

[9] “Spring Data JPA.” [Online]. Available: http://projects.spring.io/spring-data-jpa/. [Accessed: 06-Sep-2014].

[10]    “Querydsl - Reference Documentation.” [Online]. Available: http://www.querydsl.com/static/querydsl/3.4.3/reference/html/. [Accessed: 06-Sep-2014].

[11]    “Spring Data MongoDB.” [Online]. Available: http://projects.spring.io/spring-data-mongodb/#quick-start. [Accessed: 06-Sep-2014].

[12]    “Spring Data REST.” [Online]. Available: http://projects.spring.io/spring-data-rest/#quick-start. [Accessed: 06-Sep-2014].

[13]    “JacksonHome - FasterXML Wiki.” [Online]. Available: http://wiki.fasterxml.com/JacksonHome. [Accessed: 06-Sep-2014].

[14]    “MongoDB.” [Online]. Available: http://www.mongodb.org/. [Accessed: 06-Sep-2014].

[15]    “Home - Enterprise Integration Patterns.” [Online]. Available: http://www.eaipatterns.com/. [Accessed: 03-Sep-2014].

[16]    “Getting Started · Building an Application with Spring Boot.” [Online]. Available: https://spring.io/guides/gs/spring-boot/. [Accessed: 06-Sep-2014].

[17]    “Spring Boot Reference Guide.” [Online]. Available: http://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#production-ready. [Accessed: 06-Sep-2014].

[18]    “Get Started - Flyway • Database Migrations Made Easy.” [Online]. Available: http://flywaydb.org/getstarted/. [Accessed: 06-Sep-2014].

[19]    “Spring Security Reference.” [Online]. Available: http://docs.spring.io/autorepo/docs/spring-security/4.0.0.CI-SNAPSHOT/reference/htmlsingle/. [Accessed: 06-Sep-2014].

[20]    “HDIV: Java Web Application Security Framework.” [Online]. Available: http://hdiv.org/. [Accessed: 06-Sep-2014].

[21]    “Spock Framework Reference Documentation — Spock 1.0-SNAPSHOT.” [Online]. Available: http://docs.spockframework.org/en/latest/. [Accessed: 06-Sep-2014].

[22]    “Geb - Very Groovy Browser Automation.” [Online]. Available: http://www.gebish.org/. [Accessed: 06-Sep-2014].

[23]    “Pivotal Tracker: Getting Started.” [Online]. Available: https://www.pivotaltracker.com/help/gettingstarted. [Accessed: 06-Sep-2014].

[24]    “IntelliJ IDEA :: Features.” [Online]. Available: http://www.jetbrains.com/idea/features/. [Accessed: 06-Sep-2014].

[25]    “Tools.” [Online]. Available: http://spring.io/tools. [Accessed: 06-Sep-2014].

[26]    “Git - Git Basics.” [Online]. Available: http://git-scm.com/book/en/Git-Basics. [Accessed: 06-Sep-2014].

[27]    “About.” [Online]. Available: https://github.com/about. [Accessed: 06-Sep-2014].

[28]    “Gradle - Overview.” [Online]. Available: http://www.gradle.org/overview. [Accessed: 06-Sep-2014].

[29]    “Spring IO Platform.” [Online]. Available: http://spring.io/platform. [Accessed: 06-Sep-2014].

[30]    “checkstyle - Checkstyle 5.7.” [Online]. Available: http://checkstyle.sourceforge.net/. [Accessed: 06-Sep-2014].

[31]    “PMD – Welcome to PMD.” [Online]. Available: http://pmd.sourceforge.net/pmd-5.1.3/. [Accessed: 06-Sep-2014].

[32]    “CodeNarc -.” [Online]. Available: http://codenarc.sourceforge.net/. [Accessed: 06-Sep-2014].

[33]    “FindBugsTM - Find Bugs in Java Programs.” [Online]. Available: http://findbugs.sourceforge.net/. [Accessed: 06-Sep-2014].

[34]    “JDepend.” [Online]. Available: http://clarkware.com/software/JDepend.html. [Accessed: 06-Sep-2014].

[35]    “EclEmma - Using the Coverage View.” [Online]. Available: http://www.eclemma.org/userdoc/coverageview.html. [Accessed: 06-Sep-2014].

[36]    “SonarQubeTM » Features.” [Online]. Available: http://www.sonarqube.org/features/. [Accessed: 06-Sep-2014].

[37]    “Artifactory – The Open Source Repository Manager by JFrog.” [Online]. Available: http://www.jfrog.com/open-source/#os-arti. [Accessed: 06-Sep-2014].

[38]    “Sonatype.org: Nexus.” [Online]. Available: http://www.sonatype.org/nexus/. [Accessed: 06-Sep-2014].

[39]    “Meet Jenkins - Jenkins - Jenkins Wiki.” [Online]. Available: https://wiki.jenkins-ci.org/display/JENKINS/Meet+Jenkins. [Accessed: 06-Sep-2014].

[40]    “Continuous Integration.” [Online]. Available: http://www.martinfowler.com/articles/continuousIntegration.html. [Accessed: 06-Sep-2014].







