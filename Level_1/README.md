# Scratching the Surface
## Level 1 - Challenge 1

A file [file1.wav](https://api.tisc.csit-events.sg/file?id=ckr6sv183004v0838z5e2ioiy&name=file1.wav) was provided for analysis, which supposedly contains a secret message.

Running `file file1.wav` shows it is recognised as a valid audio file:
```
file1.wav: RIFF (little-endian) data, WAVE audio, Microsoft PCM, 16 bit, stereo 11025 Hz
```

Playing `file1.wav` I detected that the sound appears to be coming only from the left channel, even though the audio file had been encoded in stereo.

Opening `file1.wav` in **Audacity** to have a look at the audio data visually.
The second audio channel does look like it contains data (at a very low amplitude audio-wise). Perhaps the secret message is encoded in the second channel?

Extracting the right channel data into its own audio file with **ffmpeg**:
```bash
ffmpeg -i file1.wav -filter:a 'pan=mono|FC=FR' output.wav
```

Playing `output.wav` in **Audacity** I hear what sounds like Morse code!
A closer inspection of the visual waveform also confirms that the waveform seems to look like a series of "dots" and "dashes".
Transcribing the "dots" and "dashes" we get:
```
-.-. ... .. - .. ... .-.. --- -.-. .- - . -.. .. -. ... -.-. .. . -. -.-. . .--. .- .-. -.-
```

Throwing the Morse code into an online Morse Code Translator (https://morsedecoder.com/) returns the following text:
```
CSITISLOCATEDINSCIENCEPARK
```

The instructions require that this answer be in lowercase.

:triangular_flag_on_post: **Level 1 Challenge 1 flag: `TISC{csitislocatedinsciencepark}`**

<br>


## Level 1 - Challenge 2

A file [file2.jpg](https://api.tisc.csit-events.sg/file?id=ckr6swk6d006m0906vot9ga8l&name=file2.jpg) was provided for analysis. Question: What was the modifed time of the image?

Using `file file2.jpg` outputs some file metadata, including a timestamp `2003:08:25 14:55:27`:
```
file2.jpg: JPEG image data, Exif standard: [TIFF image data, little-endian, direntries=11, description=          , manufacturer=NIKON, model=E885, orientation=upper-left, xresolution=216, yresolution=224, resolutionunit=2, software=E885v1.1, datetime=2003:08:25 14:55:27], baseline, precision 8, 1024x768, components 3
```

:triangular_flag_on_post: **Level 1 Challenge 2 flag : `TISC{2003:08:25 14:55:27}`**

<br>


## Level 1 - Challenge 3

A file [file3.jpg](https://api.tisc.csit-events.sg/file?id=ckr6sxww900860838aged2020&name=file3.jpg) was provided for analysis. Apparently some secret message is encoded in it.

Running `file file3.jpg` gives us:
```
file3.jpg: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, progressive, precision 8, 360x360, components 3
```

---

Running `strings file3.jpg` does not reveal any readable text other than the string `picture_with_text.jpg` (which appears twice in the file).

---

Inspecting the contents of `file3.jpg` with a hex editor (GHex) we note that after the bytes 'FF D9' (which terminate a normal JPEG file) we see additional data beginning with the bytes 'PK' (which suggests there might be a Zip file there).

Using `dd` to extract the bytes from `file3.jpg` from the offset 0x1C10 (decimal 7184) to obtain `file3_a`:
```
dd if=file3.jpg bs=1 skip=7184 of=file3_a
```

Running `file file3_a` on it confirms it is a valid Zip file:
```
file3_a: Zip archive data, at least v2.0 to extract
```

Unzipping with `unzip file3_a` gives us the file `picture_with_text.jpg`.
But this is not a valid JPEG file.
It does not display in an image viewer and `file picture_with_text.jpg` also does not recognise it as a JPEG file:
```
picture_with_text.jpg: data
```

Inspecting `picture_with_text.jpg` in GHex we see that there are some additional bytes before the proper start of the JPEG file.
These bytes are:
```
NAFJRE GB GUVF PUNYYRATR VF URER NCCYRPNEEBGCRNE
```

The text looks like it could have been encoded with some simple replacement ciper.
Trying **ROT13** cipher in [CyberChef](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,13)&input=TkFGSlJFIEdCIEdVVkYgUFVOWVlSQVRSIFZGIFVSRVIgTkNDWVJQTkVFQkdDUk5F) produces the clear text:
```
ANSWER TO THIS CHALLENGE IS HERE APPLECARROTPEAR
```

:triangular_flag_on_post: **Level 1 Challenge 3 flag: `TISC{APPLECARROTPEAR}`**

<br>


## Level 1 Challenge 4

This and subsequent challenges in Level 1 are related to the Windows 10 virtual machine provided.
Question: What is the name of the user?

Running `whoami` from the command prompt shows:
```
desktop-8u7f1gr\adam
```

:triangular_flag_on_post: **Level 1 Challenge 4 flag: `TISC{adam}`**

<br>

## Level 1 Challenge 5

Question: What time was the user's most recent logon?

Starting up the Event Viewer (**eventvwr.msc**) to inspect the **Security** logs for previous logon events.
We are looking for the most recent logon by user account "DESKTOP-8U7F1GR\adam" before today.
We find such an event (Event ID 4624) at the following timestamp:
```
 Logged: 17/6/2021 10:41:37 am
```
Assuming this is Singapore local time (UTC+8), converting to UTC we get
```
17/06/2021 02:41:37
```

:triangular_flag_on_post: **Level 1 Challenge 5 flag: `TISC{17/06/2021 02:41:37}`**

<br>

## Level 1 Challenge 6

Question: A 7z archive was deleted, what is the value of the file CRC32 hash that is inside the 7z archive?

There appears to be something in the Recycle Bin. Opening up the Recycle Bin reveals a deleted file `sentosa-sea-aquarium.7z`, which should be the file of interest.

First we restore the file from the Recycle Bin (clicking on the "Restore" option on the context menu).
The file gets restored to the Desktop.
The Windows 10 virtual machine does not have 7zip or some other software that can work to decompress the contents of this 7z file, so I shall move this restored 7z archive off of the virtual machine and onto my host machine where I have more tools at my disposal.

On my host machine (a Linux box) I run `file sentosa-sea-aquarium.7z` to confirm the contents of the file:
```
sentosa-sea-aquarium.7z: 7-zip archive data, version 0.4
```

List the files in the 7z file using `7z l sentosa-sea-aquarium.7z`:
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

On the last line of the output on the screen, the CRC32 hash is given:
```
CRC32  for data:              040E23DA
```

:triangular_flag_on_post: **Level 1 Challenge 6 flag: `TISC{040E23DA}`**

<br>

## Level 1 Challenge 7

Question1: How many users have an RID of 1000 or above on the machine?<br>
Question2: What is the account name for RID of 501?<br>
Question3: What is the account name for RID of 503?

From the command prompt, running `wmic useraccount get name,sid` gives us the necessary information:
```
Name                SID
adam                S-1-5-21-271853984-2378250948-965456637-1002
Administrator       S-1-5-21-271853984-2378250948-965456637-500
DefaultAccount      S-1-5-21-271853984-2378250948-965456637-503
Guest               S-1-5-21-271853984-2378250948-965456637-501
WDAGUtilityAccount  S-1-5-21-271853984-2378250948-965456637-504
```

We have only 1 user (adam) with RID of above 1000 (1002).<br>
We have the user *Guest* with RID of 501.<br>
We have the user *DefaultAccount* with RID of 503.

:triangular_flag_on_post: **Level 1 Challenge 7 flag: `TISC{1-Guest-DefaultAccount}`**

<br>


