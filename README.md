# AWS_EC2_Honeypot

## Overview
I set up a cowrie honeypot on an Amazon EC2 instance. I left ports 22 and 23 open, rerouted their traffic to the crowie honeypot on 2222 and 2223. Then I left for a few hours and came back to 15 different src_ip attempting to attack the vulnerable server. I analyzed the logs with Splunk and used its virtualization features to better represent the data.

## Steps
1. I made an Ubunutu EC2 Instance (t2.micro) with port 22 and 23 open to 0.0.0.0/0 (everyone) and 8022 restricted to my IP, so I can still access my server once cowrie is running on 22. I also created a new kwypair that will be used to access the server.

  <img width="1319" height="365" alt="image" src="https://github.com/user-attachments/assets/61bf4995-4eb3-46dd-8755-40750cc8bc49" />

2. Then I connected to the instance with the below command on powershell. After that I install Cowrie on the Ubuntu instance following the steos in the cowrie documentation linked at the bottom of the page. This includes rerouting ports 22 and 23 to 2222 and 2223 respectively using iptables (must be root).
```
ssh -i "C:/../file.pem" ubuntu@your-server-public-address
```

3. Once everything is setup, I started the cowrie service, used the tail -f [log file] command, and left for a few hours. When I came back my screen was full of lots of "hacker" activity. There were thousands of lines of log data, I also had a few special folders in cowrie to dig through. Due to the volume of traffic, I decided to utilize Splunk Cloud to make the log easier to understand and visualize.

4. For the purpose of this project, I shut down cowrie at this point and downloaded the log file and uploaded it to Splunk. However, if you had the space and resources you could let cowrie run and have the logs populate in Splunk in real time. This is much closer to what a true enterprise environment would do. But I will go over the results in the below Results section.


## Results
1. After uploading my data, I discovered that there were 408 unique events between 15 unique IP addresses. In the below image you can see all of the 15 addresses, their occurences, and the query I used to get this from the log.

<img width="1358" height="507" alt="image" src="https://github.com/user-attachments/assets/0853a139-b71e-4cf9-876b-43d6e37fa32b" />

```
source="your-file.txt" host="[the host]" sourcetype="_json" | stats count by src_ip | sort - count
```

2. I could also view which of the open protocols were exploited most. To my surprise, SSH was still exploited more than the more vulnerable Telnet protocol. My guess is that this is likely due to unfamiliarity with telnet or bots that scan the internet for vulnerable servers weren't configured to attack telnet since it is often disabled anyways.

<img width="401" height="252" alt="image" src="https://github.com/user-attachments/assets/c30fbe8f-e648-4a72-9a2f-75151732c6d5" />

<img width="1354" height="99" alt="image" src="https://github.com/user-attachments/assets/837e439b-23b5-4673-b3e4-f15af41d7a47" />

3. The geolocations of the IP addresses can be viewed in the following images. However, it should be noted that VPN and proxies can skew the accuracy of the traffic's true origin.

<img width="897" height="388" alt="image" src="https://github.com/user-attachments/assets/01bdd3d4-48c4-423c-b3e5-57599a15e981" />


<img width="1351" height="521" alt="image" src="https://github.com/user-attachments/assets/c12135a8-9bb2-43c3-9d56-e581bce98de8" />

```
source="your-file.txt" host="[the host]" sourcetype="_json" | stats count by src_ip | iplocation src_ip | geostats count by src_ip
```

4. I also wanted to see the most frequently occuring usernames and passwords used by the attackers. In the below images, I show the top and bottom 10 most frequently occuring password and username. Not really any surprises to what the most common ones were.

```
source="your-file.txt" host="[the host]" sourcetype="_json" | stats count by username | sort -count | head
```

<img width="1362" height="350" alt="image" src="https://github.com/user-attachments/assets/8b188328-1ebd-4208-a373-c65161141540" />

<img width="1352" height="295" alt="image" src="https://github.com/user-attachments/assets/5f00536f-5d3a-4ff4-9183-011d776d6fb4" />


```
source="your-file.txt" host="[the host]" sourcetype="_json" | stats count by password | sort -count | head
```

<img width="692" height="343" alt="image" src="https://github.com/user-attachments/assets/1643b7b3-6c9a-4068-877b-ac9f4e304d26" />

<img width="1357" height="296" alt="image" src="https://github.com/user-attachments/assets/d1eb2cf7-1e0a-4cc4-86e8-2f731ea0ad03" />

4. One of the most notable events was the executable that got uploaded to the honeypot. In the ~/cowrie/downloads folder, I found the shasum of an executable file that had been left by one of the attackers (our Taiwan friend). This could be further analyzed in a sandbox environment to see what this actually does.
   
<img width="1337" height="580" alt="image" src="https://github.com/user-attachments/assets/e51e6ef5-e793-4f84-a525-4b197a63c649" />

<img width="548" height="47" alt="image" src="https://github.com/user-attachments/assets/3baf31b8-c27c-4ebe-9afd-5762bccdd010" />



6. The last folder I went through was the tty folder in cowrie. This folder logs sessions of succesfull connections and displays some of the commands that were used. The first session looks like the user was doing some early recon to discover things like OS and then lost connection or terminated on their own. The second sessions actually shows the file upload and the chmod +x shows that the executable was made runable.

<img width="1351" height="309" alt="image" src="https://github.com/user-attachments/assets/988322cd-9078-4e42-bf67-0eea31ecb2df" />

## Conclusion
This project allowed me to practice several skills such as threat analysis, cloud computing, and data analysis. After only a few short hours, I got loads of logs to sift through and learn what the real attackers are actually doing. One of the most notable takeaways were the username and passwords that were most frequently used (root, admin, 123456, etc.). Another was what one of the attackers did after gaining access. The executable file being downloaded and made executable is a sign of a successful attack. This could have been a backdoor or maybe a form of malware that could have been used to further exploit my instance (although it's now terminated). It was also interesting to see how varied the traffic was as it came from Chicago, Germany, South Korea, Japan, and many more locations. All in all, I leared a little more about what the real attackers are doing and feel that I will likely set up another honeypot in the future to continue practicing threat analysis.

## Helpful Resources
- https://docs.cowrie.org/en/latest/splunk/README.html
- https://www.codygula.com/posts/setting-up-a-cowrie-ssh-honeypot-on-aws/
