---
title: "オライリー学習プラットフォームのAIが便利だった"
emoji: "⚙️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["nginx", "apache", "phpfpm", "AI", "OReilly"]
published: true
publication_name: "gmomedia"
---

# O'Reilly AIに聞いてみた - NginxとApacheのスレッディングモデルの違い

GMOメディアでSREを担当している安保です。
今回はGMOメディアの福利厚生である[Tech Try](https://www.gmo.media/recruit/environment/?utm_source=zenn&utm_medium=display&utm_campaign=zenn)という制度を使用してオライリー学習プラットフォームを使用してみた感想を書きます。

Tech Tryは技術書購入、カンファレンスの参加費用、オンラインコースの受講費用などの技術的挑戦に関わる費用を会社が一部負担してくれる制度です。

この制度を利用して私は[O'Reilly学習プラットフォーム](https://www.oreilly.com/online-learning/)を契約してみました。
この学習プラットフォームには[O’Reilly Answer](https://www.oreilly.com/online-learning/feature-answers.html)というAIを利用している機能があり、質問をして生成AIが回答してくれます。
オライリーの書籍や資料を元にした回答が得られるため、技術的な質問に対しても信頼性の高い情報を得ることができます。

現在、仕事でnginxの設定を行っているのですが、nginxとapacheのスレッディングモデルの違いについて調べるために下記の質問をしてみました。

日本語には対応していないため英語となります。

> "What are the fundamental differences in threading models between nginx, Apache, and how does this affect PHP execution?"
> （NginxとApacheのスレッディングモデルの根本的な違いは何で、それがPHPの実行にどう影響するか？）

## AIからの回答(日本語訳)

以下は、O'Reilly AIからの回答を日本語に翻訳したものです。

### イントロダクション

NginxとApacheのスレッディングモデルの根本的な違い、そしてそれがPHP実行にどう影響するかを理解することは、Webサーバーのパフォーマンスとスケーラビリティを把握する上で非常に重要です。

### スレッディングモデル

Nginxは**非同期のイベント駆動型アーキテクチャ**を採用しており、少ないメモリ使用量で数千の同時接続を処理できます。一方、Apacheは**プロセスベース**または**スレッドベースのモデル**を使用しており、特に多数の同時接続を処理する場合にメモリ使用量とCPUオーバーヘッドが高くなる可能性があります。

例えば、Apacheのプロセスベースモデルは`mpm_prefork`モジュールを使用するように設定でき、これはリクエストごとに新しいプロセスを作成します。対照的に、Nginxのイベント駆動型アーキテクチャは、リクエストごとに新しいプロセスやスレッドを作成せずに、非同期でリクエストを処理できます。

### PHP実行

Apacheは外部言語プロセッサ（PHP-FPMなど）を使用するように設定して、パフォーマンスを向上させることができます。PHP-FPMを使用すると、Apacheは非同期でPHPリクエストを処理でき、パフォーマンスとスケーラビリティが向上します。

一方、Nginxは動的コンテンツを処理するためにPHP-FPMなどの外部プロセッサに依存しています。例えば、Nginxは以下のコードを使用してPHP-FPMと連携するように設定できます：

```nginx
location ~ \.php$ {
    try_files $uri =404;
    fastcgi_pass unix:/var/run/php5-fpm.sock;
    fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
    include fastcgi_params;
}
```

このコードはNginxがPHPリクエストをPHP-FPMに渡すように設定し、PHP-FPMがリクエストを非同期で処理します。

### パフォーマンス比較

NginxとApacheのパフォーマンスの違いは、特に多数の同時接続を処理する場合に顕著になります。Nginxのイベント駆動型アーキテクチャは、少ないメモリ使用量で数千の同時接続を処理できますが、Apacheのプロセスベースモデルはメモリ使用量とCPUオーバーヘッドが高くなる可能性があります。

例えば、あるベンチマークテストでは、Nginxは10,000の同時接続を10msの応答時間で処理できましたが、Apacheは1,000の同時接続を100msの応答時間でしか処理できませんでした。

```bash
ab -n 1000 -c 100 http://example.com/
```

このコードは`ab`ツールを使用してWebサーバーのパフォーマンスをベンチマークしており、NginxとApacheのパフォーマンスを比較するのに役立ちます。

# O'Reilly学習プラットフォームの利点

AIの回答はNginxとApacheの根本的なアーキテクチャの違いを分かりやすく説明していました。特に高トラフィックのWebサイトを運用する場合、これらの違いを理解することは重要ですね。

生成AIが作成した回答にはリファレンスがついており、どの書籍から引用されたかがわかります。これにより、さらに深く学びたい場合は、その書籍を参照することができます。

![O'Reilly AI interface](/images/oreilly/ai.png)

# まとめ
O'Reilly学習プラットフォームは、技術的な質問に対して信頼性の高い情報を提供してくれる素晴らしいツールです。今回はNginxとApacheのスレッディングモデルの違いについての理解を深めるために非常に役立ちました。

今後もこのプラットフォームを活用して、技術的な知識を深めていきたいと思います。
