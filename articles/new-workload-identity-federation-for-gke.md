---
title: "新しくなった Workload Identity Federation for GKE を試してみる"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['GKE', 'WorkloadIdentity','IAM', 'ServiceAccount', 'GoogleCloud']
published: true
---
# はじめに
GKE の Workload Identity 連携（Workload Identity Federation for GKE）がアップデートされたということで、早速試してみました。

https://cloud.google.com/kubernetes-engine/docs/concepts/workload-identity#configure-workloads

# なにが変わったのか

これまで、GKE で Workload Identity 連携を使う場合、Google Cloud の Service Account（以降：GSA） を Kubernetes の Service Account（以降：KSA） に紐づけ、権限借用をすることでアクセスする必要がありました。

下記の表にもあるように、GSAが必要なくなり、KSAを直接 bind できるようになり、両者の ServiceAccount の連携が不要となりました！

![alt text](/images/new-workload-identity-federation-for-gke/GJ4BfE-agAAvnNR.jpeg)

【引用元】
- https://twitter.com/_techcet_/status/1773865010651173293
- https://twitter.com/_techcet_/status/1773865012320440390

# External Secrets Operator を使用して、Secret を取得する時で検証してみる
従来のやり方は以下の記事を参考にして下さい。
（External Secrets Operator の導入や使い方についてはここでは割愛します）

https://zenn.dev/yokoo_an209/articles/external-secret-operator


## Step1 : Secret Manager に secret を作成
Secret Manager に、`test-secret`という key で`hogehoge` という value を設定します

![alt text](</images/new-workload-identity-federation-for-gke/screenshot.png>)

## Step2 : ESO 展開
GKE の Workload Identity連携を有効にする（Autopilot モードだとデフォルトで有効になっています。）
https://cloud.google.com/kubernetes-engine/docs/how-to/workload-identity?hl=ja#enable


ESO 関連リソースを展開する下記マニフェストを使用します。

:::details マニフェスト例 

```yaml:workload-identify-for-gke-with-secret.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-secrets-serviceaccount
  labels:
    app: test-application

---
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: test-application-secretstore
  labels:
    app: test-application
spec:
  provider:
    gcpsm:
      projectID: "your_projectID"
      auth:
        workloadIdentity:
          clusterLocation: "your_clusterLocation"
          clusterName: "your_clusterName"
          serviceAccountRef:
            name: external-secrets-serviceaccount

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: test-application-externalsecret
  labels:
    app: test-application
spec:
  refreshInterval: 1m
  secretStoreRef:
    kind: SecretStore
    name: test-application-secretstore
  target:
    name: secret-store
  data:
    - remoteRef:
        key: test-secret
      secretKey: test-secret-key
```
:::


続いて上記マニフェストを使用して、ESO の helm chart の展開と、マニフェストの apply を行います。

```bash
# install ESO
helm repo add external-secrets https://charts.external-secrets.io

helm install external-secrets \
   external-secrets/external-secrets \
    -n external-secrets \
    --create-namespace

# apply manifest
$ k apply -f workload-identify-for-gke.yaml
serviceaccount/external-secrets-serviceaccount created
secretstore.external-secrets.io/test-application-secretstore created
externalsecret.external-secrets.io/test-application-externalsecret created
```


## Step3 : KSA bind と Secret の取得
KSA を Google Cloud の IAM Policy に binding します。
`[PROJECT_NUMBER]`, `[PROJECT_ID]`には各々に合わせた形で修正してください

```bash
# policy binding
gcloud secrets add-iam-policy-binding test-secret \
  --member=principal://iam.googleapis.com/projects/[PROJECT_NUMBER]/locations/global/workloadIdentityPools/[PROJECT_ID].svc.id.goog/subject/ns/default/sa/external-secrets-serviceaccount \
  --role="roles/secretmanager.secretAccessor"

# 実行結果
Updated IAM policy for secret [test-secret].
bindings:
- members:
  - principal://iam.googleapis.com/projects/[PROJECT_NUMBER]/locations/global/workloadIdentityPools/[PROJECT_ID].svc.id.goog/subject/ns/default/sa/external-secrets-serviceaccount
  role: roles/secretmanager.secretAccessor
etag: BwYU74IDDlw=
version: 1
```

secret の中身を確認すると、無事にsecretの値（`hogehoge`）が取得できていることがわかります。
```bash
# secret の中身を確認
$ k get secrets secret-store -o jsonpath="{.data.test-secret-key}" | base64 -d
hogehoge%
```

# まとめ
今回は Workload Identity 連携を通して、secret の取得を例に扱いましたが、もちろん他にも Google Cloud のリソースにアクセスする際にも利用できます。

このアップデートにより、annotationの設定やGSAの管理の手間が削減され、よりわかりやすく柔軟に GKE の Workload Idintity 連携が実現できるようになりました😊


# おまけ
気になったので、KSA に紐づいている Google Cloud のpolicy はどこで確認できるのか調査してみました

```bash
# Kubernetes Service AccountとGoogle Cloud Service Accountの紐付けを確認
$ k describe sa external-secrets-serviceaccount
Name:                external-secrets-serviceaccount
Namespace:           default
Labels:              app=test-application
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   <none>
Tokens:              <none>
Events:              <none>
```

`Annotations` で紐づいているわけではなさそう🧐

GKE メタデータサーバーから

<details><summary> マニフェスト例 </summary></summary>

```yaml:test-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: workload-identity-test
  namespace: default
spec:
  containers:
  - image: google/cloud-sdk:slim
    name: workload-identity-test
    command: ["sleep","infinity"]
  serviceAccountName: external-secrets-serviceaccount
  nodeSelector:
    iam.gke.io/gke-metadata-server-enabled: "true"
```
</details>

```bash
# pod に入って確認
k exec -it workload-identity-test -- /bin/bash

# メタデータサーバーから取得
curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/

# KSA に紐づいている GSA の scopes を確認
curl -H "Metadata-Flavor: Google" http://169.254.169.254/computeMetadata/v1/instance/service-accounts/test-yokoo.svc.id.goog/scopes
```