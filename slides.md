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
<div class="text-center text-red-500 text-5xl">
→ チップチューンとローファイ音の<br/>夢の融合...！！！
</div>
</v-click>

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
  - <span v-mark="{ at: 2, type: 'box', strokeWidth: 12, color: 'red' }">`thumbv4t-none-eabi`（16ビット命令）</span> <span v-click="2" class="text-red-500 font-bold ml-2"><br/>カートリッジを読み取るバスの幅が16ビットしかない、等の理由で16ビットの方がパフォーマンスが出る</span>

</v-click>

---

# なんなら Rust で GBA のゲームをつくるフレームワークもすでにある

- agb: ゲーム開発のための機能が揃っているフレームワーク。ドキュメントもしっかりしてる。
- gba: より低レベルなフレームワーク。ドキュメントはあまりない。

---

# agb の方が色々整っててよさげ？

<img src="/agb_crate.png" class="mx-auto max-h-80" />

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
