NGINX TS Dockerfile
=====================

Based on this repo [https://github.com/brocaar/nginx-rtmp-dockerfile](https://github.com/brocaar/nginx-rtmp-dockerfile).

This Dockerfile installs NGINX configured with `nginx-ts-module`, ffmpeg
and some default settings for HLS live streaming.

How to use
----------

1. Build and run the container (`docker build -t nginx_ts .` &
   `docker run -d -p 8080:8080 --rm nginx_ts`).

2. Stream your live content to `http://127.0.0.1:8080/publish/stream_name` where
   `stream_name` is the name of your stream. E.g. broadcasting a single-bitrate mp4 file:
```
ffmpeg -re -i ~/Movies/sintel.mp4 -bsf:v h264_mp4toannexb
         -c copy -f mpegts http://127.0.0.1:8080/publish/sintel
```
Broadcasting an mp4 file in multiple bitrates. For proper HLS generation streams should be grouped into MPEG-TS programs with the -program option of ffmpeg:
```
ffmpeg -re -i ~/Movies/sintel.mp4 -bsf:v h264_mp4toannexb
         -map 0:0 -map 0:1 -map 0:0 -map 0:1
         -c:v:0 copy
         -c:a:0 copy
         -c:v:1 libx264 -b:v:1 100k
         -c:a:1 libfaac -ac:a:1 1 -b:a:1 32k
         -program "st=0:st=1" -program "st=2:st=3"
         -f mpegts http://127.0.0.1:8080/publish/sintel
```

3. In Safari, VLC or any HLS compatible browser / player, open
   `http://127.0.0.1:8080/publish/stream_name/index.m3u8`. Note that the first time,
   it might take a few (10-15) seconds before the stream works. This is because
   when you start streaming to the server, it needs to generate the first
   segments and the related playlists.

HLS in HTML:

```
<body>
  <video width="640" height="480" controls autoplay
         src="http://127.0.0.1:8080/play/hls/sintel/index.m3u8">
  </video>
</body>
```

MPEG-DASH in HTML using the dash.js player:

```
<script src="http://cdn.dashjs.org/latest/dash.all.min.js"></script>

<body>
  <video data-dashjs-player
         width="640" height="480" controls autoplay
         src="http://127.0.0.1:8080/play/dash/sintel/index.mpd">
  </video>
</body>
```


Links
-----

* http://nginx.org/
* https://github.com/arut/nginx-ts-module
* https://www.ffmpeg.org/
* https://obsproject.com/
* https://github.com/brocaar/nginx-rtmp-dockerfile
