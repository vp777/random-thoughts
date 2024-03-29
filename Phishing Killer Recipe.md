# Phishing Killer: Recipe

This recipe primarily targets the staff phishing with email and web flavors.

## Ingredients

We will need:

 1. Speargun: can kill φish(ing), at least certain species of it
	 - Mark externally received emails. (e.g. prepend a message on subject or body)
	 	- On Subject you could prepend the text: "EXTERNAL:"
		- On Email body, the message should be concise, self-explanatory, clearly visible (maybe some nice red highlighted text), and you should avoid including redundant statements. For example if you utilize applocker (properly configured) with macros disabled on office or maybe an endpoint sandbox solution, then you probably don't care much whether users click links/download files/execute attachments.
		> Note: You may want to check with your legal/marketing departments on modifying received emails. Another consideration is the validity of digital signatures, especially after changing email body.
	 - Awareness training on what that prepended message means
	 > In most of the situations, before the user falls into the phishing trap, there is a moment of doubt/confusion created by the contents of the email (e.g. asking the user to do something unusual). You want your users in that exact moment, to have clear instructions on how they should react. They should create a connection that when in doubt+that red annoying line at the beginning of the email=notify security (ingredient [5] should help with forging that equation into their minds)
	 > 
	 > Note: The only issue here is that some people get disturbed by the brutality of killing φish this way.
	 - Bonus: Another interesting approach is to transfer the concept discussed above on the web gateway. One way to accomplish this is by configuring the web gateway to add a [red frame](https://imgur.com/a/M17pf1y) for example (inject html) in every external website. Then you can easily prevent credential theft by some basic awareness training on what that red frame means.
	 > The catch here is that the injection code should work reliably most of the time and this is not a trivial thing without native support from browsers.
 2. Φishing bait: catches specific types of φish(ing)
	 - Proactively blacklist potential phishing domains (i.e. domains "similar" with yours) on email/web gateway ([dnstwist](https://github.com/elceef/dnstwist) is a good start)
	 - Set up an alert when the above rule is triggered, you may have some "sophisticated" attackers (or red-teamers) coming your way.
 3. Anti-anti-φishing bait: detects the attempt of getting around the φishing bait
	 - Configure the email clients to display internationalized domain names in emails with their punycode representation.
	 - Configure email gateway to strip display name when it matches potentially some c-level executive's name. (the match ideally should be based on a similarity threshold, if not possible, dnstwist can come in handy)
 4. Sonar: detects φish(ing)
	 - Create a correlation rule with the traffic to your web servers where the "Referer" http field is different than "your.domain". (e.g. catch simple reverse proxy, external resource usage, simple redirection when phishing is done). It will take some time before false positives are eliminated.
	 - Create a correlation rule for characteristics found in official emails of your organization. (e.g. your email signature). Be careful with reply to reply to avoid false positives.
	 - Enrich web gateway logs with a boolean value (phishy) populated based on the identification of the text "@your.domain" in the request body. This will be an indicator that user credentials may have been leaked.
	 - Create the capability of monitoring the links opened through email. Couple of ways to achieve this:
		 - Asynchronously: 
            - Enrich email gateway logs to include a field with the urls found in the emails.
            > It should be easy to deploy, and it will probably be available out of the box in case url reputation check is enabled on email gw.
		 - Synchronously: 
			- Specifically for Windows, monitor the [process creation event](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4688). When a link from an email is clicked, a new process of the default internet browser is created and the url is passed as an argument to that process.
            > Ok, almost synchronous, but to the point that allow us to reap most of the benefits of synchronous monitoring.
            > 
            > It should be noted that the process creation event logging is disabled by default as well as the logging of command line arguments. Both should be enabled to have this going. One limitation is that the parent process image is not included in the logs (just its pid), so it would be hard to tell for example when a link was clicked through outlook. 
            > Finally, a word of caution, the logging of command line arguments could reveal sensitive information in some instances.
            > 
            > An alternative solution without any of the shortcomings described above is to configure sysmon. The only drawback is the effort that would take to set it up at the beginning. It should be noted that with the visibility offered through sysmon, we essentially unlock super detection powers on the endpoints. One example, is the ability to detect files downloaded through the web (i.e. monitoring the FileCreateStreamHash event for the [Mark of The Web](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-fscc/6e3f7352-d11c-4d76-8c39-2516a9df36e8)). So know you can find out whether a link send via email, led to a file being downloaded and ultimately executed.
            - Create a rule on email gateway that will append (or prepend, i.e. domain) something unique like: "/sandbox-static-guid" in every link found in emails, essentially adding them a signature. Then if that signature is found in the proxy logs, it should be safe to assume that the session originated from an email. (of course the proxy will then have to redirect the user to the original link)
            > Truly synchronous, but careful planning will be required before deployment. Other than modifying the email body (so breaking potential digital signatures), its functionality depends on the presence of a "collaborating" web gateway, which might not be always the case. (e.g. maybe when a user is connected through WiFi)
     - With asynchronous email link monitoring:    
		 - Create a correlation rule using the new proxy field, phishy, and pop an alert when its domain is found within an email. 
		 - Create a correlation rule based on the navigation to resources (e.g. compressed files, documents) from the web gateway when the domain matches one from an email. (medium false-positive, can be calibrated)
         > Note: Before building these correlations you should take into consideration the asynchronous nature of our link from email detection capability.  For example, there could be considerable amount of time difference between the point an email is sent and the point it's actually read. (e.g. an email sent on Saturday is likely to be read on Monday).
	 - With synchronous email link monitoring:
		 - Create a correlation rule that will be activated when an email link is clicked and within 5 minutes, a phishy url is detected.
		 - Create a correlation rule that will be activated when an email link is clicked and within 5 minutes, there is navigation to "suspicious" resources (e.g. compressed files, documents). (medium false-positive, can be calibrated)
         > Note: Here we don't have the complications of the asynchronous mode since we know fairly well when a link from an email is clicked.
         > 
         > Additionally, with the appended "signature" synchronous mode, we could go beyond detection and with the help of the Web Gateway enable prevention. This can be achieved by setting up stricter rules to be applied on a session, when is detected to originate from an email. (e.g. prevent users submitting data/credentials, block file downloading, or just apply the red (frame) speargum).
 5. Spearfisherman: maintains and operates all the above, may catch an octopus occasionally by himself but to be effective he needs his tools
	 - Simulated phishing campaigns

Water: email source verification (e.g. dmarc), antivirus scan, attachment extension blacklist/whitelist, maybe some threat emulation and html sanitization (the controls here are supposed to be applied on email gateway)

## Execution

You carefully knead the first four ingredients together with water and store the dough in the fridge.

Then every couple of months, throw some spearfisherman in the mix to keep it fresh and ready.

## Notes

This recipe is supposed to be easy in its execution, other maybe than finding a good speargun, and should have some decent taste as well.

For further enhancement of the taste, and when the price (not only money) is not an issue, one should consider:

 1. Separate environments (ideally in both physical and network level) for user and administration tasks. (e.g [privileged access workstation](https://docs.microsoft.com/en-us/windows-server/identity/securing-privileged-access/privileged-access-workstations)). It will help minimizing the impact in case of malware infection.
 2. Unphishable multi-factor authentication. It should eliminate the impact of credential theft.
 3. Deploy a remote browser isolation solution. It could minimize the impact of malware infection, and decrease the likelihood of credential theft.
 
On the end point:
 1. applocker. When properly configured,  it will considerably decrease the likelihood of malware infection
 2. sysmon/EDR. By setting up the appropriate rules, it will decrease significantly the impact of malware infection (i.e. quick detection). It could also decrease the impact of credential theft. (e.g. assist in detection through the synchronous email link discovery capability described above)
 3. Sandbox for applications. A solution with capabilities similar to Sandboxie, can eliminate the impact of malware infection. It could also be a cheap substitute of a browser isolation solution.
