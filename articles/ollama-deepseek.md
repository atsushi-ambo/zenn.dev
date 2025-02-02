---
title: "MacBook ProでDeepseek-R1を動かしてみた"
emoji: "🤖"
type: "tech"
topics: ["ollama", "AI", "deepseek", "LLM"]
published: true
publication_name: "gmomedia"
---

# MacBook ProでDeepseek-R1を動かしてみる

最近、注目を集めている[Deepseek-R1](https://github.com/deepseek-ai/DeepSeek-R1)をローカルLLMの実行環境[Ollama](https://github.com/ollama/ollama)で使用してみました。
GMOインターネットグループでは[業務での利用は禁止](https://x.com/m_kumagai/status/1885263436160065694)されていますが、性能を確認してみるためローカルで動かしてみます。

## 環境

- MacBook Pro (M3 Pro)
- メモリ: 36GB
- ollama version: 0.5.7

## Ollamaのインストール

まず、Ollamaをインストールします。Homebrewを使用すると簡単にインストールできます：

```bash
brew install ollama

ollama --version
ollama version is 0.5.7
```

## Deepseek-R1のダウンロードと実行

Deepseek-R1の32Bモデルを使用するには、以下のコマンドを実行します：

```bash
ollama run deepseek-coder:32b
```

初回実行時は、モデルのダウンロードが始まります。モデルサイズは約20GB程度ですが、ダウンロードには結構時間がかかります。

## メモリ使用量について

Deepseek-R1 32Bモデルは、かなりのメモリを必要とします。MacBook Pro M3の36GBメモリでも、以下のような特徴がありました。
長時間動かすとファンが回り始めるくらいでした。

- モデル実行時のメモリ使用量: 約25-30GB
- 応答速度: 1回の応答に1,2分くらい(質問によります)

## 注意点

1. 初回ダウンロードに時間がかかる
2. メモリ使用量が大きい
3. GPUがない環境では応答が遅くなる可能性がある

## 実際に使ってみた
簡単な計算の質問を出してみました。
```
ollama run deepseek-r1:32b
>>> A train travels from city A to city B at 60 km/h and takes 2 hours. On the return journey, due to headwind, it travels at 40 km/h. 
... 1) What is the total distance between cities A and B?
... 2) How long does the return journey take?
... 3) What is the average speed for the entire round trip?
... Please show all calculations.
```

::::details 途中計算 (think)
:::message
```bash
% ollama run deepseek-r1:32b
>>> A train travels from city A to city B at 60 km/h and takes 2 hours. On the return journey, due to headwind, it travels at 40 km/h. 
... 1) What is the total distance between cities A and B?
... 2) How long does the return journey take?
... 3) What is the average speed for the entire round trip?
... Please show all calculations.
<think>
Okay, so I have this problem about a train traveling from city A to city B and then returning. Let me try to understand what's being asked here.

First, there are three questions:

1) What is the total distance between cities A and B?
2) How long does the return journey take?
3) What is the average speed for the entire round trip?

Alright, let's tackle each one step by step.

Starting with the first question: "What is the total distance between cities A and B?"

The problem says that the train travels from city A to city B at a speed of 60 km/h and takes 2 hours. I remember that distance can be calculated using the 
formula:

Distance = Speed × Time

So, if the train's speed is 60 km/h and it takes 2 hours, then the distance from A to B should be:

Distance = 60 km/h × 2 h = 120 km

Wait a second, does this mean that the total distance between cities A and B is 120 km? That seems straightforward.

Okay, moving on to the second question: "How long does the return journey take?"

For the return journey, the train faces a headwind and thus travels at a slower speed of 40 km/h. I need to find out how much time it takes for this return trip.

I think I can use the same distance formula rearranged to solve for time:

Time = Distance / Speed

We already know from the first part that the distance between A and B is 120 km. So, applying this to the return journey:

Time = 120 km / 40 km/h = 3 hours

So, it takes the train 3 hours to travel back from city B to city A.

Now, onto the third question: "What is the average speed for the entire round trip?"

Average speed can be a bit tricky because it's not just the average of the two speeds. Instead, I need to consider the total distance traveled and the total time 
taken.

First, let me calculate the total distance of the round trip. The train goes from A to B and then returns back to A, so that's:

Total Distance = 120 km (A to B) + 120 km (B to A) = 240 km

Next, I need to find the total time taken for the entire round trip. We know that going from A to B took 2 hours, and returning took 3 hours. So:

Total Time = 2 h (A to B) + 3 h (B to A) = 5 hours

Now, average speed is defined as Total Distance divided by Total Time:

Average Speed = Total Distance / Total Time = 240 km / 5 h = 48 km/h

So, the average speed for the entire round trip is 48 km/h.

Wait, let me double-check these calculations to make sure I didn't make any mistakes.

For question 1:
Speed = 60 km/h
Time = 2 hours
Distance = Speed × Time = 60 × 2 = 120 km ✔

For question 2:
Distance remains the same, so Distance = 120 km
Return speed = 40 km/h
Time = Distance / Speed = 120 / 40 = 3 hours ✔

For question 3:
Total Distance = 120 + 120 = 240 km
Total Time = 2 + 3 = 5 hours
Average Speed = 240 / 5 = 48 km/h ✔

Everything seems to check out. I think I've got it!
</think>
:::
::::

回答
```
**Answers:**

1) **What is the total distance between cities A and B?**
   - The train travels at 60 km/h for 2 hours.
   - Distance = Speed × Time = 60 km/h × 2 h = **120 km**

2) **How long does the return journey take?**
   - The return speed is 40 km/h, and the distance remains 120 km.
   - Time = Distance / Speed = 120 km / 40 km/h = **3 hours**

3) **What is the average speed for the entire round trip?**
   - Total Distance = 120 km (A to B) + 120 km (B to A) = **240 km**
   - Total Time = 2 h (A to B) + 3 h (B to A) = **5 hours**
   - Average Speed = Total Distance / Total Time = 240 km / 5 h = **48 km/h**

**Final Answers:**
1. The total distance between cities A and B is **120 km**.
2. The return journey takes **3 hours**.
3. The average speed for the entire round trip is **48 km/h**.
```
Gemini, GPT-o1, Claude 3.5にも同じ質問を聞いてみましたが、全部同じ回答でした。
私個人として性能についてはもう少し確認してみる必要がありますが、DeepSeekが発表している[ベンチマーク](https://github.com/deepseek-ai/DeepSeek-R1)だとかなり高い性能みたいです。

## まとめ

MacBook Pro M3でDeepseek-R1を動かしてみましたが、36GBのメモリだとギリギリ32bモデルがローカルで動きました。

ただし、メモリ使用量が大きいため、使用する際は他のアプリケーションの終了を検討するなど、リソース管理には注意が必要ですね。

中国企業ということでセキュリティ観点がかなり心配されているので、利用には十分気をつけてください。