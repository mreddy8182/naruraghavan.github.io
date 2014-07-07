---
layout: post
title: "Public Cloud for Corporate Web Applications - Part 1"
modified: 2013-07-07 12:35:57 -0400
tags: [Information Technology & Management, Security Management, Defensive Hacking]
image:
  feature: abstract-5.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---
What does “going to the public cloud” mean to organizations?  Other than the benefits such as reduced time to market, less upfront cost, rapid elasticity in terms of hardware and software resources, there are several security implications associated with such a move. 

Today’s corporations not only create and manage web applications to make their corporate presence known on the Internet, but also invite their employees, suppliers, and dealers/retailers to conduct business online. At this point, the question becomes “Where should an organization stop or draw the line when making decisions to go to the pubic cloud?” Because the case for public-facing web applications going to the public cloud has already been made. The purpose of the paper is to discuss the viability of engaging public clouds for all web applications irrespective of the user segment that they support. This paper will also outline a development process and an environment for successfully deploying secure web applications on the public cloud.


## I.	INTRODUCTION
***Cloud*** is a model for enabling convenient, on-demand network access to a shared pool of configurable computing resources (For example, networks, servers, storage, applications, and services) that can be rapidly provisioned and released with minimal management effort or service provider interaction [1].  
![NIST Definition of the Cloud]({{ site.url }}/images/nist-definition-of-the-cloud.png)
{: .pull-right}

*Infrastructure as a Service (IaaS)* – In this cloud service model, the provider offers raw computing power, storage, and network infrastructure so that the consumer can load his/her own software components such as operating systems, applications, etc. in this infrastructure.

*Platform as a Service (PaaS)* - PaaS builds on IaaS and adds a set of software components such as operating systems, middleware products (application servers), and Web servers so that the consumer can develop and deploy applications in this platform.

*Software as a Service (SaaS)* - SaaS builds on top of PaaS and adds fully developed applications such as billing, collaboration, digital asset management, web content management, customer relationship management, point of sale applications to name a few. 

To keep the scope manageable, this paper will only discuss SaaS and PaaS service models that are deployed using the public deployment model as highlighted in Figure 1. In the SaaS service model, an organization does not manage or control either the underlying infrastructure or the application that is deployed in the infrastructure. In the PaaS service model, the organization does not control or manage the underlying infrastructure, but does have control over the applications and data that are deployed in the infrastructure [2] as shown below:
![Who Manages What?]({{ site.url }}/images/who-manages-what.png)
{: .pull-right}

Public Cloud means that both the underlying platform (PaaS) and the applications (SaaS) that are hosted on the infrastructure are available to the general public and is owned and/or operated by an organization selling those cloud services. 

### A.	Roles and responsibilities in the Cloud
In order to study the security implications of using the public cloud for the intended service models, we have to understand the roles and responsibilities of the major actors in the cloud-computing domain [3] as shown below:
![Roles and Responsibilities]({{ site.url }}/images/roles-and-responsibilities.png)
{: .pull-right}

#### 1)	Cloud Consumer
A cloud consumer is an organization that makes use of the services provided by the cloud providers and brokers.

#### 2)	Cloud Provider
A cloud provider is an organization or entity that offers one or more cloud service models (IaaS, PaaS, SaaS) via one or more deployment models (public, private, hybrid) to interested parties. 

#### 3)	Cloud Broker
As the application needs increase, the service models could get extremely complex. At this juncture, the cloud consumer could seek the help of a broker instead of going directly to the cloud provider. The cloud broker’s prime responsibility is to aggregate, enhance, or augment the services provided by one or more cloud service providers as shown in the diagram [3] below:
![Cloud Broker Model]({{ site.url }}/images/cloud-broker-model.png)
{: .pull-right}
In the above scenario, the cloud consumer does not need to know about the underlying cloud providers since the service level agreement (SLA) will be created between the consumer and the broker. The cloud broker in turn will have SLAs with the cloud providers to satisfy the service levels that were committed to the consumer.

#### 4)	Cloud Auditor
The other important actor from the reference model is the cloud auditor. A cloud consumer, broker, or provider could employ an auditor to verify the security controls, privacy impact, and performance of services provided by the cloud provider.  

#### 5)	Cloud Carrier
A cloud carrier is an intermediary that provides connectivity and transport of cloud services from the cloud provider to the consumer.
![Cloud Carrier Model]({{ site.url }}/images/cloud-carrier-model.png)
{: .pull-right}
In the above scenario [3], a cloud provider could seek the help of a cloud carrier to provide dedicated and/or encrypted connections between the provider’s services and the consumer.

## II.	SECURITY IMPLICATIONS
As shown in Figure 1, more control or management of components means responsibility for security controls. In the PaaS environment, since the cloud consumer is responsible for building and deploying applications, layering in an additional level of security controls at the application level falls under the cloud consumer. By the same token, the cloud provider is responsible for establishing security controls for all the other components including the middleware environment. Since there is very little extensibility option available for the consumer in the SaaS model, the provider bares responsibility for establishing security controls for the entire environment. 

### A.	Multi-tenancy
One of the major characteristics of cloud computing is this idea of multi-tenancy. Multi-tenancy implies the use of same applications and resources by multiple cloud consumers that may belong to multiple organizations. Multi-tenancy allows cloud providers to provide economies of scale, availability, segmentation, isolation, and operational efficiency to their consumers [4].  Since PaaS builds on the infrastructure provided by IaaS and SaaS builds on PaaS, there is quite a bit of technology components that work together to enable multi-tenancy as shown in the following diagram:
![Multi-tenancy Model 1]({{ site.url }}/images/multi-tenancy-1.png)
{: .pull-right}
In the case of IaaS, the same hardware resources are shared across multiple consumers using virtualization technologies as shown in the diagram below:
![Multi-tenancy Model 2]({{ site.url }}/images/multi-tenancy-2.png)
{: .pull-right}

##### a)	Virtual Machine Manager (VMM) also known as Hypervisor
VMM is the virtualization layer that runs on top of a server platform and manages CPU, memory, hard disk, network, and other hardware resources of the server host [5]. 

##### b)	VM
A Virtual Machine (VM) is a software implementation of a physical computing environment in which the operating system and other application programs can be installed and run [5]. VMs emulates a physical computing environment and requests hardware resources such as CPU, memory etc. from the VMM. Some of the most common virtualization solutions in the market today include Xen, VMware, Hyper-V, and Oracle. 

In the case of PaaS, middleware and other components could be installed and run within the VM. In the case of SaaS, a wide variety of applications can be installed and run on top of the middleware that is running within the VM. Therefore, the cloud consumer of PaaS or SaaS ends up in a virtual machine as shown in Figure 5. Using the virtualization technology, cloud providers could virtualize the hardware/platform, desktop, software, memory, storage, data, and network to successfully orchestrate services such as IaaS, PaaS, and SaaS.

#### 2)	Data Breaches and Shared technology threats
One of the security challenges of the virtualization technology is how isolated these individual virtual machines are from each other. In other words, ensuring sufficient isolation between VMs is the security challenge. As shown in Figure 5, a security compromise in the virtualization layer means that all the VMs running on the virtualized layer are also compromised thereby resulting in data breaches. The Cloud Security Alliance (CSA) has elevated this threat ranking from 5 to 1 in 2013 [6]. There are several articles on the Internet such as [7] that demonstrate as to how a malicious VM could extract fine-grained information from a victim’s VM running on the same physical computer. There are also vulnerabilities present in the virtualization technology that could compromise the entire fleet of VMs running on the VMMs [8]. One of the articles, [9], demonstrates the use of VASTO (Virtualization Assessment Tool) within the Metasploit framework (MSF) to compromise VMware hosts with dictionary-based brute force login, man-in-the-middle (MITM) attack etc. VASTO in conjunction with MSF could be used to compromise hosts that are running Xen, Hyper-V, and Oracle VMMs as well. 

##### a)	Controls
With the help of the cloud provider, a third-party cloud auditor could be employed to assess as to how secure or vulnerable the target virtual environment is. There are several tools such as VASTO, Netwrix, Blackbird Auditor, and VMinformer that could be successfully employed to audit the virtual environment in terms of client, hypervisor, support, management, and internal components. These tools check for the presence of misconfiguration, missing security patches, bad network scheme, weaknesses in the VMM layer, and Storage misconfiguration [9].  

#### 3)	More Threats
Along with data breaches and shared technology threats as outlined in the previous section, there are several other threats such as data loss, account or service traffic hijacking, insecure interfaces and APIs by the cloud provider, denial of service, malicious insiders, and insufficient due diligence [6].

##### a)	Controls
To combat the various threats – see previous paragraph, establishment of a proper organization-wide risk management framework is essential. This is also part of doing due diligence that a cloud consumer is expected to carry out before embarking on the journey to move to the public cloud. Along with a solid risk management framework, there is also a greater need for securing web applications against a wide variety of attacks such as cross-site scripting (XSS), thread safety, access control, hidden form field manipulation, parameter manipulation, weak session cookies, SQL injection, insecure Web services, and Fail open authentication attacks [10]. Next set of sections will discuss the risk management framework along with a model for engineering secure web applications by taking the Open Security Architecture’s cloud computing security pattern, NIST’s cloud computing documents, CSA’s security reference model, OWASP’s web application security guidelines, SAFECode guidelines for Agile process frameworks into account. 

I'll contiunue my discussion in the next post.

---
[1]	NIST. (2011). The NIST Definition of Cloud Computing. Available: http://csrc.nist.gov/publications/nistpubs/800-145/SP800-145.pdf

[2]	B. Lyons. (2011). Applying a Holistic Defense-in-Depth approach to the Cloud. Available: http://www.niksun.com/presentations/day1/NIKSUN_WWSMC_July25_BarryLyons.pdf

[3]	NIST. (2011). NIST Cloud Computing Reference Architecture. Available: http://www.nist.gov/customcf/get_pdf.cfm?pub_id=909505

[4]	CSA. (2011). Security Guidance for Critical Areas of Focus in Cloud Computing V3.0 : Cloud Security Alliance. Available: https://cloudsecurityalliance.org/download/security-guidance-for-critical-areas-of-focus-in-cloud-computing-v3

[5]	A. Desai. (2011). What is virtual machine (VM)? - Definition from WhatIs.com. Available: http://searchservervirtualization.techtarget.com/definition/virtual-machine

[6]	CSA. (2013). The Notorious Nine: Cloud Computing Top Threats in 2013 : Cloud Security Alliance. Available: https://cloudsecurityalliance.org/download/the-notorious-nine-cloud-computing-top-threats-in-2013

[7]	Y. Zhang, A. Juels, M. K. Reiter, and T. Ristenpart. (2012). Cross-VM side channels and their use to extract private keys. Available: http://doi.acm.org/10.1145/2382196.2382230

[8]	M. Schwartz. (2012). New Virtualization Vulnerability Allows Escape To Hypervisor Attacks -- InformationWeek. Available: http://www.informationweek.com/news/security/app-security/240001996

[9]	S. Chauhan. (2012). Virtualization Security: Hacking VMware with VASTO. Available: http://resources.infosecinstitute.com/virtualization-security

[10]	OWASP. (2013). Category:OWASP WebGoat Project - OWASP. Available: https://http://www.owasp.org/index.php/Category:OWASP_WebGoat_Project



