# Secure DDNS - Namecheap SSL With DDClient

> Keep Namecheap DNS records updated automatically with DDClient over SSL. Configure dynamic DNS for roaming public IP addresses with encrypted HTTPS protocol.

Source: https://exitcode0.net/posts/secure-ddns-namecheap-ssl-with-ddclient/
Author: Tom Cocking (https://tomcocking.com)
Published: 2020-01-25
Updated: 2026-10-08
Tags: ddclient, ddns, namecheap



[https://ddclient.net](https://ddclient.net/ "https://ddclient.net/")
A quote short post on how to secure your DDNS updates with Namecheap, SSL and DDClient. For those of us who use dynamic DNS to work around roaming IP addresses, it is important to make sure that you are updating your DNS records securely with SSL.

The default DDclient config has an example configuration file for using Namecheap’s DDNS service, however, it does not use SSL to check for your IP. In theory that connection could be manipulated and a false IP result could be returned – updating your DNS records to a wrong, malicious IP could cause a number of problems.

Namecheap SSL DDClient Config Example
-------------------------------------

```
ssl=yes
use=web, web=dynamicdns.park-your-domain.com/getip
protocol=namecheap
server=dynamicdns.park-your-domain.com
login=<your domain goes here>
password=<your ddns password goes here>
<your sub domain goes here>
```

Something worth noting, Namecheap also have not made the effort to put this option to use SSL in their example config:  
[https://www.namecheap.com/support/knowledgebase/article.aspx/583/11/how-do-i-configure-ddclient](https://www.namecheap.com/support/knowledgebase/article.aspx/583/11/how-do-i-configure-ddclient "https://www.namecheap.com/support/knowledgebase/article.aspx/583/11/how-do-i-configure-ddclient")



---
Markdown version of https://exitcode0.net/posts/secure-ddns-namecheap-ssl-with-ddclient/ for agents and readers who prefer plain text. The HTML page has the comments, cover image and related posts.
