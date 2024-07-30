---
title: ""
emoji: "🗂"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# GKE のメトリクス
ここの表に書いてあるのはデフォルトで取れるメトリクス
これ以外のメトリクスを取得する場合は kube-state-metrics を使う


https://cloud.google.com/kubernetes-engine/docs/how-to/kube-state-metrics?hl=ja


ステップ 2: デフォルトのメトリクス収集
GKE Autopilot では、標準的な Kubernetes メトリクスが自動的に収集され、Managed Service for Prometheus に送信されます。これには、ノード、Pod、コンテナのリソース使用状況（CPU、メモリーなど）や、クラスターの健全性に関連するメトリクスが含まれます。

ステップ 3: 追加のメトリクス収集
kube-state-metrics や他のカスタムエクスポーターからのメトリクスを収集したい場合は、それらをクラスターにデプロイし、Managed Service for Prometheus で収集するための設定を行う必要があります。Managed Service for Prometheus では、Prometheus のサービスディスカバリ機能を使って、これらの追加のメトリクスソースを自動的に検出し、収集することが可能です。



# kube state metrics 
- metrics server と kube-state-metrics はそもそも違う


- kube-state-metricsは異なるnamespaceのメトリクスを取れないのか？
  - kube-state-metricsはKubernetesクラスター内のオブジェクト（Pods、Nodes、Deploymentsなど）の状態をメトリクスとして公開するサービスです。このサービスはクラスターレベルのリソースを監視し、どのネームスペースにデプロイされていても、クラスター内のすべてのネームスペースのオブジェクトに関するメトリクスを収集します。

https://medium.com/@seifeddinerajhi/monitoring-kubernetes-clusters-with-kube-state-metrics-2b9e73a67895