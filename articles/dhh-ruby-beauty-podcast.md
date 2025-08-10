---
title: "Rails作者のDHHが語るRubyの美しさ"
emoji: "💎"
type: "idea"
topics: ["Ruby", "Rails", "Programming"]
published: true
publication_name: "gmomedia"
---

# Lex Fridman PodcastのDHH回が面白かった

GMOメディアでSREチームに所属している安保です。Zennブログ委員や図書委員なども兼任しています。
Lex Fridman Podcastはよく聴くのですが、今回DHHが出演した回がめちゃくちゃ面白かったので紹介します。
特に、DHHがなぜRubyを「他の言語より美しい」と語ったのか、その理由をコード例とともに書きます。

## そもそもDHHとは誰？

David Heinemeier Hansson（通称DHH）は、Ruby on Railsの生みの親であり、BasecampやHEYを提供する37signalsの共同オーナー・CTOです。『REWORK』『REMOTE』『It Doesn't Have to Be Crazy at Work』『Getting Real』の共著者であり、少人数・高生産性・シンプルなソフトウェアづくりを提唱してきました。  
2022年にはThe Rails Foundationを設立し、2024年11月からはShopifyの取締役に就任。[Hotwire](https://hotwired.dev/)、[Kamal](https://kamal-deploy.org/)、[Omakub](https://omakub.org/)、[Omarchy](https://omarchy.org/)などのOSSも公開しています。また、レーシングドライバーとしても活動し、2014年にはル・マン24時間レースで自身のクラスで優勝した経験もあります。

### ポッドキャストの概要

DHHがプログラミング、テック、人生観まで広く語った6時間超のインタビューです。Rubyに恋に落ちた理由（「人間が読みやすく、幸福になれる言語設計」）、現代の過度な複雑性（マイクロサービスやクラウド依存）への懐疑、AI時代でも「自分の手でタイプして学ぶ」姿勢の重要性などが語られました。

- 参考: [YouTube: Future of Programming, AI, Ruby on Rails, Productivity & Parenting](https://youtu.be/vagyIcmIGOQ?list=TLGGBhloLLCM-vswOTA4MjAyNQ)

---

## Rubyが「人間的に」美しいと感じる理由

DHHが語ったRubyの美しさを、実際のコード例にしてみました。比較のため他のコードの場合も書いています。  
私は普段Pythonくらいで、そこまでプログラミング言語に触ることはないので完全には理解できていないのですが、確かにシンプルでわかりやすいなと思いました。
（※私の解釈が間違っていたらごめんなさい）

### 1. `5.times` とイテレーションの読みやすさ

Ruby（整数もオブジェクトなので、メソッドを呼べる）

```ruby
5.times do |i| # i は 0..4
  puts i
end
```

Python（範囲イテレータ）

```python
for i in range(5):
    print(i)
```

Java（古典的 for）

```java
for (int i = 0; i < 5; i++) {
  System.out.println(i);
}
```

PHP

```php
for ($i = 0; $i < 5; $i++) {
  echo $i . PHP_EOL;
}
```

DHHは「数字の5に対してdotを付けて、timesメソッドを呼ぶ。これは他のプログラミング言語では5回のイテレーションに必要な『line noise』（余計な記号）を取り除いた、例外的に美しい表現」と語っています。

### 2. 条件分岐：if／unless／後置if と「?」述語メソッド

Ruby

```ruby
if user.admin?
  upgrade(user)
end

user.upgrade if user.admin?     # 文の後ろに条件を置く
user.downgrade unless user.admin?  # unless で否定の ! を避ける
```

Python（明快だが否定は not を伴う）

```python
if user.is_admin():
    upgrade(user)

if not user.is_admin():
    downgrade(user)
```

Java（括弧とブレースの儀式が基本）

```java
if (user.isAdmin()) {
  upgrade(user);
}
if (!user.isAdmin()) {
  downgrade(user);
}
```

PHP

```php
if ($user->isAdmin()) {
  upgrade($user);
}
if (!$user->isAdmin()) {
  downgrade($user);
}
```

DHHは「`admin?` の `?` はコンピュータには意味がないが、人間には意味がある。Rubyは人間とのコミュニケーションツールとして `?` を純粋に追加している」と説明しています。また、`unless` について「否定の感嘆符は line noise だ。なぜ `if` と感嘆符を使うのか？それは醜い」と語っています。

### 3. 初期化：initialize vs __init__ vs コンストラクタ

Ruby

```ruby
class User
  def initialize(name)
    @name = name
  end
end
```

Python（__init__ と self）

```python
class User:
    def __init__(self, name):
        self.name = name
```

Java（型・可視性・フィールド宣言の儀式）

```java
class User {
  private final String name;
  public User(String name) { this.name = name; }
}
```

PHP（__construct）

```php
class User {
  private string $name;
  public function __construct(string $name) {
    $this->name = $name;
  }
}
```

DHHはPythonの__init__記法を「すべてが私の美意識の核心を侮辱している」と強く批判し、その二重アンダースコアやselfを含む構文を「醜い」「line noise」と呼んでいます。これはコンピュータのための余分な文字が多く、人間にとっての読みやすさを損ねているためです。対照的に、彼はRubyのinitializeがシンプルな「人間中心」のデザインであり 「line noise」を排除していることを称賛しています。

### 4. 人間的な時間表現（5.days など）

Railsに含まれるActiveSupport拡張

```ruby
Rails.cache.fetch("key", expires_in: 5.days) do
  compute_heavy_result
end
5.days.ago
2.hours.from_now
```

素のRubyでも概念実装は可能（概念例）

```ruby
class Numeric
  def days
    self * 24 * 60 * 60
  end
end

cache_expires_in 5.days # => 432000（秒）
```

「5日」をプログラムの母語にできるのは、Rubyが開発者を信頼して拡張可能にしているから。読み手の脳内負荷を減らす設計思想が表れる部分です。

---

## おわりに

RubyやRailsのコード例を見ただけでも、その「人間にわかりやすく」「必要十分なシンプルさ」が伝わってきます。
DHHは「プログラミングを学びたい初心者にはRubyから始めることを勧める」と語っており、始めるには最適な言語だと感じます。

このブログではポッドキャストでDHHが語った本当に一部しか触れていません。
DHHの考え方や開発哲学、さらには彼のレーシングドライバーとしての経験や家族観に興味が湧いたら、ぜひポッドキャスト本編を聴いてみてください。