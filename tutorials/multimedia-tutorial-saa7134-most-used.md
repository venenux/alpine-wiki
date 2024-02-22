

## Introduction and differncies

USB or PCI, both technologies, have more bandwidth than is necessary to tune 
television channels and to handle audio signals if we talk about a single 
management channel (a single tuner and a single sound device with only one 
or two applications or sources. audio being processed)

What will make the difference is the TV tuner chip, whether or not there is 
hardware encoding or decoding. Today almost everything that is sold is without 
hardware decoding since the processors are so powerful that they do it through 
software, an old PCI TV Card almost always comes with a hardware encoder, which 
allows your machine to be more free of work and even make better use of mixes 
simply because it frees up work. In short, USB devices are heavier and require 
extra work from your machine.

1. If you have multiple tuners, the PCI bus is larger than the USB2 bus, so you 
will eventually be able to saturate the USB bus before the PCI bus.
2. PCI tuner cards will sometimes have better driver availability on more 
operating systems than USB cards.

### Introduction too SAA7134/SAA7133/SAA7135

This chip its handled in many TV cards, and most of those already icludes an build-in sound card (basic but a capable sound card)

Notable mentions are:

* Hauppauge WinTV-HVR-1120 (only in older kernels, ironic) https://linuxtv.org/wiki/index.php/Hauppauge_WinTV-HVR-1120
* Hauppauge WinTV-HVR-1110 / HVR-1100 https://linuxtv.org/wiki/index.php/Hauppauge_WinTV-HVR-1110
* Gigabyte GT-P8000 (limited support) https://linuxtv.org/wiki/index.php/Gigabyte_GT-P8000
* Bona TV-PCI https://linuxtv.org/wiki/index.php/Bona_TV-PCI
* The Pinnacle PCTV Stereo (hardware based) https://linuxtv.org/wiki/index.php/Pinnacle_PCTV_Stereo
* KWorld DVB-T 210 (software based) https://linuxtv.org/wiki/index.php/KWorld_DVB-T_210
* KNC One TV-Station RDS https://linuxtv.org/wiki/index.php/KNC_One_TV-Station_RDS
* Sabrent SBT-TVFM https://linuxtv.org/wiki/index.php/Sabrent_SBT-TVFM
* Auvisio TV Tuner & Video Capture Card https://linuxtv.org/wiki/index.php/Auvisio_TV_Tuner_%26_Video_Capture_Card
* AVerMedia AVerTV GO 007 https://linuxtv.org/wiki/index.php/AVerMedia_AVerTV_GO_007
* LifeView FlyDVB-T Duo CardBus https://linuxtv.org/wiki/index.php/LifeView_FlyDVB-T_Duo_CardBus
* Tevion MD 9717 https://linuxtv.org/wiki/index.php/Tevion_MD_9717
* LifeView FlyVideo3000 PAL-N/NTSC/etc  https://linuxtv.org/wiki/index.php/LifeView_FlyVideo3000_PAL-N
* LifeView FlyVideo2000 https://linuxtv.org/wiki/index.php/LifeView_FlyVideo2000
* LifeView FlyVideo3000FM NTSC https://linuxtv.org/wiki/index.php/LifeView_FlyVideo3000FM_NTSC
* AVerMedia Super 007 https://linuxtv.org/wiki/index.php/AVerMedia_Super_007
* Pinnacle PCTV 50i https://linuxtv.org/wiki/index.php/Pinnacle_PCTV_50i
* Encore ENLTV or Encore ENLTV Argentina https://linuxtv.org/wiki/index.php/Encore_ENLTV_Argentina
* FlyDVB-T Hybrid https://linuxtv.org/wiki/index.php/LifeView_FlyDVB-T_Hybrid
* FlyDVB-T Hybrid CardBus https://linuxtv.org/wiki/index.php/LifeView_FlyDVB-T_Hybrid_CardBus
* FlyDVB-T Duo CardBus https://linuxtv.org/wiki/index.php/LifeView_FlyDVB-T_Duo_CardBus
* FlyTV Platinum  https://linuxtv.org/wiki/index.php/LifeView_FlyTV_Platinum

Unsupported due stupid developers work:

* Pinnacle PCTV Dual Hybrid Pro PCI Express (3010i) https://linuxtv.org/wiki/index.php/Pinnacle_PCTV_Dual_Hybrid_Pro_PCI_Express_(3010i)
* TerraTec Cinergy 600 / TerraTec Cinergy 400 https://linuxtv.org/wiki/index.php/TerraTec_Cinergy_600

### status of supported cards

Most of those cards works perfectly \-- it gets 640x480 frames, audio directly 
from the card, and closed captioning. Inclusive with up to five cards
in one computer, though you need a powerful CPU to keep up with the data
flow.

For detailed images of the various slightly different models, see [bttv
gallery](http://www.bttv-gallery.de/). Most of LifeView\'s saa713x cards
fit both 32-bit PCI slots and 64-bit PCI-X slots \-- see the telltale
second slit in the foot of the card. Most of them are old PCI bus and 
there is not yet a PCIe bus based!

### Identification

Most of cards are just rebranded, excelent example is the LifeView series, 
those are same as \'FlyTV\' range also called the FlyTV Prime 33 or
also FlyTV Prime 33FM. The \'33\' indicates that this card uses the SAA7133 chip
Ths is not the same as SAA7134 of this document. There are also \'Prime 30(FM)\'
and \'Prime 34(FM)\' product lines, indicating the use respective use of
the SAA7130 and SAA7134 chips.

` lspci -vv | grep  -e Multimedia -e Phillips -e Video -e SAA -A 1`

You will have something like:
```
0000:00:0a.0 Multimedia controller: Philips Semiconductors SAA7134 FlyView ...
       Subsystem: Unknown device 5169:0138
```

Most of those cards have the
[saa7133HL-v101](http://www.semiconductors.philips.com/pip/SAA7133HL_V101.html)
chip. But Some of those already has a multimedia sound card build-in, so we 
need to setup in parts:

## Testing saa7134 support modules: decoder, tuner and sound

The card needs the saa7134 module, which should be built as a module in
the kernel. When the module is inserted, your particular variant may be
autodetected or you may need to manually configure it. Those cards in fact 
dont have good autodetection process so its better to remove and reload 
to test the right configuration, also you will need the saa7134-alsa for the 
build-in sound card and also specify the build-in tunner sometimes separatly:


1. Try first `modprobe -f -r saa7134 saa7134-alsa`, mostly 
in debian it load automatically, so you must remove it first 
and later try with `modprobe saa7134` to load again !
2. If the card is not working right, and you have a recent kernel, try
removing `modprobe -f -r saa7134 saa7134-alsa` and then loading again 
with this: `modprobe saa7134 i2c_scan=1` to check 
3. If both fail, you may need to configure the card manually by passing
values for the card and/or tuner. See next section for it!

#### Selecting the tuner

You must check the Documentation/video4linux/CARDLIST.saa7134. As example, 
some versions of FlyVideo 3000 are shipped with different tunners to find yours, 
check then at the Documentation/video4linux/CARDLIST.tuner list!

For the NTSC tuner on the FlyVideo 3000 (without the radio), It found 17
was the right one. The FlyVideo3000FM has a different tuner, and it found
43 works, but only gives you channels below 60. Tuner 39 gives you all
channels. In kernel 2.6.12 I had to reboot to reset the tuner \--
modprobe didn\'t do it \-- but in kernel 2.6.18 the tuner reset without
a reboot. With 2.6.19, I switched to tuner 95.

### Inserting the module

So then for those cards, if autodetection fails, you can insert the module with
the following options:

`modprobe saa7134 card=2 tuner=39`

The command of example was for Flyvideo 3000 that doe snot have radio ( 3000FM does)

When you\'ve found a method that works \-- autodetection or manual
configuration \-- include the working options in your modules
configuration setup.

### Getting PCI audio

You can get the audio from the card through the PCI bus instead of
through the audio out jack and a patch cable to your sound card.

For kernels 2.6.15 and later, see
[Saa7134-alsa](Saa7134-alsa "wikilink") for details on both ALSA and OSS
sound.

For earlier kernels, adding \"oss=1\" as an insmod parameter will create
/dev/dsp1 and /dev/mixer1. For detailed instructions, see [Gentoo\'s
saa7134 wiki](http://gentoo-wiki.com/HARDWARE_saa7134).

In earlier kernels, you use

`  aumix -d /dev/mixer1 -I`

to set the recording channel to 1.

### Recording with mencoder

Use something like this:

`  mencoder `[`tv://`](tv://)` -tv driver=v4l2:device=/dev/video1:fps=30000/1001:chanlist=us-cable:audiorate=32000:`\
`  adevice=/dev/dsp1:input=0:amode=1:normid=4 -ffourcc DX50 -ovc lavc -lavcopts vcodec=mpeg4:mbd=2 `\
`  -oac mp3lame -lameopts cbr:br=128 -endpos 60 -o output.avi`

This gives me stereo audio with medium-quality video. I found that if I
included \"-of mpeg\" to create a true mpeg stream, the audio got
delayed \-- there are no sync problems with the avi file.

### Recording with transcode {#recording_with_transcode}

You can also use transcode to record:

`  transcode -x v4l2,v4l2 -M 2 -i /dev/video$DEV -p /dev/dsp1 -e 32000,16,2 -y ffmpeg -F mpeg4 `\
`  -c 00:30 -g 640x480 -f 29.970,4 -I 1 -u 1024 -Q 3 -E 32000,16,2 --lame_preset medium -o output.avi`

Note the \"-e 32000,16,2\", letting transcode know about the parameters
of the audio stream from the saa7134 card.

The files produced in this manner are almost twice as large as the ones
produced by the mencoder command above, but show significantly less
pixillation. CPU utilization is also about double. For the saa7134,
48000Hz is only valid for external audio input to the card; for internal
audio input (directly via the PCI bus), you have to use 32000Hz.

### Loading multiple cards {#loading_multiple_cards}

If you have more than one card in the same machine, and want to control
which devices they create, you can use this sort of thing with kernels
before 2.6.15:

`saa7134 video_nr=1,2,3 vbi_nr=1,2,3 mixer_nr=1,2,3 radio_nr=1,2,3 card=2,2,2 tuner=39,39,17 `

With 2.6.15 and later kernels, use this instead (drop the mixer and add
a line for the audio driver):

`saa7134 card=2,2,2 tuner=39,39,17 video_nr=1,2,3 vbi_nr=1,2,3 radio_nr=1,2,3`\
`saa7134-alsa index=1,2,3`

In /etc/modprobe.d/saa7134, I use this for five different cards
(including two [LifeView FlyTV
Platinum](LifeView_FlyTV_Platinum "wikilink") cards):

`     options saa7134 card=2,54,2,54,2 tuner=39,54,39,54,17 video_nr=1,2,3,4,5 \`\
`     vbi_nr=1,2,3,4,5 radio_nr=1,2,3,4,5 disable_ir=1,1,1,1,1`\
`     install saa7134 /sbin/modprobe --ignore-install saa7134; /sbin/modprobe saa7134-alsa`\
`     options saa7134-alsa index=1,2,3,4,5`

This creates video1, vbi1, radio1, and the alsa device hw:1 for the
first card, and so on up.

### Remote control {#remote_control}

With the FlyVideo 3000(FM), you should have the small grey credit-card
sized remote. And yes, this is
[supported](http://www.mail-archive.com/video4linux-list@redhat.com/msg04784.html)
and should be working (I haven\'t tested it).

For instructions, see [Saa713x devices: Generic SAA7134 Card
Installation#Remote_setup](Saa713x_devices:_Generic_SAA7134_Card_Installation#Remote_setup "wikilink").

### Closed captioning {#closed_captioning}

Closed captioning on saa713x is now working; Michael Schimek has added
support to libzvbi (mid-May 2005); see [Text
capture](Text_capture "wikilink").

## External Links {#external_links}

-   [LifeView product
    page](http://www.lifeview.com.tw/html/products/internal_tv/flytv_prime33.htm)

[Category:Analog PCI Cards](Category:Analog_PCI_Cards "wikilink")