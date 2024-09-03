---
title: "Github Actions で見る tfsec と Trivy の今 ~ terraform 静的解析"
emoji: "💧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Github Actions', 'tfsec', 'Trivy', 'terraform']
published: true
---

Github Actionsでtfsecを実装する際に、以下を発見して良いなと思ったので試してみた

https://tech.crassone.jp/posts/how_tfsec-pr-commenter-action_was_introduced_to_significantly_reduce_the_execution_time_of_terraform_security_scans

```yaml
# https://aquasecurity.github.io/tfsec/v1.0.11/getting-started/configuration/github-actions/pr-commenter/

name: Run tfsec PR commenter
run-name: tfsec PR commenter
on:
  pull_request:
    types: [synchronize, opened]
jobs:
  tfsec:
    runs-on: ubuntu-latest
    steps:
      - name: Clone repo
        uses: actions/checkout@master
      - name: Terraform security scan
        uses: tfsec/tfsec-pr-commenter-action@main
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}

```

しかし、最後のリリースは Nov 3, 2022 とあまり最近はメンテされてない様子だった

https://github.com/aquasecurity/tfsec-pr-commenter-action

# あれ、tfsec で Aqua に買収されてなかったけ？

そいういえば、tfsec ってAqua に買収されていなかったっけ？と思って調べてみると確かに買収されており、trivy ファミリーに加わっていた

https://github.com/aquasecurity/tfsec/discussions/1994

https://knqyf263.hatenablog.com/entry/2021/07/13/063729

tfsec 自体は最近もメンテはされている様子だが追従しているのかは正直わからない🧐
（どなたか知ってる方いたら教えていただきたいです）

リポジトリ内のREADME.mdの冒頭に「tfsec to Trivy Migration」という移行を促すアナウンスが追加されています（2023年8月31日）

https://github.com/aquasecurity/tfsec

# ということで tfsec から trivy に移行

https://zenn.dev/acntechjp/articles/9f0e3d4813e36d

[Overview - Trivy](https://aquasecurity.github.io/trivy/v0.54/)

migrate ガイドも出ている模様

[https://github.com/aquasecurity/tfsec/blob/master/tfsec-to-trivy-migration-guide.md](https://github.com/aquasecurity/tfsec/blob/master/tfsec-to-trivy-migration-guide.md)

# Trivy for terraform

もともとものモチベーションであったterraformの静的解析をtrivyで実施してみる

今回は、[tfsec-pr-commenter-action](https://github.com/aquasecurity/tfsec-pr-commenter-action)を使っていたこともあり、Github Actionsで実行する前提とする

terraform でのtrivy 利用に関する内容で、宇宙から配信したような音質で使い方の動画があった👽

https://www.youtube.com/watch?v=yf8JC-aNIV8

以下のような感じで、純粋な trivy でのコマンドをワークフローに組み込んでも良いが、公式が出しているActionsがあるのでそちらを使用する

```yaml
name: trivy-scan
run-name: trivy scan for terraform
on:
  pull_request:
    types: [synchronize, opened]

jobs:
    trivy_scan:
      steps: test
            - name: trivy
              uses: aquasec/trivy:latest
              run: | 
                trivy config --exit-code 1 --severity HIGH,CRITICAL .
```

ワークフローは公式が出しているこちらを実行

https://github.com/aquasecurity/trivy-action

ということで実装

```yaml
name: trivy-scan
run-name: trivy scan for terraform
on:
  pull_request:
    types: [synchronize, opened]

jobs:
  trivy_scan:
    name: Run Trivy Scan
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
      pull-requests: write
    steps:
      - name: Clone repo
        uses: actions/checkout@v4

      - name: Trivy Scan
        uses: aquasecurity/trivy-action@master
        with:
          scan-type: "config"
          severity: "HIGH,CRITICAL"
          scan-ref: "." # デフォルトのスキャン対象はカレントディレクトリ.ゆくゆくは変更のあったファイルのみを対象にする
          output: trivy-scan-result.txt

      - name: Format Trivy Scan Result
        run: |
          if [ -s trivy-scan-result.txt ]; then
            # ファイルに内容がある場合
            echo -e "## 脆弱性スキャン結果\n<details><summary>詳細</summary>\n\n\`\`\`\n$(cat trivy-scan-result.txt)\n\`\`\`\n</details>" > formatted-trivy-result.md
          else
            # ファイルが空の場合
            echo -e "## 脆弱性スキャン結果\n脆弱性が検知されませんでした。" > formatted-trivy-result.md
          fi

        # PR にコメントを追加
      - name: Comment PR with Trivy scan results
        uses: marocchino/sticky-pull-request-comment@v2
        with:
          recreate: true
          GITHUB_TOKEN: ${{ secrets.github_token }}
          path: formatted-trivy-result.md
```

脆弱性が検知されない場合は、以下のようなコメントがPRに追加される
![alt text](/images/trivy-for-terraform/trivy-result.png)

Misconfiguration Scanning を実現するには、以下を試してみると良さそう

[https://aquasecurity.github.io/trivy/v0.29.2/docs/misconfiguration/scanning/](https://aquasecurity.github.io/trivy/v0.29.2/docs/misconfiguration/scanning/)

Google Cloud , AWS それぞれでの **Misconfiguration Scanning** に設定されているルールは以下から確認できる

- Google Cloud 
  - [https://avd.aquasec.com/misconfig/google/](https://avd.aquasec.com/misconfig/google/)
- AWS
  - [https://avd.aquasec.com/misconfig/aws/](https://avd.aquasec.com/misconfig/aws/)

# 参考

[https://qiita.com/kohei_infra_eng/items/d19e8ec3d82265b6c009](https://qiita.com/kohei_infra_eng/items/d19e8ec3d82265b6c009)

[https://zenn.dev/cloud_ace/articles/494d4c8ae8a0e8#iac-を用いたインフラストラクチャのセキュリティに関する課題](https://zenn.dev/cloud_ace/articles/494d4c8ae8a0e8#iac-%E3%82%92%E7%94%A8%E3%81%84%E3%81%9F%E3%82%A4%E3%83%B3%E3%83%95%E3%83%A9%E3%82%B9%E3%83%88%E3%83%A9%E3%82%AF%E3%83%81%E3%83%A3%E3%81%AE%E3%82%BB%E3%82%AD%E3%83%A5%E3%83%AA%E3%83%86%E3%82%A3%E3%81%AB%E9%96%A2%E3%81%99%E3%82%8B%E8%AA%B2%E9%A1%8C)

[https://kakakakakku.hatenablog.com/entry/2023/08/30/084331](https://kakakakakku.hatenablog.com/entry/2023/08/30/084331)