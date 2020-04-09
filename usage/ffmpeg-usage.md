

[toc]

## 获取视频文件时长
```
ffprobe -i $url -show_entries format=duration -v quiet -of csv="p=0"
```

## json方式打印流信息
```
ffprobe -v quiet -print_format json -show_format -show_streams $filename
```

## packet queue size
```
-max_muxing_queue_size 4096
```
## GDB signal
```
handle SIGINT nostop print pass
```
## open file limit
```
vim /etc/security/limits.conf
* soft nofile 65535
* hard nofile 65535

sudo sh -c "ulimit -n 65535 && exec su $LOGNAME"

for ubuntu 18.04 above
sudo vim /etc/systemd/user.conf
sudo vim /etc/systemd/system.conf
DefaultLimitNOFILE=65535

ulimit -a
netstat -ant|awk '/^tcp/ {++S[$NF]} END {for(a in S) print (a,S[a])}'
cat /proc/`ps aux|grep -v grep|grep traffic_ser|awk '{print $2}'`/status|grep Vm
```

## 编译命令
```
--extra-libs='-lpthread -lm -lz' --disable-ffserver --disable-autodetect --disable-avdevice --disable-iconv --enable-pic --enable-debug --enable-openssl --enable-libx264 --enable-libx265 --enable-libfdk-aac --enable-gpl --enable-nonfree --enable-sdl2 --enable-libsrt
```

## 音视频同步
```
audio_hw_params->frame_size = av_samples_get_buffer_size(NULL, audio_hw_params->channels, 1, audio_hw_params->fmt, 1);
audio_hw_params->bytes_per_sec = av_samples_get_buffer_size(NULL, audio_hw_params->channels, audio_hw_params->freq, audio_hw_params->fmt, 1);
```

## 截图
```
ffmpeg -y -i /home/leo/Videos/120.ts -an -vframes 100 frame-%d.png
```

## ffplay 解码信息视觉化
```
使用video filter: codecview
1. Visualize forward predicted MVs of all frames using ffplay:
ffplay -flags2 +export_mvs input.mp4 -vf codecview=mv_type=fp

2. Visualize multi-directionals MVs of P and B-Frames using ffplay:
ffplay -flags2 +export_mvs input.mp4 -vf codecview=mv=pf+bf+bb
```

## 去除nal unit
```
For example, to remove all non-VCL NAL units from an H.264 stream:

ffmpeg -i INPUT -c:v copy -bsf:v 'filter_units=pass_types=1-5' OUTPUT

To remove all AUDs, SEI and filler from an H.265 stream:

ffmpeg -i INPUT -c:v copy -bsf:v 'filter_units=remove_types=35|38-40' OUTPUT
```

## 循环读取输入
```
ffmpeg -loglevel error -re -stream_loop -1 -i /var/www/html/vod/123.ts -c copy -hls_time 1 -hls_segment_filename /var/www/html/hls/live/seg-%6d.ts -hls_list_size 3 -hls_delete_threshold 3 -hls_flags delete_segments /var/www/html/hls/live/index.m3u8
```

## ffmpeg + qsv
```
./configure --disable-autodetect --disable-avdevice --disable-iconv --enable-pic --enable-debug --enable-openssl --enable-libx264 --enable-libx265 --enable-libfdk-aac --enable-gpl --enable-nonfree --enable-vaapi --enable-libmfx --extra-libs='-lmfx -lrt -ldl -lva -lva-drm -lpthread -lm -lz' --extra-cflags='-I/opt/intel/mediasdk/include' --extra-ldflags='-L/opt/intel/mediasdk/lib/lin_x64'
./ffmpeg -y -hwaccel qsv -c:v h264_qsv -i ./120.ts -c:v h264_qsv -b:v 2M -look_ahead 1 output.mp4
```
libmfx.pc
```
prefix=/opt/intel/mediasdk
exec_prefix=${prefix}
libdir=${exec_prefix}/lib/lin_x64/
includedir=${prefix}/include/

Name: libmfx
Description: Media SDK dispatcher
Version: 1.0.0
Cflags: -I${includedir}
Libs: -L${libdir} -lmfx -ldl -lstdc++ -lrt -lva -lva-drm
Libs.private; -Lstdc++ -ldl
```
```
ffmpeg -y -stats -analyzeduration 48000000 -probesize 40000000  -hwaccel qsv  -c:v h264_qsv  -i /home/leo/120.ts -movflags +faststart -map 0:v:0  -vf 'vpp_qsv=framerate=0,scale_qsv=w=640:h=360' -c:v h264_qsv  -b:v 1100k  -bufsize 1100k  -maxrate 1100k  -max_muxing_queue_size 9999 -map 0:a -c:a aac -ac 2 -ar 44100 -b:a 128k /var/www/html/hls/index_640x360b1100.mp4

ffmpeg -y -stats -analyzeduration 48000000 -probesize 40000000  -hwaccel qsv  -c:v h264_qsv  -i /home/leo/120.ts -movflags +faststart -map 0:v:0  -vf 'vpp_qsv=framerate=0,scale_qsv=w=640:h=360' -c:v h264_qsv  -b:v 1100k  -bufsize 1100k  -maxrate 1100k  -max_muxing_queue_size 9999 -map 0:a -c:a aac -ac 2 -ar 44100 -b:a 128k /var/www/html/hls/index_640x360b1100.mp4 -movflags +faststart -map 0:v:0  -vf 'vpp_qsv=framerate=0,scale_qsv=w=1280:h=720' -c:v h264_qsv  -b:v 1800k  -bufsize 1800k  -maxrate 1800k  -max_muxing_queue_size 9999 -map 0:a -c:a aac -ac 2 -ar 44100 -b:a 128k /var/www/html/hls/index_1280x720b1800.mp4
```

## 添加/去除水印
```
添加水印
ffmpeg -i /home/leo/Videos/120.ts -c:v libx264 -b:v 2300k -vf "movie=1.jpg,scale=120:120[watermask];[in][watermask] overlay=10:10[out]" -y out.mp4
去除水印（透明化处理）
ffmpeg -i out.mp4 -c:v libx264 -b:v 2300k -vf delogo=x=10:y=10:w=120:h=120 -c:a copy 1.mp4
```

## 添加实时时间戳
```
ffmpeg -y -re -i /home/leo/Videos/39.ts -s 640x360 -c:v libx264 -b:v 850k -maxrate 1100k -c:a aac -b:a 96k -vf drawtext="fontfile=FreeSans.ttf:x=8:y=8:fontsize=32:fontcolor=red:text='%{localtime\:%H-%M-%S %s}'" 123.ts

ffplay -i D:\document\演示\音视频同步演示\演示程序\test2.ts -vf drawtext="fontfile=FreeSans.ttf:x=8:y=8:fontcolor=white:fontsize=30:text='%{pts}'" 

text=%{n}: 添加帧数
text=%{pts}: 添加时间戳
text=%{gmtime}: 添加时间戳
```

## 视频质量评价 vmaf/psnr/ssim
```
libvmaf: https://github.com/Netflix/vmaf.git
./configure --enable-libvmaf --enable-version3

./ffmpeg -i /home/leo/Videos/dst.ts -i /home/leo/Videos/120.ts -filter_complex "[0:v]scale=1920x1080:flags=bicubic[main];[main][1:v]libvmaf=psnr=1:ssim=1:log_fmt=json" -f null -
./ffmpeg -i /home/leo/Videos/dst.ts -i /home/leo/Videos/120.ts -filter_complex "[1:v]libvmaf=psnr=1:ssim=1:log_path=1.log:log_fmt=json" -an -f null -

psnr:
ffmpeg -i dst.mp4  -i ref.mp4  -lavfi psnr="stats_file=psnr.log" -f null -
ssim:
ffmpeg -i dst.mp4  -i ref.mp4  -lavfi ssim="stats_file=ssim.log" -f null -
```

## 多码率输出
```
ffmpeg -i /home/leo/Videos/xaa.mkv -max_muxing_queue_size 9999 -filter_complex \
    "[0:v:0]split=2[s0][s1]; [s0]scale=w=1280:h=720[v0]; [s1]scale=w=640:h=360[v1]" \
    -c:v:0 libx264 -b:v:0 2000k \
    -c:v:1 libx264 -b:v:1 1000k \
    -map [v0] -map [v1] -map 0:a -c:a aac -b:a 128k \
    -f tee \
    "[select=\'v:0,a\']local0.ts|[select=\'v:1,a\']local1.ts"
```

## 音画不同步选项
```
-first_pts         <int64>      ....A.... Assume the first pts should be this value (in samples). (from I64_MIN to I64_MAX) (default I64_MIN)
    点播转码时可使用，设置为0，当片首音频时间戳落后视频时插入静音帧，加快播放时解析速度
-max_interleave_delta <int64>      E........ maximum buffering duration for interleaving (from 0 to I64_MAX) (default 1e+07)
    点播转码时可使用，适当设置大一些，结合 sortdts 使用对时间戳进行排序， 尤其是不转码视频只转码音频

AVFormatContext AVOptions:
  -avioflags         <flags>      ED....... (default 0)
     direct                       ED....... reduce buffering
  -probesize         <int64>      .D....... set probing size (from 32 to I64_MAX) (default 5e+06)
  -formatprobesize   <int>        .D....... number of bytes to probe file format (from 0 to 2.14748e+09) (default 1.04858e+06)
  -packetsize        <int>        E........ set packet size (from 0 to INT_MAX) (default 0)
  -fflags            <flags>      ED....... (default autobsf)
     flush_packets                E........ reduce the latency by flushing out packets immediately
        降低延迟
     ignidx                       .D....... ignore index
     genpts                       .D....... generate pts
     nofillin                     .D....... do not fill in missing values that can be exactly calculated
     noparse                      .D....... disable AVParsers, this needs nofillin too
     igndts                       .D....... ignore dts
     discardcorrupt               .D....... discard corrupted frames
     sortdts                      .D....... try to interleave outputted packets by dts
     fastseek                     .D....... fast but inaccurate seeks
     nobuffer                     .D....... reduce the latency introduced by optional buffering
        不缓存，可降低播放延迟
     bitexact                     E........ do not write random/volatile data
     shortest                     E........ stop muxing with the shortest stream
     autobsf                      E........ add needed bsfs automatically

-avoid_negative_ts integer (output)
    Possible values:

    'make_non_negative'
    Shift timestamps to make them non-negative. Also note that this affects only leading negative timestamps, and not non-monotonic negative timestamps.

    'make_zero'
    Shift timestamps so that the first timestamp is 0.

    'auto (default)'
    Enables shifting when required by the target format.

    'disabled'
    Disables shifting of timestamp.

    When shifting is enabled, all output timestamps are shifted by the same amount. Audio, video, and subtitles desynching and relative timestamp differences are preserved compared to how they would have been without shifting.

-max_muxing_queue_size packets (output,per-stream)
    When transcoding audio and/or video streams, ffmpeg will not begin writing into the output until it has one packet for each such stream. While waiting for that to happen, packets for other streams are buffered. This option sets the size of this buffer, in packets, for the matching output stream.

    The default value of this option should be high enough for most uses, so only touch this option if you are sure that you need it.

-copyts
    Do not process input timestamps, but keep their values without trying to sanitize them. In particular, do not remove the initial start time offset value.

    Note that, depending on the vsync option or on specific muxer processing (e.g. in case the format option avoid_negative_ts is enabled) the output timestamps may mismatch with the input timestamps even when this option is selected.

-mpegts_copyts boolean
    Preserve original timestamps, if value is set to 1. Default value is -1, which results in shifting timestamps so that they start from 0.
```

## MPTS
```
ffmpeg -i ~/Videos/123.ts -i ~/Videos/124.ts -map 0:0 -map 0:1 -map 1:0 -map 1:1 -c copy -program title=program-avc:st=0:st=1 -program title=program-hevc:st=2:st=3 -f mpegts mpts.ts
```

## 多音轨修复
中间插入的音轨，前面缺少部分时间段的音频，从完整音轨中提取拼接
```
echo "file null.ts" >concat.txt
echo "file 0.ts" >>concat.txt

ffmpeg -i s.ts -map 0:a:0 -c copy -y 0.ts
ffmpeg -loglevel error -xerror -f lavfi -t 8 -i anullsrc -c:a aac -b:a 128k -ac 2 -ar 48000 -y null.ts
ffmpeg -f concat -i concat.txt -c copy a.ts

ffmpeg -y -i s.ts -i a.ts -map 0:v:0 -map 1:a -map 0:a:1 -map 0:a:2 -metadata:s:a:0 language=por -c copy -copyts -avoid_negative_ts disabled -mpegts_copyts 1 d.ts
```
concat txt
```
file null.ts
file 0.ts
```

## 静音音频
```
ffmpeg -loglevel error -xerror -f lavfi -t 60 -i anullsrc -c:a aac -b:a 128k -ac 2 -y a.ts
```

## hls master
```
ffmpeg -i ~/Videos/test/nf.mp4 -map 0:a:0 -map 0:a:1 -map 0:a:2 -map 0:v -c copy -f hls -var_stream_map "a:0,agroup:all,language:por a:1,agroup:all,language:eng,default:yes a:2,agroup:all,language:spa v:0,agroup:all" -hls_playlist_type vod -master_pl_name index.m3u8 -hls_segment_filename stream_%v_%d.ts -hls_time 6 stream_%v.m3u8
```

## PCR计算
```
 时间"03:02:29.012"的PCR计算如下:
03:02:29.012 = ((3 * 60) + 2) * 60 + 29.012 = 10949.012s


PCR_base = ((27 000 000 * 10949.012) / 300) % 2^33 = 98 541 080
PCR_ext   = ((27 000 000 * 10949.012) / 1  ) % 300  = 0 
PCR = 98 541 080 * 300 + 0 = 295 623 324 000

PCR / 27 000 000 == DTS / 90000
```

## 录屏录音
```
ffmpeg -list_devices true -f dshow -i dummy
ffmpeg -f dshow -rtbufsize 8000000 -i video="HP HD Camera":audio="@device_cm_{33D9A762-90C8-11D0-BD43-00A0C911CE86}\wave_{28426CBF-4783-4C43-8470-DE16F22B06B0}" -y -c:v libx265 -s 640x360 -b:v 850k -c:a aac -ac 2 -b:a 64k av.mp4
```
