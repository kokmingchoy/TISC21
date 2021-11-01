# Level 2 - Part 1

![image](https://user-images.githubusercontent.com/82754379/139622184-2c5a46b3-5704-4943-80a6-b1217fa1e746.png)

Viewing the provided `traffic.pcap` file in **Wireshark**, I noted that it contained data for 19,487 packets of DNS query responses for various query names in the following format:<br>
> d33d**xxxxxxxxx**.tentopspot.net

![image](https://user-images.githubusercontent.com/82754379/139656227-75c722ba-2e62-42ac-85b6-4dff1c31797e.png)

The interesting part of the query name was the varying 9-character portion after "d33d".
I collected those characters into a single file with the following procedures:

First, use **tshark** to pipe the data from `traffic.pcap` through **sed** to extract the interesting 9-character string from each of the DNS packets:
```bash
tshark -r traffic.pcap | sed -n -r 's/.*?d33d(.*?)\.tentopspot.*/\1/p' >> dns.txt
```

The resultant `dns.txt` had line breaks which needed to be removed:
```bash
tr -d '\n' < dns.txt > dns2.txt
```

Now I had `dns2.txt` which looked something like the following:
```
21NRXXEZL05NEBUXA4453VNUQGI0433MN5ZC02A43JOQQ02GC3LFOQ17WCAQKCI01NCEKRSH01JBEUUS
201MJVHE6U01CRKJJVI09VKWK5MF01SWRQGEZ01DGNBVGY013TQOLBM01JRWIZLG09M5UGS2T17LNRWW
43043QOFZHG205DVOZ3X16Q6L2FMX33SA3LBMV23RWK3TBO04MQHM33M29OV2HAYL17UEBRW6305TENF
WWK013TUOVWS01AZLHMVZ11XIYLTFY33QHAZLMN07RSW45DF01ONYXKZJ01AOZUXIY01LFEBYG6204TU
(truncated for brevity)
```

I investigated the contents of `dns2.txt` using [CyberChef](https://gchq.github.io/CyberChef/) in an attempt to figure out what kind of encoding it may be in, but did not have any conclusive results initially.

Then I noticed that the characters "d33d" in the domain name were _always_ followed by 2 digits. I thought it would be very unlikely that any encoding scheme would have 2 digits occurring at regular intervals, separated by 7 other characters. So I decided to ignore those 2 digits following the characters "d33d" and collected the 7-character strings into a single file:
```bash
tshark -r traffic.pcap | sed -n -r 's/.*?d33d..(.*?)\.ten.*/\1/p' >> dns3.txt
tr -d '\n' < dns3.txt > dns4.txt
```

The resultant `dns4.txt` now looked like this:
```
NRXXEZLNEBUXA43VNUQGI33MN5ZCA43JOQQGC3LFOQWCAQKCINCEKRSHJBEUUS2MJVHE6UCRKJJVIVKWK5
MFSWRQGEZDGNBVGY3TQOLBMJRWIZLGM5UGS2TLNRWW433QOFZHG5DVOZ3XQ6L2FMXSA3LBMVRWK3TBOMQH
M33MOV2HAYLUEBRW63TENFWWK3TUOVWSAZLHMVZXIYLTFYQHAZLMNRSW45DFONYXKZJAOZUXIYLFEBYG64
TUORUXI33SEB2HK4TQNFZSYIDTMVSCAZTBMNUWY2LTNFZSA2LQON2W2LRAMR2WS4ZAOZSWYIDJNZ2GK4TE
(truncated for brevity)
```

I noted that there were only uppercase characters in there, so the data could not be Base64-encoded.<br>
(Base64 uses the character set `A-Za-z0-9+/=`).
What about Base32 encoding? (Base32 uses the character set `A-Z2-7=` and the digits '0' and '1' were noticeably missing from the data in `dns4.txt`)

Using [CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Base32('A-Z2-7%3D',true)&input=TlJYWEVaTE5FQlVYQTQzVk5VUUdJMDQzM01ONVpDMDJBNDNKT1FRMDJHQzNMRk9RMTdXQ0FRS0NJMDFOQ0VLUlNIMDFKQkVVVVMyMDFNSlZIRTZVMDFDUktKSlZJMDlWS1dLNU1GMDFTV1JRR0VaMDFER05CVkdZMDEzVFFPTEJNMDFKUldJWkxHMDlNNVVHUzJU) with the "From Base32" recipe revealed readable text:

![image](https://user-images.githubusercontent.com/82754379/139676359-ba09f563-0e50-4f47-9fbf-b224ae55c96c.png)

More importantly, it revealed our flag!

🚩 **Level 2 Part 1 flag: `TISC{n3vEr_0dd_0r_Ev3n}`**

<br>


# Level 2 - Part 2

![image](https://user-images.githubusercontent.com/82754379/139627876-19431090-dff4-46c9-a286-fb7291b39771.png)
