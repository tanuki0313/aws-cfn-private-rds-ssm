# aws-cfn-private-rds-ssm
CloudFormationを用いたEC2、RDS、SSMの自動構築プレート

## 概要（何を作ったか・目的）


## 構成 / アーキテクチャ

![構成図](images/Cfn_RDS_SSM.png)

## 検証時の構成

## 使用技術

## CloudFormation構成の説明

## 工夫・学習したポイント

## 開発中に直面した課題と解決策

## 今後の改善 / 本番想定での構成

## 改善履歴

### 問題1
EC2サブネットにIGWおよびルートテーブルが適切に関連付けられていなかったため、
外部接続が確立できなかった。

→ ルートテーブルおよびIGWアタッチをCloudFormationに追加。

### 問題2
EC2のSecurityGroupアウトバウンドにRDS接続（TCP 3306）が未定義だったため、
RDSへ接続不可となった。

→ EC2 SecurityGroupに3306アウトバウンドを追加。
