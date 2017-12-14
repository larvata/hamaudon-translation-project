https://www.bilibili.com/video/av14053219/index_3.html


 ffmpeg -i 1080p.mp4 -vframes 1 firstFrame.png

ffmpeg version 3.3.3 Copyright (c) 2000-2017 the FFmpeg developers
  built with Apple LLVM version 8.0.0 (clang-800.0.42.1)
  configuration: --prefix=/usr/local/Cellar/ffmpeg/3.3.3 --enable-shared --enable-pthreads --enable-gpl --enable-version3 --enable-hardcoded-tables --enable-avresample --cc=clang --host-cflags= --host-ldflags= --enable-libmp3lame --enable-libx264 --enable-libxvid --enable-opencl --enable-videotoolbox --disable-lzma --enable-vda
  libavutil      55. 58.100 / 55. 58.100
  libavcodec     57. 89.100 / 57. 89.100
  libavformat    57. 71.100 / 57. 71.100
  libavdevice    57.  6.100 / 57.  6.100
  libavfilter     6. 82.100 /  6. 82.100
  libavresample   3.  5.  0 /  3.  5.  0
  libswscale      4.  6.100 /  4.  6.100
  libswresample   2.  7.100 /  2.  7.100
  libpostproc    54.  5.100 / 54.  5.100
Input #0, png_pipe, from 'firstFrame.png':
  Duration: N/A, bitrate: N/A
    Stream #0:0: Video: png, rgb24(pc), 1920x1080 [SAR 1:1 DAR 16:9], 25 tbr, 25 tbn, 25 tbc
Input #1, mov,mp4,m4a,3gp,3g2,mj2, from '1080p.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1mp41
    encoder         : Lavf56.1.0
  Duration: 02:17:42.28, start: 0.000000, bitrate: 5733 kb/s
    Stream #1:0(und): Video: h264 (Main) (avc1 / 0x31637661), yuv420p, 1920x1080 [SAR 1:1 DAR 16:9], 5524 kb/s, 29.94 fps, 59.94 tbr, 90k tbn, 59.94 tbc (default)
    Metadata:
      handler_name    : VideoHandler
    Stream #1:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, stereo, fltp, 199 kb/s (default)
    Metadata:
      handler_name    : SoundHandler
Guessed Channel Layout for Input Stream #2.0 : stereo
Input #2, wav, from 'kakeana.wav':
  Duration: 00:02:32.13, bitrate: 2822 kb/s
    Stream #2:0: Audio: pcm_f32le ([3][0][0][0] / 0x0003), 44100 Hz, stereo, flt, 2822 kb/s


<!-- extract kakeana audio -->
ffmpeg -i 720p.mp4 -t 183 -c:a copy kakeana.aac

<!-- build kakeana part -->
ffmpeg -r 30000/1001 -loop 1 -i firstFrame.png -i kakeana.aac -t 183 -map 0:v:0 -c:a copy -map 1:a:0 -c:v libx264 -preset slow -crf 18 kakeana.mp4

<!-- 1080p -->





<!-- 480p for editing subtitle timeline -->
ffmpeg  -i kakeana.mp4 -ss 2 -i 1080p.mp4 -filter_complex "concat=n=2:v=1:a=1 [v] [a]; [v]scale=-1:480[v2]; [a]volume=2.5[a2]" -map "[v2]" -map "[a2]" -c:v libx264 -vsync passthrough -preset veryslow -crf 22 -c:a libfdk_aac -vbr 4 -r 30000/1001 480p_full_veryslow.mp4


<!-- 360p for test -->
ffmpeg -r 30000/1001 -i kakeana.mp4 -ss 2 -i 1080p.mp4 -filter_complex "concat=n=2:v=1:a=1 [v] [a]; [v]scale=-1:360[v2]; [a]volume=2.5[a2]" -map "[v2]" -map "[a2]" -c:v libx264 -preset veryfast -crf 22 -c:a libfdk_aac -vbr 4 360p_full_veryfast.ts


ffmpeg -ss 0 -t 183 -i 720p.mp4 -ss 2 -i 1080p.mp4 -filter_complex "[1:v:0]trim=end_frame=1[s]; [s]loop=setpts=N/1/TB[kake]; [kake] [0:a:0] [1:v:0] [1:a:0] concat=n=2:v=1:a=1 [v] [a]; [v]scale=-1:360[v2]; [a]volume=2.5[a2]" -map "[v2]" -map "[a2]" -c:v libx264 -vsync passthrough -preset veryfast -crf 22 -c:a libfdk_aac -vbr 4 -r 30000/1001 360p_full_veryfast.ts


<!-- good -->
ffmpeg  -i kakeana.mp4 -ss 2 -i 1080p.mp4 -filter_complex "concat=n=2:v=1:a=1 [v] [a]; [v]scale=-1:480[v2]; [a]volume=2.5[a2]" -map "[v2]" -map "[a2]" -c:v libx264 -preset veryslow -async 1 -crf 25 -c:a libfdk_aac -vbr 4 480p_full_veryfast_vfr_async1_preview.mp4

-af "aresample=async=1000"

ffmpeg -i kakeana.mp4 -ss 2 -i 1080p.mp4 -vsync passthrough -filter_complex "concat=n=2:v=1:a=1 [v] [a]; [v]scale=-1:480[v2]; [a]volume=2.5,aresample=async=1000[a2]" -map "[v2]" -map "[a2]" -c:v libx264 -preset veryslow -crf 22 -c:a libfdk_aac -vbr 4 480p_full_veryslow_vfr_async1_vsync_pass_preview.mp4


ffmpeg -i in.mp4 -i watermark.png -filter_complex "[0]fps=10,scale=320:-1:flags=lanczos[bg];[bg][1]overlay=W-w-5:H-h-5,palettegen" palette.png
ffmpeg -i in.mp4 -i watermark.png -i palette.png -filter_complex "[0]fps=10,scale=320:-1:flags=lanczos[bg];[bg][1]overlay=W-w-5:H-h-5[x];[x][2]paletteuse=dither=bayer:bayer_scale=3" output.gif



ffmpeg -f image2 -r 12 -i frame-%04d.png -pix_fmt rgb24 -r 12 -s 640x360 -vf "movie=credits.png [watermark]; [in][watermark] overlay=10:10 [out]" anim.gif







ffmpeg -i kakeana.mp4 -ss 2 -i 1080p.mp4 -i logo.gif -ignore_loop 0 -vsync passthrough -filter_complex "concat=n=2:v=1:a=1 [v] [a]; [v][2:v]overlay=W-w-5:5:enable='between(t,0,18)'[v2]; [v2]scale=-1:720[v3]; [a]volume=2.5,aresample=async=1000[a2]" -map "[v3]" -map "[a2]" -c:v libx264 -preset veryslow -crf 22 -c:a libfdk_aac -vbr 4 360p_full_veryslow_vfr_async1_vsync_pass_preview.ts


<!-- ok -->
ffmpeg -i kakeana.mp4 -ss 2 -i 1080p.mp4 -i logo-full.gif -vsync passthrough -filter_complex "concat=n=2:v=1:a=1 [v] [a]; [2:v]fade=out:st=13:d=2:alpha=0[logo]; [v][logo]overlay=W-w-5:5:enable='between(t,0,15)'[v2]; [v2]scale=640:480[v3]; [a]volume=2.5,aresample=async=1000[a2]" -map "[v3]" -map "[a2]" -c:v libx264 -preset veryslow -crf 22 -c:a libfdk_aac -vbr 4 480p_full_veryslow_vfr_async1_vsync_pass_preview.mp4



<!-- final -->
ffmpeg -i kageana.mp4 -ss 2 -i 1080p.mp4 -i logo-full.gif -vsync passthrough -filter_complex "concat=n=2:v=1:a=1 [v] [a]; [2:v]fade=out:st=13:d=2:alpha=0[logo]; [v][logo]overlay=W-w-5:5:enable='between(t,0,15)'[v2]; [v2]scale=1280:720[v3]; [v3]ass=480p_full_veryslow_vfr_async1_vsync_pass_preview.ass[v4]; [a]volume=2.5,aresample=async=1000[a2]" -map "[v4]" -map "[a2]" -c:v libx264 -preset veryslow -crf 22 -c:a libfdk_aac -vbr 4 720p_full_veryslow_vfr_async1_vsync_pass_preview.ts




ffmpeg -i kageana.mp4 -ss 2 -i 1080p.mp4 -i logo-full.gif -vsync passthrough -filter_complex "concat=n=2:v=1:a=1 [v] [a]; [2:v]fade=out:st=13:d=2:alpha=0[logo]; [v][logo]overlay=W-w-5:5:enable='between(t,0,15)'[v2]; [v2]scale=1280:720[v3]; [v3]ass=480p_full_veryslow_vfr_async1_vsync_pass_preview.ass[v4]; [a]volume=2.5,aresample=async=1000[a2]" -map "[v4]" -map "[a2]" -c:v libx264 -preset veryslow -b:v 1750k -pass 1 -profile:v high -level 4.1 -tune film -c:a libfdk_aac -c:a libfdk_aac -b:a 128k -f mp4 /dev/nul && \
ffmpeg -i kageana.mp4 -ss 2 -i 1080p.mp4 -i logo-full.gif -vsync passthrough -filter_complex "concat=n=2:v=1:a=1 [v] [a]; [2:v]fade=out:st=13:d=2:alpha=0[logo]; [v][logo]overlay=W-w-5:5:enable='between(t,0,15)'[v2]; [v2]scale=1280:720[v3]; [v3]ass=480p_full_veryslow_vfr_async1_vsync_pass_preview.ass[v4]; [a]volume=2.5,aresample=async=1000[a2]" -map "[v4]" -map "[a2]" -c:v libx264 -preset veryslow -b:v 1750k -pass 2 -crf 25 -profile:v high -level 4.1 -tune film -c:a libfdk_aac -c:a libfdk_aac -b:a 128k 720p_full_veryslow_vfr_async1_vsync_pass_2pass_bili.mp4








