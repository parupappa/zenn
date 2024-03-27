---
title: "ArgoCD App of Apps と MultiPullSourcesはなにが嬉しいのか？"
emoji: "💬"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

# MultiPullSources

## ツラミ / 良さみ
- ツラミ
外部から取得するHelm Chartsをデプロイする場合にApp of Appsの ${repoURL}/${path} にダウンロードしたHelmをいれておくの変更大変な気がするんだよな
```yaml
 source:
    path: nginx
    repoURL: https://github.com/xxx/argocd-sample-app.git
    targetRevision: main
```

- 良い点
Multiple Sources 使えば Git 管理するの values ファイルだけで良いですね  

https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/#helm-value-files-from-external-git-repository


# App of Apps 
