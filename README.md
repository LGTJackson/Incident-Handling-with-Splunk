# Incident Handling with Splunk 
### By tryhackme & Dex01
---
# Introduction:
An incident from a security perspective is "Any event or action, that has a negative consequence on the security of a user/computer or an organization is considered an incident"
**Examples of events:**
- Crashing the system
- Execution of an unwanted program
- Access to sensitive information from an unauthorize user
- A website being defaced by the attacker
- The user of USB devices when there is a restriction in usage against the company's policy
## Learning Objective:
- Learn to leverage OSINT sites during an investigation
- How to map an attacker's activities to Cyber Kill Chain Phases
- How to utilize effective Splunk searches to investigate logs
- Understand the importance of host-centric and network-centric log sources

## Incident Handling Life Cycle Overview:
1. Preparation
2. Detection and Analysis
3. Containment, Eradication, and Recovery
4. Post-Incident Activity/ Lessons Learned
---
# Scenario:
A Big corporate organization **Wayne Enterprises** has recently faced a cyber-attack where the attackers broke into their network, found their way to their web server, and have successfully defaced their website http://www.imreallynotbatman.com. Their website is now showing the trademark of the attackers with the message **YOUR SITE HAS BEEN DEFACED** as shown below.
![image](https://github.com/LGTJackson/Incident-Handling-with-Splunk/assets/47001339/b1e277fc-2923-47ee-9587-8f60657f3577)

They have requested "**US**" to join them as a **Security Analyst** and help them investigate this cyber attack and find the root cause and all the attackers' activities within their network.

The good thing is, that they have Splunk already in place, so we have got all the event logs related to the attacker's activities captured. We need to explore the records and find how the attack got into their network and what actions they performed.
___
# Recon Phase

In this step the first I'm just gathering a little information. My initial query is just to narrow down the index I'm looking in and logs that contain the domain name of the site being investigated. 
```
index=botsv1 imreallynotbatman.com
```
The search time will be set to "all time" for now(It's generally not best practice because when there are a large amount of logs it can cause Splunk to be slow) and "Verbose Mode"

![image](https://github.com/LGTJackson/Incident-Handling-with-Splunk/assets/47001339/7520a2fe-1bba-401b-9b8b-948cb26533b2)

From here we can click on "sourcetype" to get an idea of the different log sources
![image](https://github.com/LGTJackson/Incident-Handling-with-Splunk/assets/47001339/d13596dc-20fd-4584-8633-9c84c4f2d706)

I'll begin by checking "stream:http" by adding to the query because the http logs should include information about the source IP of the attacker
``` Splunk
index=botsv1 imreallynotbatman.com sourcetype=stream:http
```
![image](https://github.com/LGTJackson/Incident-Handling-with-Splunk/assets/47001339/db46e599-d422-4029-a570-59eeef29fab0)

There were two source IPs listed. I'll investigate "40.80.148.42" because it contributed to a large portion of events(93%) by adding it to the query.

Now that I've identified a source IP, I can change my log source type to suricata(IDS) logs.
```
index=botsv1 imreallynotbatman.com sourcetype=suricata src_ip=40.80.148.42
```

## There are now four questions to answer within this section:

**1. One suricata alert highlighted the CVE value associated with the attack attempt. What is the CVE value?**
To answer this I will click on "signature" under SELECTED FIELDS and take a look at some of the signatures assigned to the logs

The answer to this question is in the above screenshot but if I needed to narrow down the results some more we could simply add "CVE" to the query
```Splunk
index=botsv1 imreallynotbatman.com sourcetype=suricata src_ip=40.80.148.42 CVE
```

**2. What is the CMS our web server is using?**
To answer this I really only need to click on "http.url" under "INTERESTING FIELDS" and see where the majority of top values are originating from:

![image](https://github.com/LGTJackson/Incident-Handling-with-Splunk/assets/47001339/7a9f6e17-3c7d-468a-8a42-acde950bea54)

**3. What is the web scanner, the attacker used to perform the scanning attempts?**
To answer this, I'll just be adding "SCAN" to the search query
```Splunk
index=botsv1 imreallynotbatman.com sourcetype=suricata src_ip=40.80.148.42 SCAN
```
The answer should be within the signature of the logs returned

![image](https://github.com/LGTJackson/Incident-Handling-with-Splunk/assets/47001339/a5e8ef5e-f2a9-4865-a330-eb0e1d50ef25)

**4. What is the IP address of the server imreallynotbatman.com?**
The answer for this is included in the image provided for the previous question. But the destination IP should be included under the "INTERESTING FIELDS" and by clicking on "dest_ip", then just looking for which destination IP received the majority of the traffic

![image](https://github.com/LGTJackson/Incident-Handling-with-Splunk/assets/47001339/30e73fb0-79df-4724-b35a-5a1c1873d9a4)

A screenshot of the questions and answers:

![image](https://github.com/LGTJackson/Incident-Handling-with-Splunk/assets/47001339/970c3d74-6b44-4986-beed-6a3fea4deaa8)

___
