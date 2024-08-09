---
title: "Google Cloud Next Tokyo’24 勝手にRecap コンテナ最新アップデート紹介 ~ Cloud Run 編 ~"
emoji: "📑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['CloudRun', 'GoogleCloud', 'GKE' ]
published: true
---
# これはなに？

先日、Google Cloud Next Tokyo’24が開催されました。

[Google Cloud Next Tokyo ’24](https://cloudonair.withgoogle.com/events/next-tokyo-24) の セッションを（勝手に）Recap したものになります

前回の GKE 編に引き続き、今回は Cloud Runについての最新アップデートのご紹介です！


# セッション

**進化するコンテナ環境: Google Kubernetes Engine と Cloud Run の最新アップデートと使い所を徹底解説**

https://cloudonair.withgoogle.com/events/next-tokyo-24?talk=d1-app-03

# 内容
![alt text](/images/google-next-recap-cloud-run/cloud-run.png)

今回は、みんな大好き Cloud Run についての最新アップデートのご紹介です！

Cloud Run のアップデートは、[Google Cloud Next ’24 in Las Vegas](https://cloud.withgoogle.com/next) のセッション[『Cloud Run: What's new』](https://cloud.withgoogle.com/next/session-library?session=DEV205#all) で結構述べられているものが多かったです。

詳しい解説はこちらに載っているので、この記事では、Google Next Tokyo’24でのセッション内容をさらっと紹介するに留めます。

https://qiita.com/AoTo0330/items/3e47dfbbcfe8b38591e1#multi-region-services

また、Cloud Run のもはや辞書と言っても良いぐらい素敵な技術書が [@msy78](https://x.com/msy78)さん、[@HorseVictory](https://x.com/HorseVictory)さんより出版されていますので、気になる方はぜひ読んでみてください！（控えめに言って、本当に神本です）

https://techbookfest.org/product/jQP5REJPx6pwMFEyx8QHPN?productVariantID=qGHnmQt1X84CDs89pCeuJ

## Direct VPC Egress
**🖊️公式doc**
https://cloud.google.com/run/docs/configuring/vpc-direct-vpc?hl=ja

![alt text](/images/google-next-recap-cloud-run/direct-vpc-egress.png)

- すでに使っている方も多いと思いますが、Cloud Run から VPC に直接アクセスできるようになりました。これにより、VPC のリソースに直接アクセスできるようになりました
- これまでの Cloud Run は、VPC にアクセスするためには、Serverless VPC Access Connector を利用する必要がありました
  - Serverless VPC Access Connector は、Cloud Run と VPC の間に中継するためのもので、Serverless とはいいつつ、インスタンスのサイジングが必要だったり、Cloud Run と VPC 内の Cloud SQL のネットワーク遅延の問題があったりと、いくつかの課題がありました


- せっかくなのでもう少し詳しく、違いを見てみましょう

    | 機能 | Direct VPC Egress | Serverless VPC Access Connector |
    | --- | --- | --- |
    | レイテンシ | 低い | 高い |
    | スループット | 高い | 低い |
    | IP 割り振り | ほとんどの場合、より多くの IP アドレスを使用する | IP アドレスの使用が少なくなる |
    | 費用 | VM の追加料金はなし | VM の追加料金が発生 |
    | スケーリング速度 | [Cloud Run サービスでのインスタンスの自動スケーリング](https://cloud.google.com/run/docs/about-instance-autoscaling?hl=ja)は、新しい VPC ネットワーク インターフェースが作成されるときのトラフィックの急増で遅くなる | ネットワーク レイテンシは、VPC ネットワーク トラフィックが急増している間に、より多くのコネクタ インスタンスを作成することで発生 |
    | Google Cloud コンソール | サポート対象 | サポート対象 |
    | Google Cloud CLI | サポート対象 | サポート対象 |
    | リリース ステージ | GA（Cloud Run ジョブは除く） | GA |


  - 参考: [ダイレクト VPC 下り（外向き）と VPC コネクタの比較  |  Cloud Run Documentation  |  Google Cloud](https://cloud.google.com/run/docs/configuring/connecting-vpc?hl=ja)
  - 参考: [VPC ネットワークによるダイレクト VPC 下り（外向き）  |  Cloud Run Documentation  |  Google Cloud](https://cloud.google.com/run/docs/configuring/vpc-direct-vpc?hl=ja)


さらに詳しくは[こちら💡](https://qiita.com/AoTo0330/items/3e47dfbbcfe8b38591e1#volume-mounts)

## Volume Mount
**🖊️公式doc**
https://cloud.google.com/blog/ja/products/serverless/introducing-cloud-run-volume-mounts

![alt text](/images/google-next-recap-cloud-run/volume-mount.png)

- Cloud Run では共有ボリュームを利用することができず、用途としてはステートレスなワークロードに限定されていました。
- [サイドカーによるマルチコンテナ対応](https://cloud.google.com/run/docs/deploying?hl=ja#sidecars)と同時に、[インメモリボリューム](https://cloud.google.com/run/docs/configuring/services/in-memory-volume-mounts?hl=ja#command-line)の対応がされましたが、依然としてデータの永続化(大容量のデータを扱う場合やデータの信頼性が求められる場合)はファイル共有などを実装することが求められていました。

詳しくは[こちら💡](https://qiita.com/AoTo0330/items/3e47dfbbcfe8b38591e1#volume-mounts)


## App Hosting
**🖊️公式doc**
https://firebase.google.com/docs/hosting/cloud-run?hl=ja

![alt text](/images/google-next-recap-cloud-run/app-hosting.png)

- Firebase Hosting と Cloud Run を組み合わせることで、Web アプリケーションを簡単にデプロイできるようになりました
  - SSRに必要なリソースを自動プロビジョニングすることができるので、開発者は SSR の設定に時間をかけることなく、Web アプリケーションをデプロイできるようになりました

## Automatic Security Update
**🖊️公式doc**
https://cloud.google.com/container-optimized-os/docs/concepts/auto-update?hl=ja

![alt text](/images/google-next-recap-cloud-run/auto-security-update.png)


詳しくは[こちら💡](https://qiita.com/AoTo0330/items/3e47dfbbcfe8b38591e1#automatic-security-updates)



## Multi-Region Load Balancing
**🖊️公式doc**
https://cloud.google.com/run/docs/multiple-regions?hl=ja

![alt text](/images/google-next-recap-cloud-run/multi-region-lb.png)

- Cloud Run service をカスタムドメインを用いて公開する方法は、Global External Application Load Balancer の使用が一般的でしたが、Multi-Region Load Balancing を使用することで、複数のリージョンにまたがる Cloud Run service を公開することができるようになりました
    ```bash
    gcloud run deploy <SERVICE_NAME> \
    --region=><REGION1>,<REGION2>,<REGION3> \
    --domain=<CUSTOM_DOMAIN>
    ```

詳しくは[こちら💡](https://qiita.com/AoTo0330/items/3e47dfbbcfe8b38591e1#multi-region-services)

## Disable the default URL
**🖊️公式doc**
https://cloud.google.com/run/docs/securing/ingress?hl=en#disable-url

- 少し余談ですが、Cloud Run でデフォルト URL を無効化する機能が Preview 公開されました。
- 無効化すると Cloud Run service の起動は Cloud Load Balancing 経由のみに限定され、セキュリティ強化につながりますが、Preview版ということもあり、本番環境での運用はまだ推奨されていません。パブリックエンドポイントでアクセスする Pub/Sub や BigQuery リモート関数からの呼び出しも不可になるので注意がそこは注意が必要です。


## Application Canvas
**🖊️公式doc**
https://cloud.google.com/blog/ja/products/containers-kubernetes/google-clouds-container-platform-for-the-next-decade-of-ai

![alt text](/images/google-next-recap-cloud-run/application-canvas.png)

- Geminiを利用し、自然言語によってアプリケーションコンポーネントを構築することができるようになりました。
- Cloud Run と統合されている、Firestore, Cloud SQL, Memorystore などのサービスを利用することができます。


詳しくは[こちら💡](https://qiita.com/AoTo0330/items/3e47dfbbcfe8b38591e1#application-canvas)


# Cloud Run まとめ
![alt text](/images/google-next-recap-cloud-run/cloud-run-matome.png)

- Cloud Run の利用におけるかゆいところに手が届くようなアップデートが多く、開発者にとっては非常にありがたいアップデートが多かったです
  - サーバーレスであるCloud Runがよりサーバーレスになっていくとともに、Geminiとの統合により、より開発者の認知負荷や開発負荷を軽減することができるようになった印象です

