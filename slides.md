---
title: Rustでゲームボーイアドバンスを楽器にする
theme: default
background: /background.png
fonts:
  sans: Noto Sans JP
  mono: Fira Code
  weights: 600,900
class: text-center
mdc: true
favicon: /favicon.ico
# seoMeta:
#  ogImage: https://cover.sli.dev
---

# Rustで<br/>ゲームボーイアドバンスを<br/>楽器にする

2026/11/20 TechRAMEN前夜祭

@yutannihilation

---
layout: image-right
image: "/icon.jpg"
backgroundSize: 70%
---

# ドーモ！

## 名前:

湯谷啓明

## 自己紹介:

- 株式会社MIERUNEでGISエンジニア見習いをしています。
- 好きなラーメンは、水煮牛肉麵です。


---
layout: section
---

# なぜ今、<br/>ゲームボーイアドバンスが<br/>アツいのか？


---

# ゲームボーイアドバンス（GBA）

- 2001年発売
- ゲームボーイとも互換性がある

<img src="/gba.png" class="mx-auto max-h-80" />

---

# なぜ今、GBA がアツいのか？

<v-click>

- 今年5月、GBA 向けの新作音楽ソフトが発売！！

<Youtube id="z8wOA3V2KJo" width="640" height="360" class="mx-auto" />

</v-click>

---

# GBA の楽器としての可能性

- 音源チップで音楽を鳴らす最後のハードウェア
- 一方で、FM 音を出せる程度にパワフルなCPUも積んでいる

<v-click>
<div class="my-auto text-center text-red-500 text-5xl">
→ チップチューンとローファイ音の<br/>夢の融合...！！！
</div>
</v-click>

---
layout: image
image: /diagram.png
backgroundSize: contain
---

---

# GBA の楽器としての可能性

- これまで、ゲームボーイは楽器として注目されてきた（LSDj など）
- しかし、GBA はこれまで見過ごされてきた
- まだまだ新しい楽器としての可能性があるはず！

---
layout: section
---

# 開発環境：Rust

---

# Rust で GBA 向けソフトは作れるのか？

<v-click>

- 実は Rust は GBA の CPU（ARM7TDMI）をサポートしている（tier 3）
  - `armv4t-none-eabi`（32ビット命令）
  - <span v-mark="{ at: 2, type: 'box', strokeWidth: 12, color: 'red' }">`thumbv4t-none-eabi`（16ビット命令）</span> <span v-click="2" class="text-red-500 font-bold ml-2"><br/><br/>カートリッジを読み取るバスの幅が16ビットしかない、等の理由で16ビットの方がパフォーマンスが出る</span>

</v-click>

---

# なんなら Rust で GBA のゲームをつくるフレームワークもすでにある

- agb: ゲーム開発のための機能が揃っているフレームワーク。ドキュメントもしっかりしてる。
- gba: より低レベルなフレームワーク。ドキュメントはあまりない。

---

# agb の方が色々整っててよさげ？

<img src="/agb_crate.png" class="mx-auto max-h-100" />

---

# ただ、WAV を鳴らす API しかない...

```rs
use agb::{
    include_wav,
    sound::mixer::SoundData,
};

static BGM: SoundData = include_wav!("a.wav");

let mut bgm = SoundChannel::new(BGM);
bgm.stereo();
```

<v-click>
<div class="text-center text-red-500 text-5xl">
※世の中の人は GBA で音楽をやらない
</div>
</v-click>

---

# ということは、こっち？

- agb: ゲーム開発のための機能が揃っているフレームワーク。ドキュメントもしっかりしてる。
- <span v-mark="{ at: 1, type: 'box', strokeWidth: 12, color: 'red' }">gba: より低レベルなフレームワーク。ドキュメントはあまりない。</span>

---

# gba

- Lokathor さん（bytemuch や tinyvec の作者）が主に開発している。
- 気の利いた API はない代わりに、すべての機能にアクセスできる。よくも悪くも低レイヤー。
- ただしドキュメントはあまりない。

---

# gba のコード例

```rs {|5}
const freq: u16 = 100;

TONE1_FREQUENCY.write(
    ToneFrequency::new()
        .with_frequency(freq)
        .with_enabled(true),
);
```

<v-clicks>

- 周波数なんだろうけど、値の単位は...？
- 関数定義だと引数は `u16` なので最大値は `2^16`？

</v-clicks>

---

# docs.rs「そこになければないですね」

![](/docs-rs.png)

---

# `with_frequency(val: u16)`

<v-clicks>

- このパラメータが直接周波数になるわけではない
- 周波数 = $2^{17}/(2048-val)$ で計算できる
- あと、この式を見て明らかなように、最大値は 2047（$=2^{11}-1$）
  - Rust 上では`u16`になってるけど、ゲームボーイアドバンスに渡せるのは11ビットだけ

</v-clicks>

---
class: flex flex-col
---

# ...みたいな情報はどこで知ればいいの？

<v-click>
<div class="my-auto text-center text-red-500 text-5xl">
古のインターネット！
</div>
</v-click>

---

# ...みたいな情報はどこで知ればいいの？

- Rust の情報はないけど、20年前の C の情報ならある
- 手書き HTML のやさしさ
- まとまってないけどめっちゃ詳しい

---

# ...みたいな情報はどこで知ればいいの？

- [No$GBA](https://problemkaputt.de/gbatek-gba-reference.htm)
- [Tonc](https://gbadev.net/tonc/)
- [Deku's sound mixing tutorial ](https://stuij.github.io/deku-sound-tutorial/)

---

# No$GBA

<img src="/nocash_gba1.png" class="mx-auto max-h-100" />

---

# No$GBA

- GBA のエミュレーターのドキュメント
- ハードウェアの仕様については一番詳しい

<img src="/nocash_gba1.png" class="mx-auto max-h-80" />

---

# No$GBA

- GBA のエミュレーターのドキュメント
- ハードウェアの仕様については一番詳しい

<img src="/nocash_gba2.png" class="mx-auto max-h-80" />

---

# Tonc

<img src="/tonc1.png" class="mx-auto max-h-100" />

---

# Tonc

- GBA でゲームをつくるチュートリアル
- 音楽面では、矩形波の音源チップの使い方を解説している（ノイズ、PCM はなし）

<img src="/tonc1.png" class="mx-auto max-h-100" />


---

# Deku's sound mixing tutorial

<img src="/deku1.png" class="mx-auto max-h-100" />

---

# Deku's sound mixing tutorial

- GBAのサウンドに特化したチュートリアル。
- PCM 音源の使い方を説明している（矩形波、ノイズはなし）

<img src="/deku1.png" class="mx-auto max-h-80" />

---
layout: image
image: /diagram.png
backgroundSize: contain
---

---

# GBA のサウンド関連ハードウェア

- **PSG 音源**：ゲームボーイ互換の部分。音を鳴らす専用のチップから音が出る。
- **CPU 音源**：CPU で生成した波形を鳴らす。

<img src="/diagram.png" class="mx-auto max-h-70" />

---

# すべてをカバーしているドキュメントはない...

- **No$GBA**：レジスタの定義は詳しいが、具体的な使い方の説明はない。
- **Tonc**：PSG 音源、しかも矩形波の説明しかない。
- **Deku**：CPU 音源の説明しかない。

---
layout: section
---

# 矩形波を鳴らしてみる

---

# 矩形波とは？

- こんなやつ

<img src="/sqrwave.png" class="mx-auto h-80" />
<span class="block text-center mt-2">出典：Tonc</span>

---

# 矩形波関連のレジスタ

- 周波数、音のトリガ
- 1音の長さ、デューティ比、エンベロープ関連
- 周波数スイープ（矩形波1のみ）

---

# 周波数、音のトリガ

- **R**: 周波数を定めるパラメータ。周波数 = $2^{17}/(2048-R)$ で計算できる。
- **T**: 設定した1音の長さを無視するかどうか
- **Re**: 音をトリガするかどうか

<img src="/reg1.png" class="mx-auto h-40" />

---

# 周波数、音のトリガのレジスタ

- つまり、音を鳴らしたいときはこんなバイト列をレジスタに書き込む
  - `F`: 音をトリガする
  - `E`: 指定した長さで音を止める
  - `A`-`0`: 鳴らしたい周波数から逆算した値

|F|E|D|C|B|A|9|8|7|6|5|4|3|2|1|0|
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|1|1| | | |?|?|?|?|?|?|?|?|?|?|?|

---

# Rustのコードで言うとこんな感じ

- `note_to_rate()`は欲しい周波数が得られるパラメータを計算する関数

```rs
TONE1_FREQUENCY.write(
    ToneFrequency::new()
        .with_enabled(true)
        .with_stop_when_expired(true)
        .with_frequency(note_to_rate(...)),
);
```

---

# `note_to_rate()`の例

- 1オクターブ分の音は計算しておく
- 1オクターブ上がると周波数は倍になるのがミソ

```rs
const BASE_RATES: [u16; 12] = [
    8013, 7566, 7144, 6742, 6362, 6005, 5666, 5346, 5048, 4766, 4499, 4246,
];

fn note_to_rate(i: usize, octave: u8) -> u16 {
    let base_freq = BASE_RATES[i];
    2048 - (base_freq >> octave)
}
```

---

# エンベロープ関連のレジスタ

- **L**: 1音の長さ。$(64-L)/256$秒に設定される。
- **D**: デューティ比（1/8、1/4、1/2、3/4）。
- **EST**: エンベロープの増加/減少の速さ。
- **ED**: エンベロープが増加か減少か。
- **EIV**: エンベロープの初期値。

<img src="/reg2.png" class="mx-auto h-40" />

---

# そもそもエンベロープとは？

- 音が鳴ってから消えるまでの時間的な音量の変化のこと。
- ざっくり言うと、音の大きさや長さ。
- （デューティ比はエンベロープとは関係ない）

---

# デューティ比

- オンとオフの時間の割合。音色に影響する。

<img src="/duty_cycle.gif" class="mx-auto h-80" />
<span class="block text-center mt-2">出典：<a href="https://commons.wikimedia.org/wiki/File:PWM_duty_cycle_with_label.gif">Wikipedia</a></span>

---

# Rustのコードで言うとこんな感じ

```rs
TONE1_PATTERN.write(
    TonePattern::new()
        .with_length(8)    // 1音の長さ
        .with_duty(2)      // デューティ比（2=50%）
        .with_step_time(4) // 音量変化の滑らかさ
        .with_step_increasing(false)
        .with_volume(12),  // 音量の大きさ
);
```

---

# 周波数スイープのレジスタ

（省略）

---

# ミキサー関連のレジスタ

- 図では省略したけど色々設定がある

```rs
SOUND_ENABLED.write(
    SoundEnable::new()
        .with_enabled(true)
        .with_tone1_playing(true),
);
```

```rs
SOUND_MIX.write(
    SoundMix::new().with_psg(PsgMix::_100)
);
```

---

# 全体のボリューム等のレジスタ

```rs
LEFT_RIGHT_VOLUME.write(
    LeftRightVolume::new()
        .with_left_volume(7)
        .with_right_volume(7)
        .with_tone1_left(true)
        .with_tone1_right(true),
);
```

---

# デモ


---
layout: section
---

# まとめ

---

# 今日話せなかったこと

- 矩形波以外の音源チップの使い方
- CPU での音の作り方
- グラフィックの操作やボタン入力
- コンパイルしたバイナリを実際に GBA で実行する方法

---

# まとめ

- GBA は、チップチューンとローファイ音楽の夢の融合！
- Rust で GBA 向けのソフトウェアを開発することができる
- ドキュメントはないが、インターネットの海に漂う知識が助けてくれる

---

# 余談：FMSの最新のアップデート

<v-click>

<img src="/fms_update.png" class="mx-auto max-h-100" />

</v-click>

---

# えっ！？

<img src="/fms_update2.png" class="mx-auto max-h-100" />

---

# FMS、チップチューンやめるってよ

- もともと FMS は、音源チップはノイズしか使ってなかった
- しかし、音源チップのノイズよりも、ソフトウェアのノイズの方が多彩な音を出せることが判明...

<v-click>

- 実は、GBAはもうアツくない...？

</v-click>

---

# でも FMS はアツいので応援したい

<Youtube id="7KFQ_VzE8Po" width="640" height="360" class="mx-auto" />
