# Level 3

![image](https://user-images.githubusercontent.com/82754379/139781511-3e6d3825-0434-4930-b8ab-0cb5841a12a2.png)

<br>

First, some basic info gathering with **file** and **binwalk** :

## 1.bmp

```bash
1.bmp: PC bitmap, Windows 3.x format, 145 x 145 x 8, image size 21460, resolution 3780 x 3780 px/m, 256 important colors, cbSize 22538, bits offset 1078

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             PC bitmap, Windows 3.x format,, 145 x 145 x 8
1666          0x682           Broadcom 96345 firmware header, header size: 256, firmware version: "asIn", board id: "voker' uiAccess='false' />", ~CRC32 header checksum: 0x6F6D3A61, ~CRC32 data checksum: 0x3A736368
2020          0x7E4           XML document, version: "1.0"

```

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
      <requestedPrivileges
```

and then ended at the _beginning_ of `bmp1_1`:
```
        <requestedExecutionLevel level='asInvoker' uiAccess='false' />
      </requestedPrivileges>
    </security>
  </trustInfo>
</assembly>
```

As unfamiliar as I am with the application manifest or how it may be packed into a final executable, I do not know if this is normal or intentionally placed out-of-sequence. Even if I were to be able to re-assemble the manifest in the correct order, I do not know where to put this XML code in this mass of bytes, to derive the final file which I may then load into a Windows debugger.

😞 **I am officially out-of-my-depth here**
