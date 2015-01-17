# HTTPCallback

SRS does not support server-side script, but support
http-callback, read [ServerSide script](https://github.com/winlinvip/simple-rtmp-server/wiki/v1_EN_ServerSideScript).

## Compile

Use `--with-http-callback` to enable HttpCallback, while `--without-http-callback` to disable it.

For more information, read [Build](https://github.com/winlinvip/simple-rtmp-server/wiki/v2_EN_Build)

## Config SRS

The config of http hooks is:

```
    http_hooks {
        # whether the http hooks enalbe.
        # default off.
        enabled         on;
        # when client connect to vhost/app, call the hook,
        # the request in the POST data string is a object encode by json:
        #       {
        #           "action": "on_connect",
        #           "client_id": 1985,
        #           "ip": "192.168.1.10", "vhost": "video.test.com", "app": "live",
        #           "tcUrl": "rtmp://video.test.com/live?key=d2fa801d08e3f90ed1e1670e6e52651a",
        #           "pageUrl": "http://www.test.com/live.html"
        #       }
        # if valid, the hook must return HTTP code 200(Stauts OK) and response
        # an int value specifies the error code(0 corresponding to success):
        #       0
        # support multiple api hooks, format:
        #       on_connect http://xxx/api0 http://xxx/api1 http://xxx/apiN
        on_connect      http://127.0.0.1:8085/api/v1/clients http://localhost:8085/api/v1/clients;
        # when client close/disconnect to vhost/app/stream, call the hook,
        # the request in the POST data string is a object encode by json:
        #       {
        #           "action": "on_close",
        #           "client_id": 1985,
        #           "ip": "192.168.1.10", "vhost": "video.test.com", "app": "live"
        #       }
        # if valid, the hook must return HTTP code 200(Stauts OK) and response
        # an int value specifies the error code(0 corresponding to success):
        #       0
        # support multiple api hooks, format:
        #       on_close http://xxx/api0 http://xxx/api1 http://xxx/apiN
        on_close        http://127.0.0.1:8085/api/v1/clients http://localhost:8085/api/v1/clients;
        # when client(encoder) publish to vhost/app/stream, call the hook,
        # the request in the POST data string is a object encode by json:
        #       {
        #           "action": "on_publish",
        #           "client_id": 1985,
        #           "ip": "192.168.1.10", "vhost": "video.test.com", "app": "live",
        #           "stream": "livestream"
        #       }
        # if valid, the hook must return HTTP code 200(Stauts OK) and response
        # an int value specifies the error code(0 corresponding to success):
        #       0
        # support multiple api hooks, format:
        #       on_publish http://xxx/api0 http://xxx/api1 http://xxx/apiN
        on_publish      http://127.0.0.1:8085/api/v1/streams http://localhost:8085/api/v1/streams;
        # when client(encoder) stop publish to vhost/app/stream, call the hook,
        # the request in the POST data string is a object encode by json:
        #       {
        #           "action": "on_unpublish",
        #           "client_id": 1985,
        #           "ip": "192.168.1.10", "vhost": "video.test.com", "app": "live",
        #           "stream": "livestream"
        #       }
        # if valid, the hook must return HTTP code 200(Stauts OK) and response
        # an int value specifies the error code(0 corresponding to success):
        #       0
        # support multiple api hooks, format:
        #       on_unpublish http://xxx/api0 http://xxx/api1 http://xxx/apiN
        on_unpublish    http://127.0.0.1:8085/api/v1/streams http://localhost:8085/api/v1/streams;
        # when client start to play vhost/app/stream, call the hook,
        # the request in the POST data string is a object encode by json:
        #       {
        #           "action": "on_play",
        #           "client_id": 1985,
        #           "ip": "192.168.1.10", "vhost": "video.test.com", "app": "live",
        #           "stream": "livestream"
        #       }
        # if valid, the hook must return HTTP code 200(Stauts OK) and response
        # an int value specifies the error code(0 corresponding to success):
        #       0
        # support multiple api hooks, format:
        #       on_play http://xxx/api0 http://xxx/api1 http://xxx/apiN
        on_play         http://127.0.0.1:8085/api/v1/sessions http://localhost:8085/api/v1/sessions;
        # when client stop to play vhost/app/stream, call the hook,
        # the request in the POST data string is a object encode by json:
        #       {
        #           "action": "on_stop",
        #           "client_id": 1985,
        #           "ip": "192.168.1.10", "vhost": "video.test.com", "app": "live",
        #           "stream": "livestream"
        #       }
        # if valid, the hook must return HTTP code 200(Stauts OK) and response
        # an int value specifies the error code(0 corresponding to success):
        #       0
        # support multiple api hooks, format:
        #       on_stop http://xxx/api0 http://xxx/api1 http://xxx/apiN
        on_stop         http://127.0.0.1:8085/api/v1/sessions http://localhost:8085/api/v1/sessions;
        # when srs reap a dvr file, call the hook,
        # the request in the POST data string is a object encode by json:
        #       {
        #           "action": "on_dvr",
        #           "client_id": 1985,
        #           "ip": "192.168.1.10", "vhost": "video.test.com", "app": "live",
        #           "stream": "livestream",
        #           "cwd": "/usr/local/srs",
        #           "file": "./objs/nginx/html/live/livestream.1420254068776.flv"
        #       }
        # if valid, the hook must return HTTP code 200(Stauts OK) and response
        # an int value specifies the error code(0 corresponding to success):
        #       0
        on_dvr          http://127.0.0.1:8085/api/v1/dvrs http://localhost:8085/api/v1/dvrs;
    }
```

Note: For more information, read conf/full.conf the section hooks.callback.vhost.com

## HTTP callback events

SRS can call the http callback, for events:

<table>
<tr>
<th>Event</th><th>Data</th><th>Description</th>
</tr>
<tr>
<td>on_connect</td>
<td>
<pre>
{
    "action": "on_connect",
    "client_id": 1985,
    "ip": "192.168.1.10", 
    "vhost": "video.test.com", 
    "app": "live",
    "tcUrl": "rtmp://x/x?key=xxx",
    "pageUrl": "http://x/x.html"
}
</pre>
</td>
<td>When client connected at the specified vhost and app.</td>
</tr>
<tr>
<td>on_close</td>
<td>
<pre>
{
    "action": "on_close",
    "client_id": 1985,
    "ip": "192.168.1.10", 
    "vhost": "video.test.com", 
    "app": "live"
}
</pre>
</td>
<td>When client close connection, or server disconnect the connection.</td>
</tr>
<tr>
<td>on_publish</td>
<td>
<pre>
{
    "action": "on_publish",
    "client_id": 1985,
    "ip": "192.168.1.10", 
    "vhost": "video.test.com", 
    "app": "live",
    "stream": "livestream"
}
</pre>
</td>
<td>When client publish stream, for example, use flash or FMLE publish stream to server.</td>
</tr>
<tr>
<td>on_unpublish</td>
<td>
<pre>
{
    "action": "on_unpublish",
    "client_id": 1985,
    "ip": "192.168.1.10", 
    "vhost": "video.test.com", 
    "app": "live",
    "stream": "livestream"
}
</pre>
</td>
<td>When client stop publish stream.</td>
</tr>
<tr>
<td>on_play</td>
<td>
<pre>
{
    "action": "on_play",
    "client_id": 1985,
    "ip": "192.168.1.10", 
    "vhost": "video.test.com", 
    "app": "live",
    "stream": "livestream"
}
</pre>
</td>
<td>When client start play stream.</td>
</tr>
<tr>
<td>on_stop</td>
<td>
<pre>
{
    "action": "on_stop",
    "client_id": 1985,
    "ip": "192.168.1.10", 
    "vhost": "video.test.com", 
    "app": "live",
    "stream": "livestream"
}
</pre>
</td>
<td>When client stop play.</td>
</tr>
<tr>
<td>on_dvr</td>
<td>
<pre>
{
    "action": "on_dvr",
    "client_id": 1985,
    "ip": "192.168.1.10", 
    "vhost": "video.test.com", 
    "app": "live",
    "stream": "livestream",
    "cwd": "/opt",
    "file": "./l.xxx.flv"
}
</pre>
</td>
<td>When reap a dvr file.</td>
</tr>
</table>

Note:
* Event: When this event occur, callback the specified HTTP url.
* HTTP url: Can be multiple urls, splits by space, SRS will notice all one by one.
* Data: SRS will POST the data to specified HTTP api.
* Return Code: SRS requires the response is an int, indicates the error, 0 is success.
SRS will disconnect the connection when response is not 0, or http status is not 200.

## SRS HTTP callback Server

SRS provides a default HTTP callback server, use cherrypy.

To start it: `python research/api-server/server.py 8085`

```bash
[winlin@dev6 srs]$ python research/api-server/server.py 8085
[2014-02-27 09:42:25][trace] api server listen at port: 8085, static_dir: /home/winlin/git/simple-rtmp-server/trunk/research/api-server/static-dir
[2014-02-27 09:42:25][trace] start cherrypy server
[27/Feb/2014:09:42:25] ENGINE Listening for SIGHUP.
[27/Feb/2014:09:42:25] ENGINE Listening for SIGTERM.
[27/Feb/2014:09:42:25] ENGINE Listening for SIGUSR1.
[27/Feb/2014:09:42:25] ENGINE Bus STARTING
[27/Feb/2014:09:42:25] ENGINE Started monitor thread '_TimeoutMonitor'.
[27/Feb/2014:09:42:25] ENGINE Started monitor thread 'Autoreloader'.
[27/Feb/2014:09:42:25] ENGINE Serving on 0.0.0.0:8085
[27/Feb/2014:09:42:25] ENGINE Bus STARTED
```

## Publish and Play

Publish stream to SRS, SRS will call the http callback:

```bash
[2014-02-27 09:41:33][trace] post to clients, req={"action":"on_connect","client_id":4,"ip":"192.168.1.179","vhost":"__defaultVhost__","app":"live","pageUrl":""}
[2014-02-27 09:41:33][trace] srs on_connect: client id=4, ip=192.168.1.179, vhost=__defaultVhost__, app=live, pageUrl=
127.0.0.1 - - [27/Feb/2014:09:41:33] "POST /api/v1/clients HTTP/1.1" 200 1 "" "srs(simple rtmp server)0.9.2"
```

Play stream on SRS, SRS will call the http callback:

```bash
[2014-02-27 09:41:50][trace] post to clients, req={"action":"on_connect","client_id":5,"ip":"192.168.1.179","vhost":"__defaultVhost__","app":"live","pageUrl":"http://dev.chnvideo.com:3080/players/rtmp/"}
[2014-02-27 09:41:50][trace] srs on_connect: client id=5, ip=192.168.1.179, vhost=__defaultVhost__, app=live, pageUrl=http://dev.chnvideo.com:3080/players/rtmp/
127.0.0.1 - - [27/Feb/2014:09:41:50] "POST /api/v1/clients HTTP/1.1" 200 1 "" "srs(simple rtmp server)0.9.2"
```

Winlin 2015.1