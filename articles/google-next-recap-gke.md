---
title: "Google Cloud Next Tokyo’24 勝手にRecap コンテナ最新アップデート紹介 ~ GKE 編 ~"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['GKE', 'GoogleCloud', 'CloudRun' ]
published: false
---

# これはなに？

先日、Google Cloud Next Tokyo’24が開催されました。

Google Cloud Next Tokyo’24 の セッションを（勝手に）Recap したものになります

[Google Cloud Next Tokyo ’24](https://cloudonair.withgoogle.com/events/next-tokyo-24)

基調講演のアーカイブも公開されているので、見逃した方はぜひご覧ください

https://cloudonair.withgoogle.com/events/next-tokyo-24

気になる箇所だけ見たい方は、以下の目次から飛んでください！

- [これはなに？](#これはなに)
- [セッション](#セッション)
- [内容](#内容)
- [GKE](#gke)
  - [Workload Identity Federation for GKE](#workload-identity-federation-for-gke)
  - [Pod Bursting](#pod-bursting)
  - [Stateful HA Operator](#stateful-ha-operator)
  - [Secret Manager add-on with Google Kubernetes Engine](#secret-manager-add-on-with-google-kubernetes-engine)
  - [GCS FUSE Read Cache Support](#gcs-fuse-read-cache-support)
  - [Container Image Preloading](#container-image-preloading)
  - [Cluster-wide Network Policy](#cluster-wide-network-policy)
  - [GKE Enterprise](#gke-enterprise)
    - [GKE Threat Detection](#gke-threat-detection)
    - [GKE Compliance](#gke-compliance)
  - [Cloud Service Mesh（CSM）](#cloud-service-meshcsm)
  - [GKE 延長サポート](#gke-延長サポート)
  - [GKE まとめ](#gke-まとめ)
  - [さいごに - GKE編](#さいごに---gke編)
- [Cloud Run](#cloud-run)
- [さいごに](#さいごに)


# セッション

**進化するコンテナ環境: Google Kubernetes Engine と Cloud Run の最新アップデートと使い所を徹底解説**

- https://cloudonair.withgoogle.com/events/next-tokyo-24?talk=d1-app-03

# 内容

![alt text](</images/google-next-recap-gke/Google Cloud Next Tokyo 24 Recap.png>)


# GKE

![IMG_6478.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/920b1966-ee9b-4756-aa6a-edbc81eea72f.png)

まずは、お馴染みGKEについての説明です

Speaker の[@kkuchima](https://x.com/kkuchima) さんが、過去に登壇されている資料も大変有益なので参考として掲載します。

https://lp.cloudplatformonline.com/rs/808-GJW-314/images/GoogleCloud_Session_UchimaKazuki_CloudNativeDaysTokyo2022.pdf

私も過去にAutopilotに焦点を当てて、話しているので、気になれば見てみてください🥹

https://speakerdeck.com/parupappa2929/xun-su-nixie-eru-gke-autopilotniyoruyunibasarumodanakitekutiyanoshi-jian

## Workload Identity Federation for GKE

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**
https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity?hl=ja#configure-workloads

</aside>

![IMG_6479.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/IMG_6479.jpg)

これまで、GKE のWorkload Identity 連携は Google Cloud の Service Account（以降：SA）とKubernetesのSAをそれぞれ作成し、bindingする形式でしたが、Google Cloud のSAを作らずに Kubernetes のSAを直接メンバーとして権限を紐づけられるようになりました

特にこれをterraform で記述する際はけっこうめんどくさかったので、個人的には結構嬉しいアップデートです！

Cynthia Thomas-sanからの発表コメント

- [https://twitter.com/](https://twitter.com/)*techcet*/status/1773865010651173293
- [https://twitter.com/](https://twitter.com/)*techcet*/status/1773865012320440390

試してみた記事を書いたので、もしよければ🙋‍♂️

- https://zenn.dev/yokoo_an209/articles/new-workload-identity-federation-for-gke

## Pod Bursting

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/kubernetes-engine/docs/how-to/pod-bursting-gke

</aside>

![IMG_6481.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/IMG_6481.jpg)

![IMG_6482.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/IMG_6482.jpg)

![IMG_6483.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/3d39b829-84cc-4001-b557-3f747515280f.png)

- **Pod bursting in GKE**
    - release note はこちら
        - [https://cloud.google.com/kubernetes-engine/docs/release-notes#April_02_2024](https://cloud.google.com/kubernetes-engine/docs/release-notes#April_02_2024)
        - [https://cloud.google.com/blog/products/containers-kubernetes/introducing-gke-autopilot-burstable-workloads?hl=en](https://cloud.google.com/blog/products/containers-kubernetes/introducing-gke-autopilot-burstable-workloads?hl=en)
- Autopilot でこの機能をサポートするクラスタでは Pod を作成する際の最小リソース設定値も更新されています。
    - [https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-resource-requests?hl=ja#min-max-requests](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-resource-requests?hl=ja#min-max-requests)
    - 例 : Spot と組み合わせて Pod を作ると、**約月額 $0.7~** で利用可能
        - このため、お試しで使用したい場合や、小リソースで済みミニマムコストでGKE上で運用していきたい場合にまず使ってみるとより低価格でGKE を利用することができるようになると思います。
        
        ```yaml
        cat <<EOF | kubectl apply -f -
        apiVersion: v1
        kind: Pod
        metadata:
          name: hello
        spec:
          nodeSelector:
            cloud.google.com/gke-spot: "true"
          containers:
          - name: app
            image: us-docker.pkg.dev/google-samples/containers/gke/hello-app:2.0
            resources:
              requests:
                cpu: 50m
                memory: 52Mi
                ephemeral-storage: 10Mi
              limits:
                cpu: 250m
        ```
        
- 試してみた系の記事だとこちらも参考になります
    - https://blog.g-gen.co.jp/entry/gke-burstable-pod

## Stateful HA Operator

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/kubernetes-engine/docs/how-to/stateful-ha?hl=ja

</aside>

![IMG_6484.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/61a0d6fb-7fc2-4a94-ac98-49c3e38fe3b1.png)

- Stateful High Availability（HA）Operator を使用することで、GKE の組み込みの[リージョン Persistent Disk](https://cloud.google.com/compute/docs/disks/high-availability-regional-persistent-disk?hl=ja) との統合を使用して、[StatefulSet](https://cloud.google.com/kubernetes-engine/docs/concepts/statefulset?hl=ja) Pod のフェイルオーバーの速度を自動化および制御できるようになります。
- これまでは、マルチゾーンでのHA構成を実現する場合、マルチノードでステートフル状態をディスクで管理する場合、ノードに異常が発生した場合、同じディスクにノードがつなぐまでに8~10分ほど時間がかかっていました
- 一方、マルチレプリカ構成を組む場合、レプリケーションにより少し高額になってしまう状態でした。今回の構成により、安価で接続不可の時間を低減してフェイルオーバーが可能になります。

## Secret Manager add-on with Google Kubernetes Engine

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/secret-manager/docs/secret-manager-managed-csi-component

</aside>

![IMG_6485.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/efed5f4b-41ac-4af4-bfea-371edc6452f7.png)

- Google CloudのManaged な Secret Manager に対するCSI Driver となります。
    - これにより、CSIとしてマウントして参照する形となるため、これまでstoreしていた kubernetes 上の secrete store 使わなくて良くなる
    - Pod からマウントするときは、前述の Workload Identity Federation for GKE を使用することでマウント可能となる
- GKE 1.29以降（Preview）
    - Preview 中は Pod にマウントした秘密情報を Secret に同期する機能は使えないそうです

## GCS FUSE Read Cache Support

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/storage/docs/gcsfuse-cache?hl=ja

</aside>

![IMG_6486.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/e798ad1f-62ab-4995-af83-6adc963a3de2.png)

- FUSEを使用して、GKE から GCS をマウントする際のオーバーヘッドが効率化
    - 都度、GCSに対するAPIアクセスが不要になり、Read Cache（Persistent Disk / Local SSD, メモリ）に保存することで、同じデータへの都度アクセスが不要になりました。
    - AI/ML ワークロードなどの大容量のマウントを必要とするユースケースに最適になりそうです

## Container Image Preloading

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/kubernetes-engine/docs/how-to/data-container-image-preloading

</aside>

![IMG_6487.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/IMG_6487.jpg)

- Use secondary boot disks to preload data or container images
    - GKE Node Pool にセカンダリディスクを追加する機能
        - 事前にロードされたコンテナ イメージまたはセカンダリ ブート ディスク上のデータを使うことで、ワークロードをより迅速に起動できます
        - これによりコンテナイメージやデータを Node 上に Preload し、AI/MLワークロードなどサイズの大きいコンテナの立ち上げ速度が向上します。
        - ノード プールにセカンダリ ブート ディスクを追加しても、プロビジョニング時間は増加しません
        - CI/CD パイプラインで ディスク イメージの作成を自動化 することもできます
            - https://github.com/GoogleCloudPlatform/ai-on-gke/tree/main/tools/gke-disk-image-builder#cloud-build
        - GKE Standard では 1.28.3-gke.1067000、GKE Autopilot では 1.30.1-gke.1329000 以降で利用できます
- これまでも、[image streming](https://cloud.google.com/kubernetes-engine/docs/how-to/image-streaming?hl=ja#enable_on_clusters) を使用することで、image pull にかかる時間の短縮をすることはできましたが、Image Preloading ではそれ以上に大きな効果となっています。

[GKE-image-preloading.pdf](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/GKE-image-preloading.pdf)

## Cluster-wide Network Policy

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/kubernetes-engine/docs/how-to/configure-cilium-network-policy?hl=ja

</aside>

![IMG_6488.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/13f7a7d4-b664-45d7-a0f2-b1eb55406580.png)

- そもそも、NetworkPolicyとは、Podに対してPod間の通信や外部のエンドポイントへの通信を制御するためのリソースです
- 通常の、Namespace スコープであるNetwork Policyとは異なり、Cluster 全体に対する制御が可能となりました
- マニフェスト
    
    ```yaml
    apiVersion: "cilium.io/v2"
    kind: CiliumClusterwideNetworkPolicy
    metadata:
      name: "l4-rule-ingress"
    spec:
      endpointSelector:
        matchLabels:
          role: backend
      ingress:
        - fromEndpoints:
            - matchLabels:
                role: frontend
          toPorts:
            - ports:
                - port: "80"
                  protocol: TCP
    ```
    

## GKE Enterprise

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/kubernetes-engine/enterprise/docs/concepts/overview?hl=ja

</aside>

![IMG_6489.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/437a3356-bc71-4d68-a941-d6ab076b6846.png)

![IMG_6490.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/dd74a96e-50b0-4804-adcf-9b56c32d4f28.png)

- GKE Enterpriseに関するアップデートも最近活発になっております！
- そもそも、GKEには “モード” と “エディション” という棲み分けがあり、ややこしいが以下のようになっています
    - https://lp.cloudplatformonline.com/rs/808-GJW-314/images/GoogleCloud_mod0301_breakout-c5.pdf#page=21.00

![screenshot 2024-08-05 at 16.21.19.png](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/screenshot_2024-08-05_at_16.21.19.png)

### GKE Threat Detection

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/kubernetes-engine/docs/concepts/about-gke-threat-detection?hl=ja

</aside>

![IMG_6491.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/b45b9afc-8462-4037-8feb-e98237cb2ae2.png)

- `Preview` 機能にはなりますが、GKE Threat Detectionは、Google Cloud の KSPMである GKE Security Postureに統合された機能として提供されます
- GKE Enterprise だと追加料金なしで利用可能

### GKE Compliance

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/blog/ja/products/containers-kubernetes/gke-compliance-reports-on-cluster-and-workload-posture

</aside>

![IMG_6492.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/af2bce20-dacb-49a6-bfca-1b3185f8ae4d.png)

- こちらも`Preview` 機能にはなりますが、 GKE Complianceは、Google Cloud の KSPMである GKE Security Postureに統合された機能として提供されます。
- CISベンチマークをはじめとする、業界水準レベルのコンテナコンプライアンスに対応しているかどうかのアセスメントを効率的に行えるようになります。

## Cloud Service Mesh（CSM）

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**
https://cloud.google.com/blog/products/networking/introducing-cloud-service-mesh?hl=en

</aside>

![IMG_6493.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/5d9fb7c2-6cda-4437-9215-aaf68c081181.png)

- ちょっと前にアナウンスがありましたが、`Anthos Service Mesh`と`Traffic Director` が１つのプロダクトに統合され、横断的な管理や制御を行っていくことが容易になるようです。
- GKEがメインでしたが将来的にはCloud Runもサービスメッシュの中に入れられるようになるとのことです

![Cloud Service Mesh Part 1.png](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/Cloud_Service_Mesh_Part_1.png)

- Cloud Service Mesh についてはこちらがとても参考になるのでぜひぜひ🙋‍♂️

https://zenn.dev/t_hayashi/articles/ad115c602203da

## GKE 延長サポート

<aside>
<img src="https://www.notion.so/icons/bell-notification_gray.svg" alt="https://www.notion.so/icons/bell-notification_gray.svg" width="40px" /> **公式doc**

https://cloud.google.com/blog/ja/products/containers-kubernetes/announcing-gke-extended-support

</aside>

![IMG_6494.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/IMG_6494.jpg)

- 新たなリリースチャンネルとして、**`Extended` リリースチャンネル** が登場
- 標準サポート期間（14ヶ月） + 追加オプションとして10ヶ月のサポート期間を増やすことが可能になる

## GKE まとめ

![IMG_6503.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/ed7c2771-bcd3-41b9-b8a9-5bf098ceae98.png)

- AI / ML ワークロード用途として GKE を利用することが多くなってきたこともあり、それに寄り添ったアップデートが増えている印象がありました
- また、Autopilot の利用をより加速させるような寄り添ったアップデートも多くなり、ますますGKEの第一選択肢として Autopilot の選定が加速するような内容も多くなってきました
- GKE Enterprise によるガバナンスやコンプライアンス、セキュリティといったエンタープライズに求められる要素も今後さらに GKE と統合されたエコシステムで実現できる機運が高まってきそうです！（ワクワク）

## さいごに - GKE編

- 個人的に気になった GKE のアップグレードに関しては、ここにもまとめているので（抜け漏れあり）、もし興味あれば覗いてみてください👀

https://zenn.dev/yokoo_an209/scraps/1796702689a4d5

- Kubernetes全体に関することであれば、弊社のk8sマスターこと [@toVersus](https://x.com/toversus26) のtimesとZenn の Scrap をぜひご覧ください！！！

https://zenn.dev/toversus/scraps/06c7c9181ff3c1

# Cloud Run

![IMG_6495.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/c798384b-d5a4-4190-9fed-487485b2f592.png)

![IMG_6496.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/e922483a-42de-455a-8529-6e1c74b6afb9.png)

![IMG_6497.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/4c041761-3e27-4c3f-8e7f-358719e85ee6.png)

![IMG_6498.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/19268114-2880-44a3-8f0a-9eb732004c6b.png)

![IMG_6500.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/093650a5-e61f-43c2-a189-755e5e50deef.png)

![IMG_6501.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/080010dd-a6a4-4f97-ba49-6d59fd620797.png)

![IMG_6502.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/715d93ec-e407-4623-99cc-9a901a8f679c.png)

![IMG_6503.JPG](Google%20Cloud%20Next%20Tokyo%E2%80%9924%20%E5%8B%9D%E6%89%8B%E3%81%ABRecap%20~%20GKE%20Cloud%20Ru%2049f8445043b24fb08eae1f541ce4bf2d/2c28e819-a215-4811-a31e-00c393bbb690.png)

# さいごに

同じようにセッションレポートを上げてくださっている記事もあるのでそちらと合わせて、参照いただけるとより詳しい内容を追うことができるかともいます。

https://iret.media/112497

https://iret.media/111691