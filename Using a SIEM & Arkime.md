Malcolm is a SIEM integration tool that bundles Suricata, Zeek, and Arkime logs into one pane. You can use the Dashboards view to aggregate findings. It also has the Netbox application for asset tracking.

Once you get into the Dashboard, ensure your timetable is correct in the top-right:

<img width="541" height="108" alt="Pasted image 20260917121059" src="https://github.com/user-attachments/assets/da7d8be4-8e5e-4907-acea-cf44a8a99535" />

The following are a few 'if' scenarios. For example, if you wanted to find the application protocol with the most usage, you would simply go into the Overview dashboard and the list is on the initial view: 

<img width="569" height="497" alt="Pasted image 20260917121806" src="https://github.com/user-attachments/assets/98b64276-d8dc-4b8c-a282-ac1c8c3771a1" />


2. If you were trying to find the specific PDU reference number of a command, you can click into one of the available options provided on the left-side in Malcolm. Once you do, you'll be given a brea

<img width="378" height="128" alt="Pasted image 20260917122143" src="https://github.com/user-attachments/assets/a1a4ade6-3ea2-4740-a724-cfa92efb8b6e" />

After you get into the IOT dashboards, you can scroll down to the Zeek logs ot the bottom. One of these has a PLC_Stop command we'll get the PDU reference number from. Some other log attributes are captured for comparison

<img width="425" height="178" alt="Pasted image 20260917122800" src="https://github.com/user-attachments/assets/60dd600d-91bb-4ba4-b600-3f8313558c90" />

3. If you were trying to track an Uninventoried Host, you can look at the Zeek Known Summary to find known and unknown hosts. The bottom of the dashboard will give you Uninventoried Observed Hosts.

<img width="1059" height="182" alt="Pasted image 20260917123019" src="https://github.com/user-attachments/assets/120c83e3-6b74-4c27-8ca6-f9d5cfb3eafd" />

4. In order for Zeek, Suricata, and other log tools to agree on 'single sources of truth' and correlate data, they use what's called a CommunityID. This ID is shared among all log tools.  You can make this ID searchable in Arkime by putting it in the soarch tool. You'll see that the top of the screenshot contains the search, and the bottom contains the flag for this particular CTF.

<img width="834" height="998" alt="Pasted image 20260917124721" src="https://github.com/user-attachments/assets/677a1eaf-cd63-437d-84c6-a2a75470452b" />

5. If you want to find information contained within data transmissions, you would usually use either Arkime (big data) or Wireshark (one-off, smaller data). You can also use Malcolm (dashboard) to find specific logs (Zeek/Suricata), then track them down in Arkime. 

	Malcolm has severity ratiings for a lot of the logs that it ingests. a high-severity event would likely be a password sent in cleartext. If you're prompted to find data transmissons over FTP, you can likely find these passwords.

	Once in the 'severity' dashboard, scroll down to application protocol, where you'll find FTP:

<img width="779" height="386" alt="Pasted image 20260917221605" src="https://github.com/user-attachments/assets/95a21038-493f-40d3-bba4-5cbbaabece7b" />

	Click into FTP. This will take you to Arkime. Filter based on your time period you're looking at, and you'll get a few data transmissions using FTP. Click through them, and look at their attributes.

<img width="1682" height="607" alt="Pasted image 20260917221720" src="https://github.com/user-attachments/assets/7f6a2fed-d4cb-47f5-b6ba-f51f4acc856a" />

	
	You'll want to specifically click into the log type of the application (FTP) you're wanting to work with. Here, you'll find the cleartext password at the bottom!
	
<img width="1009" height="792" alt="Pasted image 20260917221833" src="https://github.com/user-attachments/assets/0216f278-768f-4036-88c2-83dd7b94b47c" />

7. If you want to look at files that your SIEM extracts, that's usually available as an option as well. In Malcolm, going to the Files dashboard will give you a table that looks like this:

<img width="1201" height="728" alt="Pasted image 20260917222135" src="https://github.com/user-attachments/assets/5e8d40f5-5325-4d36-b6fb-a3e7bd26dc29" />

	
	If you want to download a file, simply click the `zeek.files.extracted_uri` link for the file you want, and that will start the download process. These files will usually download as .zip files, but you can reliably see the contents with 7-zip. Here's an example of the FTP file we saw that you can pull!
	
<img width="896" height="792" alt="Pasted image 20260917222526" src="https://github.com/user-attachments/assets/59fd09d4-3bb9-4df6-8560-edbbbb0ce481" />

8. Sometimes, if you want to find a specific value for a specific log, you'll have to related items, or pieces of the item, then drill down further by further tracing the log. 

	Here, I have to find the Status Code (zeek.opcua_binary_status_code_detail.status_code) for the 'OPCUA Binary - Action' `Create Monitored Items`. If I look up what OPCUA is, I'll find out that it's another IOT protocol. From earlier, I know that I can filter based off of these protocols, so I'll start there. I'm met with this page:

<img width="1883" height="911" alt="Pasted image 20260917223342" src="https://github.com/user-attachments/assets/798059b6-242d-4396-b456-7ee0e5b275af" />

	You'll notice that the opcua_binary_status_code is listed in the `OPCUA Binary - Log Count`. If we hover our mouse over the option, we'll get an option to filter just based off of that. Now, we'll only get logs with that attribute. There are still 29 remaining logs, so let's see if we can find the one that says 'Create Monitored Items.' If we scroll down, we'll see the rest of the logs: 

<img width="2475" height="794" alt="Pasted image 20260917223644" src="https://github.com/user-attachments/assets/4a56261c-6ffd-4f74-b05b-e255a0d9a239" />
	
	And, you'll notice that the CreateMonitoredItems action is listed near the very bottom. Clicking into the log further reveals all of its attributes, including the `zeek.opcua_binary_status_code_detail.status_code`:

<img width="640" height="598" alt="Pasted image 20260917223836" src="https://github.com/user-attachments/assets/fdf72d58-4827-4ac7-9b3a-df0fbb0f8d40" />

10. Lastly, there's something called a communityID that the log sources share to agree on a single start to a communication session. If I wanted to find that ID, I would start by narrowing down the related protocol/IP/timestamp that I wanted to find. Once I had that and I was sure on the start of the communication timeline, I would simply filter by the event ID, then go to the start of the communication session in Arkime to track it down as an attribute.
