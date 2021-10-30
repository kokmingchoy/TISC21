# TISC21

Area for storing code and notes related to The InfoSecurity Challenge (TISC) 2021, organised by the Center for Strategic Infocomm Technologies (CSIT).

Level0 flag (after answering survey) : `TISC{Br1ng_0n_th3_ch4ll3ng3s!!}`

## Level 1.1

A file [file1.wav](https://api.tisc.csit-events.sg/file?id=ckr6sv183004v0838z5e2ioiy&name=file1.wav) was provided for analysis, which supposedly contains a secret message.

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

