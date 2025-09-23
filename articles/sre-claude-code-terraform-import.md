---
title: "SREがClaude Codeを活用してIaC化した方法"
emoji: "🔧"
type: "tech"
topics: ["SRE", "Terraform", "ClaudeCode", "IaC", "AWS"]
published: true
---

## はじめに

SREとして日々インフラ管理に携わる中で、既存のAWSリソースをIaC（Infrastructure as Code）化することはあると思います。この記事では、Claude Codeを活用してAWSリソースの棚卸しとTerraform管理への移行を効率化する方法を紹介します。

## 前提条件

この記事の手順を実行するには、以下の環境が必要です：

### Claude Code
- **Claude Code** のインストール

### 実行環境
- **AWS CLI** がインストールされたローカル環境（v2.x推奨）
  - または **AWS CloudShell** （ブラウザベースのシェル環境）
- **jq** コマンド（JSONの整形・解析用）

### Terraform実行環境
- **Terraform**: v1.5以上
- **Terraform stateファイル**: S3バケットで管理 (+DynamoDB)
- **Terraform実行**: GitHub Actionsで自動化（ローカルでは実行しない）

### 必要な権限
- **ReadOnly権限** のAWS IAMユーザーまたはロール
- 確認したいリソースを指定する：
  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": [
          "ec2:Describe*",
          "rds:Describe*",
          "s3:ListBucket",
          "s3:ListAllMyBuckets",
          "s3:GetObject",
          "s3:GetBucketLocation",
          "iam:List*",
          "iam:Get*",
          "elasticloadbalancing:Describe*",
          "ecs:List*",
          "ecs:Describe*"
        ],
        "Resource": "*"
      }
    ]
  }
  ```

### その他のツール
- Python 3.8以上
- 必要なPythonライブラリ：
  ```bash
  pip install boto3 pandas tabulate
  ```

## 解決したい課題

今回は下記の課題がありました：

1. **可視性の欠如**: コンソールから手動作成されたAWSリソースが散在している
2. **管理状態の不明瞭さ**: どのリソースがTerraform管理下にあり、どれが管理外なのか把握できていない 
3. **移行作業の複雑さ**: 既存リソースをTerraformにインポートする作業が煩雑

これらの課題を、Claude Codeを使ってスクリプト化することで解決してみます。

## Claude Codeによる解決アプローチ

### 解決ステップの概要

以下の3つのステップでClaude Codeを活用してIaC化を進めます：

| ステップ | 実施内容 | Claude Codeの活用方法 | 成果物 |
|---------|----------|---------------------|--------|
| **リソース棚卸し** | AWS CLIでリソース一覧を取得 | スクリプト生成プロンプトを送信 | `get_aws_resources.sh` |
| **管理状況分析** | Terraform状態と実リソースの比較 | 分析スクリプト生成プロンプトを送信 | `analyze_terraform_coverage.py` |
| **インポート準備** | 未管理リソースのimport blocks生成 | 実行して結果を確認 | `import_blocks.tf` |

## Claude Codeを使った実装アプローチ

### 1. AWS CLIでリソース一覧を取得

まず、Claude Codeに以下のようなプロンプトを投げます：

```
AWS CLIを使って、以下のリソースをすべて取得するスクリプトを作成してください：
- EC2インスタンス（終了済みを除く）
- RDSインスタンス
- S3バケット（バージョニング状態も含む）
- IAMロール（信頼ポリシーも含む）
- Security Groups
- ALB/NLB
- VPC
- サブネット

ただし、以下は取得時に除外してください：
- ECSタスク定義の複数リビジョン（最新のみ取得）
- デフォルトのSecurity Group
- AWS管理のIAMロール（AWS、aws-で始まるもの）

出力は構造化されたJSONファイルに保存し、エラーハンドリングを含めてください。
各リソースの取得時にプログレス表示を行い、リージョンも明示してください。
```

Claude Codeが生成したスクリプト：

```bash
#!/bin/bash

set -euo pipefail

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Function to print colored output
print_status() {
    echo -e "${GREEN}[INFO]${NC} $1"
}

print_warning() {
    echo -e "${YELLOW}[WARN]${NC} $1"
}

print_error() {
    echo -e "${RED}[ERROR]${NC} $1"
}

# Output file
OUTPUT_FILE="aws_resources_$(date +%Y%m%d_%H%M%S).json"

# Initialize output JSON
echo '{"timestamp": "'$(date -Iseconds)'", "resources": {}}' > "$OUTPUT_FILE"

# Function to add resource to JSON
add_to_json() {
    local resource_type="$1"
    local data="$2"

    # Use jq to add the resource data
    jq --arg type "$resource_type" --argjson data "$data" \
       '.resources[$type] = $data' "$OUTPUT_FILE" > temp.json && mv temp.json "$OUTPUT_FILE"
}

print_status "Starting AWS resource discovery..."

# Get current region
REGION=$(aws configure get region || echo "us-east-1")
print_status "Using region: $REGION"

# 1. EC2 Instances (exclude terminated)
print_status "Fetching EC2 instances..."
EC2_DATA=$(aws ec2 describe-instances \
    --query 'Reservations[].Instances[?State.Name!=`terminated`].{
        InstanceId: InstanceId,
        InstanceType: InstanceType,
        State: State.Name,
        LaunchTime: LaunchTime,
        Tags: Tags[?Key==`Name`].Value | [0],
        VpcId: VpcId,
        SubnetId: SubnetId,
        SecurityGroups: SecurityGroups[].GroupId
    }' \
    --output json 2>/dev/null || echo '[]')
add_to_json "ec2_instances" "$EC2_DATA"

# 2. RDS Instances
print_status "Fetching RDS instances..."
RDS_DATA=$(aws rds describe-db-instances \
    --query 'DBInstances[].{
        DBInstanceIdentifier: DBInstanceIdentifier,
        DBInstanceClass: DBInstanceClass,
        Engine: Engine,
        DBInstanceStatus: DBInstanceStatus,
        AllocatedStorage: AllocatedStorage,
        VpcSecurityGroups: VpcSecurityGroups[].VpcSecurityGroupId,
        DBSubnetGroup: DBSubnetGroup.DBSubnetGroupName
    }' \
    --output json 2>/dev/null || echo '[]')
add_to_json "rds_instances" "$RDS_DATA"

# 3. S3 Buckets with versioning info
print_status "Fetching S3 buckets..."
S3_BUCKETS=$(aws s3api list-buckets --query 'Buckets[].Name' --output text 2>/dev/null || echo "")
S3_DATA='[]'
if [[ -n "$S3_BUCKETS" ]]; then
    S3_ARRAY='[]'
    for bucket in $S3_BUCKETS; do
        VERSIONING=$(aws s3api get-bucket-versioning --bucket "$bucket" 2>/dev/null || echo '{}')
        BUCKET_INFO=$(jq -n \
            --arg name "$bucket" \
            --argjson versioning "$VERSIONING" \
            '{name: $name, versioning: $versioning}')
        S3_ARRAY=$(echo "$S3_ARRAY" | jq ". + [$BUCKET_INFO]")
    done
    S3_DATA="$S3_ARRAY"
fi
add_to_json "s3_buckets" "$S3_DATA"

# ... 他のリソース取得処理（ECS、Lambda等）も同様に続く

print_status "Resource discovery completed!"
print_status "Output saved to: $OUTPUT_FILE"

# Display summary
echo ""
echo "=== Resource Summary ==="
jq -r '
.resources | to_entries[] |
"\(.key): \(.value | length) items"
' "$OUTPUT_FILE"
```

### 2. Terraform状態との比較スクリプト

次に、取得したAWSリソースとTerraform状態を比較するための分析スクリプトを作成するプロンプトです。

```
先ほど取得したAWSリソースのJSONファイルと、Terraform stateファイルを比較して、
以下を分析するPythonスクリプトを作成してください：

1. Terraform管理されていないAWSリソースの特定
2. リソースタイプ別の管理状況の可視化
3. インポート候補リソースのリスト生成
4. Terraform import blocksの自動生成

入力：
- aws_resources_YYYYMMDD_HHMMSS.json （AWSリソース一覧）
- terraform.tfstate （S3から取得したTerraform状態ファイル）

出力：
- 管理状況レポート（テキスト・テーブル形式）
- インポート候補リスト（CSV形式）
- 自動生成されたTerraform import blocks

分析対象リソース：
- aws_instance
- aws_db_instance
- aws_s3_bucket
- aws_iam_role
- aws_security_group
- aws_lb
- aws_vpc
- aws_subnet

エラーハンドリングとプログレス表示を含めてください。
```

Claude Codeが生成した比較分析スクリプト：

```python
#!/usr/bin/env python3

import json
import pandas as pd
import argparse
import sys
from pathlib import Path
from tabulate import tabulate
from datetime import datetime

class TerraformAWSAnalyzer:
    def __init__(self, aws_data_file, tfstate_file):
        self.aws_data_file = Path(aws_data_file)
        self.tfstate_file = Path(tfstate_file)
        self.aws_resources = {}
        self.tf_resources = {}
        self.analysis_results = {}

    def load_aws_data(self):
        """AWSリソースデータを読み込み"""
        print("📋 Loading AWS resources data...")
        try:
            with open(self.aws_data_file, 'r') as f:
                data = json.load(f)
                self.aws_resources = data.get('resources', {})
            print(f"✅ Loaded AWS data from {self.aws_data_file}")
        except FileNotFoundError:
            print(f"❌ AWS data file not found: {self.aws_data_file}")
            sys.exit(1)
        except json.JSONDecodeError as e:
            print(f"❌ Invalid JSON in AWS data file: {e}")
            sys.exit(1)

    def load_terraform_state(self):
        """Terraform状態ファイルを読み込み"""
        print("🏗️  Loading Terraform state...")
        try:
            with open(self.tfstate_file, 'r') as f:
                tfstate = json.load(f)

            # Terraform管理リソースを抽出
            self.tf_resources = {
                'aws_instance': [],
                'aws_db_instance': [],
                'aws_s3_bucket': [],
                'aws_iam_role': [],
                'aws_security_group': [],
                'aws_lb': [],
                'aws_vpc': [],
                'aws_subnet': []
            }

            for resource in tfstate.get('resources', []):
                resource_type = resource.get('type')
                if resource_type in self.tf_resources:
                    for instance in resource.get('instances', []):
                        attributes = instance.get('attributes', {})
                        self.tf_resources[resource_type].append({
                            'name': resource.get('name'),
                            'id': self._get_resource_id(resource_type, attributes),
                            'attributes': attributes
                        })

            print(f"✅ Loaded Terraform state from {self.tfstate_file}")
        except FileNotFoundError:
            print(f"❌ Terraform state file not found: {self.tfstate_file}")
            sys.exit(1)
        except json.JSONDecodeError as e:
            print(f"❌ Invalid JSON in Terraform state file: {e}")
            sys.exit(1)

    def _get_resource_id(self, resource_type, attributes):
        """リソースタイプに応じて適切なIDを取得"""
        id_mappings = {
            'aws_instance': 'id',
            'aws_db_instance': 'id',
            'aws_s3_bucket': 'bucket',
            'aws_iam_role': 'name',
            'aws_security_group': 'id',
            'aws_lb': 'arn',
            'aws_vpc': 'id',
            'aws_subnet': 'id'
        }
        return attributes.get(id_mappings.get(resource_type, 'id'))

・・・省略
```

このスクリプトにより、Terraform管理状況を詳細に分析し、インポート候補を特定できます。

### 3. 実行例と結果の確認
実際にこんな感じでリソースを取得して、分析してみました。

```bash
# 1. AWSリソースの取得
$ ./get_aws_resources.sh
Current region: ap-northeast-1
Continue? (y/n): y
[2025-09-23 10:30:45] Getting EC2 instances...
[2025-09-23 10:30:47] Getting VPCs...
...
Resource collection completed!
Summary:
--------
  ec2_instances: 15 resources
  rds_instances: 3 resources
  s3_buckets: 42 resources
  ...
```

# 2. 比較分析の実行
```
$ python3 analyze_terraform_coverage.py aws_resources_20240115_103045.json terraform.tfstate
📋 Loading AWS resources data...
✅ Loaded AWS data from aws_resources_20240115_103045.json
🏗️  Loading Terraform state...
✅ Loaded Terraform state from terraform.tfstate
🔍 Analyzing resource coverage...
✅ Analysis completed!

============================================================
📊 TERRAFORM COVERAGE REPORT
============================================================

🌐 Overall Coverage:
   Total AWS Resources: 75
   Terraform Managed: 67 (89.3%)
   Unmanaged: 8 (10.7%)

📋 Resource Type Breakdown:
┌─────────────────┬───────┬─────────┬───────────┬──────────┐
│ Resource Type   │ Total │ Managed │ Unmanaged │ Coverage │
├─────────────────┼───────┼─────────┼───────────┼──────────┤
│ Ec2 Instances   │    15 │      12 │         3 │    80.0% │
│ Rds Instances   │     3 │       2 │         1 │    66.7% │
│ S3 Buckets      │    42 │      38 │         4 │    90.5% │
│ Security Groups │    15 │      15 │         0 │   100.0% │
└─────────────────┴───────┴─────────┴───────────┴──────────┘

📄 Generating import candidates list...
✅ Import candidates saved to: import_candidates_20240115_103045.csv
   Total candidates: 8

⚡ Generating Terraform import blocks...
✅ Import blocks saved to: import_blocks_20240115_103045.tf
   Total import blocks: 8

🎉 Analysis completed! Check output files in: .
```
# 3. 生成されたファイルの確認
```
$ ls -la
import_candidates_20250923_103045.csv      # インポート候補リスト
import_blocks_20250923_103045.tf           # Terraform import blocks

$ head -10 import_blocks_20240115_103045.tf
# Auto-generated Terraform import blocks
# Generated on: 2024-01-15T10:30:45

# Import ec2_instances: web-server-01
import {
  to = aws_instance.instance_imported_001
  id = "i-1234567890abcdef0"
}

# Import rds_instances: prod-mysql
import {
  to = aws_db_instance.db_instance_imported_002
  id = "prod-mysql-instance"
}
```

# 4. GitHub ActionsでTerraform planを実行
import blocksとresource definitionsをリポジトリにコミット後、
GitHub ActionsでTerraform applyを実行します。


## Claude Codeを使う際のTips

### 効果的なプロンプトの書き方

1. **明確な要件定義**
   ```
   良い例：
   「EC2インスタンスを取得する際、Name タグ、VPC ID、サブネットID、
   セキュリティグループIDも含めて構造化されたJSONで出力してください」
   
   悪い例：
   「EC2の情報を取得してください」
   ```

2. **エラーハンドリングの明示**
   ```
   「各API呼び出しでエラーが発生した場合は、
   エラーメッセージをログに記録し、空の配列を返すようにしてください」
   ```

3. **出力形式の指定**
   ```
   「プログレスバーまたは進捗状況を表示し、
   最後にサマリーテーブルを表示してください」
   ```

### 段階的な実装アプローチ

1. **初期バージョン**: 基本的な機能のみ実装
2. **エラーハンドリング追加**: 例外処理とログ出力
3. **最適化**: ページネーション、並列処理
4. **ユーザビリティ向上**: 進捗表示、カラー出力、レポート生成

## まとめ

Claude Codeを活用することで、以下のメリットが得られます：

- **作業の自動化**: 手動での棚卸し作業を大幅に削減
- **ミスの削減**: スクリプト化により人的ミスを防止
- **再現性**: 同じ手順を複数の環境で実行可能
- **安全性**: ReadOnly権限と事前確認により事故を防止
- **可視性の向上**: レポート生成により管理状態を明確化

重要なのは、生成されたスクリプトを盲信せず、必ず内容を理解してから実行することです。また、本番環境での実行前に、必ず開発環境でのテストを行いましょう。

SREとして、既存リソースのIaC化はよくある課題かなと思います。Claude CodeのようなAIツールを効果的に活用することで、効率良く作業出来ると思います。