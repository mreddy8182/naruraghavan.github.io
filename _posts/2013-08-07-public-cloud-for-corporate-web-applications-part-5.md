---
layout: post
title: "Public Cloud for Corporate Web Applications - Part 5"
modified: 2013-08-07 20:33:22 -0400
tags: [Information Technology & Management, Security Management, Defensive Hacking, Cloud Computing]
image:
  feature: abstract-5.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---
Continuing from where we left off in [**Part 4**]({{ site.url }}/public-cloud-for-corporate-web-applications-part-4), lets start exploring the Secure Development Practices in this post.

### C.  Employing Secure Development Practices

Establishing a set of security libraries to do input and output validation, error handling, authentication and authorization, session management, secure resource access, secure communications, and secure storage [26] will help implement several security controls when moving to the cloud. Using a standard library will help contain the proliferation of inconsistent security implementations across teams for some known vulnerabilities. For example, OWASP Java Encoder library [27] could be used to implement secure coding requirements such as input and output validation and combat against a whole host of vulnerabilities such as cross-site scripting (XSS), SQL injection attacks etc.

#### 1)  Input Validation

Developers have to ensure that the data received is a valid input for the software component in question. There are four techniques to make sure that the data received is valid as follows:

* Whitelisting input
  
  Using this technique, the developer defines a set of known input values. For example, a regular expression used for whitelisting an email address as input could be: 

  {% highlight java %}

Boolean isGoodEmail = Pattern.matches(email,”^([a-zA-Z0-9_-.]+)@(([[0-9]{1,3}.[0-9]{1,3}.[0-9]{1,3}.)|(([a-zA-Z0-9-]+.)+))([a-zA-Z]{2,4}|[0-9]{1,3})(]?)$”);

  {% endhighlight %}

* Blacklisting input
Based on some known vulnerabilities, the developer could actively check for combinations of characters that are usually signs of cross-site scripting and remove them as follows:

  {% highlight java %}

String clean_input = bad_input.replaceAll(Pattern.quote(“’”));

  {% endhighlight %}

* Canonicalization of Input
Canonicalization is an operation by which the input (that could have been encoded by a malicious user to send scripts via input parameters) is reduced down to its simple form. 

{% highlight java %}
String canonicalized_input = ESAPI.canonicalize(input);
{% endhighlight %}

* Encoding/Escaping of Input
OWASP has published a library of Java methods [27, 28] to successfully encode or escape input data so that the input doesn’t contain any commands/scripts in them. For example, to ensure that there is no SQL scripting in the input, one could use the ESAPI as follows:

{% highlight java %}
String db_query_input = ESAPI.encodeForSQL(DB2_Codec, input);
{% endhighlight %}

In the above code, by using the right codec for a database such as MySQLCodec, OracleCodec etc., the developer could potentially combat against SQL Injection Attacks. By maintaining a parameter-mapping table as shown below, a developer could apply necessary techniques to properly validate the input data.

![Parameter Specs for Input Validation]({{ site.url }}/images/parameter-specs-for-input-validation.png)

#### 2)  Output Validation

Cross-site scripting attacks originate through poorly validated output of a web application. Attackers often use this technique to orchestrate phishing attacks and extract valuable information from unsuspecting end user. Depending upon where the output is going to end up on a web page, developers have to choose the correct encoding technique. For example, if the output is going to end up as a URL, the output has to be encoded with URL special characters such as “?” and “&” etc. These characters have to be properly encoded with “%”. Once again, developers must start with a parameter table as follows:

![Parameter Specs for Output Validation]({{ site.url }}/images/parameter-specs-for-output-validation.png)

By using ESAPI (or a similar framework), encode the output as follows:

{% highlight java %}
String safe_output_url = ESAPI.encodeForURL(output);
{% endhighlight %}

By the same token, output destined to go within the body of the web page, use the following technique:

{% highlight java %}
String safe_output_html = ESAPI.encodeForHTML(output);
{% endhighlight %}

Using the above technique, the “output” will be properly encoded to convert special HTML characters such as “<” to “&lt;” etc. 

On a similar note, other than URL (PercentCodec) and HTML (HTMLCodec) encoding techniques, there are

JavaScriptCodec, CSSCodec, and VBScriptCodec etc. to encode the output to go within a JavaScript, CSS, or VBScript.

##### a)  Why do we need to do input validation at the server side?
Relying on client-side input validation is not recommended. By setting up a proxy such as WebScarab or BurpSuite etc., an attacker can easily bypass client-side validation completely as shown in the screenshot below:

![Modification of Input using WebScarab]({{ site.url }}/images/modification-of-input-using-webscarab.png)

Using these client-side proxy tools, an attacker will be able modify the HTTP header variables, change the values of query parameters, and add new query parameters if necessary. Therefore, it is not enough to just rely on client-side validations.

#### 3)  Error Handling

This is the most important step a developer has to pay special attention to. Attackers formulate and fine-tune their attack vectors by inducing error conditions and see how the web application reacts to those errors. When an error is not properly handled, oftentimes, the error message gets displayed on the web page. These error messages provide vital clues to the attackers as to the type of operating system, database, web server, application server, and authentication server that the web application uses or is built with. For example, ESAPI has several predefined exceptions that can be thrown and handled in the code. Some of the exceptions include EncodingException, IntrusionException, and EnterpriseSecurityException etc.

#### 4)  Secure Storage

As discussed in the responsibility chart of various cloud actors, securing the storage of certain sensitive information such as password, credit card numbers, personal identification number (PIN) etc. falls under the cloud consumer’s responsibility. Therefore, the development team from the consumer side has to ensure that appropriate techniques such as one-way hashing, encryption of sensitive information are performed in an effective manner. There are several industrial strength cryptographic packages available such as Java Cryptographic Extension (JCE), Bouncy Castle etc. that the development team could easily incorporate into their code to enable secure storage of sensitive information on the cloud.

For example, ESAPI provides Java methods that enable users to either hash or encrypt sensitive information. The difference is that hashed information cannot be decrypted and is ideal for password storage. On the other hand, encrypted data can be decrypted using a key and this technique is useful for securing the other sensitive information such as credit card numbers etc. 

{% highlight java %}
String hashed_password = ESAPI.hash(plaintext_password, salt);
String encrypted_credit_card = ESAPI.encrypt(plaintext_cc);
{% endhighlight %}

What is interesting to note is the use of “salt” in the above hashing code. A salt is used as an additional input to a one-way hash function to strengthen the randomness of the resulting ciphertext. A good “salt” will combat against dictionary attacks and rainbow-table attacks. Since the hashes are randomized effectively by the use of the salt, the attackers are forced to change the dictionary/table constantly for comparison.

A similar technique must be employed to securely store property files. The property files are used in almost all of the layers and partitions of a web application. 

#### 5)  Secure Communications

Web pages that take any type of sensitive information, as input must be sent over a secure communication layer such as Transport Layer Security (TLS) or the Secured Socket Layer (SSL). When using SSL, the use of SSLv3 is recommended over SSLv2, as SSLv2 is not considered secure. 

By following these security principles along with a pre-fabricated security library such as ESAPI [28], developers can exercise secure coding practices and successfully contribute towards implementing the security controls that the cloud consumer is responsible for.  

I'll contiunue my discussion in the next post.

---
[1] NIST. (2011). The NIST Definition of Cloud Computing. Available: http://csrc.nist.gov/publications/nistpubs/800-145/SP800-145.pdf

[2] B. Lyons. (2011). Applying a Holistic Defense-in-Depth approach to the Cloud. Available: http://www.niksun.com/presentations/day1/NIKSUN_WWSMC_July25_BarryLyons.pdf

[3] NIST. (2011). NIST Cloud Computing Reference Architecture. Available: http://www.nist.gov/customcf/get_pdf.cfm?pub_id=909505

[4] CSA. (2011). Security Guidance for Critical Areas of Focus in Cloud Computing V3.0 : Cloud Security Alliance. Available: https://cloudsecurityalliance.org/download/security-guidance-for-critical-areas-of-focus-in-cloud-computing-v3

[5] A. Desai. (2011). What is virtual machine (VM)? - Definition from WhatIs.com. Available: http://searchservervirtualization.techtarget.com/definition/virtual-machine

[6] CSA. (2013). The Notorious Nine: Cloud Computing Top Threats in 2013 : Cloud Security Alliance. Available: https://cloudsecurityalliance.org/download/the-notorious-nine-cloud-computing-top-threats-in-2013

[7] Y. Zhang, A. Juels, M. K. Reiter, and T. Ristenpart. (2012). Cross-VM side channels and their use to extract private keys. Available: http://doi.acm.org/10.1145/2382196.2382230

[8] M. Schwartz. (2012). New Virtualization Vulnerability Allows Escape To Hypervisor Attacks -- InformationWeek. Available: http://www.informationweek.com/news/security/app-security/240001996

[9] S. Chauhan. (2012). Virtualization Security: Hacking VMware with VASTO. Available: http://resources.infosecinstitute.com/virtualization-security

[10]    OWASP. (2013). Category:OWASP WebGoat Project - OWASP. Available: https://http://www.owasp.org/index.php/Category:OWASP_WebGoat_Project

[11]    OSA, "SP-011: Cloud Computing Pattern," ed, 2013.

[12]    NIST. (2011). Guidelines on Security and Privacy in Public Cloud Computing. Available: http://www.nist.gov/customcf/get_pdf.cfm?pub_id=909494

[13]    NIST. (2011). NIST Cloud Computing Security Reference Architecture (DRAFT). Available: http://collaborate.nist.gov/twiki-cloud-computing/pub/ \\
CloudComputing/CloudSecurity/NIST_Security_Reference_Architecture_2013.05.15_v1.0.pdf

[14]    CSA. (2012). Cloud Controls Matrix v1.4 : Cloud Security Alliance. Available: https://cloudsecurityalliance.org/download/cloud-controls-matrix-v1-4

[15]    B. Damele. (2009). Advanced SQL injection to operating system full control. Available: https://http://www.owasp.org/images/d/dc/AppsecEU09-Damele-A-G-Advanced-SQL-injection-slides.pdf

[16]    T. Shimeall. (2013). Tactics and Penetration Testing. Available: https://blackboard.andrew.cmu.edu/bbcswebdav/pid-449058-dt-content-rid-3403035_1/xid-3403035_1

[17]    T. Shimeall. (2013). Web Hacking. Available: https://blackboard.andrew.cmu.edu/bbcswebdav/pid-449052-dt-content-rid-3403026_1/xid-3403026_1

[18]    T. Shimeall. (2013). Strategy. Available: https://blackboard.andrew.cmu.edu/bbcswebdav/pid-449058-dt-content-rid-3403035_1/xid-3403035_1

[19]    NIST. (2008). Volume I: Guide for Mapping Types of Information and Information Systems to Security Categories. Available: http://csrc.nist.gov/publications/nistpubs/800-60-rev1/SP800-60_Vol1-Rev1.pdf

[20]    NIST. (2006). Minimum Security Requirements for Federal Information and Information Systems Available: http://csrc.nist.gov/publications/fips/fips200/FIPS-200-final-march.pdf

[21]    NIST. (2013). NIST Cloud Computing Reference Architecture Cloud Service Metrics Description. Available: http://collaborate.nist.gov/twiki-cloud-computing/pub/CloudComputing/RATax_CloudMetrics_WAs_Docs/RATAX-CloudServiceMetricsDescription-DRAFT-20130621.pdf

[22]    SAFECode. (2012). Practical Security Stories and Security Tasks for Agile Development Environments. Available: http://www.safecode.org/publications/SAFECode_Agile_Dev_Security0712.pdf

[23]    J. Rockwood, "Choose Your Weapon Wisely: A handbook for determining the right software development method best suited for your team, client, and project (Independent Study Paper). Pittsburgh, PA: School of Computer Science, Carnegie Mellon University," 2011.

[24]    H. Kniberg. (2007). Scrum and XP from the Tenches - How we do Scrum. Available: http://infoq.com/minibooks/scrum-xpfrom-the-trenches

[25]    M. Hüttermann, Agile ALM : lightweight tools and Agile strategies. Shelter Island, N.Y.: Manning, 2012.

[26]    SecurityNinja. (2009). Secure Development. Available: http://www.securityninja.co.uk/secure-development

[27]    OWASP. (2013). Projects/OWASP Java Encoder Project - OWASP. Available: https://http://www.owasp.org/index.php/Projects/OWASP_Java_Encoder_Project

[28]    OWASP, "Category:OWASP Enterprise Security API - OWASP," ed, 2013.

[29]    D. Cornell. (2013). Web application testing: The difference between black, gray and white box testing. Available: http://searchsoftwarequality.techtarget.com/tip/Web-application-testing-The-difference-between-black-gray-and-white-box-testing

