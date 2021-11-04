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

After much research into the PE format I was still getting stuck, until I switched focus to research into the BMP file format. 
That was when I gained "enlightenment" :grin: 

<br><br>

From the details the Wikipedia article - [BMP file format](https://en.wikipedia.org/wiki/BMP_file_format), this was the key piece of information:

> ...pixels are stored "bottom-up", starting in the lower left corner, going from left to right, and then row by row from the bottom to the top of the image.

Now it made perfect sense why I was seeing out-of-sequence content and the sections of the PE file appearing in reverse order.

---












## 2.bmp

```bash
2.bmp: PC bitmap, Windows 3.x format, 99 x 99 x 8, image size 9900, resolution 3780 x 3780 px/m, 256 important colors, cbSize 10978, bits offset 1078

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PC bitmap, Windows 3.x format,, 99 x 99 x 8

```

---

The indicated sub-contents in `1.bmp` looked promising. Extracted the revelant portions to separate files `bmp1_1` and `bmp1_2` with **dd**:

```bash
dd if=1.bmp of=bmp1_1 bs=1 skip=1666 count=354
dd if=1.bmp of=bmp1_2 bs=1 skip=2020
```

`bmp1_1` seemed to contain some fragment of an application manifest. <br>
`bmp1_2` contained part of what appeared to be the same application manifest, and probably also the compiled code and symbols for some Windows app. <br>

Some observations: The application manifest seemed to start at the beginning of `bmp1_2`, that is:
```
<?xml version='1.0' encoding='UTF-8' standalone='yes'?>
<assembly xmlns='urn:schemas-micr
```

and continued into the middle portion of `bmp1_1`:
```
soft-com:asm.v1' manifestVersion='1.0'>
  <trustInfo xmlns="urn:schemas-microsoft-com:asm.v3">
    <security>
      <requestedPrivileges>
```

and then ended at the _beginning_ of `bmp1_1`:
```
        <requestedExecutionLevel level='asInvoker' uiAccess='false' />
      </requestedPrivileges>
    </security>
  </trustInfo>
</assembly>
```

As unfamiliar as I am with the application manifest or how it may be packed into a final executable, I do not know if this is normal or intentionally placed out-of-sequence. Even if I were able to re-assemble the manifest in the correct order, I do not know where to put this XML snippet in the mass of bytes, to derive the final file which I may then load into a Windows debugger.

---

Observations about `2.bmp`: Although **binwalk** did not indicate other filetypes lurking within the BMP file, using **GHex** I saw that there were readable English words at the end of the file (from offset 0x4FE, or offset 1278 in decimal). I suspect these words may be used by the Windows program embedded in `1.bmp`.
Perhaps the combination of some of these words lead to the final "encryption key" that I needed to find?


😞 **I am officially out-of-my-depth here**
