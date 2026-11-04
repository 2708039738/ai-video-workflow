# MiniMax-H3 提示词格式(全参考模式)

本文件让技能自带 H3 的输出格式,不依赖外部技能。写卡时按此结构产出,语言为英文正文 + 台词原语言。

## 1. 六段结构与顺序

必须按此顺序,字段名不改写:

```
subject_definitions:     标签定义
summary:                 一段话 + 方括号任务类型前缀
retention_analysis:      每个标签一行,关系标记
detailed_description:     逐镜头正文(≤500 词)
overall_soundscape:      1–4 句环境音
non_diegetic_music:      1–3 句配乐,没有写 N/A
```

可选追加 `single_person_lock:`(防重影/防重复人),放在 `detailed_description` 与 `overall_soundscape` 之间。这是项目习惯,不属于官方六段。

## 2. 标签

| 标签 | 用途 |
|---|---|
| `<Subject N>` | 可复用的可见内容:人、场景、服装、道具、风格 |
| `<Picture N>` | 只在该图本身就是首帧/尾帧/关键帧时才单列;仅用于定义人物或场景时,把 `<Picture N>` 写在对应 `<Subject N>` 里,不单列 |
| `<Video N>` | 整段视频级关系:剪辑源、续写、镜头结构参考 |
| `<Audio N>` | 音频:整轨复用、音色参考、节奏参考 |

规则:一个标签只在 `subject_definitions` 定义一次,全文复用同一含义;正文里不得出现未定义的标签。

## 3. 任务类型前缀(`summary` 开头)

`[reference generation]`、`[keyframe completion]`、`[video editing]`、`[video continuation]`、`[audio reuse]`、`[audio reference]`。可组合,用 ` + ` 连接,不重复。

判据:
- 图像只提供人物/场景/风格 → `reference generation`
- 图像本身就是某一帧 → `keyframe completion`
- 音频原样当最终音轨 → `audio reuse`
- 只借音色、台词重新生成 → `audio reference`

## 4. retention_analysis 的关系标记

可见内容(`<Subject>`/`<Picture>`/`<Video>`):`fully_preserved` / `partially_preserved` / `attribute_transfer` / `weak_reference`

音频(`<Audio>`):`fully_copy` / `partially_copy` / `reference` / `weak_reference`

格式:`<Subject 1> (appears in [Shot 1], [Shot 2]): fully_preserved - …`
音频段不写说话人 ID。

## 5. detailed_description 写法

- **风格句放在 `[Shot 1]` 之前**,一到两句:介质(实拍/动画)、画幅、光线、质感。
- `[Shot 1]` **不带时间戳**;后续镜头写 `[Shot 2] At 00:09.000, the shot cuts to …`,时间严格递增且小于总时长。
- 每个镜头只写**一个主要动作**和**一种运镜**。运镜用词:
  `Push In / Pull Out / Pan Left|Right / Truck Left|Right / Tilt Up|Down / Pedestal Up|Down / Arc Shot / Tracking Shot / Static Shot / Shake Slightly|Strongly / POV / Roll Clockwise|Counterclockwise`
  需要时补 `with small|large amplitude`、`at slow|fast speed`,写成句子里的一部分,不要堆在句末当标签。
- 台词:`(S1)` 写在 `<d>` 外,`<d>[Chinese] 原文</d>` 内只放语言标记和原话,逐字不改。首次出现的说话人要给身份、年龄、性别、是否出镜、音色语速。
- 画外音固定写法:`says in an off-screen voiceover`,并紧跟一句"嘴唇保持闭合"。
- 屏幕可见文字用英文双引号原样包裹,如 `the glowing sign "某职业培训学校"`。**但有严格前提**,见下。
  - **仅适用于**:场景装饰性文字、不承载商业信息、允许不精确(远景招牌、模糊价签、装饰图形里的字)。
  - **不适用于**:品牌名与 Logo、价格、地址电话、促销承诺、资质证照、产品标签法定信息——这类一律转后期,不写进提示词。
  - **参考图里已有的文字**:不要用提示词"复刻"。要么取景避开,要么让它虚焦,要么生成后贴原图区域。详见 `on-image-text.md`。
  - **原因**:模型没有复制文字的能力,只有生成"像字的图形"的能力。写进提示词只会让它更努力地画,错得更离谱。
- 篇幅:生成类 350–500 词;台词密集时优先保证台词时间轴完整。

## 6. 音频两种关系,写法不同

| | 音轨复用 `fully_copy` | 音色参考 `reference` |
|---|---|---|
| 场景 | 已有成品音轨 | 只有音色样本,台词重生成 |
| 音频定义里写什么 | 总长、每句起止、句间静默 | 音色与说话方式,声明原词不搬运 |
| 正文怎么写 | 表演锚到秒,静默处闭嘴 | 台词写进 `<d>`,给说话窗口 |
| 常见错误 | 只写"参考该音频"不写时间轴 | 把样本里的原话带进正文 |

## 7. 完整可用示例(5 秒 · 单人口播 · 固定机位)

已实际生成并复核。人物参考图五视角、场景为前台、音频为音色参考。

```
subject_definitions:
<Subject 1> is the Chinese woman defined by <Picture 1>. <Picture 1> is a character sheet showing the same single person from five angles, not five different people. Her look is fixed for the whole video: round face with slightly wide cheekbones, thin gold-rimmed wire glasses, soft red lipstick, short black hair combed back into a low bun, a light-blue satin shirt with a large bow collar, a long dark olive-grey open-knit cardigan, black wide-leg trousers and black pointed heels. She is the on-camera presenter (S1).
<Subject 2> is the school reception area defined by <Picture 2>, kept exactly as photographed: a beige-gold vertically ribbed wall, the glowing school emblem above the illuminated characters "某职业培训学校", a grey door with a small gold plaque on the left, a white angled counter on the right, a crystal chandelier overhead.
<Audio 1> is the voice-timbre reference for <Subject 1> (S1): a clean recording of her cloned voice, no music, no second speaker. It supplies timbre and manner only; the speech is newly generated rather than copied.

summary:
[reference generation + audio reference] <Picture 1> fixes who she is and <Picture 2> fixes the space; both are locked. <Subject 1> (S1) stands centred in <Subject 2> in one full-body fixed frontal shot and speaks a newly generated line in the cloned timbre of <Audio 1>, opening both hands in a small welcoming gesture and closing on a smile.

retention_analysis:
<Subject 1> (appears in [Shot 1]): fully_preserved - face, glasses, lipstick, low bun, blue bow shirt, olive-grey cardigan, black trousers and pointed heels follow <Picture 1> and never change.
<Subject 2> (appears in [Shot 1]): fully_preserved - the ribbed wall, glowing emblem, the characters "某职业培训学校", the grey door and the white counter are retained in place.
<Audio 1>: reference - it supplies timbre and delivery only; the spoken signal is newly generated and none of its original words are carried over.

detailed_description:
The target video is polished live-action in vertical 9:16, shot on a fixed frontal camera at eye level, one full-body framing of <Subject 1> (S1) standing centred in <Subject 2>, her whole body inside the frame with a little air above her hair and about ten percent of the frame left as floor beneath her shoes. Warm ceiling light falls evenly on her face.
[Shot 1] Single continuous shot, no cut. <Subject 1> (S1) stands facing the lens, arms relaxed at her sides, dressed exactly as <Picture 1> shows; behind her <Subject 2> stays perfectly still and nothing in the background moves or redraws. At 00:00.400 her arms rise from her sides and open slightly outward, elbows softly bent, palms up, hands stopping at the height of her lower waist and no wider than her shoulders. Speaking on camera in the cloned voice referenced from <Audio 1>, lip movement locked to the words, she says, <d>[Chinese] 学技术来本校，我在本校等你。</d> at a natural conversational pace. From 00:03.500 her hands lower back to her sides, and from 00:04.300 her lips close and she holds a warm smile into the lens. She speaks only this line; no second voice, no music and no sound effect is heard. The camera never moves, and her position in frame stays fixed.

overall_soundscape:
Quiet indoor lobby room tone continues throughout, with a faint air-conditioning hum and nothing else; no footsteps and no other voices.

non_diegetic_music:
N/A
```

**这条的实测结果**(作者单次自述,无第三方复核;仅作"到位长什么样"的参照,不是可复现的基准):台词落在设定窗口附近、句前后全静音、全身入画未裁鞋、手部无畸形、直出音频需归一化。已知偏差:校名中间一字被头部遮挡、机位有缓慢平移漂移。具体数值见 `platform-profiles.md` 的 MiniMax-H3 条目。

## 8. 最常见的退回原因

1. 多视角角色图被读成多人 → 定义里写明"同一人 N 次"
2. 主体居中遮住背景标识 → 前置站位或偏移构图
3. 写死"0.0 秒开口" → 合成语音有 0.2–0.7 秒起音
4. 只写"参考音频"不写时间轴 → 模型自编节奏
5. 直出音频顶格 → 交付前归一化
