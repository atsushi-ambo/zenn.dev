---
title: "AWS CloudWatchの謎コストを解明した"
emoji: "📊"
type: "tech"
topics: ["aws", "cloudwatch", "athena", "cost"]
published: true
---

# AWS CloudWatchの謎コストを解明する

AWS CloudWatchは、AWSリソースやアプリケーションのモニタリングに欠かせないサービスですが、意外な隠れたコストが発生する場合があります。本記事では、AWS Cost and Usage Reports (CUR)とAthenaを活用し、CloudWatchに関連する不透明なコストの原因を解析した方法を説明します。今回は、USE1-S3-Egress-Bytesに関連するコストに注目し、その内訳を調査しました。

```
AmazonCloudWatch USE1-S3-Egress-Bytes 
USD 168.71  
$0.25 per GB of log data delivered to S3 for the first 10TB - US East (Northern Virginia)  
674.86 GB
```

## Cost and Usage Reports (CUR)の設定

[AWS Data Export](https://docs.aws.amazon.com/ja_jp/cur/latest/userguide/cur-query-athena.html)は、AWSの利用状況と詳細なコストレポートを提供します。以下の手順で設定を行ってください。

### 1. レポート作成の開始

1. AWSマネジメントコンソールにログインし、右上のアカウント名をクリック
2. 「Billing and Cost Management」を選択
3. 左側のメニューから「Data Exports」を選択
4. 「Create」をクリック

### 2. レポートの設定

レポート設定では、以下の項目を指定します。

1. **Export name**:  
   - 「Legacy CUR export」を選択  
   -  名前（例：`cloudwatch-cost-analysis`）を設定

2. **追加設定オプション**:  
   - Include resource IDs: リソースごとの詳細分析を可能にする  
   - Split cost allocation data: 各チームや機能ごとのコスト配分を明確化  
   - Data refresh settings: データ自動更新を有効化  
   - Time granularity: 時間単位で詳細なデータを取得  
   - Report versioning: データの上書き設定  
   - Report data integration: 今回はAthenaを選択

### 3. S3バケットの設定

レポートデータの保存先となるS3バケットの設定手順:

1. **新規バケットの作成または既存バケットの選択**
2. **レポートパスプレフィックスの設定**（例：`cur/cloudwatch/`）

最後に「Create report」をクリックしてください。
データが作成されるまで24時間くらいかかります。

## Athenaを使用したデータ分析

データがS3にエクスポートされたら、以下の手順で分析を実施します：

### 1. Athenaテーブルのセットアップ
- CURデータ用のデータベースを作成する  
  (例: CREATE DATABASE cost-report;)
- Parquet形式に対応するテーブルスキーマの定義  
  (S3上に出力されたファイル名の最後がcreate-table.sqlとなっているファイルのsqlを実行)
- パーティション設定による効率的なクエリ実行
  (例: MSCK REPAIR TABLE テーブル名)

### 2. 分析結果

使用したクエリ:
```
SELECT
    line_item_usage_type,
    line_item_resource_id,
    line_item_operation
FROM cost_usage_report
WHERE line_item_usage_type = 'USE1-S3-Egress-Bytes'
LIMIT 5;
```
結果:
```
#   line_item_usage_type   line_item_resource_id   line_item_operation
1   USE1-S3-Egress-Bytes   *****(マスク化)         LogDelivery
2   USE1-S3-Egress-Bytes   *****(マスク化)         LogDelivery
3   USE1-S3-Egress-Bytes   *****(マスク化)         LogDelivery
4   USE1-S3-Egress-Bytes   *****(マスク化)         LogDelivery
5   USE1-S3-Egress-Bytes   *****(マスク化)         LogDelivery
```
この結果から、LogDelivery機能の一部として利用されていることが確認できました。調査の結果、これらのログは不要と判断され、停止する予定です。

## 結論

USE1-S3-Egress-Bytesに関するコストは、CloudFrontに設定されたAWS WAFのログが原因で、CloudWatch経由でS3に配信されるために発生していました。不要なログであるため、停止および削除することで、月々USD 168.71のコスト削減が期待できます。AWSのコスト削減は、企業の利益向上に直結する重要な施策なのでどんどん実施していきたいですね。

