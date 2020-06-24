# gopro-utils-gps
This software get out GPS data from hevc (h265)

GPS output formats:
- gpx
- kml
- json
- csv

Gyro, Accel, Temperature - output format:
- csv

# Dependencies 
## Binary static version recommended
download https://ffmpeg.zeranoe.com/builds/
- copy this files to Bin Directory
ffmpeg.exe
ffplay.exe
ffprobe.exe

## Build own exe files by go language
- Build go files in Bin to got exe
All procedure is here
https://medium.com/@lucaselbert/extracting-gopro-gps-and-other-telemetry-data-fadf97ed1834
or 
https://community.gopro.com/t5/GoPro-Telemetry-GPS-overlays/Extracting-the-metadata-in-a-useful-format/gpm-p/40293

# Gopro Quik not working with windows 10 - fix it (Intel should works 100%, AMD not exactly)
https://gatetoadventures.com/how-to-fix-playback-issues-with-hevc-files-from-a-gopro-hero7-or-hero6-black-on-pc/

# GoPro Metadata Format Parser + GPMD2CSV

**I have recently switched my efforts to a more comprehensive [JavaScript version of these tools](https://github.com/JuanIrache/gopro-telemetry), but feel free to send a PR if you think you can improve this one**

Examples of what can be achieved: https://goprotelemetryextractor.com/gallery

User friendly and cross-platform tool for extracting the telemetry: https://goprotelemetryextractor.com/free

I forked stilldavid's project ( https://github.com/stilldavid/gopro-utils ) to achieve 3 things:

- Export the data in csv format from /bin/gpmd2csv/gpmd2csv.go
- Allow the project to work with GoPro's h5 v2.00 firmware
- Create a tool for easy data extraction. That's the GPMD2CSV folder. You can just drag and drop the GoPro video files on the BATCH file. If you're not used to github, you can download the tool here: https://tailorandwayne.com/gpmd2csv/

Over time, we have added other exporting tools. They all follow the same pattern when used with the extracted metadata .bin:

`gpmd2csv -i GOPR0001.bin -o GOPR0001.csv`

Aditionally to `-i` and `-o`, the gopro2gpx and gopro2kml tools allow for an `-a` accuracy option for filtering out bad GPS locations (default 1000, the lower the more accurate) and `-f` for type of fix (default 3. 0- no fix, 2 - 2D fix, 3 - 3D fix)

`gopro2gpx -i GOPR0001.bin -a 500 -f 2 -o GOPR0001.gpx`

The gpmd2csv instead allows for a `-s` option to select which data to export. It accepts the following:

- a: Accelerometer
- g: GPS
- y: Gyroscope
- t: Camera temperature

For example, in order to export gyroscope and GPS data only we would do

`gpmd2csv -i GOPR0001.bin -s yg`

If `-s` is not specified, it will export all available data. More options could be added in the future.

ToDo:

- Add other sensors to JSON export

This was my first ~~repository~~ fork. Any possible wrong practices are not intentional.

If you liked this you might like some of my [app prototyping](https://prototyping.barcelona).

Here continues Stilldavid's work:
##############################################################################################################

TLDR:

1.

```
ffmpeg -y -i GOPR0001.MP4 -codec copy -map 0:m:handler_name:"	GoPro MET" -f rawvideo GOPR0001.bin
```

Note the gap before GoPro MET should be a TAB, not a space. Also, the handler_name and position changes between camera models and frame rates. There should be a way to target always the right stream.

2. `gopro2json -i GOPR0001.bin -o GOPR0001.json`

3. There is no step 3

---

I spent some time trying to reverse-engineer the GoPro Metadata Format (GPMD or GPMDF) that is stored in GoPro Hero 5 cameras if GPS is enabled. This is what I found.

Part of this code is in production on [Earthscape](https://public.earthscape.com/); for an example of what you can do with the extracted data, see [this video](https://public.earthscape.com/videos/10231).

If you enjoy working on this sort of thing, please see our [careers page](https://churchillnavigation.com/careers/).

## Extracting the Metadata File

The metadata stream is stored in the `.mp4` video file itself alongside the video and audio streams. We can use `ffprobe` to find it:

```
[computar][100GOPRO] ➔ ffprobe GOPR0008.MP4
ffprobe version 3.2.4 Copyright (c) 2007-2017 the FFmpeg developers
[SNIP]
    Stream #0:3(eng): Data: none (gpmd / 0x646D7067), 33 kb/s (default)
    Metadata:
      creation_time   : 2016-11-22T23:42:41.000000Z
      handler_name    : 	GoPro MET
[SNIP]
```

We can identify it by the `gpmd` in the tag string - in this case it's id 3. We can then use `ffmpeg` to extract the metadata stream into a binary file for processing:

`ffmpeg -y -i GOPR0001.MP4 -codec copy -map 0:3 -f rawvideo out-0001.bin`

This leaves us with a binary file with the data.

## Data We Get

- ~400 Hz 3-axis gyro readings
- ~200 Hz 3-axis accelerometer readings
- ~18 Hz GPS position (lat/lon/alt/spd)
- 1 Hz GPS timestamps
- 1 Hz GPS accuracy (cm) and fix (2d/3d)
- 1 Hz temperature of camera

---

## The Protocol

Data starts with a label that describes the data following it. Values are all big endian, and floats are IEEE 754. Everything is packed to 4 bytes where applicable, padded with zeroes so it's 32-bit aligned.

- **Labels** - human readable types of proceeding data
- **Type** - single ascii character describing data
- **Size** - how big is the data type
- **Count** - how many values are we going to get
- **Length** = size \* count

Labels include:

- `ACCL` - accelerometer reading x/y/z
- `DEVC` - device
- `DVID` - device ID, possibly hard-coded to 0x1
- `DVNM` - devicde name, string "Camera"
- `EMPT` - empty packet
- `GPS5` - GPS data (lat, lon, alt, speed, 3d speed)
- `GPSF` - GPS fix (none, 2d, 3d)
- `GPSP` - GPS positional accuracy in cm
- `GPSU` - GPS acquired timestamp; potentially different than "camera time"
- `GYRO` - gryroscope reading x/y/z
- `SCAL` - scale factor, a multiplier for subsequent data
- `SIUN` - SI units; strings (m/s², rad/s)
- `STRM` - ¯\\\_(ツ)\_/¯
- `TMPC` - temperature
- `TSMP` - total number of samples
- `UNIT` - alternative units; strings (deg, m, m/s)

Types include:

- `c` - single char
- `L` - unsigned long
- `s` - signed short
- `S` - unsigned short
- `f` - 32 float

For implementation details, see `reader.go` and other corresponding files in `telemetry/`.


===============INSTRUCTIONS=============== v3.29

For a more user friendly and comprehensive telemetry extraction tool please use https://goprotelemetryextractor.com/free/

Just drop your GoPro Hero5 (or later) files GPMD2VS.bat. A "GoPro Metadata Extract" folder will appear alongside your files with your data in multiple file formats.

You can also drop a file on "GPMD2CSV Folder Process.bat" if you want all your video files in your folder processed.

This is an example of what can be done with the extracted data (Just Hero5 Session IMU, no GPS in this case) https://youtu.be/bg8B0Hl_au0
Give us a like if you like this tool :).

Source and bug reporting: https://github.com/JuanIrache/gopro-utils


===============OPTIONS===================

Note: You can no longer provide multiple file names to batch process files via command line because new command line options have been implemented.
It's best to use "GPMD2CSV Folder Process.bat" for batch processing files in the same directory.

You can edit GPMD2CSV.bat to change some export preferences:
	By Default your files will be exported to:
		<MP4 Source File Directory>\GoPro Metadata Extract\<Input File Name>\<Input File Name>.<File Extension>
		Example: C:\GoPro\GOPR1234.mp4 would output to: C:\GoPro\GoPro Metadata Extract\GOPR1234\GOPR1234.kml
	Change "BatchOutputFolder=GoPro Metadata Extract" to adjust the name of the export folder (or delete everything after "=" and an export folder won't be created).
	Change "IndSubDir=Yes" to enable creation of individual sub folders or not (Exports to "/<filename>/files" or just to "/files"
	Change "AccuracyFilter=1000" to adjust the Accuracy Filter
	Change "FixFilter=3" to adjust the Fix Filter.
	
You may also run "GPMD2CSV.bat" or "GPMD2CSV Folder Process.bat" via command line with options and override the settings in the file.
	If running from command line, the usage format is below.  Note that "<BatchOutputFolder>" "<IndSubDir>" "<AccuracyFilter>" "<FixFilter>" are all optional.
	GPMD2CSV.bat "<MP4 Input File>" "<BatchOutputFolder>" "<IndSubDir>" "<AccuracyFilter>" "<FixFilter>"

The script filters bad GPS locations by accuracy and type of GPS fix (GPX and KML files only).
If you'd prefer a less demanding filter you can modify "AccuracyFilter=" and "FixFilter=" options in the GPMD2VS.bat file.
A higher -a value will tolerate lower accuracy and a lower -f value will tolerate 2d fixes.

Note that if you make changes to the script you must delete the older data files if you want those same videos analysed with the new options.

================BINARIES=================
If you copied this tool from GitHub it might not include the necessary binaries.
Please download the full version from https://tailorandwayne.com/gpmd2csv/ or compile the binaries yourself and reproduce the following structure

GPMD2CSV/
  GPMD2CSV.bat
  bin/
    ffmpeg.exe
    gopro2gpx.exe
    gopro2json.exe
    gpmd2csv.exe
    gps2kml.exe
    gopro2json.exe

===============CREATED BY================
Juan Irache

Script optimisations and enhancements by Alex Denning

===============ATTRIBUTION===============

######## This software uses a custom gpmdinfo based on https://github.com/stilldavid/gopro-utils ########

BSD 2-Clause License

Copyright (c) 2017, Churchill Navigation
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

######## This software uses KonradIT's gopro2gpx and gps2kml https://github.com/KonradIT/gopro-utils ########

BSD 2-Clause License

Copyright (c) 2017
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

* Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.

* Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.


######## This software uses libraries from the FFmpeg project under the LGPLv2.1 https://www.ffmpeg.org/ ########

FFmpeg is licensed under the GNU Lesser General Public License (LGPL) version 2.1 https://www.gnu.org/licenses/old-licenses/lgpl-2.1.html

This is a FFmpeg win64 static build by Kyle Schwarz.

Zeranoe's FFmpeg Builds Home Page: <http://ffmpeg.zeranoe.com/builds/>

FFmpeg version: 20170327-d65b595

This FFmpeg build was configured with:
  --enable-gpl
  --enable-version3
  --enable-cuda
  --enable-cuvid
  --enable-d3d11va
  --enable-dxva2
  --enable-libmfx
  --enable-nvenc
  --enable-avisynth
  --enable-bzlib
  --enable-fontconfig
  --enable-frei0r
  --enable-gnutls
  --enable-iconv
  --enable-libass
  --enable-libbluray
  --enable-libbs2b
  --enable-libcaca
  --enable-libfreetype
  --enable-libgme
  --enable-libgsm
  --enable-libilbc
  --enable-libmodplug
  --enable-libmp3lame
  --enable-libopencore-amrnb
  --enable-libopencore-amrwb
  --enable-libopenh264
  --enable-libopenjpeg
  --enable-libopus
  --enable-librtmp
  --enable-libsnappy
  --enable-libsoxr
  --enable-libspeex
  --enable-libtheora
  --enable-libtwolame
  --enable-libvidstab
  --enable-libvo-amrwbenc
  --enable-libvorbis
  --enable-libvpx
  --enable-libwavpack
  --enable-libwebp
  --enable-libx264
  --enable-libx265
  --enable-libxavs
  --enable-libxvid
  --enable-libzimg
  --enable-lzma
  --enable-zlib

This build was compiled with the following external libraries:
  libmfx 1.19 <https://ffmpeg.zeranoe.com>
  bzip2 1.0.6 <http://bzip.org/>
  Fontconfig 2.12.1 <http://freedesktop.org/wiki/Software/fontconfig>
  Frei0r 20130909-git-10d8360 <http://frei0r.dyne.org/>
  GnuTLS 3.5.8 <http://gnutls.org/>
  libiconv 1.14 <http://gnu.org/software/libiconv/>
  libass 0.13.6 <https://github.com/libass/libass>
  libbluray 20170124-60e3d26 <http://videolan.org/developers/libbluray.html>
  libbs2b 3.1.0 <http://bs2b.sourceforge.net/>
  libcaca 0.99.beta19 <http://caca.zoy.org/wiki/libcaca>
  FreeType 2.7.1 <http://freetype.sourceforge.net/>
  Game Music Emu 0.6.1 <https://bitbucket.org/mpyne/game-music-emu/wiki/Home>
  GSM 1.0.13-4 <http://packages.debian.org/source/squeeze/libgsm>
  iLBC 20160404-746f8e2 <https://github.com/dekkers/libilbc/>
  Modplug-XMMS 0.8.8.5 <http://modplug-xmms.sourceforge.net/>
  LAME 3.99.5 <http://lame.sourceforge.net/>
  OpenCORE AMR 0.1.3 <http://sourceforge.net/projects/opencore-amr/>
  OpenH264 1.6.0 <https://github.com/cisco/openh264>
  OpenJPEG 2.1.2 <https://github.com/uclouvain/openjpeg>
  Opus 1.1.4 <http://opus-codec.org/>
  RTMPDump 20151223-git-fa8646d <http://rtmpdump.mplayerhq.hu/>
  Snappy 20170127-2d99bd1 <https://github.com/google/snappy>
  libsoxr 0.1.2 <http://sourceforge.net/projects/soxr/>
  Speex 1.2.0 <http://speex.org/>
  Theora 1.1.1 <http://theora.org/>
  TwoLAME 0.3.13 <http://twolame.org/>
  vid.stab 0.98 <http://public.hronopik.de/vid.stab/>
  VisualOn AMR-WB 0.1.2 <https://github.com/mstorsjo/vo-amrwbenc>
  Vorbis 1.3.5 <http://vorbis.com/>
  vpx 1.6.1 <http://webmproject.org/>
  WavPack 5.1.0 <http://wavpack.com/>
  WebP 0.6.0 <https://developers.google.com/speed/webp/>
  x264 20170123-90a61ec <http://videolan.org/developers/x264.html>
  x265 2.3 <https://bitbucket.org/multicoreware/x265/wiki/Home>
  XAVS svn-r55 <http://xavs.sourceforge.net/>
  Xvid 1.3.4 <http://xvid.org/>
  z.lib 20170202-3608f63 <https://github.com/sekrit-twc/zimg>
  XZ Utils 5.2.3 <http://tukaani.org/xz>
  zlib 1.2.8 <http://zlib.net/>

The source code for this FFmpeg build can be found at: <http://ffmpeg.zeranoe.com/builds/source/>

This build was compiled on Ubuntu 16.04.2 LTS: <https://www.ubuntu.com/>

GCC 6.3.0 was used to compile this FFmpeg build: <http://gcc.gnu.org/>

This build was compiled using the MinGW-w64 toolchain: <http://mingw-w64.sourceforge.net/>

Licenses for each library can be found in the 'licenses' folder. 
