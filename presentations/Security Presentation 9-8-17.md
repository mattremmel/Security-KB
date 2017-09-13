# Security Presentation Notes 9/8/2017

{{TOC}} 

## Resources

Its always a good idea to familiarize yourself with the most common types of attacks out there, so you can know what to defend against. You can find them at [OWASP Top Ten](https://www.owasp.org/index.php/Top_10_2013-Top_10). 2017 is in final stages of being approved, but the 2013 edition is good enough for now.

To look for known vulnerabilities in tools, libraries, frameworks, etc, use the [Common Vulnerabilities and Exposures (CVE)](https://cve.mitre.org/cve/cve.html)

To look for lists of common weaknesses that occur in software, use [Common Weakness Enumeration](https://cwe.mitre.org)

- Threat Modeling
	- [Mitre Threat Modeling Overview](https://www.owasp.org/index.php/Application_Threat_Modeling)
	- [Threat Modeling (book) - Adam Shostack](http://www.wiley.com/WileyCDA/WileyTitle/productCd-1118809998.html)
	- [Elevation of Privilege Cards - Adam Shostack - Microsoft](https://www.microsoft.com/en-us/sdl/adopt/eop.aspx)

- Security Development Lifecycle
	- [Microsoft SDL-IT](https://www.microsoft.com/en-us/sdl/)

## Tools

- [Snyk](https://snyk.io)
	- $ snyk test
- [FindBugs](https://github.com/findbugsproject/findbugs)
	- Set war file and set source code to 'src/main/java'
- [Nmap](https://nmap.org)
	- $ nmap [host]
	- $ nmap -A -vv [host]
		- 71 seconds to run
- [Nikto](https://github.com/sullo/nikto)
	- $ nikto -h [host]
- [Burp Suite](https://portswigger.net/burp)
	- Crawl site, show passive and active scanning
	- Export movie search url for SQLMap
- [SQLMap](http://sqlmap.org)
	- $ sqlmap -l [logfile]
	- $ sqlmap -l [logfile] --dump
- [Gitrob](https://github.com/michenriksen/gitrob)

---

## Presentation Notes

## Project Overview
- Reverse auction app intended to be used by manufactures and those in need of manufacturing services (Rolls Royce).
- Project has been in development for about 3 years. Parts of the application come from different places.
	- Physics simulation and algorithms inherited from a project out of MIT.
	- Part of code base inherited from GM.
- Rolls Royce had a formal review and security audit planned for September, but because they were short on resources asked us to give them a preliminary report about the functionality and security of the application to help them plan and make decisions.
- The format of the project was really nice, because it gave us the opportunity to practice and learn some new skills and put together some resources, without the pressure of having to deliver a formal audit and put our name on it in case we missed something.

### Web Stack
Angular web app with a Java Spring server, and a Postgres database.

### Security Audit Process
Began by doing a lot of automated scans and looking for quick wins and low hanging fruit. Unfortunately it wasn't hard to find a lot of low hanging fruit. Followed with some manual testing.

Had to run scans against the dev server because at the time there was no way to run the project locally (according to OpenDMC), however Clay eventually managed to accomplish it in a couple hours.

- Searched for keys committed to Github.
- Scanned for dependencies and stack for CVE's
	- Found several potential issues in outdated packages, but didn't spend time confirming they could be exploited.
		- Main issue was their build process was broken and they weren't able to update dependencies to their latest versions.
		- They were also targeting a old version of Node that was a couple major versions out of date.
- Ran basic automated attacks (no real test environment, and we weren't doing a full blown red team audit)
	- SQLMap
	- Nikto
- Played with forging http requests via a proxy.
- Static analysis of source code.
	- Revealed potential SQL injection vulnerabilities that we were able to exploit.
- Manually reviewed source code.
- Generally just put on the mind set of an attacker and tried to compromised the application.

### Discovered Issues
- **Committed Credentials** - AWS and Postgres credentials committed to Github.
- **SQL Injection** - Not properly sanitizing inputs.
- **CRUD Operations** - Only checking authentication and not properly checking authorization. Able to create, delete, and modify documents in other peoples workspaces/directories.
- **No Server Side Validation** - Some places only had client side validation.
- **Timing Attack on File Upload** - Files of the same name submitted at the same time can overwrite each other.
- **Forging a different document** as a different version of an existing document.

---

## Security Fundamentals
### Quick Terminology
- **Adversary (threat agent)**: An entity that attacks, or is a threat to, a system.
- **Attack**: An assault on system security that derives from an intelligent threat; that is, an intelligent act that is a deliberate attempt (especially in the sense of a method or technique) to evade security services and violate
the security policy of a system.
- **Countermeasure**: An action, device, procedure, or technique that reduces a threat, a vulnerability, or an attack by eliminating or preventing it, by minimizing the harm it can cause, or by discovering and reporting it so that corrective action can be taken.
- **Risk**: An expectation of loss expressed as the probability that a particular threat will exploit a particular vulnerability with a particular harmful result.
- **Security Policy**: A set of rules and practices that specify or regulate how a system or organization provides security services to protect sensitive and critical system resources.
- **System Resource (Asset)**: Data contained in an information system; or a service provided by a system; or a system capability, such as processing power or communication bandwidth; or an item of system equipment (i.e., a system component— hardware, firmware, software, or documentation); or a facility that houses system operations and equipment.
- **Threat**: A potential for violation of security, which exists when there is a circumstance, capability, action, or event, that could breach security and cause harm. That is, a threat is a possible danger that might exploit a vulnerability.
- **Vulnerability**: A flaw or weakness in a system’s design, implementation, or operation and management that could be exploited to violate the system’s security policy.

### CIA Triad
- **Confidentiality**: Preserving authorized restrictions on information access and disclosure, including means for protecting personal privacy and proprietary information. A loss of confidentiality is the unauthorized disclosure of information.
	- **Data Confidentiality**: Assures that private or confidential information is not made available or disclosed to unauthorized individuals.
	- **Privacy**: Assures that individuals control or influence what information related to them may be collected and stored and by whom and to whom that information may be disclosed.
- **Integrity**: Guarding against improper information modification or destruction, including ensuring information nonrepudiation and authenticity. A loss of integrity is the unauthorized modification or destruction of information.
	- **Data Integrity**: Assures that information and programs are changed only in a specified and authorized manner.
	- **System Integrity**: Assures that a system performs its intended function in an unimpaired manner, free from deliberate or inadvertent unauthorized manipulation of the system.
- **Availability**: Ensuring timely and reliable access to and use of information. A loss of availability is the disruption of access to or use of information or an information system. 

### Common Triad Additions
- **Authenticity**: The property of being genuine and being able to be verified and trusted; confidence in the validity of a transmission, a message, or message originator. This means verifying that users are who they say they are and that each input arriving at the system came from a trusted source. (FIPS 199 includes authenticity under integrity.)
- **Accountability**: The security goal that generates the requirement for actions of an entity to be traced uniquely to that entity. This supports nonrepudiation, deterrence, fault isolation, intrusion detection and prevention, and after-action recovery and legal action. Because truly secure systems are not yet an achievable goal, we must be able to trace a security breach to a responsible party. Systems must keep records of their activities to permit later forensic analysis to trace security breaches or to aid in transaction disputes.

### Computer Security Challenges
1. Computer security is not as simple as it might first appear to the novice. The requirements seem to be straightforward; indeed, most of the major requirements for security services can be given self-explanatory one-word labels: confidentiality, authentication, nonrepudiation, integrity. But the mechanisms used to meet those requirements can be quite complex, and understanding them may involve rather subtle reasoning.
2. Computer security is essentially a battle of wits between a perpetrator who tries to find holes and the designer or administrator who tries to close them. The great advantage that the attacker has is that he or she need only find a single weakness while the designer must find and eliminate all weaknesses to achieve perfect security.
3. There is a natural tendency on the part of users and system managers to perceive little benefit from security investment until a security failure occurs.
4. Security requires regular, even constant, monitoring, and this is difficult in today’s short-term, overloaded environment.
5. Security is still too often an afterthought to be incorporated into a system after the design is complete rather than being an integral part of the design process.
6. Many users and even security administrators view strong security as an impediment to efficient and user-friendly operation of an information system or use of information.

---

## Security Development Lifecycle

### Application Layer = Weak Point
- Attackers target the weakest point of a system. The OS Layer and Network layer are too hard these days.
- On average over 70% of IT security budget is spent on infrastructure, yet over 75% of attacks happen at the application level.
- According to Microsoft research, only 1/3 of developers are confident that they write secure code.
- More time and energy needs to be spent on hardening the application layer.

### Integrate Security
A step forward: Integrate Security Practice.

- Integrate a Secure Development Lifecycle into the normal development lifecycle.
- Train developers in secure coding practices.
- Incorporate Threat Modeling, Secure Coding Techniques, Secure Code Review, Security Focused Testing into the development process.

Lifecycles Compared:

Envision -> Risk Assessment
Design -> Threat Model / Design Review
Develop -> Internal Review
Test -> Pre-Production Assessment
Release -> Post-Production Assessment


### Purpose of SDL
A lot of hackers are really smart, and the ones who aren't use publicly available tools that are really smart on their behalf. It's becoming more and more obvious everyday that the tech world needs better security. Its seems like every week that a new company was the victim of a large hack.

- Inventory and assess applications.
- Identify and ensure resolution of security/privacy vulnerabilities found in those applications.
- Enable strategic, tactical, operational, and legal risk management.

### SDL Objectives

#### Risk Assessment
- Application Inventory
- Determine Application Risk Categorization
	- High Risk
	- Medium Risk
	- Low Risk

#### Threat Model / Design Review
- Threat modeling provides a consistent methodology for objectively evaluating threats to
applications.
- Review application design to verify compliance with security standards and best practices.
- Verify application meets security principles.
	- CIA-AA

#### Internal Review
- Review security checklists and policies.
- Team conducts 'self' code review and attack and penetration test.

#### Pre-Production Assessment
- Low Risk Applications
	- Host level scan
- High/Medium Risk Applications
	- Host Level Scan
	- White Box Code Review
		- Application team provides the source code.
		- Analysts review application code searching for security vulnerabilities.
		- Vulnerabilities logged in bug database.
		- Application team addresses all severity-1 bugs prior to going into production.

#### Post-Production Assessment
- Host Level Scan

### SDL Resources
- Elevation of Privilege Cards (Threat Model)
- Static Code Analysis
	- FindBugs(Java)
- Mitre (Common weaknesses and vulnerabilities)
- OWASP - Top 10, 2017
- Security Checklists
- Threat Modeling - Adam Shostack

---

## Secure Design Principles
- **Economy of Mechanism**: The design of security measures embodied in both hardware and software should be as simple and small as possible. The motivation for this principle is that relatively simple, small design is easier to test and verify thoroughly. With a complex design, there are many more opportunities for an adversary to discover subtle weaknesses to exploit that may be difficult to spot ahead of time. The more complex the mechanism, the more likely it is to possess exploitable flaws. Simple mechanisms tend to have fewer exploitable flaws and require less maintenance. Furthermore, because configuration management issues are simplified, updating or replacing a simple mechanism becomes a less intensive process. In practice, this is perhaps the most difficult principle to honor. There is a constant demand for new features in both hardware and software, complicating the security design task. The best that can be done is to keep this principle in mind during system design to try to eliminate unnecessary complexity.
- **Fail-safe Defaults**: Access decisions should be based on permission rather than exclusion. That is, the default situation is lack of access, and the protection scheme identifies conditions under which access is permitted. This approach exhibits a better failure mode than the alternative approach, where the default is to permit access. A design or implementation mistake in a mechanism that gives explicit permission tends to fail by refusing permission, a safe situation that can be quickly detected. On the other hand, a design or implementation mistake in a mechanism that explicitly excludes access tends to fail by allowing access, a failure that may long go unnoticed in normal use. For example, most file access systems work on this principle and virtually all protected services on client/server systems work this way.
- **Complete Mediation**: Every access must be checked against the access control mechanism. Systems should not rely on access decisions retrieved from a cache. In a system designed to operate continuously, this principle requires that, if access decisions are remembered for future use, careful consideration should be given to how changes in authority are propagated into such local memories. File access systems appear to provide an example of a system that complies with this principle. However, typically, once a user has opened a file, no check is made to see if permissions change. To fully implement complete mediation, every time a user reads a field or record in a file, or a data item in a database, the system must exercise access control. This resource-intensive approach is rarely used.
- **Open Design**: The design of a security mechanism should be open rather than secret. For example, although encryption keys must be secret, encryption algorithms should be open to public scrutiny. The algorithms can then be reviewed by many experts, and users can therefore have high confidence in them. This is the philosophy behind the National Institute of Standards and Technology (NIST) program of standardizing encryption and hash algorithms, and has led to the widespread adoption of NIST-approved algorithms.
- **Separation of Privilege**: A practice in which multiple privilege attributes are required to achieve access to a restricted resource. A good example of this is multi-factor user authentication, which requires the use of multiple techniques, such as a password and a smart card, to authorize a user. The term is also now applied to any technique in which a program is divided into parts that are limited to the specific privileges they require in order to perform a specific task. This is used to mitigate the potential damage of a computer security attack. One example of this latter interpretation of the principle is removing high privilege operations to another process and running that process with the higher privileges required to perform its tasks. Day-to-day interfaces are executed in a lower privileged process.
- **Least Privilege**: Every process and every user of the system should operate using the least set of privileges necessary to perform the task. A good example of the use of this principle is role-based access control, described in Chapter 4. The system security policy can identify and define the various roles of users or processes. Each role is assigned only those permissions needed to perform its functions. Each permission specifies a permitted access to a particular resource (such as read and write access to a specified file or directory, and connect access to a given host and port). Unless permission is granted explicitly, the user or process should not be able to access the protected resource. More generally, any access control system should allow each user only the privileges that are authorized for that user. There is also a temporal aspect to the least privilege principle. For example, system programs or administrators who have special privileges should have those privileges only when necessary; when they are doing ordinary activities the privileges should be withdrawn. Leaving them in place just opens the door to accidents.
- **Least Common Mechanism**: The design should minimize the functions shared by different users, providing mutual security. This principle helps reduce the number of unintended communication paths and reduces the amount of hardware and software on which all users depend, thus making it easier to verify if there are any undesirable security implications.
- **Psychological Acceptability**: The security mechanisms should not interfere unduly with the work of users, while at the same time meeting the needs of those who authorize access. If security mechanisms hinder the usability or accessibility of resources, users may opt to turn off those mechanisms. Where possible, security mechanisms should be transparent to the users of the system or at most introduce minimal obstruction. In addition to not being intrusive or burdensome, security procedures must reflect the user’s mental model of protection. If the protection procedures do not make sense to the user or if the user must translate his image of protection into a substantially different protocol, the user is likely to make errors.
- **Isolation**: A principle that applies in three contexts. First, public access systems should be isolated from critical resources (data, processes, etc.) to prevent disclosure or tampering. In cases where the sensitivity or criticality of the information is high, organizations may want to limit the number of systems on which that data are stored and isolate them, either physically or logically. Physical isolation may include ensuring that no physical connection exists between an organization’s public access information resources and an organization’s critical information. When implementing logical isolation solutions, layers of security services and mechanisms should be established between public systems and secure systems responsible for protecting critical resources. Second, the processes and files of individual users should be isolated from one another except where it is explicitly desired. All modern operating systems provide facilities for such isolation, so that individual users have separate, isolated process space, memory space, and file space, with protections for preventing unauthorized access. And finally, security mechanisms should be isolated in the sense of preventing access to those mechanisms. For example, logical access control may provide a means of isolating cryptographic software from other parts of the host system and for protecting cryptographic software from tampering and the keys from replacement or disclosure.
- **Encapsulation**: A specific form of isolation based on object- oriented functionality. Protection is provided by encapsulating a collection of procedures and data objects in a domain of its own so that the internal structure of a data object is accessible only to the procedures of the protected subsystem and the procedures may be called only at designated domain entry points.
- **Modularity**: In the context of security refers both to the development of security functions as separate, protected modules and to the use of a modular architecture for mechanism design and implementation. With respect to the use of separate security modules, the design goal here is to provide common security functions and services, such as cryptographic functions, as common modules. For example, numerous protocols and applications make use of cryptographic functions. Rather than implementing such functions in each protocol or application, a more secure design is provided by developing a common cryptographic module that can be invoked by numerous protocols and applications. The design and implementation effort can then focus on the secure design and implementation of a single cryptographic module, including mechanisms to protect the module from tampering. With respect to the use of a modular architecture, each security mechanism should be able to support migration to new technology or upgrade of new features without requiring an entire system redesign. The security design should be modular so that individual parts of the security design can be upgraded without the requirement to modify the entire system.
- **Layering**: The use of multiple, overlapping protection approaches addressing the people, technology, and operational aspects of information systems. By using multiple, overlapping protection approaches, the failure or circumvention of any individual protection approach will not leave the system unprotected. This technique is often referred to as *defense in depth*.
- **Least Astonishment**: A program or user interface should always respond in the way that is least likely to astonish the user. For example, the mechanism for authorization should be transparent enough to a user that the user has a good intuitive understanding of how the security goals map to the provided security mechanism.

## Threat Modeling
[OWASP Threat Modeling](https://www.owasp.org/index.php/Application_Threat_Modeling)

At a high-level, the threat model process can be broken down into three steps:

1. Decompose the Application.
2. Determine and rank threats
3. Determine countermeasures and mitigation.

### Decompose the Application
- External Dependencies
	- Servers
	- Databases
	- Firewalls
	- Protocols
- Entry Points
	- Open ports
	- Landing pages
	- Login pages
	- Login function
	- Search entry
	- Forms
- Assets
	- User details
	- Login details
	- Personal data
- Data Flow Diagrams
- System Design Diagrams

### Determine and Rank Threats
A threat categorization provides a set of threat categories with corresponding examples so that threats can be systematically identified in the application in a structured and repeatable manner.

- Identify Threats
	- STRIDE
- Identify Possible Security Controls
- Threat Analysis
- Risk Ranking of Threats
	- DREAD
- Identify Possible Countermeasures
- Mitigation Strategies

## Demo
- Mitre CWE and CVE
- OWASP
- Snyk
	- snyk test
- FindBugs
	- Set war file and set source code to 'src/main/java'
- Nmap
	- nmap 172.20.1.22
	- nmap -A -vv 172.20.1.22
		- 71 seconds to run
- Nikto
	- nikto -h 172.20.1.22
- Burp Suite
	- Crawl site, show passive and active scanning
	- Export movie search url for SQLMap
- SQLMap
	- sqlmap -l [logfile]
	- sqlmap -l [logfile] --dump

### Other Tools, attacks, and considerations
Its not always a matter of just the code we write introducing security issues, sometimes its our OS, frameworks, and libraries that cause introduce issues.

- OpenVAS
- Metasploit
- SlowLoris
- Son Flood
- XML Custom Entity - Billion Laughs Attack
- DDOS
- Computational Heavy API - DOS  
- Sanitizing Input and Output
- SSL Strip / Valid certificates
- Fuzzing
- UDP Reflection
- Kali

#### Defensive Tools
- OSSEC
- Security Onion
- Elastic / Logstash / Kibana


## Conclusion
The need for security is obvious and is becoming more and more important everyday. There is a huge demand for security analysts and secure software, and there is an equally large shortage of people who can fill those roles.

It's my opinion that we have an obligation to be developing secure code. Clients may not always want to pay for above and beyond security assessment, but I feel that we still have the obligation to do our due diligence and deliver software that is *adequately* secure. In this day and age I feel like security is just as important as functionality for the goal of delivering quality software to our clients.

Its going to be a continuous improvement process. We need to invest time in upfront activities like Threat Modeling and Design Reviews, and holistically its going to depend on the relationship of the people, the processes, and the tools. We need people providing guidance on secure application development, we need tools to help organize and optimize the design, implementation, and analysis stages of security development, and we need to integrate processes that doesn't put security as an afterthought.

To do that we are going to need support from those in leadership positions and project management positions; we need to integrate a Secure Development Lifecycle into our normal SDLC; we as developers need to continue learning how to write secure code, and recognize designs/implementations that aren't secure; and most importantly we need to have a **security first attitude**. It's really hard and expensive to try to add security after the fact, but if you integrate security from the very beginning, it can become almost transparent costing almost no extra time or money.

For those interested:

- Building and implementing a Secure Development Lifecycle that works with our current processes and context.
- Building a knowledge base of tools and resources to enable us to build more secure and reliable software; in the case that we already do that pretty well, we can at least increase our confidence that we are doing that so we can offer more assurance that we are delivering secure software.
- Being intentional about being security minded and encouraging others.
