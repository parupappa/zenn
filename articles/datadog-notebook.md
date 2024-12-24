---
title: "Datadog Notebook の使い方を中の人目線で考えてみた"
emoji: "🗒️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Datadog', 'Notebook']
published: true
publication_name: datadog
---
# はじめに
Datadog で Sales Engineer をしている [parupappa](https://x.com/866mfs) です

この記事は、[Datadog Advent Calendar 2024](https://qiita.com/advent-calendar/2024/datadog)の14日目の記事です。

---

突然ですが、みなさん、**Datadog Notebook** 使っていますか？

中の人間が言うのもなんですが、数ある Deatadog の機能の中では、なかなかニッチな(存在感が薄い)機能なのではないかと思っています🧐

今回は、そんな Datadog Notebook について改めて存在感をアピールしていこうという記事になっております！

# Datadog Notebook
Datadog Notebook について知らない方はまずはこちらをご覧いただければなんとなく掴めると思います。

https://docs.datadoghq.com/notebooks

一言で言うと、**Notion や Google Docs, Confluence などのドキュメントツールに、Datadog 独自のコンポーネントを組み込めるように拡張したもの**です。

Notebook ページの Template Gallery の中にも、How to use Daatadog Notebooks というテンプレートがあるのでそちらも合わせてご覧ください

![alt text](/images/datadog-notebook/notebook-howtouse.png)


主な使い方や Tips については 過去の Datadog Advent Calendar や Meetup で書かれた以下のブログがとても貴重なのでぜひご覧ください！
- [みんな！DatadogのNotebookを活用してくれ！](https://gomiba.co/archives/2021/12/datadog-notebook/)
- [DataDog Notebooks を活用して勉強する](https://qiita.com/raki/items/b86163a70ce5e3f55328)
- [Datadogの活用ノウハウを一挙に公開・それを支える全社管理者の工夫とは](https://techblog.zozo.com/entry/datadog-japan-meetup-2022-summer#4-%E9%9A%9C%E5%AE%B3%E8%AA%BF%E6%9F%BB%E3%82%84%E8%B2%A0%E8%8D%B7%E8%A9%A6%E9%A8%93%E3%81%AA%E3%81%A9%E3%81%A7%E3%83%A1%E3%83%88%E3%83%AA%E3%82%AF%E3%82%B9%E3%82%92%E8%A8%98%E9%8C%B2%E3%81%99%E3%82%8B%E3%81%AE%E3%81%8C%E5%A4%A7%E5%A4%89--Notebook)

なので、ここではよりユースケースに当てはめた形でご紹介しようと思います。

# How to use
## Case1 - Document Library
１つ目の使い方は、**ドキュメントライブラリー**として使用することです。

Datadog Notebook は Dashboard, Monitor, Log など Detadog 内で定義されるリソースに対して、ネイティブで統合されるため、コンポーネントを自由に埋め込むことが可能です<br>（スクリーンショットのような静的画像ではないので、時系列データとして保持したまま参照することが可能）

また、Dashboard のグラフなどは Notebook に対しても Datadog 内での操作感と同様に、ドラッグアンドドロップやコピペ（Cmd + C）で簡単に操作できるので、気持ちの良い UI/UX になっている点も推しポイントです☺️

https://www.datadoghq.com/blog/incident-management-templates-notebooks-list/

### Template Gallery
少し上でも触れましたが、 Datadog では、カスタマイズ可能なすぐに使える Notebook として [Template Gallary](https://docs.datadoghq.com/ja/notebooks/#%E3%83%86%E3%83%B3%E3%83%97%E3%83%AC%E3%83%BC%E3%83%88%E3%82%AE%E3%83%A3%E3%83%A9%E3%83%AA%E3%83%BC) を提供しています。

個人的に特にオススメなのは、以下のテンプレートです
- [Incident Postmortem](https://app.datadoghq.com/notebook/template/7/postmortem-ir-xxxx-outage-name)
- [Incident Report](https://app.datadoghq.com/notebook/template/11/incident-report) 

具体的な内容については後述します。

### Custom Temlates
またそれだけでなく、Custom Templates として、自分たちオリジナルのテンプレートを作成することができます。

なので例えば・・・
- **team ごと（SRE, Platform, Application）での Datadog に関するオンボーディング Notebook**
- **team ごとのコストレポートの雛形**

などなど、特に Notebook では team でのタグやフィルタリングを設定できるので、team ごとでのテンプレートを Custom Templates として用意してもらうと機能とマッチするのではないかと感じています。

![alt text](/images/datadog-notebook/notebook-team1.png)

## Case2 - Incident Management
2つ目の使い方として、広くインシデントマネジメントに使えるという要素があります。

少し古いので UI の違いやアップデートが入っている部分もありますが、以下のブログを見ていただくとイメージがつくと思います。
- 2017年
  - [Data-driven storytelling with Datadog Notebooks](https://www.datadoghq.com/blog/data-driven-notebooks/)
- 2020年
  - [Tell data-driven stories with Collaborative Notebooks](https://www.datadoghq.com/blog/collaborative-notebooks-datadog/)


特に、Template Gallery の Incident Report は、あらかじめ Incidet Report として必要な要素が詰まっており、何がどのくらい起きたら Incident と定義するかの Graph を設定するだけで、定期的に見ることができるダッシュボードのような使い方ができると思います。

![alt text](/images/datadog-notebook/incident-report.png)

また、左上の `Add Template Variables` を押すと、Notebook に対する変数を設定できるので、環境名や、cluster 名などお好きなタグを使用して、変数化したい部分を設定できるので、複数個同じようなテンプレートを用意する必要もないです。

![alt text](/images/datadog-notebook/template-value.png)


また Notebook は Bits AI とも統合されているので、Bits AI が Incident の経緯をまとめてくれる機能(Bits AI Autonomous Investigations)も今年の DASH で発表されています。
具体的な使い方イメージは動画を見ていただけるとイメージがつくかと思います！

https://www.youtube.com/watch?v=ZMNXNH-kJAM&t=5100s

こちらのブログも併せてご覧ください

https://www.datadoghq.com/blog/bits-ai-autonomous-investigations/

※ Bits AI Autonomous Investigations　は現在、Preview での利用となっているため、気になる方は[こちら](https://www.datadoghq.com/product-preview/)から Request をしてみてください！

## Case3 - Postmortem
すでに触れられている、Template Gallery の Incident Postmortem もオススメしたい内容です。

![alt text](/images/datadog-notebook/postmortem.png)

上で説明したような、Bits AI との統合により、Incident Postmortem のテンプレートをベースとして、各インシデントに合わせたユニークなケースを自動生成してくれるので、今後の機能拡充にさらにご期待ください！

こちらのブログで Incident Management から Postmortem までの使い方イメージが解説されています。

https://www.datadoghq.com/blog/incident-postmortem-process-best-practices/


# Others
## Copy formatted Contents
Notebook の共有の方法として、Dashboard と同じような JSON での出力はもちろんのこと `Copy formatted Contents` という出力形式もあります。

これは、Notebook の内容を HTML としてコピーした状態になるので、HTML　フォーマットに対応しているエディタ(Google Docs, MS Word)とかに貼り付けるとそのまま、

同じ見た目で PDF も Markdown も出力できるようになっています（ちなみに、Markdown はテキストだけではなく、base64エンコードされたグラフの画像もついてきます. Notion だとエクスポートすると、テキストと画像ファイルが別々にでてくるので煩わしさがあった😢）

なので、特に Datadog 上のデータを出力して自社のドキュメントライブラリーに統合させたいような場合には、このように様々な形式にエクスポートもできるので、 Tempolary 的な使い方として一次情報を残しておくという使い方も出来ると思います。

![alt text](/images/datadog-notebook/copy-formatted-contents.png)

## Terraform 対応
Datadog の terraform provier に2019年から [Support for Notebook resources #261](https://github.com/DataDog/terraform-provider-datadog/issues/261)として issue が上がっていますが、今だ採用にはいたっていません...

https://github.com/DataDog/terraform-provider-datadog/issues/261

issue の中で、機能実装を望む声は多く上がっていそうですが、使い方として難しいことが理由ではないかと推察しています。
[datadog_dashboard](https://registry.terraform.io/providers/DataDog/datadog/latest/docs/resources/dashboard) と同じく、インタラクティブに操作することがメインとなる使い方なので、仮に実装されたとしても、resource の管理面や運用の仕方に工夫が必要そうですね

# さいごに
Datadog Notebook のユースケースについて紹介する Advent Calendar をお送りしました。

他にもこんな使い方をしている！というのがあれば、ぜひぜひコメントください📝

コストはかからないので、ぜひ今後は Datadog Notebook を活用してみてください☺️
