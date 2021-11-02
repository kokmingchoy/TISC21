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





