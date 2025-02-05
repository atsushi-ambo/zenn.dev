---
title: "MacBook ProでDeepSeekの性能に近いQwenを動かしてみた"
emoji: "🤖"
type: "tech"
topics: ["ollama", "AI", "deepseek", "LLM", "qwen"]
published: true
publication_name: "gmomedia"
---

# MacBook ProでQwenを動かしてみる

最近、注目を集めている[Qwen](https://github.com/QwenLM/Qwen)をローカルLLMの実行環境[Ollama](https://github.com/ollama/ollama)で使用してみました。
Qwen2.5-Maxが一番性能良いみたいですが、今回は私のローカル環境で動くモデル(qwen2.5:32b)の性能を見てみます。

## 環境

- MacBook Pro (M3 Pro)
- メモリ: 36GB
- ollama version: 最新

## Ollamaのインストール

まず、Ollamaをインストールします。Homebrewを使用すると簡単にインストールできます：

```bash
brew install ollama

ollama --version
ollama version is 0.5.7
```

## Qwen2.5 32Bのダウンロードと実行

モデルを使用するには、以下のコマンドを実行します：

```bash
ollama run qwen2.5:32b
pulling manifest 
pulling eabc98a9bcbf... 100% ▕█████████████████████████████████████████████████████████████████████████████████████████████▏  19 GB                         
pulling 66b9ea09bd5b... 100% ▕█████████████████████████████████████████████████████████████████████████████████████████████▏   68 B                         
pulling eb4402837c78... 100% ▕█████████████████████████████████████████████████████████████████████████████████████████████▏ 1.5 KB                         
pulling 832dd9e00a68... 100% ▕█████████████████████████████████████████████████████████████████████████████████████████████▏  11 KB                         
pulling 833f525ce183... 100% ▕█████████████████████████████████████████████████████████████████████████████████████████████▏  488 B                         
verifying sha256 digest 
writing manifest 
success
```

初回実行時は、モデルのダウンロードが始まります。モデルサイズは約20GB程度ですが、ダウンロードには少し時間がかかります。

## メモリ使用量について

qwen2.5 32Bモデルは、かなりのメモリを必要とします。MacBook Pro M3の36GBメモリでも、以下のような特徴がありました。
長時間動かすとファンが回り始めるくらいでした。

- モデル実行時のメモリ使用量: 約30-34GB (実行していない時は15GB使用済み)
- 応答速度: 1回の応答に1,2分くらい(質問によります)

## 注意点

1. 初回ダウンロードに時間がかかる
2. メモリ使用量が大きい
3. GPUがない環境では応答が遅くなる可能性がある

## 実際に使ってみた
簡単な計算の質問を出してみました。
```
ollama run qwen2.5:32b
>>> A train travels from city A to city B at 60 km/h and takes 2 hours. On the return journey, due to headwind, it travels at 40 km/h. 
... 1) What is the total distance between cities A and B?
... 2) How long does the return journey take?
... 3) What is the average speed for the entire round trip?
... Please show all calculations.
```

::::details 途中計算 (think)
:::message

```bash
ollama run qwen2.5:32b
>>> >>> A train travels from city A to city B at 60 km/h and takes 2 hours. On the return journey, due to headwind, it travels at 40 km/h. 
... ... 1) What is the total distance between cities A and B?
... ... 2) How long does the return journey take?
... ... 3) What is the average speed for the entire round trip?
... ... Please show all calculations.
Sure, let's go through each part of your question step-by-step.

### 1) Total Distance Between Cities A and B

Given:
- The train travels from city A to city B at a speed of \(60 \text{ km/h}\).
- It takes \(2\) hours for this journey.

To find the distance between cities A and B, we can use the formula:

\[
\text{Distance} = \text{Speed} \times \text{Time}
\]

So,

\[
\text{Distance from A to B} = 60 \text{ km/h} \times 2 \text{ h} = 120 \text{ km}
\]

The total distance between cities A and B is \(120\) kilometers.

### 2) Duration of the Return Journey

Given:
- The train travels from city B to city A at a speed of \(40 \text{ km/h}\).

Using the same formula:

\[
\text{Time} = \frac{\text{Distance}}{\text{Speed}}
\]

So,

\[
\text{Duration of return journey} = \frac{120 \text{ km}}{40 \text{ km/h}} = 3 \text{ hours}
\]

The duration of the return journey is \(3\) hours.

### 3) Average Speed for the Entire Round Trip

To find the average speed for the entire round trip, we need to consider both the total distance and the total time taken.

- Total distance: Distance from A to B + Distance from B to A
\[
= 120 \text{ km} + 120 \text{ km} = 240 \text{ km}
\]

- Total time:
\[
= 2 \text{ hours (to B)} + 3 \text{ hours (return)}
= 5 \text{ hours}
\]

The formula for average speed is:

\[
\text{Average Speed} = \frac{\text{Total Distance}}{\text{Total Time}}
\]

So,

\[
\text{Average Speed} = \frac{240 \text{ km}}{5 \text{ h}} = 48 \text{ km/h}
\]

The average speed for the entire round trip is \(48\) kilometers per hour.
:::
::::

回答
```
### Summary

1. The total distance between cities A and B is \(120\) kilometers.
2. The duration of the return journey from city B to city A is \(3\) hours.
3. The average speed for the entire round trip is \(48\) km/h.
```
Gemini, GPT-o1, Claude 3.5にも同じ質問を聞いてみましたが、全部同じ回答でした。

## 削除
もし、不要な場合は下記のコマンドで削除できます。
```
% ollama rm qwen2.5:32b
deleted 'qwen2.5:32b'
```

## まとめ

MacBook Pro M3でqwen2.5:32bを動かしてみましたが、36GBのメモリだとギリギリ32bモデルがローカルで動きました。
ただし、メモリ使用量が大きいため、使用する際は他のアプリケーションの終了を検討するなど、リソース管理には注意が必要ですね。
私個人として性能についてはもう少し見て行く予定ですが、Qwenが発表しているQwen2.5-Maxの [ベンチマーク](https://qwenlm.github.io/blog/qwen2.5-max/)だとDeepSeek-V3やGPT-4oの性能に近いと書いてあります。Qwenは引き続き注目していきたいオープンソースLLMですね。
