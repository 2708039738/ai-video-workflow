# 生成后验收:可直接执行的命令

配合主文件「生成后验收」使用。把 `IN.mp4` 换成成片,`OUT.mp4` 换成交付名。Windows 下路径用正斜杠或加引号。

## 1. 规格与音轨

```bash
ffprobe -v error -show_entries format=duration,size \
  -show_entries stream=codec_name,width,height,r_frame_rate,sample_rate,channels \
  -of default=nw=1 IN.mp4
```

看三件事:时长是否等于请求值(常略长,如请求 5 秒出 5.26 秒)、画幅是否正确、有没有音轨。

## 2. 入画与裁切(抽帧)

```bash
mkdir -p _review
# 先取片长:T=$(ffprobe -v error -show_entries format=duration -of csv=p=0 IN.mp4)
# 下面是 5 秒片的例子;换片长时按时长换算成 0.04×T / 0.2×T / 0.5×T / 0.95×T
for t in 0.2 1.0 2.5 4.8; do
  ffmpeg -v error -y -ss $t -i IN.mp4 -frames:v 1 -vf scale=400:-1 "_review/f_${t}.jpg"
done
```

逐帧看:头顶是否留空、鞋是否被裁、主体是否在要求位置、有没有字幕水印、手指数量。

## 3. 台词完整性(转录比对)

```bash
ffmpeg -v error -y -i IN.mp4 -vn -ac 1 -ar 16000 _review/audio.wav
```

把 `_review/audio.wav` 交给语音转录(本地 CLI 或 ASR 接口),把结果与脚本**逐字**比对。文字一致才说明口播没被截断、没被音乐盖住。这一步比"听起来还行"硬。

**没有转录能力时的降级**:不要用"听了觉得对"顶替。只能标 `待核实:台词完整性(无转录条件)`,并写明缺的是哪一段;不得默认通过,也不得在交付里写"台词无误"。

## 4. 有没有多余声音

```bash
ffmpeg -hide_banner -nostats -i IN.mp4 -vn \
  -af "silencedetect=noise=-40dB:d=0.3" -f null - 2>&1 | grep -E "silence_(start|end)"
```

要把"不要音效、不要他人声"证明成静音区间。台词结束到片尾之间应当是连续静音。

## 5. 响度与真峰

```bash
ffmpeg -hide_banner -nostats -i IN.mp4 -vn -af "ebur128=peak=true" -f null - 2>&1 \
  | grep -A6 "Summary"

# 超出目标就归一化(视频流不动)
ffmpeg -y -v error -i IN.mp4 -c:v copy \
  -af "loudnorm=I=-14:TP=-1.5:LRA=11" -c:a aac -b:a 192k \
  -movflags +faststart OUT.mp4
```

目标:人声为主约 `-14 LUFS`,真峰不高于 `-1.5 dBFS`。模型直出常顶到接近 0 dBFS(具体因平台而异),必须处理。

## 6. 机位漂移(固定机位要求时)

用背景区域做模板匹配,同时搜位移和缩放:缩放恒为 `1.000` 说明没推拉,位移非零说明在缓慢漂移。

```python
import subprocess, numpy as np, io
from PIL import Image

def grab(t):
    p = subprocess.run(['ffmpeg','-v','error','-ss',str(t),'-i','IN.mp4',
                        '-frames:v','1','-f','image2pipe','-vcodec','png','-'],
                       capture_output=True)
    return np.asarray(Image.open(io.BytesIO(p.stdout)).convert('L'), dtype=np.float32)

f0 = grab(0.15)
y0, y1, x0, x1 = 380, 940, 0, 300      # 选一块纯背景、不含主体的区域
tpl = f0[y0:y1, x0:x1]

for t in [2.5, 4.8]:
    f1 = grab(t); found = []
    h, w = tpl.shape
    for sc in [0.985, 0.99, 0.995, 1.0, 1.005, 1.01, 1.015]:
        nh, nw = int(h*sc), int(w*sc)
        t2 = np.asarray(Image.fromarray(tpl.astype(np.uint8)).resize((nw, nh)), dtype=np.float32)
        for dy in range(-30, 31, 2):
            for dx in range(-30, 31, 2):
                yy = y0 + (h-nh)//2 + dy; xx = x0 + (w-nw)//2 + dx
                if yy < 0 or xx < 0 or yy+nh > f1.shape[0] or xx+nw > f1.shape[1]:
                    continue
                found.append((np.abs(f1[yy:yy+nh, xx:xx+nw]-t2).mean(), sc, dx, dy))
    found.sort()
    d, sc, dx, dy = found[0]
    print(f'{t}s: 误差={d:.2f} 缩放={sc:.3f} 位移 dx={dx}px dy={dy}px')
```

判读:`缩放≈1.000` + `位移十几像素` = 没推拉但在漂,如实写进交付;要绝对锁死只能后期校正,或重生成。

## 交付时怎么写

只写结论,不复述命令:

```
验收:入画✓ 台词逐字一致✓ 片尾静音 1.6s✓ -14.2 LUFS / TP -1.2✓
偏差:机位有轻微平移漂移(无缩放),校名中间一字被头部遮挡
```
