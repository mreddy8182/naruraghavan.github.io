---
layout: post
title: "Public Cloud for Corporate Web Applications - Part 4"
modified: 2013-08-01 20:17:31 -0400
tags: [Information Technology & Management, Security Management, Defensive Hacking, Cloud Computing]
image:
  feature: abstract-5.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---
Continuing from where we left off in [**Part 3**]({{ site.url }}/public-cloud-for-corporate-web-applications-part-3), lets start exploring the Software Engineering for the PaaS Service Model in this post.

### B.  Software Engineering for the PaaS Service Model

The software development environment has to lend itself to managing a wide variety of artifacts that emerge from a multitude of software life cycle management activities such as requirements management, change management, configuration management, build management, integration management, release management, and test management in the areas of define, design, code, test, and run [25]. In order to succeed in the security endeavor, there is a greater need to equip the development team with necessary software tools to enable complete tracing from requirements to code releases.

The following diagram illustrates the overall development environment with tasks organized in terms of Development, QA, Automated, and Security:

![Software Engineering for PaaS Service Model]({{ site.url }}/images/software-engineering-for-paas-service-model.png)

#### 1)  Development aka. Scrum Team Tasks

 ![]({{ site.url }}/images/dev-team-1.png)Product Owner creates user stories and epics in the Agile Requirements Management Server. Developers, Security experts, and the Scrum master break the user stories and epics into multiple tasks and assign appropriate estimates in terms of story points. One convention that the team could follow is to equate one story point to one ideal man-day – A day of development that can be done when there are no external distractions such as meetings, discussions etc. Based on the team’s velocity, appropriate number of stories is selected to be part of the Sprint. The team along with the security expert determines the number of security-focused stories to be included in the Sprint based on the business requirements. Each story will have a “How to Demo” field populated to assist the QA team in properly testing the various functionalities of the software component.

 ![]({{ site.url }}/images/dev-team-2.png)The development team is responsible for periodically checking in/syncing/rebasing their code into the enterprise version control system. The developer as well as the code reviewer must look for the successful completion of static code analysis. The development team is also responsible for setting up and configuring the various life cycle management servers such as the version control system, continuous integration server, dependency management, agile requirements management server etc. From time to time, the development team will also be responsible for peer code reviews, refactoring of code to improve performance and other non-functional architectural requirements. An automated code crawler that reviews the code for vulnerabilities such as OWASP Code Crawler is highly recommended over manual code reviews. Using the “defense in depth” principles, more than one type of code reviews can be done. The first pass could be the automatic code inspection through code crawlers as stated above and the second pass could be a manual code review by a peer or a security expert.


#### 2)  QA Team Tasks

 ![]({{ site.url }}/images/qa-team-1.png)The QA team is responsible for performing the acceptance testing and reporting of the results to the development team. The acceptance testing will include both functionality and security-focused testing according to SAFECode stories. The acceptance testing is typically carried out in the Test Server as shown in the above diagram. The QA team is also responsible for configuring and writing test plans for the automated test server such as Selenium. 

 ![]({{ site.url }}/images/qa-team-2.png)The QA team is also responsible for performing tests against the QA and production environments on the public cloud as shown in the above diagram.

#### 3)  Security Team Tasks

 ![]({{ site.url }}/images/security-team-1.png)One of the major tasks of the security team is to train the scrum team on security practices that will successfully implement the stipulated security controls. The security team is also responsible for installing and configuring the necessary static code analysis tools, malware scanners, and code vulnerability scanners that can be automatically run upon on a build. The Continuous Integration Server will not deploy if the successful build failed one of the security tests such as static code analysis etc.

 ![]({{ site.url }}/images/security-team-2.png)The security team is also responsible for securing the code deployment to the public cloud in terms of network equipment such as Firewalls, Intrusion Detection Systems etc. There is a need to set up firewall rules in terms of connectivity guidelines from the PaaS service provider such as protocol (HTTPS), ports, time of day restrictions etc.

 ![]({{ site.url }}/images/security-team-3.png)The security team has to work with the PaaS broker or provider to set up the necessary SLA and incorporate provisions for security controls as described in the risk management framework section. It is this team’s responsibility to ensure that the “continuous monitoring” strategy is established and followed upon at a time interval stipulated in the SLA.

#### 4)  Automated Tasks

 ![]({{ site.url }}/images/auto-task-1.png)Establishing a Continuous Integration (CI) Server is well worth the time and money. It makes a lot of sense for the development team to carve some time out to establish a version control system such as Atlassian Stash along with a CI Server (if there are none) with deployment plans such as Test, QA, and Production with appropriate build plans. As can be seen in Figure 16, the CI Server, according to the build/deployment plans setup, checks out the necessary source code from the version control system and executes those plans. Based on the deployment plan, it deploys the software component on Test, QA, or Production server as prescribed.

 ![]({{ site.url }}/images/auto-task-2.png)The integration server, such as Atlassian Bamboo, deploys the software component on the Test server. It then notifies the QA team to start the acceptance testing for the software component.

 ![]({{ site.url }}/images/auto-task-3.png)The CI Server deploys the software component to the automated test server such as Selenium. The automated tests will also include security-focus tests such as code vulnerability testing, static code analysis, and running of malware scanners. These tools upon successfully completing their tests would notify the QA, security, and development teams with results.

 ![]({{ site.url }}/images/auto-task-4.png)The CI Server deploys the software component in the PaaS environment on the public cloud. The deployment could be targeted for either the QA or the production environment. The server notifies the QA team to initiate testing on the QA or production stage.

I'll contiunue my discussion in the next post.


 ---
 [25]   M. Hüttermann, Agile ALM : lightweight tools and Agile strategies. Shelter Island, N.Y.: Manning, 2012.

