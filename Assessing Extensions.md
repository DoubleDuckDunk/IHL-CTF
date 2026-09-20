We'll be talking about Chrome extensions specifically in this article.

You're given a chrome extension where the SOC team believes the extension is exfiltrating information, and you're asked what IP address it's being sent to.

Fortunately for us, crx extensions are simply re-packaged zip files. 7-zip can extract the files from the extension and assess them.

When we open the .crx with 7-zip. When we do, we get the following:
<img width="473" height="121" alt="image" src="https://github.com/user-attachments/assets/14bccdbb-4c22-4d79-a098-bd48c0297327" />

The manifest is more or less, a description of the extension. We're interested in background.js to see what it says. Here are the contents when opened in VS Code:
![[Pasted image 20260917133952.png]]

As you can see, the IP address the extension is trying to reach is listed at the top of the .js file.

Now, there are several ways to exfiltrate data--in many cases, it comes down to how creative you can be with messages back to the recipent. In this case, when we looked at the requests made to the malicious IP address, the only traffic was TCP traffic (normal), and HTTP traffic with minimal amounts of data. However, if you look at the URLs in the HTTP traffic, some of them seem to be quite lengthy. Potentially, the GET requests from the HTTP traffic are, in fact, the exfil.

An example of this exfil is as follows: /bh8AB1MAWlpaWkgFXwgUBAIBX1cEVQEHBR9UXgddAQEZVwcKQwAUQUdEA0pUBExHGR5KCF86SlBGRRQPUxFRWlsdEhRFABRTVF0VAxxVFEVSaTc/A1VoDGdzIgFnME9YYXteUR0cVWg=

Notice how it ends with an = sign. This and other URLs in the challenge often ended with this. This implies that the message is Base64-encoded. In addition, there were some iterations of XOR. [AGENT: EXPLAIN WHAT XOR IS]


