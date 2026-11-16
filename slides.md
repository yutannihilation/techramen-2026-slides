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

# ゲームボーイアドバンス（GBA）

- 2001年発売
- ゲームボーイとも互換性がある

<img src="/gba.png" class="mx-auto max-h-80" />

---

# なぜ今GBAがアツいのか？

<v-click>

- 今年5月、GBA向けの新作音楽ソフトが発売！！

<Youtube id="z8wOA3V2KJo" width="640" height="360" class="mx-auto" />

</v-click>

---

# GBAの楽器としての可能性

- 音源チップで音楽を鳴らす最後のハードウェア
- 一方で、FM音を出せる程度にパワフルなCPUも積んでいる

<v-click>
<div class="text-center text-red-500 text-5xl">
→ チップチューンとローファイ音の<br/>夢の融合...！！！
</div>
</v-click>

---

# GBAの楽器としての可能性

- これまで、ゲームボーイは楽器として注目されてきた（LSDjなど）
- しかし、ゲームボーイアドバンスはこれまで見過ごされてきた
- まだまだ新しい楽器としての可能性があるはず！

---
layout: section
---

# 開発環境：Rust
