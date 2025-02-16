# Firewall Configuration with iptables on Kali Linux

## OBJECTIVE  
To secure a Linux lab server by configuring a robust firewall using iptables, implementing a “default deny” strategy while explicitly allowing only critical services. This project demonstrates the practical application of network security best practices in a real-world environment.


## USE CASE 
I deployed this configuration on my personal lab server which hosts internal web applications and provides remote SSH management. This server is a critical asset used for development, testing, and hosting internal tools. Securing it ensures that only authorized traffic such as web requests, SSH connections, DNS resolution, and NTP for accurate timekeeping reaches the server while all other unsolicited or potentially malicious traffic is blocked.



### 1.1 Verifying my tools
Before starting, I confirmed that iptables was installed on my Linux system by running: 

![VirtualBox_kali-linux-2024 3-virtualbox-amd64_13_02_2025_22_12_13](https://github.com/user-attachments/assets/c006b5d8-3401-48c1-8580-a1a816c586e0)


### 1.2 Starting from a Clean Slate
To avoid conflicts with pre-existing rules, I flushed all current iptables rules:

![VirtualBox_kali-linux-2024 3-virtualbox-amd64_13_02_2025_22_13_53](https://github.com/user-attachments/assets/18140236-6da5-47c7-b555-38e82055df58)

-F (Flush): Deletes all rules in the default chains.

-X (Delete): Removes any user-defined chains.

Summarily, these commands  help us start on a fresh note  ensuring that old and insecure rules do not remain.


### 2.0  Setting a Default-Deny Policy:
 To minimize the attack surface, I set the default policy for all chains to DROP. This means that, by default, all traffic is blocked unless   explicitly allowed:
 
![VirtualBox_kali-linux-2024 3-virtualbox-amd64_13_02_2025_22_28_19](https://github.com/user-attachments/assets/a80b3a4b-3440-493f-a92a-ef9c7938ff3a)

 This “default deny” stance ensures that only the traffic you trust is permitted. 





### 2.1 Allowing Essential Traffic:

#### 2.1.1  Loopback (Localhost) Traffic:
Internal communications (e.g., a web server communicating with a local database) rely on the loopback interface (127.0.0.1). Thus, it’s crucial to allow this traffic.
##### Incoming Lookback Traffic 
![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_22_24_07](https://github.com/user-attachments/assets/70f18d04-7f0f-4992-a660-69ae84e768cd)

Where -A INPUT: Append to the INPUT chain.

-i lo: Match traffic on the loopback interface.

-j ACCEPT: Allow the packet.

 ##### Outgoing Loopback Traffic
![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_22_49_33](https://github.com/user-attachments/assets/131c7a7f-18c3-4b3c-b3a7-9834b8c8aba2)

Where -A OUTPUT: Append to the OUTPUT chain.

-o lo: Match traffic going out via the loopback interface.

-j ACCEPT: Allow the packet.







#### 2.1.2  Established and Related Connections:
To ensure that responses to outbound requests aren’t blocked, I allowed traffic that is part of an existing connection or related to it.

![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_23_03_33](https://github.com/user-attachments/assets/c176aa96-5d7b-4569-b589-733b79fcbab2)



When you initiate a connection (like browsing a website or connecting via SSH), this rule makes sure that the returning data (the response) isn’t dropped by the firewall.

#### 2.1.3  SSH (Port 22):
 SSH is essential for remote server management. To allow new incoming SSH connections, I used:

 ![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_23_16_37](https://github.com/user-attachments/assets/cedabda5-53a9-43ea-a1de-5488e230a4ae)

Hence, this rule allows any new TCP connection destined for port 22 (the default SSH port), ensuring remote access remains functional.

#### 2.1.4  Web Traffic – HTTP (Port 80) & HTTPS (Port 443)
To ensure that my web server is accessible, I allowed incoming web traffic:

![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_23_18_23](https://github.com/user-attachments/assets/3b62ebb0-a920-42d3-89e4-103c40e94729)

These rules permit new incoming connections on ports 80 and 443, which are standard for HTTP and HTTPS traffic.



#### 2.1.5  DNS and NTP:
DNS (Port 53)
DNS resolution is crucial for nearly all network communication. I allowed outbound DNS traffic with:
![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_23_22_05](https://github.com/user-attachments/assets/f791e5b1-4ff4-4c35-b704-cca493c7b003)

This rule ensures that my server can translate domain names into IP addresses.




### NTP (Port 123)
NTP is used for synchronizing system time. I allowed outbound NTP traffic using:

![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_23_29_08](https://github.com/user-attachments/assets/68189d83-e1af-433d-97a3-dfba1dcdea14)

NTP operates over  UDP (not TCP) on port 123 for time synchronization. I created  this rule because accurate system timing  is essential for logging and security protocols and certificate validation.








#### 3.0 Checking Created Rules:
To verify that my rules are correctly implemented, I listed the current iptables configuration:
![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_23_32_51](https://github.com/user-attachments/assets/c4b27b7f-3fbc-43dd-a680-7df56ce1cbf5)


#### 3.1 Ensuring Persistence Across Reboots:
 Finally, to ensure that these rules persist across reboots, I installed iptables-persistent:

![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_23_33_56](https://github.com/user-attachments/assets/04607b04-94d6-403f-a9d7-22049bf80766)

This installs iptables-persistent and saves your current configuration, ensuring that your security posture remains intact after a restart.

![VirtualBox_kali-linux-2024 3-virtualbox-amd64_14_02_2025_23_38_32](https://github.com/user-attachments/assets/342d4323-9b61-4871-9d7f-23d090a471e5)

Persistence is critical in production environments so that security settings are not lost during maintenance or unexpected reboots.



### 4.0  WHAT I LEARNED 

##### Default Deny Strategy: Starting with a “default deny” posture ensures that only explicitly allowed traffic reaches the system, significantly reducing the attack surface.

##### Rule Specificity: Understanding the importance of allowing essential services (like loopback, SSH, and web traffic) while blocking everything else is crucial for maintaining both functionality and security.

##### Stateful Inspection: Using state-based rules (ESTABLISHED, RELATED) is vital for ensuring that legitimate traffic flows without interruption.

##### Persistence Matters: Employing tools like iptables-persistent guarantees that your hard-earned security settings are not lost on reboot which is critical for production environments.



## CONCLUSION

This project successfully secured my Linux lab server by implementing a robust iptables firewall configuration. By adopting a default-deny policy and selectively allowing only essential traffic, I built a strong defence against unwanted network activity. This hands-on experience not only reinforced my practical knowledge of network security but also showcased my ability to design and implement secure systems in a real-world environment.
