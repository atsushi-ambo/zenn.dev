---
title: "VSCode Reclineを使ってAWS SDK for Pythonを書かせてみた"
emoji: "🚀"
type: "tech"
topics: ["aws", "vscode", "recline", "python"]
published: true
---

# VSCode Reclineを使ってAWS SDKを実装してみた

今回は、話題のAIエージェント「VSCode Recline」を使ってAWS SDK for Python (boto3)のコードを書いてみました。

[Recline](https://github.com/julesmons/recline)は[Cline](https://github.com/cline/cline)をフォークしたものでVSCode内でGitHub Copilotを利用することが出来る拡張機能です。

Reclineを使うことで、GitHub Copilotがコードの提案や補完だけでなく、さらに高度なタスクをこなせるようになります。例えば、「この関数のテストコードを書いて」と指示するだけで、GitHub Copilotが自動的にテストコードを生成してくれます。

Reclineの大きな特徴の一つに、VS Code Language Model APIを選択することで、追加のAPIコストがかからないという点があります。これは、ReclineがGitHub Copilotの機能を拡張する形で動作するため、GitHub Copilotのサブスクリプション料金だけでReclineの機能も利用できます。

## 前提条件
- Python 3.6以上
- AWS CLI設定済み
- VS Code + Reclineインストール済み
## 実施にコードを書いてみる
現在、私はAWSのコスト削減に取り組んでいます。
そこで不要なリソースを見つけるために今回はアタッチされていないEBSボリューム一覧を出力するコードを書いてもらいました。

### 指示内容
下記のような簡単な指示をしてみました。日本語でも英語でも対応可能です。
```
describe EBS volumes not associated with any resources using AWS SDK for python and the profile is "****"
```
Reclineが自分でファイルを作成し、保存して良いか聞いてきたので承認しました。
コードを承認したら実行して良いか聞いてきたので、承認すると実行まで完了してくれました。
※profile名はマスク化してあります。

```python: describe_unattached_volumes.py
import boto3
import sys
from datetime import datetime

def describe_unattached_volumes():
    try:
        # Create session with specified profile
        session = boto3.Session(profile_name='****')
        ec2 = session.client('ec2')
        
        # Get all volumes
        response = ec2.describe_volumes(
            Filters=[
                {
                    'Name': 'status',
                    'Values': ['available']  # 'available' means not attached
                }
            ]
        )
        
        volumes = response['Volumes']
        if not volumes:
            print("No unattached EBS volumes found.")
            return
            
        print(f"\nFound {len(volumes)} unattached EBS volume(s):")
        print("-" * 80)
        
        for volume in volumes:
            # Convert creation time to readable format
            creation_time = volume['CreateTime'].strftime("%Y-%m-%d %H:%M:%S")
            
            print(f"Volume ID: {volume['VolumeId']}")
            print(f"Size: {volume['Size']} GiB")
            print(f"Volume Type: {volume['VolumeType']}")
            print(f"Creation Time: {creation_time}")
            print(f"Availability Zone: {volume['AvailabilityZone']}")
            print(f"State: {volume['State']}")
            
            # Print tags if they exist
            if 'Tags' in volume:
                print("Tags:")
                for tag in volume['Tags']:
                    print(f"  - {tag['Key']}: {tag['Value']}")
            
            print("-" * 80)

    except Exception as e:
        print(f"Error: {str(e)}")
        sys.exit(1)

if __name__ == "__main__":
    describe_unattached_volumes()
```
出力結果はこんな感じです。
```
% python3 describe_unattached_volumes.py 

Found 18 unattached EBS volume(s):
--------------------------------------------------------------------------------
Volume ID: vol-0****
Size: 30 GiB
Volume Type: gp3
Creation Time: 2019-09-06 07:28:30
Availability Zone: ap-northeast-1a
State: available
--------------------------------------------------------------------------------
Volume ID: vol-0fc***
Size: 8 GiB
Volume Type: gp2
Creation Time: 2025-01-30 10:01:05
Availability Zone: ap-northeast-1a
State: available
--------------------------------------------------------------------------------
Volume ID: vol-02**********
Size: 8 GiB
Volume Type: gp2
Creation Time: 2025-01-30 10:01:05
Availability Zone: ap-northeast-1a
State: available
--------------------------------------------------------------------------------
・・・・・・・省略
```

## まとめ
今回は、VSCode Reclineを使ってAWS SDK for Python (boto3)のコードを書いて、実行してもらいました。
このくらいの簡単なタスクならAIで出来るので本当にコードは使い捨てになったなと感じています。
Reclineはファイル作成、エラーが出たら修正、実行までしてくれるので非常に便利ですね。
GitHub Copilotをサブスクしている人は是非使ってみてください。

