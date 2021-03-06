# Jitsi-Hammer

A traffic generator for Jitsi Videobridge.

Jitsi-Hammer is a program that connects to a Jitsi-Meet conference, 
 initiates the video conference management by Focus,
 creates fake users, and generates RTP traffic for these fake users.  

Note that the hammer requires Java 1.8 because of a dependency (see pom.xml for details).

The project includes some sample video/audio files. These Pcaps are available in the ./jitsi-hammer/resources/ folder. The files are big and in order to download them correctly, you're going to need Git LFS support. You can download it from [here](https://git-lfs.github.com/).

## How to install
1. If you want to use sample input video/audio files then install git-lfs first (git or manual download will not work).
1. Clone the project.
1. When running the jitsi-hammer.sh script for the first time you will most likely get a Maven dependency issue. To mitigate this navigate to ./jitsi-hammer folder and run **mvn compile exec:java** before starting the script (this command will fail but will fetch the needed dependencies).

## How to use
1. Setup jitsi-meet server-side components.
1. Using a web browser open the jitsi-meet app and create a “room”.
1. Connect to the created room by running hammer script (see below). An exemplary command would be: **./jitsi-hammer.sh -u https://$MY_JITSI_WEB_SERVER/http-bind/ -room $MY_ROOM_NAME -length 120 -audiortpdump ./resources/badger-audio.rtpdump -videortpdump ./resources/badger-video.rtpdump -summarystats**
1. Most likely there will be some WARNINGS (corrupted .pom files, org.jitsi.impl.neomedia.codec.FFmpeg problems etc.), however despite these warnings the hammer should work. 
1. It takes some time for the hammer to start sending video/audio. The hammer participants should show-up in a browser-created room, and the sample audio/video file should begin playing. 
1. After the specified amount of time (specified by -length option) the hammer will try to exit, however it may happen that you will need to kill the created processes forcefully.

Running the **jitsi-hammer.sh** script:

```./jitsi-hammer.sh <MANDATORY_OPTIONS> [OPTIONAL_OPTIONS]```

The only **MANDATORY_OPTION** for now is :

```
-u (alias -BOSHuri) - the full URI to use for BOSH connection. Note that -u option should be used when running the jitsi-hammer.sh!
```

The **OPTIONAL_OPTIONS** are :
```
-XMPPdomain <domain name used by the XMPP server>
-BOSHhost <hostname of the BOSH server>
-MUCdomain <domain name used for the MUC>
-help <display the usage and help of the program>
-focusJID <JID of the focus user, by default this is 'focus.' + XMPPdomain>
-room <name of the MUC room (default : TestHammer)>
-port <port number of the BOSH server (default: 80)>
-users <number of fake users to create (default: 1)>
-length <length of the run in seconds (default: 0)>
-ivf <path to an ivf file for the video streams>
-videortpdump <path to a rtpdump file containing VP8 RTP packets for the video streams>
-audiortpdump <path to a rtpdump file containing Opus RTP packets for the video streams>
-overallstats <enable the logging of the overall stats at the end of the run>
-allstats <enable the logging of all the stats collected during the run>
-summarystats <enable the logging of the summary stats(min,max,mean,standard deviation) from the stats collected during the run>
-statspolling <time (in seconds) between two polling of stats (default: 5sec)>
-credentials <filepath to a file containing users credentials>
-interval <time in milliseconds between adding of users (default: 2sec)>
-nostats <disable all stats (default: stats are enabled)>
-channelLastN <"channelLastN" video conference property for initiated video conference>
-adaptiveLastN <"adaptiveLastN" video conference property for initiated video conference>
-adaptiveSimulcast <"adaptiveSimulcast" video conference property for initiated video conference>
-openSctp <"openSctp" video conference property for initiated video conference>
-startAudioMuted <"startAudioMuted" video conference property for initiated video conference>
-startVideoMuted <"startVideoMuted" video conference property for initiated video conference>
-simulcastMode <"simulcastMode" video conference property for initiated video conference>
```

Options ```XMPPdomain``` , ```BOSHhost```, ```port``` and ```MUCDomain```, if specified, will override the ones retrieved from ```BOSHuri```.
By default, the ```BOSHuri```'s hostname is used as both ```BOSHhost``` and ```XMPPdomain```, and ```MUCdomain``` is that hostname with ```conference.``` prefix, e.g. ```conference.meet.jit.si```
will be used as ```MUCdomain``` when accessing a ```BOSHuri``` "https://meet.jit.si/http-bind/" .
The ```port``` setting defaults for 80 for non-secure ```BOSHuri```, and respectively 443 will be used for HTTPS ones.

When the option ```-credentials``` is used, instead of loging in anonymously to the XMPP server, Jitsi-Hammer will login with the credentials contained in the file.
The file must be encoded in UTF-8, and should be a list of "username:password" (the password and username are separeted by a ":") separated by newlines.

You must know that when ```-length N``` is given, if N <= 0, the run will never stop.

For the audio streams, if ```-audiortpdump file``` is not given, Jitsi-Hammer will generate silence.

For the video streams, if neither ````-ivf file``` nor ```-videortpdump file``` are given, Jitsi-Hammer will generate a fading from white to black to white...

## What is IVF, and how to create IVF files
IVF is a simple video format described [here](http://wiki.multimedia.cx/index.php?title=IVF) (not official source). An IVF file basically just contains VP8 frames, with a fixed header for each.

IVF is more or less a toy format, generally used to test and debug VP8/VP9. But its simplicity makes it great to read.

You can create IVF from a webm file by using```mkvextract``` :
```
mkvextract tracks input_file.webm ID:output_file.ivf
```
with **ID** being the id of the VP8 track (generally 0, you can get its value by using ```mkvinfo input_file.webm```.

Or you can also use ```ffmpeg``` :
```
ffmpeg -i file_input.webm -vcodec copy file_output.ivf
```

## What is rtpdump, and how to create rtpdump files
rtpdump file is a binary file format, described [here](http://wiki.wireshark.org/rtpdump) and [here](http://www.cs.columbia.edu/irt/software/rtptools/), used to record/dump and save RTP packets of a RTP stream.

Wireshark can create rtpdump files from RTP streams (if you decode them as RTP).  
To do that, you first need to record with Wireshark an RTP stream, decode it as RTP, and then do the following :  
 1. go the the **Telephony** menu
 2. go in the **RTP** submenu
 3. clic on **Show All Streams**
 4. select the stream you want to record
 5. clic on **Save As** and save the stream as a rtpdump file.

## Resources available
In the **resources** directory, you can find :
 - an ivf file **big-buck-bunny_trailer_track1_eng.ivf**, extracted from the webm with the same name, that can be used with ```-ivf```
 - the rtpdump file called **rtp_opus.rtpdump** containing Opus RTP packets (recorded from the silence generated by Jitsi-Hammer) that can be used with ```-audiortpdump```
 - **hammer-opus.rtpdump** -- Bob Marley's "Hammer", recorded in a Jitsi call, with an opus target bitrate of 32kbps.
 - the rtpdump file called **rtp_vp8** containing VP8 RTP packets (recorded from the video stream generated by Jitsi-Hammer with **big-buck-bunny_trailer_track1_eng.ivf**) that can be used with ```-videortpdump```.

You can also find **dtls10-cipher-suites-only.diff** : it's a diff file used to patch older version of libjitsi to enable dtls. You don't need it.

## Statistics files
Jitsi-Hammer can log all the streams stats that it can gets from the class MediaStreamStats. You can generate 3 types of stats :
 - All the stats : with ```-allstats``` you will log the stats of ALL the streams at each turn of loop.
 - The summary stats : with ```-summarystats``` you will log the summary stats like min/max/mean/standard deviation from all the stream stats at each turn of loop.
 - The overall stats : with ```-overallstats``` you will log the overall stats of the stream for the entire run (not just at each turn of loop).

The stats will be saved in JSON in a directory depending of your system :
 - in Linux it's in /path/to/your/home/.Jitsi-Hammer/stats/date\_of\_the\_run
 - In Win7 it's in /path/to/your/user/directory/AppData/Roaming/stats/date\_of\_the\_run

## Java log
You can adjust the logging configuration of the JVM with the file ./lib/logging.properties .  

For now it is set to only display at a WARNING level, but you can set "org.jitsi.hammer.level" to INFO if you want to print the INFO log of Jitsi-Hammer (but not libjitsi).