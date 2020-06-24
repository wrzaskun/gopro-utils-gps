# gopro-utils-gps
This software get out GPS data from hevc (h265)

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
