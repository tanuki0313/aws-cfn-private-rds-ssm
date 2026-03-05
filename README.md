# aws-cfn-private-rds-ssm
CloudFormationを用いたEC2、RDS、SSMの自動構築プレート

## 概要（何を作ったか・目的）
CloudFormation を用いて、EC2、RDS,SSMを用いてSSMからEC2に接続しRDSを操作できる環境を自動構築するテンプレートを作成しました。

SSM(AWS Systems Manager)やIaC（Infrastructure as Code）の基礎理解に加え
VPC、サブネット、EC2、RDSのネットワーク設計およびセキュリティ設計の理解を目的としています。

## 構成 / アーキテクチャ

![構成図](images/Cfn_RDS_SSM.png)

## 検証時の構成
RDSをPrivate Subnetに配置し、SSM経由でEC2に接続しRDS操作ができる設計にしてます。

RDSはMulti-AZ構成としており、障害発生時には自動的にスタンバイAZへフェイルオーバーされます。

RDSのセキュリティグループで特定のEC2からの接続のみ許可しているのでセキュリティ面も考慮しています。

## 使用技術
### AWS
- Amazon VPC  
  - 新規作成
  - Public / Private Subnet を想定した設計
- Amazon RDS (MySQL)
  - MySQL 8.0
  - Multi-AZ 構成（フェイルオーバー対応）
  - DB Subnet Group を利用した AZ 分散
- Amazon EC2
  - RDSへの接続元として設計
- AWS Secrets Manager
  - RDS のマスターユーザー名 / パスワードを管理
- AWS CloudFormation
  - インフラをコードとして管理（IaC）
- AWS Systems Manager Session Manager
  - EC2への接続元として設計

### ネットワーク
- EC2 Security Group
  - SSM、RDSに対しての接続設定
  
- RDS Security Group
  - 特定のEC2 SG が付いているインスタンスからの通信のみ許可に設定

### 設計・その他
- draw.io
  - 構成図（アーキテクチャ図）の作成
- GitHub
  - テンプレートおよび README の管理

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
