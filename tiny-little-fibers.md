First, I ran `file` command against it, to see what I'm dealing with
```sh
┌──(virtu)(raz㉿kali-wsl)-[~/nahamcon2023]
└─$ file tiny-little-fibers
tiny-little-fibers: JPEG image data, JFIF standard 1.01, aspect ratio, density 1x1, segment length 16, progressive, precision 8, 2056x1371, components 3
```
According to `file` output, this is a JPEG image, so let's have a look at its metadata using `exiftool`:
```sh
┌──(virtu)(raz㉿kali-wsl)-[~/nahamcon2023]
└─$ exiftool tiny-little-fibers
ExifTool Version Number         : 12.57
File Name                       : tiny-little-fibers
Directory                       : .
File Size                       : 622 kB
File Modification Date/Time     : 2023:06:20 08:38:03+03:00
File Access Date/Time           : 2023:06:20 09:52:56+03:00
File Inode Change Date/Time     : 2023:06:20 09:52:29+03:00
File Permissions                : -rwxrwxrwx
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Profile CMM Type                : Linotronic
Profile Version                 : 2.1.0
Profile Class                   : Display Device Profile
Color Space Data                : RGB
Profile Connection Space        : XYZ
Profile Date Time               : 1998:02:09 06:49:00
Profile File Signature          : acsp
Primary Platform                : Microsoft Corporation
CMM Flags                       : Not Embedded, Independent
Device Manufacturer             : Hewlett-Packard
Device Model                    : sRGB
Device Attributes               : Reflective, Glossy, Positive, Color
Rendering Intent                : Perceptual
Connection Space Illuminant     : 0.9642 1 0.82491
Profile Creator                 : Hewlett-Packard
Profile ID                      : 0
Profile Copyright               : Copyright (c) 1998 Hewlett-Packard Company
Profile Description             : sRGB IEC61966-2.1
Media White Point               : 0.95045 1 1.08905
Media Black Point               : 0 0 0
Red Matrix Column               : 0.43607 0.22249 0.01392
Green Matrix Column             : 0.38515 0.71687 0.09708
Blue Matrix Column              : 0.14307 0.06061 0.7141
Device Mfg Desc                 : IEC http://www.iec.ch
Device Model Desc               : IEC 61966-2.1 Default RGB colour space - sRGB
Viewing Cond Desc               : Reference Viewing Condition in IEC61966-2.1
Viewing Cond Illuminant         : 19.6445 20.3718 16.8089
Viewing Cond Surround           : 3.92889 4.07439 3.36179
Viewing Cond Illuminant Type    : D50
Luminance                       : 76.03647 80 87.12462
Measurement Observer            : CIE 1931
Measurement Backing             : 0 0 0
Measurement Geometry            : Unknown
Measurement Flare               : 0.999%
Measurement Illuminant          : D65
Technology                      : Cathode Ray Tube Display
Red Tone Reproduction Curve     : (Binary data 2060 bytes, use -b option to extract)
Green Tone Reproduction Curve   : (Binary data 2060 bytes, use -b option to extract)
Blue Tone Reproduction Curve    : (Binary data 2060 bytes, use -b option to extract)
Image Width                     : 2056
Image Height                    : 1371
Encoding Process                : Progressive DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 2056x1371
Megapixels                      : 2.8
```

A JPEG file should start with `ffdb`, so let's check using `xxd`:
```sh
┌──(virtu)(raz㉿kali-wsl)-[~/nahamcon2023]
└─$ xxd tiny-little-fibers | grep ffd8
00000000: ffd8 ffe0 0010 4a46 4946 0001 0100 0001  ......JFIF......
```

Good, now we want to see the end of the file (it should be `ffd9`). Same, using `xxd`:
```sh
┌──(virtu)(raz㉿kali-wsl)-[~/nahamcon2023]
└─$ xxd tiny-little-fibers | grep ffd9
0004c130: cf3f ffd9 6600 6c00 6100 0a00 6700 7b00  .?..f.l.a...g.{.
00097ec0: 1b02 5816 72e1 1321 7115 e2f8 cf3f ffd9  ..X.r..!q....?..
```
Well, it looks like we have two EOF (means there's some extra data after the legitimate file end). the first row looks like the beginning of our flag ( `f.l.a...g.{.` ) so at this point we want to examine further. Let's grab some more rows (let's say 10): 
```sh
┌──(virtu)(raz㉿kali-wsl)-[~/nahamcon2023]
└─$ xxd tiny-little-fibers| grep -A10 0004c130
0004c130: cf3f ffd9 6600 6c00 6100 0a00 6700 7b00  .?..f.l.a...g.{.
0004c140: 3200 0a00 3200 6300 3500 0a00 3300 3400  2...2.c.5...3.4.
0004c150: 6300 0a00 3500 6100 6200 0a00 6500 6100  c...5.a.b...e.a.
0004c160: 3800 0a00 3400 6200 6600 0a00 3600 6300  8...4.b.f...6.c.
0004c170: 3100 0a00 3100 3900 3300 0a00 6500 3200  1...1.9.3...e.2.
0004c180: 3600 0a00 3300 6600 3700 0a00 3200 3500  6...3.f.7...2.5.
0004c190: 3900 0a00 6600 7d00 0a00 0a00 0000 0013  9...f.}.........
0004c1a0: a4fe 0014 5f2e 0010 cf14 0003 edcc 0004  ...._...........
0004c1b0: 130b 0003 5c9e 0000 0001 5859 5a20 0000  ....\.....XYZ ..
0004c1c0: 0000 004c 0956 0050 0000 0057 1fe7 6d65  ...L.V.P...W..me
0004c1d0: 6173 0000 0000 0000 0001 0000 0000 0000  as..............
```
We have the flag spread amongst 7 lines, so let's grab the next 6, reverse it into string and remove spaces:
```sh
┌──(virtu)(raz㉿kali-wsl)-[~/nahamcon2023]
└─$ xxd tiny-little-fibers| grep -A6 0004c130 | xxd -r | tr --delete [:space:]
�?��flag{22c534c5abea84bf6c1193e263f7259f}
```

We can also grab it without newline and send it to clipboard for easy paste (assuming you're running WSL2).
```sh
┌──(virtu)(raz㉿kali-wsl)-[~/nahamcon2023]
└─$ echo -n $(xxd -s 0x0004c130 tiny-little-fibers | head -n7 | xxd -r | tr --delete [:space:] | sed -e 's/[^[:lower:]|[:digit:]|{}]*//g' | cut -c 4-) | clip.exe

```

Brief explanation of the command, as you should never copy&paste commands that you don't understand into your terminal:

`xxd -s 0x0004c130 tiny-little-fibers` - xxd starting at the specific offset

`head -n7` - grab only the first 7 lines

`xxd -r` - reverse xxd

`tr --delete [:space:]` - delete spaces/newlines, so you get the flag as one line

`sed -e 's/[^[:lower:]|[:digit:]|{}]*//g'` - remove unwanted characters (keep only lowercase, numbers and brackets )

`cut -c 4-` - remove the first three nagging "���" characters


`echo -n $() | clip.exe` - this will echo the output for the commands chain without trailing newline and pipe it to clipboard.


That's all folks!
