# Build SRS

You can directly use the release binaries, or build SRS step by step. See: [Github: release](http://winlinvip.github.io/srs.release/releases/) or [Mirror of China: release](http://www.ossrs.net/srs.release/releases/)

## OS

* <strong>Centos6.x/Ubuntu12</strong> is proved for Usage of README.
* Recomment to use <strong>Centos6.x/Ubuntu12</strong> for demo of SRS, because it's complex to compile FFMPEG.
* Turn some features off when you need to compile SRS on other OS.

## Iptables and Selinux

Sometimes the stream play failed, but without any error message, or server cann't connect to. Please check the iptables and selinux.

Turn off iptables:

```bash
# disable the firewall
sudo /etc/init.d/iptables stop
sudo /sbin/chkconfig iptables off
```

Disable the selinux, to run `getenforce` to ensure the result is `Disabled`:

1. Edit the config of selinux: `sudo vi /etc/sysconfig/selinux`
1. Change the SELINUX to disabled: `SELINUX=disabled`
1. Rebot: `sudo init 6`

## Build and Run SRS

It's very easy to build SRS:

```
./configure && make
```

Also easy to start SRS:

```bash
./objs/srs -c conf/srs.conf
```

Publish RTMP, please see: [Usage: RTMP](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SampleRTMP)

More usages, please see: [Usage](https://github.com/winlinvip/simple-rtmp-server/tree/1.0release#usage)

## Build Options and Presets

Each big feature is controlled by a build option, while a preset provides a set of options.

SRS will apply preset first, then apply user specified options, for example, `./configure --rtmp-hls --with-http-api` will:
* apply preset: --rtmp-hls, enable rtmp ssl and hls, disable others.
* apply user specified options: --with-http-api, enable http api.

So, the built SRS will support RTMP+HLS+HttpApi.

All preset and options supported is specified by command `./configure -h`.

## jobs: Speedup Build

It will take long time to compile SRS when ffmpeg/nginx enabled. You can use multiple cpu to speedup, similar to the `make --jobs=N`.
* configure --jobs=N: to speedup when compile ffmpeg/nginx.
* make --jobs=N: to speedup when compile SRS.

The following components can be speedup, SRS will auto apply the --jobs automatically:
* SRS: support.
* st-1.9: not support, for it's small.
* http-parser: not support, for it's small.
* openssl: not support.
* nginx: support.
* ffmpeg: support.
* lame: support.
* libaacplus: not support.
* x264: support.

The usage to speedup configure:

```bash
./configure --jobs=16
```

Note: configure donot support `-jN`, only support `--jobs=N`.

The usage to speedup make SRS:

```bash
// or make --jobs=16
make -j16
```

## Package

SRS provides package script, to package the install zip on release website.

The package script will build SRS, then zip the files. See help of package script:

```bash
[winlin@dev6 srs]$ ./scripts/package.sh --help

  --help                   print this message

  --x86-x64                configure with x86-x64 and make srs. 
  --arm                    configure with arm and make srs.
```

## SRS依赖关系

SRS依赖于g++/gcc/make，st-1.9，http-parser2.1，ffmpeg，cherrypy，nginx，openssl-devel，python2。

某些依赖可以通过configure配置脚本关闭，详见下表：

<table>
<tr>
<td><strong>功能</strong></td>
<td><strong>选项</strong></td>
<td><strong>编译</strong></td>
<td><strong>依赖库</strong></td>
<td><strong>说明</strong></td>
</tr>
<tr>
<td>编译器</td>
<td>必选</td>
<td>无</td>
<td>linux,g++,gcc,make</td>
<td>基础编译环境</td>
</tr>
<tr>
<td>RTMP(Basic)</td>
<td>必选</td>
<td>无</td>
<td>st-1.9</td>
<td>RTMP服务器，st为处理并发的基础库<br/>forward,vhost,refer,reload为基础功能。<br/><br/>st-1.9没有再依赖其他库，在各种linux下都可以编译，<br/>测试过的有CentOS4/5/6，Ubuntu12，Debian-Armhf，<br/>其他问题也不大<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_DeliveryRTMP">DeliveryRTMP</td>
</tr>
<tr>
<td>RTMP<br/>(H.264/AAC)</td>
<td>可选</td>
<td>--with-ssl</td>
<td>ssl</td>
<td>RTMP分发H.264/AAC，需要支持<a href="http://blog.csdn.net/win_lin/article/details/13006803">复杂握手</a><br/><br/>简单握手的内容为1537字节随机数，<br/>而复杂握手为按一定规则加密的数据<br/><br/>srs使用自己编译的ssl库<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_RTMPHandshake">RTMPHandshake</td>
</tr>
<tr>
<td>HLS</td>
<td>可选</td>
<td>--with-hls \<br/>
--with-nginx</td>
<td>nginx</td>
<td>--with-hls<br/>将RTMP流切片成ts，并生成m3u8，<br/>即AppleHLS流分发。参考：<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_DeliveryHLS">HLS</a><br/><br/>
--with-nginx<br/>打开此功能后会编译<a href="http://nginx.org/">nginx</a>，<br/>通过nginx分发m3u8和ts静态文件<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_DeliveryHLS">DeliveryHLS</a>
</td>
</tr>
<tr>
<td>FFMPEG</td>
<td>可选</td>
<td>--with-ffmpeg</td>
<td>ffmpeg<br/>(libaacplus,<br/>lame,yasm,<br/>x264,ffmpeg)</td>
<td>转码，转封装，采集工具，<br/>FFMPEG依赖的项目实在太多，<br/>而且在老版本的linux上这些库很难编译成功，<br/><br/>因此若不需要转码功能，建议关闭此功能，<br/>若需要转码，推荐使用CentOS6.*/Ubuntu12系统<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_FFMPEG">FFMPEG</a></td>
</tr>
<tr>
<td>Transcode</td>
<td>可选</td>
<td>--with-transcode</td>
<td>转码工具<br/>譬如FFMPEG</td>
<td>将RTMP流转码后输出RTMP流，<br/>一般转码需要FFMPEG工具，<br/>或者禁用FFMPEG后指定自己的工具<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_FFMPEG">FFMPEG</a></td>
</tr>
<tr>
<td>Ingest</td>
<td>可选</td>
<td>--with-ingest</td>
<td>采集工具<br/>譬如FFMPEG</td>
<td>将文件/流/设备数据抓取后推送到SRS，<br/>一般采集需要FFMPEG工具，<br/>或者禁用FFMPEG后指定自己的工具<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_Ingest">Ingest</a></td>
</tr>
<tr>
<td>HttpCallback</td>
<td>可选</td>
<td>--with-http-callback</td>
<td>cherrypy<br/>http-parser2.1<br/>python2</td>
<td>当某些事件发生，SRS可以调用http地址<br/><br/>譬如客户端连接到服务器时，SRS会调用<br/>on_connect接口，SRS自带了一个<br/>research/api-server(使用Cherrypy)，<br/>提供了这些http api的默认实现。<br/><br/>另外，若开启了HttpCallback，<br/>players的演示默认会跳转到api-server<br/><br/>http-parser2.1在各种linux下编译问题也不大<br/><br/>python2.6/2.7在CentOS6/Ubuntu12下才有，<br/>所以CentOS5启动HttpCallback会报json模块找不到<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_HTTPCallback">HTTPCallback</td>
</tr>
<tr>
<td>HttpServer</td>
<td>可选</td>
<td>--with-http-server</td>
<td>http-parser2.1</td>
<td>SRS内嵌了一个web服务器，实现基本的http协议，<br/>主要用于文件分发。<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_HTTPServer">HTTPServer</a></td>
</tr>
<tr>
<td>HttpApi</td>
<td>可选</td>
<td>--with-http-api</td>
<td>http-parser2.1</td>
<td>SRS提供http-api（内嵌了web服务器），<br/>支持http方式管理服务器。<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_HTTPApi">HTTPApi</a></td>
</tr>
<tr>
<td>ARM</td>
<td>可选</td>
<td>--with-arm-ubuntu12</td>
<td>无额外依赖</td>
<td>SRS可运行于ARM，<br/>若需要支持<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_RTMPHandshake">复杂握手</a>则需要依赖ssl，<br/>目前在Ubuntu12下编译，<br/>debian-armhf(v7cpu)下测试通过<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SrsLinuxArm">SrsLinuxArm</td>
</tr>
<tr>
<td>librtmp</td>
<td>可选</td>
<td>--with-librtmp</td>
<td>无额外依赖</td>
<td>SRS提供客户端库<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SrsLibrtmp">srs-librtmp</a>，<br/>若需要支持<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_RTMPHandshake">复杂握手</a>则需要依赖ssl，<br/>支持客户端推RTMP流到SRS，或者播放RTMP流<br/><br/>srs-librtmp使用同步socket，协议栈和SRS<br/>服务端一致，和librtmp一样，只适合用作客户端，<br/>不可用作服务端。<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SrsLibrtmp">SrsLibrtmp</td>
</tr>
<tr>
<td>DEMO</td>
<td>可选</td>
<td>--with-ssl \<br/>--with-hls \<br/>--with-nginx \<br/>--with-ffmpeg \<br/>--with-transcode<br/></td>
<td>nginx/cherrypy</td>
<td>SRS的演示播放器/转码输出的流/编码器/视频会议，<br/>因为需要http服务器，所以依赖于nginx，<br/><br/>另外，视频会议因为需要知道大家发布的流名称，<br/>所以需要HttpCallback支持<br/>
参考: <a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SampleDemo">SampleDemo</td>
</tr>
<tr>
<td>GPERF</td>
<td>可选</td>
<td>--with-gperf</td>
<td>gperftools</td>
<td>使用Google的tcmalloc内存分配库，<br/>gmc/gmp/gcp依赖这个选项，参考：<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_GPERF">GPERF</a></td>
</tr>
<tr>
<td>GPERF(GMC)</td>
<td>可选</td>
<td>--with-gmc</td>
<td>gperftools</td>
<td>内存检查gperf-memory-check，<br/>gmc依赖gperf，参考：<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_GPERF">GPERF</a></td>
</tr>
<tr>
<td>GPERF(GMP)</td>
<td>可选</td>
<td>--with-gmp</td>
<td>gperftools</td>
<td>内存性能分析gperf-memory-profile，<br/>gmp依赖gperf，参考：<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_GPERF">GPERF</a></td>
</tr>
<tr>
<td>GPERF(GCP)</td>
<td>可选</td>
<td>--with-gcp</td>
<td>gperftools</td>
<td>CPU性能分析gperf-cpu-profile，<br/>gcp依赖gperf，参考：<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_GPERF">GPERF</a></td>
</tr>
<tr>
<td>GPROF</td>
<td>可选</td>
<td>--with-gprof</td>
<td>gprof</td>
<td>GNU CPU profile性能分析工具，<br/>参考：<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_GPROF">GPROF</a></td>
</tr>
</table>

## 自定义编译参数

SRS可以自定义编译器，譬如arm编译时使用arm-linux-g++而非g++。参考[ARM：手动编译](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SrsLinuxArm#%E6%89%8B%E5%8A%A8%E7%BC%96%E8%AF%91srs)

注意：SRS和ST都可以通过编译前设置变量编译，但是ssl需要手动修改Makefile。还好ssl不用每次都编译。

## 编译的生成项目

configure和make将会生成一些项目，都在objs目录。有些文件在research目录，configure会自动软链到objs目录。

HttpCallback(及其服务端api-server)的目录为research/api-server，没有做软链，可以直接启动。详细参考下面的方法。

<table>
<tr>
<td><strong>生成项目</strong></td>
<td><strong>使用方法</strong></td>
<td><strong>说明</strong></td>
</tr>
<tr>
<td>./objs/srs</td>
<td>./objs/srs -c conf/srs.conf</td>
<td>启动SRS服务器</td>
</tr>
<tr>
<td>./objs/research/<br/>librtmp/<br/>srs_bandwidth_check</td>
<td>./objs/research/<br/>librtmp/<br/>srs_bandwidth_check -h</td>
<td>linux测速工具</td>
</tr>
<tr>
<td>./objs/nginx</td>
<td>sudo ./objs/nginx/sbin/nginx</td>
<td>HLS/DEMO用到的nginx服务器</td>
</tr>
<tr>
<td>api-server</td>
<td>python research/api-server/<br/>server.py 8085</td>
<td>启动HTTP hooks和DEMO视频会议用到的api-server</td>
</tr>
<tr>
<td>FFMPEG</td>
<td>./objs/ffmpeg/bin/ffmpeg</td>
<td>SRS转码用的FFMPEG，DEMO推流也是用它</td>
</tr>
<tr>
<td>librtmp</td>
<td>./objs/include/srs_librtmp.h<br/>
./objs/lib/srs_librtmp.a</td>
<td>SRS提供的客户端库，参考<a href="https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SrsLibrtmp">srs-librtmp</a></td>
</tr>
<tr>
<td>DEMO<br/>(关闭HttpCallback)</td>
<td>./objs/nginx/<br/>html/players</td>
<td>SRS的DEMO的静态页面，当没有开启HttpCallback时</td>
</tr>
<tr>
<td>DEMO<br/>(开启HttpCallback)</td>
<td>research/api-server/static-dir/players</td>
<td>SRS的DEMO的静态页面，<br/>和nginx里面的静态目录是一个目录，软链到research/players，<br/>1.当HttpCallback开启（--with-http-callback)，<br/>nginx的index.html会默认跳转到HttpCallback的首页，<br/>原因是视频会议的DEMO需要HttpCallback，<br/>2.若HttpCallback没有开启，<br/>则默认浏览的是Nginx里面的DEMO，<br/>当然视频会议会无法演示</td>
</tr>
</table>

## 配置参数说明

SRS的配置(configure)参数说明如下：
* --help 配置的帮助信息
* --with-ssl 添加ssl支持，ssl用来支持复杂握手。参考：[RTMP Handshake](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_RTMPHandshake)。
* --with-hls 支持HLS输出，将RTMP流切片成ts，可用于支持移动端HLS（IOS/Android），不过PC端jwplayer也支持HLS。参考：[HLS](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_DeliveryHLS)
* --with-dvr 支持将RTMP流录制成FLV。参考：[DVR](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_DVR)
* --with-nginx 编译nginx，使用nginx作为web服务器分发HLS文件，以及demo的静态页等。
* --with-http-callback 支持http回调接口，用于认证，统计，事件处理等。参考：[HTTP callback](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_HTTPCallback)
* --with-http-api 打开HTTP管理接口。参考：[HTTP API](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_HTTPApi)
* --with-http-server 打开内置HTTP服务器，支持分发HTTP流。参考：[HTTP Server](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_HTTPServer)
* --with-ffmpeg 编译转码/转封装/采集用的工具FFMPEG。参考：[FFMPEG](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_FFMPEG)
* --with-transcode 直播流转码功能。需要在配置中指定转码工具。参考：[FFMPEG](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_FFMPEG)
* --with-ingest 采集文件/流/设备数据，封装为RTMP流后，推送到SRS。参考：[Ingest](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_Ingest)
* --with-stat 是否开启数据统计功能，SRS可以采集cpu/内存/网络/磁盘IO等数据，共监控系统通过http-api获取。（目前osx不支持）。
* --with-research 是否编译research目录的文件，research目录是一些调研，譬如ts info是做HLS时调研的ts标准。和SRS的功能没有关系，仅供参考。
* --with-utest 是否编译SRS的单元测试，默认开启，也可以关闭。
* --with-gperf 是否使用google的tcmalloc库，默认关闭。
* --with-gmc 是否使用gperf的内存检测，编译后启动srs会检测内存错误。这个选项会导致低性能，只应该在找内存泄漏时才开启。默认关闭。参考：[gperf](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_GPERF)
* --with-gmp 是否使用gperf的内存性能分析，编译后srs退出时会生成内存分析报告。这个选项会导致地性能，只应该在调优时开启。默认关闭。参考：[gperf](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_GPERF)
* --with-gcp 是否启用gperf的CPU性能分析，编译后srs退出时会生成CPU分析报告。这个选项会导致地性能，只应该在调优时开启。默认关闭。参考：[gperf](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_GPERF)
* --with-gprof 是否启用gprof性能分析，编译后srs会生成CPU分析报告。这个选项会导致地性能，只应该在调优时开启。默认关闭。参考：[gprof](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_GPROF)
* --with-librtmp 客户端推流/播放库，参考[srs-librtmp](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SrsLibrtmp)
* --with-arm-ubuntu12 交叉编译ARM上运行的SRS，要求系统是Ubuntu12。参考[srs-arm](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SrsLinuxArm)
* --jobs[=N] 开启的编译进程数，和make的-j（--jobs）一样，在configure时可能会编译nginx/ffmpeg等工具，可以开启多个jobs编译，可以显著加速。参考：[Build: jobs](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_Build#wiki-jobs%E5%8A%A0%E9%80%9F%E7%BC%96%E8%AF%91)
* --static 使用静态链接。指定arm编译时，会自动打开这个选项。手动编译需要用户自身打开。参考：[ARM](https://github.com/winlinvip/simple-rtmp-server/wiki/EN_SrsLinuxArm)

预设集：
* --x86-x64，默认预设集，一般的x86或x64服务器使用。release使用这个配置编译。
* --osx，苹果MAC OSX（Darwin）系统下编译，安装好xcode和brew后，可以使用这个选项。
* --pi，树莓派预设集，arm的子集。树莓派的release用这个配置编译。
* --cubie，在cubieboard下直接编译的选项，使用ubuntu差不多的配置集。
* --arm，ubuntu下交叉编译，等价于--with-arm-ubuntu12。release使用这个配置。
* --mips，ubuntu下交叉编译，为hiwifi的mips路由器编译。（目前srs在mips上有内存泄漏，2天左右会把路由器跑死）。
* --dev，开发选项，尽可能开启功能。
* --fast，关闭所有功能，只支持基本RTMP（不支持h264/aac），最快的编译速度。
* --pure-rtmp，支持RTMP（支持h264+aac），需要编译ssl。
* --rtmp-hls，支持RTMP和HLS，典型的应用方式。还可以加上内置的http服务器（--with-http-server）。
* --disable-all, 禁用所有功能，只支持rtmp（vp6）。
* --demo，SRS的演示编译选项。
* --full，开启SRS所有的功能。

专家选项：有可能编译失败，不是专家就不要用这个。
* --use-sys-ssl 使用系统的ssl，不单独编译ssl（在--with-ssl时有效）。

Winlin 2014.10