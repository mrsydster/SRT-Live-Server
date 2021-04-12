Introduction
============

srt-live-server(SLS) is an open source live streaming server for low latency based on Secure Reliable Tranport(SRT).
Normally, the latency of transport by SLS is less than 1 second in internet.


Requirements
============

please install the SRT first, refer to SRT(https://github.com/Haivision/srt) for system enviroment.
SLS can only run on OS based on linux, such as mac, centos or ubuntu etc.

Compile
=======

$ sudo make

bin file is generated in subdir of 'bin'.

Directivies
===========

about the config file, please see the wiki:
https://github.com/Edward-Wu/srt-live-server/wiki/Directives

Usage
=====

$ cd bin

1.help information
------------------
$ ./sls -h

2.run with default config file
------------------------------
$ ./sls -c ../sls.conf

Test
====

SLS only supports the MPEG-TS format streaming. 

1.test with ffmpeg
------------------

you can push camera live stream by FFMPEG.Please download ffmpeg sourcecode from https://github.com/FFmpeg/FFmpeg, then compile FFMPEG with --enable-libsrt. 

srt library is installed in folder /usr/local/lib64.
if "ERROR: srt >= 1.3.0 not found using pkg-config" occured when compiling FFMPEG, please check the ffbuild/config.log file and follow its instruction to resolve this issue. in most cases it can be resolved by the following command:
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:/usr/local/lib64/pkgconfig

if "error while loading shared libraries: libsrt.so.1" occured, please add srt library path to file '/etc/ld.so.conf' as the default path, then refresh by comand /sbin/ldconfig with root.


use ffmpeg to push camera stream with SRT(on my mac):

$ ./ffmpeg -f avfoundation -framerate 30 -i "0:0" -vcodec libx264  -preset ultrafast -tune zerolatency -flags2 local_header  -acodec libmp3lame -g  30 -pkt_size 1316 -flush_packets 0 -f mpegts "srt://[your.sls.ip]:8080?streamid=uplive.sls.com/live/test"


play the SRT stream with ffplay:

./ffplay -fflags nobuffer -i "srt://[your.sls.ip]:8080?streamid=live.sls.com/live/test"


2.test with OBS
---------------

the OBS supports srt protocol to publish stream when version is later than v25.0. you can use the following url:
srt://[your.sls.ip]:8080?streamid=uplive.sls.com/live/test
whith custom service.

3.test with srt-live-client
---------------------------

there is a test tool in sls, which can be used performance test because of no codec overhead but main network overhead. the slc can play a srt stream to a ts file, or push a ts file to a srt stream.

push ts file as srt url:

cd bin

./slc -r srt://[your.sls.ip]:8080?streamid=uplive.sls.com/live/test -i [the full file name of exist ts file]

play srt url

./slc -r srt://[your.sls.ip]:8080?streamid=live.sls.com/live/test -o [the full file name of ts file to save]


Note:
=====

1.SLS refer to the RTMP url format(domain/app/stream_name), example: www.sls.com/live/test. The url of SLS must be set in streamid parameter of SRT, which will be the unique identification a stream.

2.How to distinguish the publisher and player of the same stream? In conf file, you can set parameters of domain_player/domain_publisher and app_player/app_publisher to resolve it. Importantly, the two combination strings of domain_publisher/app_publisher and domain_player/app_player must not be equal in the same server block.

3.I supply a simple android app for test sls, your can download from https://github.com/Edward-Wu/liteplayer-srt

ReleaseNote
============

v1.2
----
1. update the memory mode, in v1.1 which is publisher copy data to eacc player, in v1.2 each publisher put data to a array and all players read data from this array.
2. update the relation of the publisher and player, the player is not a member of publisher. the only relation of them is array data.
3. add push and pull features, support all and hash mode for push, support loop and hash for pull. in cluster mode, you can push a stream to a hash node, and pull this stream from the same hash node.

v1.2.1
------
1. support hostname:port/app in upstreams of pull and push.

v1.3
----
1. support reload.
2. add idle_streams_timeout feature for relay.
3. change license type from gpl to mit.

v1.4
----
1. add http statistic info.
2. add http event notification, on_connect, on_close.
3. add player feature to slc(srt-live-client) tool for pressure test.

v1.4.1
------
1. add publisher feather to slc(srt-live-client) tool, which can push ts file with srt according dts.
2. modify the http bug when host is not available.

v1.4.2
------
1. add remote_ip and remote_port to on_event_url which can be as the unique identification for player or publisher.

v1.4.3
------
1. change the tcp'epoll mode to select mode for compatible MAC os.
2. modify the http check repeat bug for reopen.  

v1.4.4
------
1. OBS streaming compatible, OBS support the srt protocol which is later than v25.0.
(https://obsproject.com/forum/threads/obs-studio-25-0-release-candidate.116067/)

v1.4.5
------
1. add hls record feature.

v1.4.6
------
1. update the pid file path from "~/" to "/opt/soft/sls/"

v1.4.7
------
1. update the pid file path from to "/opt/soft/sls/" "/tmp/sls" to avoid the root authority in some case.


v1.4.8
------
1. for compatible srt v1.4.1, add the set latency method before setup method 
