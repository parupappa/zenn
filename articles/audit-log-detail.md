---
title: "【Google Cloud Audit logs】_Required バケットに書き込みできるのか"
emoji: "👻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Google Cloud', 'Audit Log']
published: true
---
# はじめに
Google Cloud の監査ログの設計を行なっていく中で、_Required バケットに書き込みできるのかを調査しました。

背景としては、クラウドセキュリティ的な観点から、監査ログ（特に、[データアクセス監査ログ](https://cloud.google.com/logging/docs/audit/configure-data-access?hl=ja)）に対するログの書き込み権限を制御できるのかという要望があったためです。


Google Cloud の監査ログとはなんぞや？という方は、以下の記事を参照してください。すごく詳しく載っているので、これだけ見ておけばおkです。

https://zenn.dev/nekoshita/articles/9fdfec20ed122b


# 結論
監査ログ用のログバケット（ _Required バケット）に対する、ログの書き込みは、ログの書き込み権限が付与されていても不可能

_Required バケットに対する制限は以下を参照

https://cloud.google.com/logging/docs/routing/overview?hl=ja#required-bucket

https://cloud.google.com/logging/docs/audit?hl=ja#storing_and_routing_audit_logs


# 作業ログ
検証方法：gcloud cli にてログを作成する

以下のコマンドを実行して、ログエントリを作成する


```bash
$ gcloud logging write test-log "write test log"                                                       
Created log entry.

$ gcloud logging write --payload-type=json my-test-log '{ "message": "My second entry", "weather": "partly cloudy"}'
Created log entry.
```

_Default バケットにログエントリが作成される


上記で作成されたログは、監査ログ用の _Required バケットへシンクされるフィルターにマッチしないため、 _Required バケットへのログエントリは作成されない

_Required バケットへシンクされるフィルターは以下（設定の変更はできない）

```bash
LOG_ID("cloudaudit.googleapis.com/activity") OR LOG_ID("externalaudit.googleapis.com/activity") OR LOG_ID("cloudaudit.googleapis.com/system_event") OR LOG_ID("externalaudit.googleapis.com/system_event") OR LOG_ID("cloudaudit.googleapis.com/access_transparency") OR LOG_ID("externalaudit.googleapis.com/access_transparency")
```

作成する LOG_ID に手を加えて、監査用のログバケットにログエントリを作成することができないか確認

```bash
$ gcloud logging write --payload-type=json cloudaudit.googleapis.com/activity '{ "message": "My second entry", "weather": "partly cloudy"}'

ERROR: (gcloud.logging.write) PERMISSION_DENIED: User not authorized. Only Google Cloud Platform services can write audit logs.
```

権限エラーでログエントリを作成できないことを確認

`cloudaudit.googleapis.com` の エンドポイントへの権限を google  にて制御していることがわかる

例えば LOG_ID を `cloudaudit.googleapis.com` に変更すると、権限エラーは出なくなり、ログエントリを作成することができる。

しかし、監査ログ用のバケットへシンクされるフィルタリング条件にはマッチしないので、 _Default バケットへルーティングされる


```bash
$ gcloud logging write --payload-type=json cloudaudit.googleapis.co/activity/test '{ "message": "My second entry", "weather": "partly cloudy"}'
Created log entry.
```
