# ffmpeg 拼装手册（两次踩坑后的定稿方案）

## 总原则
- **不要用 xfade 做长链**：ffmpeg 8.1.2 下多段 xfade 链会吞掉转场后的尾部帧（内容对、时长错），行为不可靠。改用 concat demuxer 硬拼 + 把过渡"烤进"单镜。
- 链式续接的段本就是首尾相接，镜内硬拼肉眼无感。

## Stage 1：单镜剪辑（每镜一次）
输入：该镜 2–3 个 3.75s 分段。先写 concat 列表，再一条命令完成拼接→变速→调色→颗粒→白闪：

```bash
# concat 列表
printf "file '%s'\n" clips/S01_seg1.mp4 clips/S01_seg2.mp4 clips/S01_seg3.mp4 > edits/concat_S01.txt
# base=该镜拼接后时长（2段=7.5，3段=11.25）；target=目标时长×GF
ffmpeg -y -f concat -safe 0 -i edits/concat_S01.txt -vf \
"setpts=PTS*<target/base>,fps=24,\
colorbalance=bs=0.12:gs=0.04:rs=-0.10:bh=-0.08:gh=0.03:rh=0.10:bm=0.05:gm=0.01:rm=-0.03,\
eq=contrast=1.07:saturation=1.08:brightness=0.01,\
noise=alls=9:allf=t,setsar=1,format=yuv420p" \
-c:v libx264 -preset fast -crf 18 -an edits/shot_S01.mp4
```

**变速数学（曾写反过，警惕）**：`setpts=PTS*(target/base)`。target<base 是加速（因子<1），target>base 是减速。后面必须跟 `fps=24` 把帧数重采样到整齐的 24fps，否则时长仍由原帧数决定。环境空镜（如雨、云）1.4x 加速可接受；人物动作镜尽量接近 1.0。

## 白闪过渡（烤进单镜，不占用额外时间）
- S14 结尾：`,fade=t=out:st=<target-0.35>:d=0.35:color=white`
- S15 开头：`,fade=t=in:st=0:d=0.35:color=white`
- 同理 S20→S21（0.45s，高潮闪白）、S27→S28（0.5s）。淡入淡出都发生在镜内，总时长不变。

## 标题
- 片头：`color=c=black:s=864x480:r=24:d=2` + drawtext，alpha 前 0.5s 淡入、后 0.5s 淡出。
- 片尾：S29 最后 4s 标题淡入（不淡出），位置 `x=(w-text_w)/2:y=h-text_h-60`。
- 字体：`/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf`，fontsize 52（480P 时间线，升格后等比放大），白色 + 黑色阴影。
- drawtext 的 `alpha` 表达式示例：`alpha='if(lt(t,1),t,if(gt(t,1.5),max(0,(2-t)/0.5),1))'`。

## Stage 2：总装
```bash
# final_concat.txt: title_black.mp4 + shot_S01..S29（按顺序）
ffmpeg -y -f concat -safe 0 -i edits/final_concat.txt -i song.mp3 -filter_complex \
"[0:v]<片尾drawtext>,tpad=stop=0.5:stop_mode=clone,scale=1920:1080:flags=lanczos,format=yuv420p[vfin]; \
 [1:a]afade=t=out:st=<歌曲时长-3>:d=3,aresample=48000[afin]" \
-map "[vfin]" -map "[afin]" -c:v libx264 -preset medium -crf 19 \
-c:a aac -b:a 192k -t <歌曲时长> -movflags +faststart final/MV.mp4
```
注意：`aac` 是编码器不是滤镜，不要写进 `-filter_complex`（曾因此整条失败）。

## 验片
- `ffprobe -count_frames` 核对视频帧数 ≈ 时长×24；音频流时长 == 歌曲时长。
- 抽帧看：片头标题、高潮闪白点、中段、片尾标题。
- 分享版：`-c:v libx264 -preset veryfast -crf 27 -c:a aac -b:a 128k`（胶片颗粒会让体积偏大，属正常）。

## 调色说明（青橙 teal-ember）
阴影偏青（bs=+0.12, rs=−0.10）、高光偏暖（rh=+0.10, bh=−0.08），对比 1.07、饱和 1.08，`noise=alls=9:allf=t` 盖住 480P 短板。升格 1080P 用 lanczos；如实告知用户 480P 源放大的偏软。
