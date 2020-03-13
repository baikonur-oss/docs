<p align="center">
  <br>
  <img width="800" src="./images/logo_l_black@4x.png" alt="logo of baikonur/docs repository">
  <br>
  <br>
</p>


# Baikonur プロジェクトホームページ

Baikonurへようこそ！

Baikonurは、[株式会社サイバーエージェント](https://www.cyberagent.co.jp/) によるオープンソースの[Terraform Module](https://www.terraform.io/docs/modules/usage.html) プロジェクトです。

Baikonurは2018年に社内プロジェクトとしてスタートし、すぐに50モジュールを超えました。最も人気のある、ユニークなモジュールをここでリリースしています。

Baikonur プロジェクトの名前は、人類初の人工衛星の打ち上げと人類初の有人宇宙飛行で知られるカザフスタンにある宇宙港である[バイコヌール宇宙基地](https://ja.wikipedia.org/wiki/バイコヌール宇宙基地) に由来します。 <!-- _ -->

ここでリリースされているモジュールは、Terraform Public Module Registryの[Baikonur トップページ](https://registry.terraform.io/modules/baikonur-oss)にも公開されています。

## 使い方
各リポジトリのREADMEまたは[Terraform Module Registry のページ](https://registry.terraform.io/modules/baikonur-oss)に従ってください。リポジトリでの使用例を参照し、適切な [バージョン指定](https://www.terraform.io/docs/configuration/modules.html#module-versions) をご利用ください。

## モジュールとツール

- AWS
  - [iam-nofile](https://github.com/baikonur-oss/terraform-aws-iam-nofile) - AWS IAMロールの作成をより簡単にするためのモジュール。インライン (heredoc) 構文でロールポリシーを記述できるため、[テンプレートレンダリング](https://www.terraform.io/docs/providers/template/d/file.html) を使用することなくポリシー内で変数を使うことが可能です。

  - ECSモジュール
    - [fargate-scheduled-task](https://github.com/baikonur-oss/terraform-aws-fargate-scheduled-task) - CloudWatch Eventsを活用してECSでのタスクの定期実行を簡単に構築するためのモジュール。

  - KinesisとLambdaを活用したロギングのためのモジュール
    - [lambda-kinesis-to-fluent](https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-fluent) - Kinesis Data Streamsから Fluentエンドポイントへの転送 <!-- -->
    - [lambda-kinesis-to-s3](https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-s3) - Kinesis Data StreamsからS3への保存 <!-- -->
    - [lambda-kinesis-to-es](https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-es) - Kinesis Data Streamsから Elasticsearchへの格納 <!-- --> 
    - [lambda-es-cleaner](https://github.com/baikonur-oss/terraform-aws-lambda-es-cleaner) - Elasticsearch Serviceの古いインデックスを自動的に消すためのツール
    - [lambda-kinesis-forward](https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-forward) - Kinesis Data Streamsから Kinesis Data Streamsへの転送、ログのルーティング <!-- -->

  - eden (ECS Dynamic Environment Manager、ECS動的開発環境マネージャ)
    - [aws-eden-cli](https://github.com/baikonur-oss/aws-eden-cli) - eden CLI
    - [lambda-eden-api](https://github.com/baikonur-oss/terraform-aws-lambda-eden-api) - eden API (Lambda)
    - [aws-eden-core](https://github.com/baikonur-oss/aws-eden-core) - eden core package (shared between CLI and API)


- GCP
  - 近日公開


## コントリビューション
baikonur-ossオーガニゼーション下配下のすべてのリポジトリへのプルリクエスト、イシューやその他のフィードバック大歓迎です! :smile: 

Baikonurに寄付したいモジュールがある場合は、このリポジトリのイシューでお知らせください。

<!-- メンターとコントリビューター募集中! -->

## ライセンス

このオーガニゼーション配下のすべてのリポジトリは、MITライセンスで提供されます（各リポジトリのLICENSEファイルを参照）。

BaikonurのロゴはCC0ライセンスで提供しています。
