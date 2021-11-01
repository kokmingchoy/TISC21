# Level 2 - Part 1

![image](https://user-images.githubusercontent.com/82754379/139622184-2c5a46b3-5704-4943-80a6-b1217fa1e746.png)

Viewing the provided `traffic.pcap` file in Wireshark, I noted that it contained data for 19,487 packets of DNS query responses for the query name in the following format:<br>
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
21NRXXEZL05NEBUXA4453VNUQGI0433MN5ZC02A43JOQQ02GC3LFOQ17WCAQKCI01NCEKRSH01JBEUUS201MJVHE6U01CRKJJVI09VKWK5MF01SWRQGEZ01DGNBVGY013TQOLBM01JRWIZLG09M5UGS2T17LNRWW43043QOFZHG205DVOZ3X16Q6L2FMX33SA3LBMV23RWK3TBO04MQHM33M29OV2HAYL17UEBRW6305TENFWWK013TUOVWS01AZLHMVZ11XIYLTFY33QHAZLMN07RSW45DF01ONYXKZJ01AOZUXIY01LFEBYG6204TUORUX01I33SEB202HK4TQNF49ZSYIDTM01VSCAZTB23MNUWY2L53TNFZSA214LQON2W248LRAMR2W28S4ZAOZS40WYIDJNZ182GK4TEO38VWSA3LJ28FQQGC5B40AMRQXA218LCOVZSA32YLVM52W22KLRANVX08XEYTJEB383HK3DQO49V2GC5DF26EB2WY5D24SNFRWSZ14LTEB3HK303DQOV2G12C5DFFYQ40GK5DJMF34WSAYJA ...
```

<br>

# Level 2 - Part 2

![image](https://user-images.githubusercontent.com/82754379/139627876-19431090-dff4-46c9-a286-fb7291b39771.png)
