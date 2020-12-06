# jsDelivr CDN 视频床

### 什么是 M3U8、TS 文件？
**M3U8**
M3U8 是指 UTF-8 编码格式的 M3U 文件 (M3U 使用 Latin-1 字符集编码)
M3U 文件是一个记录索引的纯文本文件，打开它时播放软件并不会播放它
而是根据它的索引找到对应的音视频文件的网络地址进行在线播放

**TS**
ts 是日本高清摄像机拍摄下进行的封装格式，全称为 MPEG2-TS。ts 即”Transport Stream” 的缩写。
将一个视频文件 (MP4) 切片分为很多个 TS 文件，一个 TS 文件的视频时常可以自定义，比如切片为 5 秒
那么其他 ts 文件也是 5 秒，但是这个不是完全准确，也就是说会有误差，会产生 4-7 秒左右的 ts 视频文件

那他是这么工作的呢？(以下图片是本地运行过程)

![](https://cdn.jsdelivr.net/gh/eallion/static@typora/images/m3u8-ts.png)

### 视频切片

> ffmpeg <https://ffmpeg.org/download.html>

1. 将 mp4 转成 ts 格式，1 对 1，转换后视频质量与大小无变化。(执行下方代码得到 eallion.ts)
```
./ffmpeg.exe -y -i douyin.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb douyin.ts
```

2. 按时间隔分片，1 对 N，下面的 5 即每个分片5秒，自行修改
`-segment_list eallion.m3u8` 为切片后得到的 m3u8 文件
`-segment_time 5 eallion%03d.ts` 为切片后得到的 ts 文件名 5 代表每个 ts 文件 5 秒播放时常 (有误差，不完全 5 秒)
```
./ffmpeg -i douyin.ts -c copy -map 0 -f segment -segment_list douyin.m3u8 -segment_time 5 douyin%03d.ts
```

3. Push 切片

### HLS 技术
什么是 HLS 技术？

HLS (HTTP Live Streaming) 是 Apple 的动态码率自适应技术。主要用于 PC 和 Apple 终端的音视频服务。
包括一个 m3u (8) 的索引文件，TS 媒体分片文件和 key 加密串文件。(摘抄自百度百科)

CDN：<https://cdn.jsdelivr.net/npm/hls.js>

### 视频播放

使用 video 标签播放视频
|属性	|值	|说明|
| -------- | -------- | ------------------------------------------------------------ |
| autoplay | autoplay | 如果出现该属性，则视频在就绪后马上播放。                     |
| controls | controls | 如果出现该属性，则向用户显示控件，比如播放按钮。             |
| height   | pixels   | 设置视频播放器的高度。                                       |
| loop     | loop     | 如果出现该属性，则当媒介文件完成播放后再次开始播放。         |
| muted    | muted    | 规定视频的音频输出应该被静音。                               |
| poster   | URL      | 规定视频下载时显示的图像，或者在用户点击播放按钮前显示的图像。 |
| preload  | preload  | 如果出现该属性，则视频在页面加载时进行加载，并预备播放。 如果使用 “autoplay”，则忽略该属性。 |
| src      | url      | 要播放的视频的 URL。                                         |
| width    | pixels   | 设置视频播放器的宽度。                                       |

```
<script src="https://cdn.jsdelivr.net/npm/hls.js"></script>
<video id="video" preload muted loop autoplay style="height: 100%;width: 100%;object-fit: cover;">
</video>
<script>
  var video = document.getElementById('video');
  var videoSrc = 'https://cdn.jsdelivr.net/gh/eallion/static/video/douyin/douyin.m3u8';
  if (Hls.isSupported()) {
    var hls = new Hls();
    hls.loadSource(videoSrc);
    hls.attachMedia(video);
    hls.on(Hls.Events.MANIFEST_PARSED, function() {
      video.play();
    });
  }
</script>
```

- <https://blog.lete114.top/article/Jsdeliver-video.html>