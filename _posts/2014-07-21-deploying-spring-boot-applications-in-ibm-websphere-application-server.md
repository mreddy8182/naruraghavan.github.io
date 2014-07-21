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