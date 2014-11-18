# SRS librtmp

librtmp is a client side library, seems from rtmpdump.

## Use Scenarios

The use scenarios of librtmp:
* Play or suck RTMP stream: For example rtmpdump, dvr RTMP stream to flv file.
* Publish RTMP stream: Publish RTMP stream to server.
* Use sync block socket: It's ok for client.
* ARM: Can used for linux arm, for some embed device, to publish stream to server.
* Publish h.264 raw stream: SRS2.0 supports this feature, read [publish-h264-raw-data](https://github.com/winlinvip/simple-rtmp-server/wiki/v2_EN_SrsLibrtmp#publish-h264-raw-data)

Note: About the openssl, complex and simple handshake, read [RTMP protocol](https://github.com/winlinvip/simple-rtmp-server/wiki/v1_EN_RTMPHandshake)

Note: To cross build srs-librtmp for ARM cpu, read [srs-arm](https://github.com/winlinvip/simple-rtmp-server/wiki/v1_EN_SrsLinuxArm)

## librtmp For Server

librtmp or srs-librtmp only for client side application, impossible for server side.

## Why SRS provides srs-librtmp

SRS provides different librtmp:
* The code of librtmp is hard to maintain.
* The interface of librtmp is hard to use.
* No example, while srs-librtmp provides lots of examples at trunk/research/librtmp.
* Min depends, SRS extract core/kernel/rtmp modules for srs-librtmp.
* Min library requires, srs-librtmp only depends on stdc++.
* NO ST, srs-librtmp does not depends on st.
* Provides bandwidth api, to get the bandwidth data to server, read [Bandwidth Test](https://github.com/winlinvip/simple-rtmp-server/wiki/v1_EN_BandwidthTestTool)
* Provides tracable log, to get the information on server of client, read [Tracable log](https://github.com/winlinvip/simple-rtmp-server/wiki/v1_EN_SrsLog)
* Supports directly publish h.264 raw stream, read [publish-h264-raw-data](https://github.com/winlinvip/simple-rtmp-server/wiki/v2_EN_SrsLibrtmp#publish-h264-raw-data)
* Exports SRS to srs-librtmp as single project which can be make to .h and .a, or exports SRS to a single .h and .cpp file, read [export srs librtmp](https://github.com/winlinvip/simple-rtmp-server/wiki/v2_EN_SrsLibrtmp#export-srs-librtmp)

In a word, SRS provides more efficient and simple client library srs-librtmp.

## Export Srs Librtmp

SRS2.0 provides options for configure to export srs-librtmp to single project to make .h and .a, or to single .h and .cpp file.

Usage for export project:

```
dir=/home/winlin/srs-librtmp &&
rm -rf $dir &&
./configure --export-librtmp-project=$dir &&
cd $dir && make &&
./objs/research/librtmp/srs_play rtmp://ossrs.net/live/livestream
```

SRS export srs-librtmp project can be make to .h and .a, and compile all release/librtmp examples.

SRS can also export srs-librtmp to a single .h and .cpp file, and generate a simple example:

```
dir=/home/winlin/srs-librtmp &&
rm -rf $dir &&
./configure --export-librtmp-single=$dir &&
cd $dir && gcc example.c srs_librtmp.cpp -g -O0 -lstdc++ -o example && 
strip example && ./example
```

Note: The export librtmp support both relative and absolute dir path.

## Build srs-librtmp

When make SRS, the srs-librtmp will auto generated when configure with librtmp:

```bash
./configure --with-librtmp --without-ssl && make
```

All examples are built, read [Examples](https://github.com/winlinvip/simple-rtmp-server/wiki/v2_EN_SrsLibrtmp#srs-librtmp-examples).

<strong>Note: Recomment to disable ssl, for librtmp does not depends on ssl.</strong>

<strong>Note: srs-librtmp provides only simple handshake, without complex handshake, eventhough configure with ssl.</strong>

When build ok, user can use .h and .a library to build client application.

## Build srs-librtmp on Windows

srs-librtmp only depends on libc++, so can be build on windows.

SRS 2.0 can export srs-librtmp to single project, or a .h and a .cpp file, read [export srs librtmp](https://github.com/winlinvip/simple-rtmp-server/wiki/v2_EN_SrsLibrtmp#export-srs-librtmp).

Need to port some linux header files.

Note: Donot need ssl and st.

## RTMP packet specification

This section descrips the RTMP packet specification, for the srs-librtmp api to read or write data.

The api about data:
* Read RTMP packet from server: int srs_read_packet(int* type, u_int32_t* timestamp, char** data, int* size)
* Write RTMP packet to server: int srs_write_packet(int type, u_int32_t timestamp, char* data, int size)
* Write h.264 raw data to server, read [publish-h264-raw-data](https://github.com/winlinvip/simple-rtmp-server/wiki/v2_EN_SrsLibrtmp#publish-h264-raw-data)

The RTMP packet(char* data) for api, is format in flv Video/Audio, read the trunk/doc [video_file_format_spec_v10_1.pdf](https://raw.github.com/winlinvip/simple-rtmp-server/master/trunk/doc/video_file_format_spec_v10_1.pdf)
* Audio data, read `E.4.2.1 AUDIODATA`，p76, for example, the aac codec audio data.
* Video data, read  `E.4.3.1 VIDEODATA`，p78, for example, the h.264 video data.
* Script data, read `E.4.4.1 SCRIPTDATA`，p80, for example, onMetadata call.

The RTMP packet type(int type) defines (in `E.4.1 FLV Tag`，page 75)：
* Audio: 8, the macro SRS_RTMP_TYPE_AUDIO
* Video: 9, the macro SRS_RTMP_TYPE_VIDEO
* Script: 18, the macro SRS_RTMP_TYPE_SCRIPT

Other parameters, for instance, the timestamp, pass by args.

About the flv specification:
* flv header format: `E.2 The FLV header`，p74。
* flv body format: `E.3 The FLV File Body`，p74。
* tag tag header: `E.4.1 FLV Tag`，p75。

Why use flv format as srs-librtmp api data format:
* Flv is simple enough.
* FFMPEG also use flv for rtmp.
* Only need to add flv tag header, then we can write to flv file.
* When publish flv file to server, only need to parse the tag header, the tag body is the data.

## Publish H.264 Raw Data

srs-librtmp provides api to publish h.264 raw stream to RTMP server.

Please read http://blog.csdn.net/win_lin/article/details/41170653

When convert h.264 raw stream to RTMP packet:

1. The h.264 raw stream does not specifies the dts and pts, which is calculated by encoder.
1. The RTMP sequence header, always sent in the first video packet.
1. The RTMP-packet = 5bytes(RTMP-header) + h.264-header + h.264-NALU-data. Refer to SrsAvcAacCodec::video_avc_demux
1. The srs-librtmp provides api to directly send h.264 raw stream, while the raw stream should starts with annexb header N[00] 00 00 01, where N>=0.

The api of srs-librtmp to send h.264 raw stream:

```
/**
* write h.264 raw frame over RTMP to rtmp server.
* @param frames the input h264 raw data, encoded h.264 I/P/B frames data.
*       frames can be one or more than one frame,
*       each frame prefixed h.264 annexb header, by N[00] 00 00 01, where N>=0, 
*       for instance, frame = header(00 00 00 01) + payload(67 42 80 29 95 A0 14 01 6E 40)
*       about annexb, @see H.264-AVC-ISO_IEC_14496-10.pdf, page 211.
* @paam frames_size the size of h264 raw data. 
*       assert frames_size > 0, at least has 1 bytes header.
* @param dts the dts of h.264 raw data.
* @param pts the pts of h.264 raw data.
* 
* @remark, user should free the frames.
* @remark, the tbn of dts/pts is 1/1000 for RTMP, that is, in ms.
* @remark, cts = pts - dts
* 
* @return 0, success; otherswise, failed.
*/
extern int srs_h264_write_raw_frames(srs_rtmp_t rtmp, 
    char* frames, int frames_size, u_int32_t dts, u_int32_t pts
);
```

For the sample h.264 file: http://winlinvip.github.io/srs.release/3rdparty/720p.h264.raw

The data is:

```
// SPS
000000016742802995A014016E40
// PPS
0000000168CE3880
// IFrame
0000000165B8041014C038008B0D0D3A071.....
// PFrame
0000000141E02041F8CDDC562BBDEFAD2F.....
```

The sps and pps can be sent togother, or not:

```
// SPS+PPS
srs_h264_write_raw_frame('000000016742802995A014016E400000000168CE3880', size, dts, pts)
// IFrame
srs_h264_write_raw_frame('0000000165B8041014C038008B0D0D3A071......', size, dts, pts)
// PFrame
srs_h264_write_raw_frame('0000000141E02041F8CDDC562BBDEFAD2F......', size, dts, pts)
```

The NALU can be sent together, or not:

```
// SPS
srs_h264_write_raw_frame('000000016742802995A014016E4', size, dts, pts)
// PPS
srs_h264_write_raw_frame('00000000168CE3880', size, dts, pts)
// IFrame
srs_h264_write_raw_frame('0000000165B8041014C038008B0D0D3A071......', size, dts, pts)
// PFrame
srs_h264_write_raw_frame('0000000141E02041F8CDDC562BBDEFAD2F......', size, dts, pts) 
```

About the api, read https://github.com/winlinvip/simple-rtmp-server/issues/66#issuecomment-62240521

About to use the api, read https://github.com/winlinvip/simple-rtmp-server/issues/66#issuecomment-62245512

## srs-librtmp Examples

The examples for srs-librtmp, automatically build when build SRS:
* research/librtmp/srs_play.c: Use srs-librtmp to play RTMP stream.
* research/librtmp/srs_publish.c: Use srs-librtmp to publish RTMP stream.
* research/librtmp/srs_ingest_flv.c: Use srs-librtmp to read local flv to publish as RTMP stream.
* research/librtmp/srs_ingest_rtmp.c: Use srs-librtmp to read RTMP then publish as RTMP stream.
* research/librtmp/srs_bandwidth_check.c: Use srs-librtmp to check bandwidth to server.
* research/librtmp/srs_flv_injecter.c: Use srs-librtmp to inject flv keyframes offset for flv vod stream.
* research/librtmp/srs_flv_parser.c: Use srs-librtmp to show the flv file.
* research/librtmp/srs_detect_rtmp.c: Use srs-librtmp to detect the RTMP stream status.
* research/librtmp/srs_h264_raw_publish.c: Use srs-librtmp to publish h.264 raw stream to RTMP server.

## Run Examples

Start SRS:

```bash
make && ./objs/srs -c srs.conf 
```

The publish example:

```bash
make && ./objs/research/librtmp/objs/srs_publish rtmp://127.0.0.1:1935/live/livestream
```

Note: the publish stream send random data, cannot play by player.

The play example:

```bash
make && ./objs/research/librtmp/objs/srs_play rtmp://ossrs.net/live/livestreamsuck rtmp stream like rtmpdump
```

Winlin 2014.11