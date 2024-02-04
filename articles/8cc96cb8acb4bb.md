---
title: "Google CloudのObservability : Personalized Service Health"
emoji: "🤸"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['gc24', 'Google Cloud', 'Observability', 'Personalized Service Health']
published: false
---

# はじめに

先日、これまで[プレビュー版](https://cloud.google.com/blog/ja/products/management-tools/introducing-personalized-service-health)だった、Google Cloudの稼働監視サービスの**Personalized Service Health**がGAになりました！

[Personalized Service Health is now generally available | Google Cloud Blog](https://cloud.google.com/blog/products/devops-sre/personalized-service-health-is-now-generally-available/?hl=en)

基本的な使用方法や設定はこちらのブログにて詳しく解説されていますので、ここでは運用面の機能を中心にPersonalized Service Healthを見ていきます。

[Google Cloudの障害情報をプロジェクト単位でキャッチしたい!! Personalized Service Healthを使ってみた](https://zenn.dev/monicle/articles/4734ab6244d653)

# Google CloudのObservability

Google Cloudの各種サービスステータスは、Google Cloud Service Healthにて公開されています

[Google Cloud Service Health](https://status.cloud.google.com/)

一方、Personalized Service Healthでは、使用中である関連するプロダクトのインシデント、そのステータス、ロケーションなどの情報が提供される他、公開されていないインシデントに加え、Google Cloud Service Health からのインシデントも、インシデント対応状況の証跡も含めてカバーされます。

インシデントが発生すると、以下のようにダッシュボード上に警告**！**の表示がつきます。


![スクリーンショット 2024-02-04 11.25.44.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9061398-9d73-43e2-a89a-963f6a3bc2e8/39972fe0-af2a-49e5-ba4b-02c0b37243f4/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88_2024-02-04_11.25.44.png)

また Personalized Service Healthでは **Service Health event** と呼ばれるイベント情報がCloud LoggingにLoggingされます

インシデント管理ワークフローとして組み込み可能な以下のような手法があります。

- Dashboard
- Alert（Cloud Monitoring, Cloud Logging）
- API
    
    ![スクリーンショット 2024-02-04 11.27.06.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/b9061398-9d73-43e2-a89a-963f6a3bc2e8/bddc9657-3c29-4ab2-ace7-342ec4ae2b2a/%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88_2024-02-04_11.27.06.png)
    

[Personalized Service Health shows emerging incidents | Google Cloud Blog](https://cloud.google.com/blog/products/networking/personalized-service-health-shows-emerging-incidents?hl=en)

まさに、**Google Cloudの可観測性（Observability）** を提供してくれていると言って良いのではないでしょうか

# ログ集約

Personalized Service Healthはプロジェクトごとに有効化の設定を行うこともできますし、組織またはフォルダー内のすべてのプロジェクトに対してPersonalized Service Healthを適用することも可能です。

そのため、組織レベルで1つのプロジェクトに対してLog Sinkを行うことでログを集約し分析ワークロードを組むことも可能です。

[Enable Personalized Service Health for all projects in an organization or folder  |  Google Cloud](https://cloud.google.com/service-health/docs/script-enable-api)

# Terraform対応

Personalized Service Health固有のresource定義については現在のところ特にありません

アラートの設定は、[Cloud Monitoringのアラートポリシー](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/monitoring_alert_policy)を使用する形で実装できます

`PROJECT_ID` , `NOTIFICATION_CHANNEL`は自身の環境に合わせて修正してください

フィルターを変更することで、インデントアラートに対するスコープを組織レベルからプロジェクトレベルまで変更できます（ここでは、すべてのインシデントとそれに伴うアップデートを通知する例を示します）

```hcl

# Enable Personalized Service Health API
resource "google_project_service" "project" {
  service = "servicehealth.googleapis.com"
}

# Alert Policy for Personalized Service Health
resource "google_monitoring_alert_policy" "alert_policy_all" {
  display_name = "All incidents, all updates"
  combiner     = "OR"
  enabled      = "true"
  conditions {
    display_name = "test condition"
    condition_matched_log {
      filter     = "resource.type = \"servicehealth.googleapis.com/Event\" AND jsonPayload.category = \"INCIDENT\" AND jsonPayload.@type = \"type.googleapis.com/google.cloud.servicehealth.logging.v1.EventLog\""
      label_extractors = {
          state = "EXTRACT(jsonPayload.state)"
          description = "EXTRACT(jsonPayload.description)"
          impactedProducts = "EXTRACT(jsonPayload.impactedProducts)"
          startTime = "EXTRACT(jsonPayload.startTime)"
          title = "EXTRACT(jsonPayload.title)"
          impactedLocations = "EXTRACT(jsonPayload.impactedLocations)"
      }
    }
  }

  documentation {
    content = "### $${log.extracted_label.title}\nCheck out [Personalized Service Health dashboard](https://console.cloud.google.com/servicehealth/eventDetails/projects%2F$${resource.labels.resource_container}%2Flocations%2F$${resource.labels.location}%2Fevents%2F$${resource.labels.event_id}) for more details.<br><br>  Description: $${log.extracted_label.description}<br><br>  Impacted products: $${log.extracted_label.impactedProducts}<br><br> Impacted locations: $${log.extracted_label.impactedLocations}<br><br>  Incident start time: $${log.extracted_label.startTime}<br><br>  State: $${log.extracted_label.state}"
    mime_type = "GoogleCloud稼働監視"
  }
  alert_strategy {
    notification_rate_limit {
      period = "300s"
    }
  }

  notification_channels =  ["projects/PROJECT_ID/notificationChannels/NOTIFICATION_CHANNEL"]

  user_labels = {  # Add any extra labels that might be helpful
    scope = "all"
  }
}
```

https://cloud.google.com/service-health/docs/configure-alerts-terraform

個人的には、認証周りをterraformにて管理するのは運用負荷が上がってしまうため、通知チャンネルは管理コンソールから手動で追加し、以下のようdataで取得するようにしたい派です。

以下は、通知チャンネルにemailを設定した例です

```hcl

data "google_monitoring_uptimecheck_config" "email"{
	display_name = "your-email-display-name"
}

notification_channels =  [
	data.google_monitoring_uptimecheck_config.email.id
]
```

[Configure alerts in Terraform  |  Personalized Service Health  |  Google Cloud](https://cloud.google.com/service-health/docs/configure-alerts-terraform)

user_labelを設定することで、Alert Policyごとに制御範囲を任意で識別することも可能です

```hcl
user_labeles = { "severity" : "info" }
```

# まとめ

Personalized Service Healthを用いることで、Google Cloud上でのサービスの健全性をより詳細に、かつ柔軟に監視・管理することができるようになります。

また、Cloud MonitoringやCloud Loggingと統合されたインシデントワークフローの構築により、インシデント発生時の迅速な対応と詳細な容易な管理が可能になります。

使っていこう！Personalized Service Health🥳