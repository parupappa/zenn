---
title: "Github ActionsでCross ProjectのDirect Workload Identity Federationを試す"
emoji: "🧬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Google Cloud', 'Workload Identity', 'Github Actions', 'Terraform']
published: false
---

# はじめに

現在、Workload Identity Federation の認証は以下2種類があります。

- **Direct Workload Identity Federation**（推奨、本記事の内容）
- **Workload Identity Federation through a Service Account**（従来の方法）

今回は、Github Actions上で、**Direct Workload Identity Federation** を使用して Google Cloud の Cross Project（プロジェクトをまたぎ）でのアクセスが可能か検証します。

# Direct Workload Identity Federation の概要

Workload Identityは、OIDC (OpenID Connect) を用いて期限付きのトークンを発行することで、セキュアに**認証と認可**の両方を行い、外部IDに対して任意のサービスアカウントになりすますことができるロールを付与する仕組みです。

Workload Identityは外部IDを管理しているエンティティです。

dev、stg、prodなど環境が分かれている場合は、外部IDに対してより細かな権限制御を行うために複数のプールを作成することが推奨されます。

https://cloud.google.com/iam/docs/workload-identity-federation?hl=ja#pools


現在、[Direct Workload Identity](https://cloud.google.com/iam/docs/workload-identity-federation?hl=ja#direct-resource-access) に対応しているリソースには制限があります。
対応しているリソースの一覧は[こちら](https://cloud.google.com/iam/docs/federated-identity-supported-services?hl=ja#list)

では対応していない場合どうするかというと、従来の[**Workload Identity Federation through a Service Account**](https://cloud.google.com/iam/docs/workload-identity-federation?hl=ja#direct-resource-access) が推奨されます


両者の違いについて、詳しい説明は[こちら](https://paper2.hatenablog.com/entry/2024/06/29/143947#Direct-Workload-Identity-Federation%E3%81%A8%E3%81%AF)を見てもらえたらと思います



## Direct Workload Identity Federation をシングルプロジェクトで試す
[@paper2parasol](https://x.com/paper2parasol)さんの書かれている、以下記事がとても参考になるので、基本的にこれを試してもらえれば良いです

https://paper2.hatenablog.com/entry/2024/06/29/143947


## Direct Workload Identity Federation を Cross Project（プロジェクト跨ぎ）で試す
さて、今回の本題となる Cross Project での Direct Workload Identity Federation について説明します

今回実現したいこととなった前提は以下です。

【前提】
- tfstate 保管用のプロジェクトとして、`manegement` というプロジェクトを作成し、state分割をせず、一元管理とする
- `management` プロジェクトには `tfstate-bucket` という GCS があり、tfstate が保管されている
- リソースの展開先は、`dev` というプロジェクトがあり、ここにWorkload Identity Pool をはじめとするリソースを作成する
  - リソースを展開するプロジェクトは、dev, stg, prd など複数あるとします


以下の図にあるように、Github Actions から `dev` のWorkload Identity Federation を利用して、`management` プロジェクトの GCS（`tfstate-bucket`） にアクセスすることを目指します

![alt text](/images/direct-workloadidentity-federation/cross-project.png)


### Terraform での設定
**`variable.tf` に resouece 作成に必要なvariableを用意します。**
`tfstate_project_id` を追加し、state参照先プロジェクトを指定します.

リソースの参照・作成する先のプロジェクトは、provider と resource 側で設定することとします

```hcl
# Projectに関するvariable
variable "tfstate_project_id" {
  type    = string
  default = "management"
}
variable "project_id" {
  type    = string
  default = "dev"
}

variable "github_org" {
  type    = string
  default = "<YOUR_GITHUB_ORG>"
}

variable "repo_name" {
  type    = string
  default = "<YOUR_REPO_NAME>"
}
```

**Workload Identity Poolを作成します**
```hcl
# Workload Identity
resource "google_iam_workload_identity_pool" "github_actions_pool" {
  provider                  = google-beta
  project                   = var.project_id
  workload_identity_pool_id = "github-actions-pool"
  display_name              = "github-actions-pool"
  description               = "for github actions workflows"
}
```

**Workload Identity Pool Provider を作成します**

github_org には、Github Organization を指定します
`attribute-condition`オプションでは、Workload Identityの利用をgithub_orgで指定したOrganizationに制限しています

```hcl
resource "google_iam_workload_identity_pool_provider" "provider" {
  project                   = var.project_id
  workload_identity_pool_id = google_iam_workload_identity_pool.github_actions_pool.workload_identity_pool_id

  workload_identity_pool_provider_id = "github-provider"
  display_name                       = "github-provider"
  description                        = "for github actions workflows"

  attribute_mapping = {
    "google.subject"             = "assertion.sub"
    "attribute.repository"       = "assertion.repository"
    "attribute.repository_owner" = "assertion.repository_owner"
  }
  attribute_condition = "assertion.repository_owner == '${var.github_org}'"

  oidc {
    issuer_uri = "https://token.actions.githubusercontent.com"
  }
}
```

**Workload Identity Poolに権限を付与します**

ポイントは以下で、プリンシパルセットに設定するアクセスをCross Projectの参照先（今回だと、tfstateの存在するプロジェクト：`management`）とすることです

```hcl
# tfstate に対するアクセス権限を付与
resource "google_project_iam_member" "the_principal_tfstate" {
  for_each = toset([
    "roles/storage.objectViewer",
  ])
  project = var.tfstate_project_id
  role    = each.value
  member  = "principalSet://iam.googleapis.com/${google_iam_workload_identity_pool.github_actions_pool.name}/attribute.repository/${var.github_org}/${var.repo_name}"
}
```

そうすることで、参照先プロジェクトに対して、Workload Identity のプリンシパルセットをプリンシパルとするIAMアクセスが作成されます

作成した `main.tf` ファイル全体は以下のようになっています

:::details main.tf
```hcl
# Workload Identity
resource "google_iam_workload_identity_pool" "github_actions_pool" {
  provider                  = google-beta
  project                   = var.project_id
  workload_identity_pool_id = "github-actions-pool"
  display_name              = "github-actions-pool"
  description               = "for github actions workflows"
}

resource "google_iam_workload_identity_pool_provider" "provider" {
  project                   = var.project_id
  workload_identity_pool_id = google_iam_workload_identity_pool.github_actions_pool.workload_identity_pool_id

  workload_identity_pool_provider_id = "github-provider"
  display_name                       = "github-provider"
  description                        = "for github actions workflows"

  attribute_mapping = {
    "google.subject"             = "assertion.sub"
    "attribute.repository"       = "assertion.repository"
    "attribute.repository_owner" = "assertion.repository_owner"
  }
  attribute_condition = "assertion.repository_owner == '${var.github_org}'"

  oidc {
    issuer_uri = "https://token.actions.githubusercontent.com"
  }
}

# Workload Identity プリンシパルに対するアクセス権限を付与（Resource の作成・更新・削除）
resource "google_project_iam_member" "the_principal" {
  for_each = toset([
    "roles/editor",
  ])
  project = var.project_id
  role    = each.value
  member  = "principalSet://iam.googleapis.com/${google_iam_workload_identity_pool.github_actions_pool.name}/attribute.repository/${var.github_org}/${var.repo_name}"
}

# tfstate に対するアクセス権限を付与
resource "google_project_iam_member" "the_principal_tfstate" {
  for_each = toset([
    "roles/storage.objectViewer",
  ])
  project = var.tfstate_project_id
  role    = each.value
  member  = "principalSet://iam.googleapis.com/${google_iam_workload_identity_pool.github_actions_pool.name}/attribute.repository/${var.github_org}/${var.repo_name}"
}

terraform {
  required_version = "1.9.5"
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 6.0.1"
    }
    google-beta = {
      source  = "hashicorp/google-beta"
      version = "~> 6.0.1"
    }
  }
  backend "gcs" {
    bucket = "tfstate-bucket"
    prefix = "dev"
  }
}

provider "google" {
  project = var.tfstate_project_id
  region  = var.region
}

provider "google-beta" {
  project = var.tfstate_project_id
  region  = var.region
}

```
:::

次に、Github Actionsのワークフローを作成していきます。

認証に必要なProvider の情報を以下で取得します。
ここでは認証先のPROJECT_IDとなるので、今回だと`dev` プロジェクトのIDを指定します
```bash
$ gcloud iam workload-identity-pools providers describe "github-pool" \
  --project="${PROJECT_ID}" \
  --location="global" \
  --workload-identity-pool="github-pool" \
  --format="value(name)"

=> projects/<PROJECT_ID>/locations/global/workloadIdentityPools/github-pool/providers/github-provider
```

取得した情報を以下のように設定します
```bash
- id: "auth"
uses: "google-github-actions/auth@v2"
with:
    project_id: "test-direct-workload-identity"
    workload_identity_provider: "projects/<PROJECT_ID>/locations/global/workloadIdentityPools/github-actions-pool/providers/github-provider"
```

ワークフローの設定ファイルは以下のようになります
```yaml:actions.yaml
name: Test Direct Workload Identity Federation
on:
  push:
    branches:
      - "main"

jobs:
  access-gcs:
    name: Access GCS
    runs-on: ubuntu-latest
    permissions:
      contents: "read"
      id-token: "write"

    steps:
      - uses: "actions/checkout@v4"

      - id: "auth"
        uses: "google-github-actions/auth@v2"
        with:
          project_id: "test-direct-workload-identity"
          workload_identity_provider: "projects/<PROJECT_ID>/locations/global/workloadIdentityPools/github-actions-pool/providers/github-provider"

      - name: "Set up Cloud SDK"
        uses: "google-github-actions/setup-gcloud@v2"

      - name: "Access GCS"
        run: gcloud storage ls --recursive gs://tfstate-bucket/**

```


ワークフローを実行すると、GCS にアクセスできることが確認できます

![alt text](/images/direct-workloadidentity-federation/result-github-actions.png)


これで、Cross Project の Direct Workload Identity Federation が実現できました

## gcloud コマンドで実現
今回は、設定をterraformで行いましたが、gcloud コマンドで実現する場合も同じで、[Workload Identity Poolに権限を付与する](https://paper2.hatenablog.com/entry/2024/06/29/143947#Workload-Identity-Pool%E3%81%AB%E6%A8%A9%E9%99%90%E3%82%92%E4%BB%98%E4%B8%8E%E3%81%99%E3%82%8B)で権限を付与する部分で、Cross Project の参照先プロジェクトに対して権限を付与することで実現できます。

```bash
# --project="${CROSS_PROJECT_ID}" \の部分を参照先プロジェクトに変更
$ gcloud secrets add-iam-policy-binding "my-secret" \
  --project="${CROSS_PROJECT_ID}" \
  --role="roles/secretmanager.secretAccessor" \
  --member="principalSet://iam.googleapis.com/${WORKLOAD_IDENTITY_POOL_ID}/attribute.repository/${REPO}"

```


# まとめ

GitHub ActionsからGoogle Cloudへの認証に利用するgoogle-github-actions/authのDirect Workload Identity Federationについて terraform を利用するやり方で紹介しました。

Google も推奨と言っていることですし、Direct Workload Identity Federationの利用をまずは基本路線として、対応していないリソースがあれば、従来のWorkload Identity Federation through a Service Accountを利用すると良いでしょう。


# 参考

https://zenn.dev/amazyra/articles/workloadidentityfederation

https://zenn.dev/cloud_ace/articles/7fe428ac4f25c8

https://zenn.dev/optimind/articles/googlecloud-githubactions-terraform-cicd