# Scratching the Surface
## Level 1 - Challenge 1

![image](https://user-images.githubusercontent.com/82754379/139614121-27c8ff25-c8b9-47b7-8416-4f997bfb93d9.png)

Running `file file1.wav` showed it is recognised as a valid audio file:
```
file1.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 11025 Hz
```

Playing `file1.wav` I detected that the sound appeared to be coming only from the left channel, even though the audio file had been encoded in stereo.

Opening `file1.wav` in **Audacity** to have a look at the audio data visually.

![image](https://user-images.githubusercontent.com/82754379/139616997-6c916f9e-8251-4d2f-9052-b0cbfd8f045d.png)

The second audio channel looked like it contained data (at a very low amplitude audio-wise). Perhaps the secret message was encoded in the second channel?

Extracting the right channel data into its own audio file with **ffmpeg**:
```bash
ffmpeg -i file1.wav -filter:a 'pan=mono|FC=FR' output.wav
```

Playing `output.wav` in **Audacity** I heard what sounded like Morse code!
A closer inspection of the visual waveform also confirmed that the waveform seemed to look like a series of "dots" and "dashes".

![image](https://user-images.githubusercontent.com/82754379/139616096-8841626d-8727-44e8-9c38-df7f42f8d182.png)

Transcribing the "dots" and "dashes" we got: 
```
-.-. ... .. - .. ... .-.. --- -.-. .- - . -.. .. -. ... -.-. .. . -. -.-. . .--. .- .-. -.-
```

Throwing the Morse code into an online Morse Code Translator (https://morsedecoder.com/) returned the following text:
```
CSITISLOCATEDINSCIENCEPARK
```

![image](https://user-images.githubusercontent.com/82754379/139617226-a522288a-c7ac-4ccd-8a76-1012d61cb62a.png)


The instructions required that this answer be in lowercase.

:triangular_flag_on_post: **Level 1 Challenge 1 flag: `TISC{csitislocatedinsciencepark}`**

<br>


## Level 1 - Challenge 2

![image](https://user-images.githubusercontent.com/82754379/139614313-e143e304-b6e8-4579-93d9-43cced08ceb2.png)


Using `file file2.jpg` provided some file metadata, including a timestamp `2003:08:25 14:55:27`:
```
file2.jpg: JPEG image data, Exif standard: [TIFF image data, little-endian, direntries=11, description=          , manufacturer=NIKON, model=E885, orientation=upper-left, xresolution=216, yresolution=224, resolutionunit=2, software=E885v1.1, datetime=2003:08:25 14:55:27], baseline, precision 8, 1024x768, components 3
```

:triangular_flag_on_post: **Level 1 Challenge 2 flag : `TISC{2003:08:25 14:55:27}`**

<br>


## Level 1 - Challenge 3

![image](https://user-images.githubusercontent.com/82754379/139614379-106d7f89-8c84-4435-a5eb-7d4bd4171999.png)

Running `file file3.jpg` gave:
```
file3.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, progressive, precision 8, 360x360, components 3
```

---

Running `strings file3.jpg` did not reveal any readable text other than the string `picture_with_text.jpg` (which appeared twice in the file).

---

Inspecting the contents of `file3.jpg` with a hex editor (GHex) I noted that after the bytes 'FF D9' (which terminate a normal JPEG file) I saw additional data beginning with the bytes 'PK' (which suggested there might be a Zip file there).

![image](https://user-images.githubusercontent.com/82754379/139616617-30ed6016-1a4c-4b3b-b0ad-6c3f2fa0ec8c.png)


Using `dd` to extract the bytes from `file3.jpg` from the offset 0x1C10 (decimal 7184) to obtain `file3_a`:
```
dd if=file3.jpg bs=1 skip=7184 of=file3_a
```

Running `file file3_a` on it confirmed it is a valid Zip file:
```
file3_a: Zip archive data, at least v2.0 to extract
```

Unzipping with `unzip file3_a` produced the file `picture_with_text.jpg`.
But this was not a valid JPEG file.
It did not display in an image viewer and `file picture_with_text.jpg` also did not recognise it as a JPEG file:
```
picture_with_text.jpg: data
```

Inspecting `picture_with_text.jpg` in GHex I saw that there were some additional bytes before the proper start of the JPEG file.
These bytes were:
```
NAFJRE GB GUVF PUNYYRATR VF URER NCCYRPNEEBGCRNE
```

![image](https://user-images.githubusercontent.com/82754379/141346481-bd45f84b-0583-438a-9f55-79b3b2d152f5.png)

<br>

The text looked like it could have been encoded with some simple substition ciper.
Trying **ROT13** cipher in [CyberChef](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,13)&input=TkFGSlJFIEdCIEdVVkYgUFVOWVlSQVRSIFZGIFVSRVIgTkNDWVJQTkVFQkdDUk5FCg) produced the clear text:
```
ANSWER TO THIS CHALLENGE IS HERE APPLECARROTPEAR
```

![image](https://user-images.githubusercontent.com/82754379/141345980-cc43a387-580c-4c0d-ad8c-96eff1727a41.png)


:triangular_flag_on_post: **Level 1 Challenge 3 flag: `TISC{APPLECARROTPEAR}`**

<br>


## Level 1 Challenge 4

![image](https://user-images.githubusercontent.com/82754379/139614542-dd1d1ad5-d4e1-43f4-a035-e0dfab6551d3.png)

This and subsequent challenges in Level 1 were related to the Windows 10 virtual machine provided.<br>

Running `whoami` from the command prompt shows:
```
desktop-8u7f1gr\adam
```

:triangular_flag_on_post: **Level 1 Challenge 4 flag: `TISC{adam}`**

<br>


## Level 1 Challenge 5

![image](https://user-images.githubusercontent.com/82754379/139614611-6975e690-faca-4fd1-b451-0361ab49b5d7.png)

Starting up the Event Viewer (**eventvwr.msc**) to inspect the **Security** logs for previous logon events.<br>
I was looking for the most recent logon by user account "DESKTOP-8U7F1GR\adam" before today.<br>
I found such an event (Event ID 4624) at the following timestamp:
```
 Logged: 17/6/2021 10:41:37 am
```

![image](https://user-images.githubusercontent.com/82754379/139615104-fb46c75b-d525-42bd-9917-364d04cffaf2.png)


Assuming this was Singapore local time (UTC+8), converting to UTC I got:
```
17/06/2021 02:41:37
```

:triangular_flag_on_post: **Level 1 Challenge 5 flag: `TISC{17/06/2021 02:41:37}`**

<br>


## Level 1 Challenge 6

![image](https://user-images.githubusercontent.com/82754379/139615174-4b660bf5-9e4e-41fe-b9d9-a6683be762dc.png)


There appeared to be something in the Recycle Bin. Opening up the Recycle Bin revealed a deleted file `sentosa-sea-aquarium.7z`, which should be the file of interest.

First I restored the file from the Recycle Bin (clicking on the "Restore" option on the context menu).
The file got restored to the Desktop.
The Windows 10 virtual machine did not have 7zip or some other software that could decompress the contents of this 7z file, so I moved the restored 7z archive off of the virtual machine and onto my host machine where I had more tools at my disposal.

On my host machine (a Linux box) I ran `file sentosa-sea-aquarium.7z` to confirm the the file was a 7-zip archive:
```
sentosa-sea-aquarium.7z: 7-zip archive data, version 0.4
```

Here was the list of files in the 7z file using `7z l sentosa-sea-aquarium.7z`:
```
   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2021-06-17 09:48:03 ....A        45620        45624  sentosa-sea-aquarium.jpg
------------------- ----- ------------ ------------  ------------------------
2021-06-17 09:48:03              45620        45624  1 files
```

There was only one file (sentosa-sea-aquarium.jpg) in the 7z archive.

Extracting with the option to calculate the CRC32 hash: 
```
7z e -scrcCRC32 sentosa-sea-aquarium.7z
```

On the last line of the output on the screen, the CRC32 hash was given:
```
CRC32  for data:              040E23DA
```

:triangular_flag_on_post: **Level 1 Challenge 6 flag: `TISC{040E23DA}`**

<br>


## Level 1 Challenge 7

![image](https://user-images.githubusercontent.com/82754379/139615309-73fea1f0-6c61-4e13-bde7-1cded6cb5c8c.png)

From the command prompt, running `wmic useraccount get name,sid` gave the necessary information:
```
Name                SID
adam                S-1-5-21-271853984-2378250948-965456637-1002
Administrator       S-1-5-21-271853984-2378250948-965456637-500
DefaultAccount      S-1-5-21-271853984-2378250948-965456637-503
Guest               S-1-5-21-271853984-2378250948-965456637-501
WDAGUtilityAccount  S-1-5-21-271853984-2378250948-965456637-504
```

There was only 1 user (adam) with RID of above 1000 (1002).<br>
There was the user *Guest* with RID of 501.<br>
There was the user *DefaultAccount* with RID of 503.

:triangular_flag_on_post: **Level 1 Challenge 7 flag: `TISC{1-Guest-DefaultAccount}`**

<br>


## Level 1 Challenge 8

![image](https://user-images.githubusercontent.com/82754379/139615369-a9074dcb-887a-46e8-b032-8632b16d47b0.png)

For this challenge I needed to inspect the *History* file of Microsoft Edge, which I found at:
```
C:\Users\adam\AppData\Local\Microsoft\Edge\User Data\Default\History
```

As this was an SQLite database file, I transfered the *History* file over to my host machine and from there, uploaded the file to the online [SQLite Viewer]( https://inloop.github.io/sqlite-viewer/).

The *urls* table in the database showed the visited URLs and also the *visit_count* for each visited URL.

![image](https://user-images.githubusercontent.com/82754379/139615493-9439ea35-3222-42d0-b8a0-2204e6d3bfdd.png)

I saw 2 visits to `https://www.csit.gov.sg/about-csit/who-we-are` but no visits to either `https://www.facebook.com` or `https://www.live.com`.

:triangular_flag_on_post: **Level 1 Challenge 8 flag: `TISC{2-0-0}`**

<br>


## Level 1 Challenge 9

![image](https://user-images.githubusercontent.com/82754379/139615572-a484659f-24de-4abe-9f09-9368670cad96.png)

The relevant Registry key may be viewed using **regedit**.
The key I looked at was:
```
Computer\HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MountPoints2
```
where I saw an entry for something that could have been the VirtualBox share (i.e. `\\VBoxSvr\vm-share`):
```
##VBoxSvr#vm-share
```

![image](https://user-images.githubusercontent.com/82754379/139615672-85321f42-3cb2-4087-98a2-276dcc76df45.png)


The "label" of the share would have been "vm-share".

:triangular_flag_on_post: **Level 1 Challenge 9 flag: `TISC{vm-share}`**

<br>


## Level 1 Challenge 10

![image](https://user-images.githubusercontent.com/82754379/139615763-878436d2-f43d-4635-b01c-617151ff61c5.png)

First, I checked against the [VirusTotal database](https://www.virustotal.com/gui/search/0D97DBDBA2D35C37F434538E4DFAA06FCCC18A13) in case this SHA1 hash is for a known Windows executable. There was no result returned by VirusTotal, so I assumed the SHA1 hash is for a custom file created for this challenge, and likely to be in one of the folders owned by the user "adam".

Starting at the folder location `c:\users\adam\` I generated the SHA1 hashes for all the files found under that folder, using Window's **certutil** program and placed them in a file called `results.txt`.
```
for /r %i in (*.*) do certutil -hashfile "%i" sha1 >> c:\users\adam\results.txt
```

It took a couple of minutes to generate the `results.txt`. I performed a case-insensitive search of the hash in the file using the **findstr** command:
```
findstr /I "0D97DBDBA2D35C37F434538E4DFAA06FCCC18A13" c:\users\adam\results.txt
```

It found a match! So next I opened the file in **notepad** and performed the search again for the hash, to find the following:
```
SHA1 hash of c:\Users\adam\AppData\Roaming\Microsoft\Windows\Recent\otter-singapore.lnk:
0d97dbdba2d35c37f434538e4dfaa06fccc18a13
```

I submitted the flag `TISC{otter-singapore.lnk}` but this was rejected. Perhaps the "file that is of interest" referred to the file this LNK file was pointing to. Looking into `otter-singapore.lnk` with **notepad** revealed that the LNK file pointed to `C:\Users\adam\Downloads\otter-singapore.jpg`.

:triangular_flag_on_post: **Level 1 Challenge 10 flag: `TISC{otter-singapore.jpg}`**

---

👍 **Level 1 Challenges have been completed!** 
