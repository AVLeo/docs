
## vod转码兼容性问题

1. vod源视频编码参数信息解析失败  
  增大 探测时间‘analyzeduration’和缓存‘probesize’ 大小，默认analyzeduration=5000000（微妙），probesize=5000000（字节）
  ```
  ffmpeg -analyzeduration 45000000 -probesize 45000000 -i 
  ```

2. vod源视频头部几十秒只有视频没有音频  
  向头部插入静音帧，保持音视频的起始时间戳对齐。使用音频重采样选项‘first_pts’
  ```
  ffmpeg -analyzeduration 45000000 -probesize 45000000 -i [] -c:a aac -b:a 128k -ar 2 -ar 44100 -af aresample=first_pts=0 -y []
  ```
  此时需要对音视频包进行缓存和时间戳排序，需设置选项'max_muxing_queue_size'和‘max_interleave_delta’
  ```
  ffmpeg -y -analyzeduration 45000000 -probesize 45000000 -i [] -max_muxing_queue_size 102400 -max_interleave_delta 45000000 -c:a aac -b:a 128k -ar 2 -ar 44100 -af aresample=first_pts=0 -y []
  ```

3. 转码生成mpegts格式文件某些播放器seek后出现播放视频失败  
  mpegts默认只在头部插入了sps/pps信息，seek后某些播放器会重新查找sps/pps进行解码。通过使用‘-bsf:v dump_extra=freq=k’在每个视频关键帧插入sps/pps信息
  ```
  ffmpeg -y -analyzeduration 45000000 -probesize 45000000 -i [] -max_muxing_queue_size 102400 -max_interleave_delta 45000000 -map 0:a -c:a aac -b:a 128k -ar 2 -ar 44100 -af aresample=first_pts=0 -map 0:v -c:v libx264 -bsf:v dump_extra=freq=k dst.ts
  ```

4. mp4视频转mpegts视频失败  
  使用‘-bsf:v hevc_mp4toannexb’或hevc_mp4toannexb
