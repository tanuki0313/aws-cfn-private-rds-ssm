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
- IAMロール
  - EC2にSSM接続用のAmazonSSMManagedInstanceCore権限を付与する設計

### ネットワーク
- EC2 Security Group
  - SSM、RDSに対しての接続設定
- RDS Security Group
  - 特定のEC2 SG が付いているインスタンスからの通信のみ許可に設定
- IGW
  - 作成したEC2が入っているVPCがSSMと接続できるよう、IGWを設定
- ルートテーブル
  - 上記作成したIGWとVPCを繋ぐルートテーブルを設定

### 設計・その他
- draw.io
  - 構成図（アーキテクチャ図）の作成
- GitHub
  - テンプレートおよび README の管理

## CloudFormation構成の説明
本テンプレートでは、CloudFormation を用いて 
VPC、サブネット、SG、SSM(Session Manager)、IGW、ルートテーブル、IAMロール、EC2、RDS（MySQL）の構築を自動化しています。

DB の認証情報は AWS Secrets Manager により管理し、テンプレート内に認証情報を平文で記載しない構成としています。

また、DB Subnet Group に複数 AZ のサブネットを指定し、MultiAZ: true を設定することで、高可用性を考慮した RDS 構成を実現しています。

スタックはネットワーク、セキュリティ、Secrets Manager、EC2、RDSをそれぞれ分けて作成することで、
どのスタックがどのような役割、処理をしているか判断できるように設計しております。

## 工夫・学習したポイント

・Parameters を活用することで、異なる環境でも再利用可能な CloudFormation テンプレート設計を意識しました。

・Multi-AZ を有効化し、障害発生時にスタンバイインスタンスへ自動フェイルオーバーされる高可用性構成を実装しました。

・DB の認証情報は AWS Secrets Manager で管理し、CloudFormation テンプレート内にパスワードを平文で記載しないことで、セキュリティを考慮した設計としました。

・Outputs を定義し、作成した RDS のエンドポイントや Secret ARN を即座に参照できるようにし、運用や他サービス連携を考慮した設計としました。

・ネットワーク、セキュリティ、Secrets Manager、EC2、RDSそれぞれの処理をスタック事に作成したことで一目でどのスタックが何の処理をしてるか明確にしました

・SSM経由でEC2に接続できるように設計したことにより、SSH接続よりセキュリティ面を考慮した設計としました。

・RDS SGで特定のEC2 SG が付いているインスタンスからの通信のみ許可することでセキュリティ面を考慮した設計としました。

## 開発中に直面した課題と解決策

### 問題1
EC2サブネットにIGWおよびルートテーブルが適切に関連付けられていなかったため、
外部接続が確立できなかった。

→ ルートテーブルおよびIGWアタッチをCloudFormationに追加。

### 問題2
EC2のSecurityGroupアウトバウンドにRDS接続（TCP 3306）が未定義だったため、
RDSへ接続不可となった。

→ EC2 SecurityGroupに3306アウトバウンドを追加。
