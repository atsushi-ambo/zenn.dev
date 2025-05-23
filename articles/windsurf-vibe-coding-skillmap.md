---
title: "WindsurfでVibe Coding体験"
emoji: "🏄‍♂️"
type: "tech"
topics: ["AI", "Windsurf", "VibeCoding"]
published: true
publication_name: "gmomedia"
---

# Skillmap.devを作ってみた

GMOメディアSREの安保です。今回はAIコーディングエージェント[Windsurf](https://windsurf.com/)を使い、実際にバイブコーディングスタイルでエンジニア向け学習プロジェクト[Skillmap.dev](https://atsushi-ambo.github.io/skillmap.dev/)を立ち上げてみました。その体験をシェアします。

## Windsurfとは？

Windsurfは、AIエージェントとペアプロしながら開発を進める“バイブコーディング”体験を実現する次世代のAI開発エディタです。

2025年春、Bloomberg等の報道によればOpenAIによる買収で合意したとされており、今後のAI開発基盤としても注目されています。

### 主な特徴・機能
- **ペアプロUI**：チャットでAIエージェントと対話しながら、設計・実装・デバッグまで一気通貫
- **コード自動修正・提案**：既存コードの一部修正やリファクタもAIが提案
- **テストコード生成・実行**：テストの自動生成やコマンド実行もサポート
- **Webアプリの即時プレビュー**：ローカルWebサーバのプレビューやデプロイもワンクリック
- **Git/GitHub連携**：AIの提案をそのままコミット・Push可能
- **CLI/GUI両対応**：エンジニアの好みに合わせて使い分け可能

従来の「AIにまとめて指示」ではなく、会話しながら構造化・分割し、プロンプトを洗練させていく“バイブコーディング”が体験できます。

## Skillmap.devとは？

Skillmap.devはエンジニア向けの実践演習環境を提供するオープンソースプロジェクトです。  
- [GitHubリポジトリ](https://github.com/atsushi-ambo/skillmap.dev)

### 主な特徴
- マークダウンベースのドキュメント
- ハンズオン演習
- Mermaid図による可視化

## WindsurfでSkillmap.devを作るまで

### 1. 新規リポジトリ作成

WindsurfのエージェントUIで「学習用の教材サイトを作りたい」と伝え、READMEやディレクトリ構成、必要なドキュメントの雛形まで一気に生成してもらいました。

### 2. 反復的な開発

- 「Docker Composeでドキュメントサーバーを立ち上げたい」
- 「Mermaid図を入れたい」
- 「セキュリティに配慮したガイドも追加したい」
など、要件を都度伝えると、AIがコードやドキュメントを提案・修正してくれます。

### 3. GitHub公開

AIが生成した成果物をそのままGitHubにpushし、[リポジトリ](https://github.com/atsushi-ambo/skillmap.dev)として公開しました。

## 実際に使ってみて感じたこと

### 良かった点

- コードやドキュメントの雛形作成が爆速。プロジェクト初期の立ち上げが圧倒的に早い
- 会話形式で細かい要望もすぐ反映。反復的な修正・改善がストレスフリー
- コマンドやDocker構成の自動化も得意。Webサーバのプレビューやデプロイもワンクリック
- Git連携やファイル操作もAIから直接実行できる

### 改善してほしい点・苦戦した点

- テストコード生成や複雑な依存関係の理解はやや苦手。既存テストとの連携やファイル分割も難しい場合あり
- 長い指示や複雑な要件を一度に伝えると、AIが意図を誤解することがある
- 日本語の技術用語や表現でたまに解釈違いが発生
- コマンド実行やファイル修正が失敗した場合、リカバリには追加の会話が必要

## 作業時間・工夫した点

- Skillmap.devの立ち上げ～教材雛形作成、GitHub公開まで**半日ほど**で完了
- テストコード生成をWindsurfに指示したが、うまく動作せず苦戦
- リポジトリ全体を[repomix](https://github.com/yamadashy/repomix)でまとめてChatGPT o3に渡すことで理解してもらい、エラー発生時の修正プロンプトを作成しました。そのプロンプトでWindsurfに指示することで問題解決しました。

## まとめ

WindsurfはAIエージェントとの「対話型開発」を実現し、特にプロトタイピングや教材系プロジェクトに最適なプラットフォームです。

OpenAIによる買収合意で今後の進化にも期待が高まります。半日で教材サイトをゼロから公開できるスピード感、AIとの対話による新しい開発体験は非常に刺激的でした。

一方で、テスト生成の精度や複雑な要件の伝達など課題もありましたが、AIの進化とともに今後改善されていくはずです。

今後もWindsurfを使い込み、より高度な開発やAI時代の新しいコーディングスタイルに挑戦していきたいと思います！
