# TISC21

Area for storing code and notes related to The InfoSecurity Challenge (TISC) 2021, organised by the Center for Strategic Infocomm Technologies (CSIT).

Level0 flag (after answering survey) : `TISC{Br1ng_0n_th3_ch4ll3ng3s!!}`

## Level 1.1

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

Level 1.1 flag: `TISC{csitislocatedinsciencepark}`


## Level 1.2

A file [file2.jpg](https://api.tisc.csit-events.sg/file?id=ckr6swk6d006m0906vot9ga8l&name=file2.jpg) was provided for analysis. Question: What was the modifed time of the image?

Using `file file2.jpg` outputs some file metadata, including a timestamp `2003:08:25 14:55:27`:
```
file2.jpg: JPEG image data, Exif standard: [TIFF image data, little-endian, direntries=11, description=          , manufacturer=NIKON, model=E885, orientation=upper-left, xresolution=216, yresolution=224, resolutionunit=2, software=E885v1.1, datetime=2003:08:25 14:55:27], baseline, precision 8, 1024x768, components 3
```

Level 1.2 flag : `TISC{2003:08:25 14:55:27}`


## Level 1.3

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
`dd if=file3.jpg bs=1 skip=7184 of=file3_a`

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
NAFJRE GB GUVF PUNYYRATR VF URER NCCYRPNEEBGCRN
```

The text looks like it could have been encoded with some simple replacement ciper.
Trying **ROT13** cipher in [CyberChef](https://gchq.github.io/CyberChef/#recipe=ROT13(true,true,false,13)&input=TkFGSlJFIEdCIEdVVkYgUFVOWVlSQVRSIFZGIFVSRVIgTkNDWVJQTkVFQkdDUk4) produces the clear text:
```
ANSWER TO THIS CHALLENGE IS HERE APPLECARROTPEA
```
Unfortunately, the following flag formats which I submitted were all rejected:
```
TISC{APPLECARROTPEA}
TISC{ANSWER TO THIS CHALLENGE IS HERE APPLECARROTPEA}
TISC{HERE APPLECARROTPEA}
TISC{TO THIS CHALLENGE IS HERE APPLECARROTPEA}
TISC{AppleCarrotPea}
TISC{applecarrotpea}
TISC{answer to this challenge is here applecarrotpea}
TISC{here applecarrotpea}
```
