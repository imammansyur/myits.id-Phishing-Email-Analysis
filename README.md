# myits.id-Phishing-Email-Analysis
This is an analysis of a phishing email that mimics a login page from Sepuluh Nopember Institute of Technology (ITS) Surabaya.

## How It Started
I got the email on my university email inbox at September 18, 2024 with the subject of "Verify email account ...". I noticed that the sender has a weird domain `myits.id` when the real one supposed to be `its.ac.id`. It uses a similar domain from `my.its.ac.id` that was the login address of ITS at that time.  
<img width="342" height="83" alt="image" src="https://github.com/user-attachments/assets/d1f4b9dd-b416-43e7-8fec-b209c70fb4ab" />  
I was curious, so I opened it anyway, which I shouldn't have and immediately regretted when I found out what opening a phishing email can do. That was done before I knew I could just download the email and read the content safely. And that's a lesson learned.

## URL Analysis
The email has a button with the text "Verify Email" that leads to:
``` url
http://email.myits.id/c/eJxEjL2KwzAQhJ9GKsXualc_hYqDw_XBFaltr8ACJwRbCeTtg4qQKaaZ-T4tCbGmYGvBSCEQZ0a7Fao-EomurHEBpspSVbKgLiwro23fPyQfATwgOQ4QKDn2CJApi3imDBBSNAzXV-una2r3svV-P43_MTQZmj6D8dMx-vevXeZ5_7dHASHExDAiQ3L2h9Zbd4OY12F7FnoHAAD__13JNKw
```
Then, I checked where the link lead to using urlscan.io. It redirected to `https://myits.id/?rid=PiWaalS` at IP 63.142.251.217 with this view:  
<img width="70%" alt="1716ac61-9fe4-4bdd-90b1-f76785222b38" src="https://github.com/user-attachments/assets/39415cdf-bddb-4304-bf40-c994415af404" />  
URLScan result: https://urlscan.io/result/1716ac61-9fe4-4bdd-90b1-f76785222b38/  
This is the exact view of the `my.its.ac.id` at that time! Specifically, this is a credential harvesting website that mimics `my.its.ac.id` login page. The email is also targetted to someone within ITS since I received it. Unfortunately, it somehow pass the spam check and got into the inbox. It's unclear who or how many received the email, opened the link, and entered their login credential there.  
The WHOIS of the IP address and the domain might have already changed from the time the email was sent. As of August 15, 2025, here's the WHOIS info:  
<img width="549" height="337" alt="image" src="https://github.com/user-attachments/assets/6f35a789-079f-42f4-8c19-5519902b26f4" />
<img width="673" height="426" alt="image" src="https://github.com/user-attachments/assets/7e551679-7233-45e3-9218-464bf1591dae" />  

## Email Header Analysis
``` eml
Received: from a4.mail.mailgun.net (198.61.254.63) by
 HK2PEPF00006FAF.mail.protection.outlook.com (10.167.8.5) with Microsoft SMTP
 Server (version=TLS1_3, cipher=TLS_AES_256_GCM_SHA384) id 15.20.7918.13 via
 Frontend Transport; Wed, 18 Sep 2024 02:02:21 +0000
Received: from myits.id (unknown [63.142.251.217]) by 190cda81a40d with SMTP id
 66ea34adbfefb1e9acea8a1a (version=TLS1.3, cipher=TLS_AES_128_GCM_SHA256);
 Wed, 18 Sep 2024 02:02:21 GMT
```
This part of the email header shows that the attacker from `myits.id` uses a mail API service called Mailgun to send the email. Mailgun, just like any other email services, can be used to send spam or phishing emails. It has been reported 15 times on abuseipdb.
<img width="638" height="451" alt="image" src="https://github.com/user-attachments/assets/74626b93-2b21-4f8b-916b-ddec12dcccef" />

## Email Content Analysis
``` eml
Content-Type: text/html; charset="utf-8"
Content-Transfer-Encoding: quoted-printable
```
The content type set to HTML to enable the attacker to include link in a stylized button and images.  
Then they set the encoding to quoted printable to make it harder for spam scanner to detect any link inside the email. In this case, it works by encoding the `=` into `=3D` (`3D` is the hex value of `=`) and separating the long text with `=` at the end of the lines.  
<img width="609" height="189" alt="image" src="https://github.com/user-attachments/assets/1f14ed64-45fa-4a00-a11a-1136a37dc9af" />  
Then, the content also includes an attempt to track the email receiver.
``` html
<p><img alt="" src="http://localhost/track?rid=bJnnw5U" style="display: none"></p>

<p><img alt="" src="http://myits.id/track?rid=e96331b" style="display: none"></p>

<p><img alt="" src="http://localhost/track?rid=ckvXF0b" style="display: none"></p>

<p><img alt="" style="display: none" src="https://myits.id/track?rid=PiWaalS"></p>
<img width="1px" height="1px" alt="" src="http://email.myits.id/o/eJxEyj2OwyAQR_HTLCX6zxfMFBzGBqx1sVvETqTcPqLKa99vNCeaXtJsVLkU1lBKv626dBMENjl2jznFYq9kXYYx9yOdXw-XCgiIsxYU9qxCQHCYiXIAxeuP4u993lc-R3o0GBO5YmXrXfdzzP87L7H1hV6NPwEAAP__vt0nqw">
```
They use the img tag from HTML that retrieves an image from their website. I'm not sure if the img with display set to none would still be rendered and send the request, but the one with the size set to 1x1 pixel would. With this they can track how many have opened the email and their IP address. That's why you shouldn't open suspicious email.

## Lessons learned
- Don't open any suspicious email. Download the email if you're curious about the content.
- Always redact any personally identifiable information (PII) or sensitive information from anything that you want to share. That could be EXIF data or even hard-coded API Key.
- Make sure to set the default app to open .eml or .msg file to any text viewer so you don't accidentally open the email.
- Defang the link or make it unclickable.
- Those random `3D` on the email content is from quoted printable encoding.
