---
title: "Google Cloud Next Tokyo’24 勝手にRecap コンテナ最新アップデート紹介 ~ GKE 編 ~"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['GKE', 'GoogleCloud', 'CloudRun' ]
published: false
---

# これはなに？

先日、Google Cloud Next Tokyo’24が開催されました。

[Google Cloud Next Tokyo ’24](https://cloudonair.withgoogle.com/events/next-tokyo-24) の セッションを（勝手に）Recap したものになります

基調講演のアーカイブも公開されているので、見逃した方はぜひご覧ください

https://cloudonair.withgoogle.com/events/next-tokyo-24

気になる箇所だけ見たい方は、目次から飛んでください！


# セッション

**進化するコンテナ環境: Google Kubernetes Engine と Cloud Run の最新アップデートと使い所を徹底解説**

https://cloudonair.withgoogle.com/events/next-tokyo-24?talk=d1-app-03

# 内容

![alt text](/images/google-next-recap-gke/recap.png)


# GKE

![alt text](/images/google-next-recap-gke/gke.png)

まずは、お馴染みGKEについての説明です

Speaker の[@kkuchima](https://x.com/kkuchima) さんが、過去に登壇されている資料も大変有益なので参考として掲載します。

https://lp.cloudplatformonline.com/rs/808-GJW-314/images/GoogleCloud_Session_UchimaKazuki_CloudNativeDaysTokyo2022.pdf

私も過去にAutopilotに焦点を当てて、話しているので、気になれば見てみてください🥹

https://speakerdeck.com/parupappa2929/xun-su-nixie-eru-gke-autopilotniyoruyunibasarumodanakitekutiyanoshi-jian

## Workload Identity Federation for GKE

**🖊️公式doc**
https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity?hl=ja#configure-workloads

![alt text](/images/google-next-recap-gke/workload-identity-gke.jpg)

これまで、GKE のWorkload Identity 連携は Google Cloud の Service Account（以降：SA）とKubernetesのSAをそれぞれ作成し、bindingする形式でしたが、Google Cloud のSAを作らずに Kubernetes のSAを直接メンバーとして権限を紐づけられるようになりました

特にこれをterraform で記述する際はけっこうめんどくさかったので、個人的には結構嬉しいアップデートです！

Cynthia Thomas-sanからの発表コメント

- [https://twitter.com/](https://twitter.com/)*techcet*/status/1773865010651173293
- [https://twitter.com/](https://twitter.com/)*techcet*/status/1773865012320440390

試してみた記事を書いたので、もしよければ🙋‍♂️

- https://zenn.dev/yokoo_an209/articles/new-workload-identity-federation-for-gke

## Pod Bursting

**🖊️公式doc**
https://cloud.google.com/kubernetes-engine/docs/how-to/pod-bursting-gke

![alt text](/images/google-next-recap-gke/pod-bursting-0.jpg)

![alt text](/images/google-next-recap-gke/pod-bursting-1.png)

![alt text](/images/google-next-recap-gke/pod-bursting-2.jpg)


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

**🖊️公式doc**
https://cloud.google.com/kubernetes-engine/docs/how-to/stateful-ha?hl=ja


![alt text](/images/google-next-recap-gke/stateful-ha-operator.png)


- Stateful High Availability（HA）Operator を使用することで、GKE の組み込みの[リージョン Persistent Disk](https://cloud.google.com/compute/docs/disks/high-availability-regional-persistent-disk?hl=ja) との統合を使用して、[StatefulSet](https://cloud.google.com/kubernetes-engine/docs/concepts/statefulset?hl=ja) Pod のフェイルオーバーの速度を自動化および制御できるようになります。
- これまでは、マルチゾーンでのHA構成を実現する場合、マルチノードでステートフル状態をディスクで管理する場合、ノードに異常が発生した場合、同じディスクにノードがつなぐまでに8~10分ほど時間がかかっていました
- 一方、マルチレプリカ構成を組む場合、レプリケーションにより少し高額になってしまう状態でした。今回の構成により、安価で接続不可の時間を低減してフェイルオーバーが可能になります。

## Secret Manager add-on with Google Kubernetes Engine

**🖊️公式doc**
https://cloud.google.com/secret-manager/docs/secret-manager-managed-csi-component


![alt text](/images/google-next-recap-gke/secrete-manager-addon.png)

- Google CloudのManaged な Secret Manager に対するCSI Driver となります。
    - これにより、CSIとしてマウントして参照する形となるため、これまでstoreしていた kubernetes 上の secrete store 使わなくて良くなる
    - Pod からマウントするときは、前述の Workload Identity Federation for GKE を使用することでマウント可能となる
- GKE 1.29以降（Preview）
    - Preview 中は Pod にマウントした秘密情報を Secret に同期する機能は使えないそうです

## GCS FUSE Read Cache Support

**🖊️公式doc**
https://cloud.google.com/storage/docs/gcsfuse-cache?hl=ja

![alt text](/images/google-next-recap-gke/gcs-fuse-read-cache.png)

- FUSEを使用して、GKE から GCS をマウントする際のオーバーヘッドが効率化
    - 都度、GCSに対するAPIアクセスが不要になり、Read Cache（Persistent Disk / Local SSD, メモリ）に保存することで、同じデータへの都度アクセスが不要になりました。
    - AI/ML ワークロードなどの大容量のマウントを必要とするユースケースに最適になりそうです

## Container Image Preloading

**🖊️公式doc**
https://cloud.google.com/kubernetes-engine/docs/how-to/data-container-image-preloading


![alt text](/images/google-next-recap-gke/container-image-preloading.jpg)

- Use secondary boot disks to preload data or container images
    - GKE Node Pool にセカンダリディスクを追加する機能
        - 事前にロードされたコンテナ イメージまたはセカンダリ ブート ディスク上のデータを使うことで、ワークロードをより迅速に起動できます
        - これによりコンテナイメージやデータを Node 上に Preload し、AI/MLワークロードなどサイズの大きいコンテナの立ち上げ速度が向上します。
        - ノード プールにセカンダリ ブート ディスクを追加しても、プロビジョニング時間は増加しません
        - CI/CD パイプラインで ディスク イメージの作成を自動化 することもできます
            - https://github.com/GoogleCloudPlatform/ai-on-gke/tree/main/tools/gke-disk-image-builder#cloud-build
        - GKE Standard では 1.28.3-gke.1067000、GKE Autopilot では 1.30.1-gke.1329000 以降で利用できます
- これまでも、[image streming](https://cloud.google.com/kubernetes-engine/docs/how-to/image-streaming?hl=ja#enable_on_clusters) を使用することで、image pull にかかる時間の短縮をすることはできましたが、Image Preloading ではそれ以上に大きな効果となっています。



## Cluster-wide Network Policy

**🖊️公式doc**
https://cloud.google.com/kubernetes-engine/docs/how-to/configure-cilium-network-policy?hl=ja

![alt text](/images/google-next-recap-gke/cluster-wiede-network.png)


- そもそも、NetworkPolicyとは、Podに対してPod間の通信や外部のエンドポイントへの通信を制御するためのリソースです
- 通常の、Namespace スコープであるNetwork Policyとは異なり、Cluster 全体に対する制御が可能となりました

:::details マニフェスト
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
:::


## GKE Enterprise

**🖊️公式doc**
https://cloud.google.com/kubernetes-engine/enterprise/docs/concepts/overview?hl=ja


![alt text](/images/google-next-recap-gke/gke-enterprise-1.png)

![alt text](/images/google-next-recap-gke/gke-enterprise-all.png)

- GKE Enterpriseに関するアップデートも最近活発になっております！
- そもそも、GKEには “モード” と “エディション” という棲み分けがあり、ややこしいが以下のようになっています
    - https://lp.cloudplatformonline.com/rs/808-GJW-314/images/GoogleCloud_mod0301_breakout-c5.pdf#page=21.00

    ![alt text](/images/google-next-recap-gke/gke-enterprise-edtion.png)

### GKE Threat Detection

**🖊️公式doc**
https://cloud.google.com/kubernetes-engine/docs/concepts/about-gke-threat-detection?hl=ja


![alt text](/images/google-next-recap-gke/gke-threat-detection.png)

- `Preview` 機能にはなりますが、GKE Threat Detectionは、Google Cloud の KSPMである GKE Security Postureに統合された機能として提供されます
- GKE Enterprise だと追加料金なしで利用可能

### GKE Compliance
**🖊️公式doc**
https://cloud.google.com/blog/ja/products/containers-kubernetes/gke-compliance-reports-on-cluster-and-workload-posture

![alt text](/images/google-next-recap-gke/gke-compliance.png)

- こちらも`Preview` 機能にはなりますが、 GKE Complianceは、Google Cloud の KSPMである GKE Security Postureに統合された機能として提供されます。
- CISベンチマークをはじめとする、業界水準レベルのコンテナコンプライアンスに対応しているかどうかのアセスメントを効率的に行えるようになります。

## Cloud Service Mesh（CSM）

**🖊️公式doc**
https://cloud.google.com/blog/products/networking/introducing-cloud-service-mesh?hl=en


![alt text](/images/google-next-recap-gke/cloud-service-mesh.png)

- ちょっと前にアナウンスがありましたが、`Anthos Service Mesh`と`Traffic Director` が１つのプロダクトに統合され、横断的な管理や制御を行っていくことが容易になるようです。
- GKEがメインでしたが将来的にはCloud Runもサービスメッシュの中に入れられるようになるとのことです

![alt text](</images/google-next-recap-gke/Cloud Service Mesh Part 1.png>)

- Cloud Service Mesh についてはこちらがとても参考になるのでぜひぜひ🙋‍♂️

https://zenn.dev/t_hayashi/articles/ad115c602203da

## GKE 延長サポート
**🖊️公式doc**
https://cloud.google.com/blog/ja/products/containers-kubernetes/announcing-gke-extended-support

![alt text](/images/google-next-recap-gke/gke-extended.jpg)

- 新たなリリースチャンネルとして、**`Extended` リリースチャンネル** が登場
- 標準サポート期間（14ヶ月） + 追加オプションとして10ヶ月のサポート期間を増やすことが可能になる

## GKE まとめ

![alt text](/images/google-next-recap-gke/gke-matome.png)

- AI / ML ワークロード用途として GKE を利用することが多くなってきたこともあり、それに寄り添ったアップデートが増えている印象がありました
- また、Autopilot の利用をより加速させるような寄り添ったアップデートも多くなり、ますますGKEの第一選択肢として Autopilot の選定が加速するような内容も多くなってきました
- GKE Enterprise によるガバナンスやコンプライアンス、セキュリティといったエンタープライズに求められる要素も今後さらに GKE と統合されたエコシステムで実現できる機運が高まってきそうです！（ワクワク）

## さいごに - GKE編

- 個人的に気になった GKE のアップグレードに関しては、ここにもまとめているので（抜け漏れあり）、もし興味あれば覗いてみてください👀

https://zenn.dev/yokoo_an209/scraps/1796702689a4d5

- Kubernetes全体に関することであれば、弊社のk8sの神こと [@toVersus](https://x.com/toversus26) のZenn の Scrap をぜひご覧ください！！！

https://zenn.dev/toversus/scraps/06c7c9181ff3c1


また、同じようにセッションレポートを上げてくださっている記事もあるのでそちらと合わせて、参照いただけるとより詳しい内容を追うことができるかと思います。

https://iret.media/112497

https://iret.media/111691