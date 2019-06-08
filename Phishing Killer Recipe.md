# Phishing Killer: Recipe

This recipe primarily targets the staff phishing with email and web flavors.

## Ingredients

We will need:

 1. Speargun: can kill φish(ing), at least certain species of it
	 - Mark externally received emails. (e.g. prepend a message on subject or body)
	 	- On Subject it could be a simple: "EXTERNAL:"
		- On Email body, the message should be concise, self-explanatory, clearly visible (maybe some nice red highlighted text), and you should avoid including reduntant statements. For example if you utilize applocker with good coverage with macros disabled on office or maybe an endpoint sandbox solution, then you probably don't care much whether users click links/download files/execute attachments.
		> Note: You may want to check with your legal/marketing departments on modifying received emails. Another consideration is the validity of digital signatures, especially after changing email body.
	 - Awareness training on what that prepended message means
	 > In most of the situations, before the user falls into the phishing trap, there is a moment of doubt/confusion created by the contents of the email (e.g. asking the user to do something unusual). You want your users in that exact moment, to have clear instructions on how they should react. They should create a connection that when in doubt+that red annoying line at the beginning of the email=notify security (ingredient [5] should help with forging that equation into their minds)
	 > 
	 > Note: The only issue here is that some people get disturbed by the brutality of killing φish this way.
	 - Experimental, but could still have some great results, is to transfer the concept above in the web gateway. One way to accomplish this is by configuring the web gateway to add a [red frame](https://imgur.com/a/M17pf1y) for example (inject html) in every external website. Then you can easily prevent credential theft by some basic awareness training on what that red frame means.
	 > The catch here is that the injection code should work reliably most of the time and this is not a trivial thing without native support from browsers.
 2. Φishing bait: catches specific types of φish(ing)
	 - Proactively blacklist potential phishing domains on email/web gateway ([dnstwist](https://github.com/elceef/dnstwist) is a good start)
	 - Set up an alert when the above rule is triggered, you may have some "sophisticated" attackers (or red-teamers) coming your way.
 3. Anti-anti-φishing bait: detects the attempt of getting around the φishing bait
	 - Configure the email clients to display internationalized domain names in emails with their punycode representation.
	 - Configure email gateway to strip display name when it matches potentially some c-level executive's name. (the match ideally should be based on a similarity threshold, if not possible, dnstwist can come in handy)
 4. Sonar: detects φish(ing)
	 - Create a correlation rule with the traffic to your web servers where the "Referer" http field is different than "your.domain". (e.g. catch simple reverse proxy, external resource usage, simple redirection when phishing is done). It will take some time before false positives are eliminated.
	 - Create a correlation rule for characteristics found in official emails of your organization. (e.g. your email signature). Be careful with reply to reply to avoid false positives.
	 - Enrich web gateway logs with a boolean value (phishy) populated based on the identification of the text "@your.domain" in the request body. This will be an indicator that user credentials may have been leaked.
	 - Create the capability of monitoring the links opened through email. Couple of ways to achieve this:
		 - Asynchronously: Enrich email gateway logs to include the urls found in the emails (let's call this field eurl).
		 - Synchronously: Create a rule on email gateway that will append (or prepend, i.e. domain) something unique like: "/sandbox-static-guid" in every link found in emails, essentially adding them a signature. Then if that signature is found in the proxy logs, it should be safe to assume that the session originated from an email and the proxy can redirect the user to the original link. 
         > Asynchronous capability will normally be easier to deploy, and probably be available out of the box with url reputation check. The Synchronous capability on the other hand may need some careful planning before deployment, since except from the fact that it requires the modification of the message body (so breaking potential digital signatures), its functionality depends on the presence of a "collaborating" web gateway, which might not be always the case. (e.g. maybe when a user is connected through WiFi)
     - With asynchronous email link monitoring:    
		 - Create a correlation rule using the new proxy field, phishy, with the eurl and pop an alert when the domains match. 
		 - Create a correlation rule based on the navigation to resources (e.g. compressed files, documents) from the web gateway when the domain matches an eurl. (medium false-positive, can be calibrated)
         > Note: Before building this correlation you should take into consideration the potential time difference between the point an email is sent and the point it's actually read. (e.g. an email sent on Saturday is likely to be read on Monday).
	 - With synchronous email link monitoring:
		 - Create a correlation rule that will be activated when an email link is clicked and within 5 minutes, a phishy url is detected.
		 - Create a correlation rule that will be activated when an email link is clicked and within 5 minutes, there is navigation to "suspicious" resources (e.g. compressed files, documents). (medium false-positive, can be calibrated)
         > Note: Here we don't have the complications of the asynchronous mode since we know exactly when a link from an email is clicked. This of course comes at the cost of being more "invasive" in detecting the links.
         > 
         > Additionally, in the synchronous mode, we could go beyond detection and with the help of the Web Gateway enable prevention. This can be achieved by setting up stricter rules to be applied on a session, when is detected to originate from an email. (e.g. prevent users submitting data/credentials, block file downloading, or just apply speargun #2).
 5. Spearfisherman: maintains and operates all the above, may catch an octopus occasionally by himself but to be effective he needs his tools
	 - Simulated phishing campaigns

Water: email source verification (e.g. dmarc), antivirus scan, attachment extension blacklist/whitelist, maybe some threat emulation and html sanitization

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
 2. sysmon/EDR. By setting up the appropriate rules, it will increase significantly the probability of detecting malware infection.
 3. Sandbox for applications. A solution with capabilities similar to Sandboxie, can eliminate the impact of malware infection. It could also be a cheap substitute of a browser isolation solution.
