---
title: "Terraform / GKE で実現する ExternalSecretOperator テンプレート"
emoji: "🔑"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['GoogleCloud', 'GKE', 'Terraform', 'Helm', 'ExternalSecretOperator']
published: true
---
# はじめに

GKEのSecretの管理に External Secret Operator を利用することがありました。


備忘も兼ねて、周辺知識も補足しながら、今後の使い回しのできるようにTerraform / GKEでのExternal Secret Operatorのテンプレートを作成しました。

【前提】
- Cluster : GKE Autopilot 1.27
  - GKEのWorkload Identityは有効化されている
- マニフェスト管理
  - helm : helmfile
- IaC : Terraform

今回は、Service Account キーの使用ではなく、Workload Identityを利用しています。
基本的な流れはこちらの公式docに詳しく書いてあるので、参考にしてください。

https://cloud.google.com/kubernetes-engine/docs/tutorials/workload-identity-secrets?hl=ja


# Terraform

```hcl
variable "roles" {
  default = ["roles/secretmanager.secretAccessor", "roles/iam.serviceAccountTokenCreator", "roles/cloudkms.cryptoKeyEncrypterDecrypter"]
}
```

Service Account に対して必要な権限を定義します。

GKEでのアプリケーションレイヤでのSecretの暗号化が有効になっている場合は、Cloud KMSへのアクセスが必要なため、`roles/cloudkms.cryptoKeyEncrypterDecrypter`を追加します。

https://cloud.google.com/kubernetes-engine/docs/how-to/encrypting-secrets?hl=ja


また、アクセストークンの生成に必要なロール `roles/iam.serviceAccountTokenCreator` も付与します。



```hcl
# Service Account(ExternalSecretsOperator用ユーザ)
resource "google_service_account" "external_secrets" {
  account_id   = "${var.prefix}-${var.service}-eso-${var.env}"
  display_name = "External Secrets Operator Service Account"
}

# Role Binding(ESOへSecretsを読み込む権限を)
resource "google_project_iam_binding" "secret-pull" {
  for_each = toset(var.roles)

  project = var.project_id
  role    = each.key

  members = [
    "serviceAccount:${google_service_account.external_secrets.email}",
  ]
}

# GKEに対して、WorkloadIdentityUserを付与する
resource "google_service_account_iam_member" "external_secrets_workload_identity_user" {
  service_account_id = google_service_account.external_secrets.name
  role               = "roles/iam.workloadIdentityUser"
  member             = "serviceAccount:${var.project_id}.svc.id.goog[default/external-secrets-serviceaccount]"
}
```
`google_project_iam_binding`に対するRoleのバインドは1つしかできないため、for_eachを利用して複数のRoleをバインドします。

`member`の値には、kubernetes serviceaccount（KSA）の存在するNamespsaceと、KSAの名前を指定します。
ここでは、default Namespaceに`external-secrets-serviceaccount`という名前のKSAを作成しています。


# Manifest
環境差分がありそうな部分に関しては、`values.yaml`に定義していますので、参照する形で作成してください。

```yaml:serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-secrets-serviceaccount
  labels:
    app: test-application
  # ESOに対してSSM/SecretsManagerからSecretsを取得することを許可するServiceAccount(IAM Role Binding)
  {{ - with Values.serviceAccount.annotations }}
  annotations:
    {{to nindent 4 .}}
  {{- end }}
```
kubernetes の ServiceAccount を作成します。

```yaml:secretstore.yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: test-application-secretstore
  labels:
    app: test-application
spec:
  provider:
    gcpsm:
      projectID: {{ .Values.projectID}}
      # ref: https://external-secrets.io/v0.5.7/spec/#external-secrets.io/v1beta1.GCPSMAuth
      auth:
        workloadIdentity:
          clusterLocation: {{ .Values.clusterLocation }}
          clusterName: {{ .Values.clusterName }}
          serviceAccountRef:
            name: external-secrets-serviceaccount
```

今回は Namespace ごとの制御にしたいため、`kind: ClusterSecretStore` ではなく、`kind: SecretStore` を利用しています。

違いはこちらに詳しく書いてあります。

https://mixi-developers.mixi.co.jp/compare-eso-with-secret-csi-846ed8b1c9b

また、`serviceAccountRef` に対して Namespace を指定することはできないので、気をつけてください


```yaml:externalsecret.yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: test-application-externalsecret
  labels:
    app: test-application
spec:
  # 外部providerに対してデータ参照を行いSecretの値を更新する頻度
  refreshInterval: 1h
  # 該当データをどのように取得するかを定義したSecretStoreを指定する
  secretStoreRef:
    kind: SecretStore
    name: test-application-secretstore
  # ExternalSecretから作成される `kind: Secret` リソースの情報を定義する.ExternalSecretにつき1つだけ定義できる 
  target:
    name: secret-store
  # Secret Managerから取得するパラメータを定義する
  data:
  - remoteRef:
      key: foo
    secretKey: bar
```

```yaml:values.yaml
projectID: ${PROJECT_ID}
clusterName: ${CLUSTER_NAME}
clusterLocation: ${CLUSTER_LOCATION}
serviceAccount:
  annotations:
    iam.gke.io/gcp-service-account: ${PROJECT_ID}.svc.id.goog[NAMESPACE/KSA_NAME]
```

# 参考
- [Google Secret Manager と External Secrets Operator を連携して GKE で secret を自動生成](https://www.creationline.com/tech-blog/66988)

- [ExternalSecretsOperator(ESO)をEKSとGKEに導入して各SecretsManagerからシークレットを取得する](https://qiita.com/sokasanan/items/0011ed478c0a060539b8)

- [GKE の Secret 手動管理を脱却すべく External Secrets Operator を導入しました](https://medium.com/arigatobank-tech-blog/gke-%E3%81%AE-secret-%E6%89%8B%E5%8B%95%E7%AE%A1%E7%90%86%E3%82%92%E8%84%B1%E5%8D%B4%E3%81%99%E3%81%B9%E3%81%8F-external-secrets-operator-%E3%82%92%E5%B0%8E%E5%85%A5%E3%81%97%E3%81%BE%E3%81%97%E3%81%9F-7c7722c5e114)

- [External Secrets Operator ＋ GCP Secret Manager でSecretを管理する](https://qiita.com/scum/items/09d8187fcb5eee1618ba)

- [Kubernetes External Secrets が非推奨になるので External Secrets Operator と Secret Storage CSI を比較する](https://mixi-developers.mixi.co.jp/compare-eso-with-secret-csi-846ed8b1c9b)

- [[Kubernetes]External Secrets Operator入門](https://zenn.dev/nameless_gyoza/articles/external-secrets-operator)