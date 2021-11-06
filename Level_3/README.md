# Level 3

![image](https://user-images.githubusercontent.com/82754379/139781511-3e6d3825-0434-4930-b8ab-0cb5841a12a2.png)

<br>

## 1.bmp

First, some basic information gathering using **file** and **binwalk**:

```bash
1.bmp: PC bitmap, Windows 3.x format, 145 x 145 x 8, image size 21460, resolution 3780 x 3780 px/m, 256 important colors, cbSize 22538, bits offset 1078

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PC bitmap, Windows 3.x format,, 145 x 145 x 8
1666          0x682           Broadcom 96345 firmware header, header size: 256, firmware version: "asIn", board id: "voker' uiAccess='false' />", ~CRC32 header checksum: 0x6F6D3A61, ~CRC32 data checksum: 0x3A736368
2020          0x7E4           XML document, version: "1.0"

```

Displaying `1.bmp` to screen using an image viewer just showed garbled pixels. <br>
Inspecting with **Ghex** revealed that there were what appeared to be portions of an executable file (i.e. Windows Portable Executable or PE format) embedded in the `1.bmp`, for example:

![image](https://user-images.githubusercontent.com/82754379/140280315-d3061160-6429-4846-a996-acb8cb97d02c.png)

<br>

There were also indications that debugging information for the executable was also embedded in `1.bmp` :

![image](https://user-images.githubusercontent.com/82754379/140280634-05ca111b-a9f2-4cb3-befe-ee970fd6120c.png)

![image](https://user-images.githubusercontent.com/82754379/140280870-c1fe7846-2866-4539-8c4b-938d26a309ec.png)

<br>
However, the content in there appeared to be out-of-sequence in many places. I thought that might have been done intentionally to thwart the easy extraction of the hidden content, and that I might be forced to manually re-assemble the sections of the PE file in the correct sequence based on knowledge of the PE format.

<br>
<br>

After much research into the PE format I was still getting stuck, until I switched focus to research into the BMP file format. 
That was when I gained "enlightenment" :grin: 

<br>

From the details the Wikipedia article - [BMP file format](https://en.wikipedia.org/wiki/BMP_file_format), this was the key piece of information:


> ...pixels are stored "bottom-up", starting in the lower left corner, going from left to right, and then row by row from the bottom to the top of the image.


Now it made perfect sense why I was seeing out-of-sequence content and the sections of the PE file appearing in reverse order.

---

From the output of `file 1.bmp`, we know that the actual "image" data in `1.bmp` starts at offset 1078, so first we extract that "image" data into a separate file `hidden_exe`:

```bash
dd if=1.bmp of=hidden_exe bs=1 skip=1078
```

The file size of `hidden_exe` was 21460 bytes (which agreed with the _image size_ indicated by the `file 1.bmp` command). <br>
21460 divided by 145 "pixels" (at 8 bits or 1 byte per pixel) across for each row of data in the image gives us 148 bytes per row. <br>

To recover the original content embedded in `1.bmp` I would need to read from `hidden_exe`, one "line" at a time (where each line is 148 bytes) but write out the "lines" of data in reverse order from when they were read. There are 3 bytes (148-145=3) of padding at the end of each "line" of data, which need to be excluded  when writing the output.

The following Python script will extract the bytes from `hidden_exe` and write them out in the correct sequence to `final_exe` (21025 bytes):

```python
rows_of_data = []
with open("hidden_exe", "rb") as file:
    for i in range(145):        # There are 145 rows of data expected
        bytes_read = file.read(148)     # Reading 148 bytes per row, which includes padding
        rows_of_data.append(bytes_read[:-3])  # Drop the last 3 bytes of padding

with open("final_exe", "wb") as output:
    for i in range(len(rows_of_data)-1, -1, -1):   # Write rows in reverse order from when they were read in
        output.write(bytes(rows_of_data[i]))
```

<br>

---

I moved `final_exe` over to a Windows machine and renamed the file to `a.exe` (since that was the original name of the compiled executable, following the observation that the readable text `D:\writeup\TISC\ChallengeDesign\repo\a\Release\a.pdb` was seen in the executable).

I loaded `a.exe` into **Ghidra** for analysis:

![image](https://user-images.githubusercontent.com/82754379/140459420-c735250c-f9fb-435b-875a-8b5cb5e619a4.png)

When run without command line parameters, the unknown program `a.exe` simply printed out:

[image](https://user-images.githubusercontent.com/82754379/140600711-37c97814-611d-4d09-b046-b093f099f857.png)

I managed to find the specific function which output that text.



<br><br>

---

## 2.bmp

Basic information-gathering for `2.bmp` with **binwalk** and **file**:

```bash
2.bmp: PC bitmap, Windows 3.x format, 99 x 99 x 8, image size 9900, resolution 3780 x 3780 px/m, 256 important colors, cbSize 10978, bits offset 1078

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PC bitmap, Windows 3.x format,, 99 x 99 x 8

```

Inspecting with **Ghex** revealed that there were English words embedded in the `2.bmp` file.
I used the same approach as I did with `1.bmp` to extract the embedded content into a separate file `hidden_words` (9900 bytes):

```bash
dd if=2.bmp of=hidden_words bs=1 skip=1078
```

<br>

`2.bmp` has different dimensions (99 x 99) so the Python code to extract the hidden content needed to be adjusted accordingly, in order to obtain the `final_words` file (9801 bytes):

```python
rows_of_data = []
with open("hidden_words", "rb") as file:
    for i in range(99):        # There are 99 rows of data expected
        bytes_read = file.read(100)     # Reading 100 bytes per row, which includes padding
        rows_of_data.append(bytes_read[:-1])  # Drop the last 1 byte of padding

with open("final_words", "wb") as output:
    for i in range(len(rows_of_data)-1, -1, -1):    # Write rows in reverse order from when they were read in
        output.write(bytes(rows_of_data[i]))
```

---
