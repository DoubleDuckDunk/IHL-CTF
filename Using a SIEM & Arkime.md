Malcolm is a SIEM integration tool that bundles Suricata, Zeek, and Arkime logs into one pane. You can use the Dashboards view to aggregate findings. It also has the Netbox application for asset tracking.

Once you get into the Dashboard, ensure your timetable is correct in the top-right:
![[Pasted image 20260917121059.png]]

The following are a few 'if' scenarios. For example, if you wanted to find the application protocol with the most usage, you would simply go into the Overview dashboard and the list is on the initial view: ![[Pasted image 20260917121806.png]]

2. If you were trying to find the specific PDU reference number of a command, you can click into one of the available options provided on the left-side in Malcolm. Once you do, you'll be given a brea
![[Pasted image 20260917122143.png]]

After you get into the IOT dashboards, you can scroll down to the Zeek logs ot the bottom. One of these has a PLC_Stop command we'll get the PDU reference number from. Some other log attributes are captured for comparison
![[Pasted image 20260917122800.png]]


3. If you were trying to track an Uninventoried Host, you can look at the Zeek Known Summary to find known and unknown hosts. The bottom of the dashboard will give you Uninventoried Observed Hosts.
![[Pasted image 20260917123019.png]]

4. In order for Zeek, Suricata, and other log tools to agree on 'single sources of truth' and correlate data, they use what's called a CommunityID. This ID is shared among all log tools.  You can make this ID searchable in Arkime by putting it in the soarch tool. You'll see that the top of the screenshot contains the search, and the bottom contains the flag for this particular CTF.
![[Pasted image 20260917124721.png]]


5. If you want to find information contained within data transmissions, you would usually use either Arkime (big data) or Wireshark (one-off, smaller data). You can also use Malcolm (dashboard) to find specific logs (Zeek/Suricata), then track them down in Arkime. 

	Malcolm has severity ratiings for a lot of the logs that it ingests. a high-severity event would likely be a password sent in cleartext. If you're prompted to find data transmissons over FTP, you can likely find these passwords.

	Once in the 'severity' dashboard, scroll down to application protocol, where you'll find FTP:
	![[Pasted image 20260917221605.png]]

	Click into FTP. This will take you to Arkime. Filter based on your time period you're looking at, and you'll get a few data transmissions using FTP. Click through them, and look at their attributes.
	![[Pasted image 20260917221720.png]]
	You'll want to specifically click into the log type of the application (FTP) you're wanting to work with. Here, you'll find the cleartext password at the bottom!
	
	![[Pasted image 20260917221833.png]]


6. If you want to look at files that your SIEM extracts, that's usually available as an option as well. In Malcolm, going to the Files dashboard will give you a table that looks like this:
	 ![[Pasted image 20260917222135.png]]
	If you want to download a file, simply click the `zeek.files.extracted_uri` link for the file you want, and that will start the download process. These files will usually download as .zip files, but you can reliably see the contents with 7-zip. Here's an example of the FTP file we saw that you can pull!
	![[Pasted image 20260917222526.png]]

7. Sometimes, if you want to find a specific value for a specific log, you'll have to related items, or pieces of the item, then drill down further by further tracing the log. 

	Here, I have to find the Status Code (zeek.opcua_binary_status_code_detail.status_code) for the 'OPCUA Binary - Action' `Create Monitored Items`. If I look up what OPCUA is, I'll find out that it's another IOT protocol. From earlier, I know that I can filter based off of these protocols, so I'll start there. I'm met with this page:![[Pasted image 20260917223342.png]]

	You'll notice that the opcua_binary_status_code is listed in the `OPCUA Binary - Log Count`. If we hover our mouse over the option, we'll get an option to filter just based off of that. Now, we'll only get logs with that attribute. There are still 29 remaining logs, so let's see if we can find the one that says 'Create Monitored Items.' If we scroll down, we'll see the rest of the logs: ![[Pasted image 20260917223644.png]]
	And, you'll notice that the CreateMonitoredItems action is listed near the very bottom. Clicking into the log further reveals all of its attributes, including the `zeek.opcua_binary_status_code_detail.status_code`:
	![[Pasted image 20260917223836.png]]

8. Lastly, there's something called a communityID that the log sources share to agree on a single start to a communication session. If I wanted to find that ID, I would start by narrowing down the related protocol/IP/timestamp that I wanted to find. Once I had that and I was sure on the start of the communication timeline, I would simply filter by the event ID, then go to the start of the communication session in Arkime to track it down as an attribute.