---
title: "OpenTelemetry - Community Demo を Datadog で試す"
emoji: "👓"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['Datadog', 'OpenTelemetry', 'observability', 'Kubernetes']
published: true
---

# はじめに

OpenTelemetry には [Community Demo](https://opentelemetry.io/ja/docs/demo/) が用意されている。demo では[Kubernetes](https://opentelemetry.io/docs/demo/kubernetes-deployment/)と [Docker](https://opentelemetry.io/docs/demo/docker-deployment/)でのお試し方法が記載されている。

今回は、minikube を使用し、Kubernetes 上で試したいと思ったのだが、
demoでは OpenTelemetry の Exporter のバックエンドとして、Jaeger, Prometheus(Grafana) が使用されているが、その他のバックエンドは自分で構築してね〜となっている

https://opentelemetry.io/docs/demo/kubernetes-deployment/#bring-your-own-backend

なので、これを Datadog に送信する方法を試してみる。


# TL;DR
observabilityバックエンドを Datadog にする方法をいろいろ調べていく中で、Datadog 公式から以下のような情報が出ていたので結論としてはこれを試すことになった😮‍💨
（ここだけ見ればいいかもしれない）

https://docs.datadoghq.com/ja/opentelemetry/guide/otel_demo_to_datadog/?tab=kubernetes


# お試し

minikube を起動するのだが、デモの動作環境メモリが
**「6 GB of free RAM for the application」** なので、minikube 起動時にメモリ容量を 6GB 以上に指定

```bash
$ minikube start --cpus='4' --memory='8G'
```

Datadog の API Key と SiteParameter を k8s Secret に保存

※ DD_SITE_PARAMETERの確認: https://docs.datadoghq.com/ja/getting_started/site/#datadog-%E3%82%B5%E3%82%A4%E3%83%88%E3%81%AB%E3%82%A2%E3%82%AF%E3%82%BB%E3%82%B9%E3%81%99%E3%82%8B
```bash
$ kubectl create secret generic dd-secrets --from-literal="DD_SITE_PARAMETER=<Your API Site>" --from-literal="DD_API_KEY=<Your API Key>"
```


```bash
$ helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
```

demoの Helm Chart は[こちら](https://github.com/open-telemetry/opentelemetry-helm-charts/blob/main/charts/opentelemetry-demo/values.yaml#L674)にあるが、この設定では Datadog の設定が含まれていないので、values ファイルを追加して、DataDog の設定を追加する

```bash
$ helm install my-otel-demo open-telemetry/opentelemetry-demo --values my-values-file.yaml

NAME: my-otel-demo
LAST DEPLOYED: Wed Aug 21 15:22:19 2024
NAMESPACE: otel-demo
STATUS: deployed
REVISION: 1
NOTES:
=======================================================================================


 ██████╗ ████████╗███████╗██╗         ██████╗ ███████╗███╗   ███╗ ██████╗
██╔═══██╗╚══██╔══╝██╔════╝██║         ██╔══██╗██╔════╝████╗ ████║██╔═══██╗
██║   ██║   ██║   █████╗  ██║         ██║  ██║█████╗  ██╔████╔██║██║   ██║
██║   ██║   ██║   ██╔══╝  ██║         ██║  ██║██╔══╝  ██║╚██╔╝██║██║   ██║
╚██████╔╝   ██║   ███████╗███████╗    ██████╔╝███████╗██║ ╚═╝ ██║╚██████╔╝
 ╚═════╝    ╚═╝   ╚══════╝╚══════╝    ╚═════╝ ╚══════╝╚═╝     ╚═╝ ╚═════╝


- All services are available via the Frontend proxy: http://localhost:8080
  by running these commands:
     kubectl --namespace otel-demo port-forward svc/my-otel-demo-frontendproxy 8080:8080

  The following services are available at these paths once the proxy is exposed:
  Webstore             http://localhost:8080/
  Grafana              http://localhost:8080/grafana/
  Load Generator UI    http://localhost:8080/loadgen/
  Jaeger UI            http://localhost:8080/jaeger/ui/
```


```yaml
opentelemetry-collector:
  extraEnvsFrom:
    - secretRef:
        name: dd-secrets
  config:
    exporters:
      datadog:
        traces:
          span_name_as_resource_name: true
          trace_buffer: 500
        hostname: "otelcol-helm"
        api:
          site: ${DD_SITE_PARAMETER}
          key: ${DD_API_KEY}

    processors:
      resource:
        attributes:
          - key: deployment.environment
            value: "otel"
            action: upsert

    connectors:
      datadog/connector:
        traces:
          span_name_as_resource_name: true

    service:
      pipelines:
        traces:
          processors: [resource, batch]
          exporters: [otlp, debug, spanmetrics, datadog, datadog/connector]
        metrics:
          receivers: [httpcheck/frontendproxy, otlp, redis, spanmetrics, datadog/connector]
          processors: [resource, batch]
          exporters: [otlphttp/prometheus, debug, datadog]
        logs:
          processors: [resource, batch]
          exporters: [opensearch, debug, datadog]
```


# Datadog Exporter
Datadog には OpenTelemetry の Exporter(Datadog Exporter) が用意されている。
Datadog Agent を使用した収集のパターンもあるが、今回は、以下のように Datadog Exporter を使用する

![alt text](/images/otel-datadog-demo/datadog-exporter.png)

https://docs.datadoghq.com/ja/opentelemetry/


https://docs.datadoghq.com/ja/opentelemetry/ingestion_sampling_with_opentelemetry/


Datadog Exporter に関する詳しい説明は以下のあたりに載っている

https://www.datadoghq.com/ja/blog/ingest-opentelemetry-traces-metrics-with-datadog-exporter/

EXporter に関する config のサンプルはこちら

https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/exporter/datadogexporter/examples/collector.yaml


# 確認

OpenTelemetry Host Metrics Dashboard を Datadog に追加することで確認できた

![alt text](/images/otel-datadog-demo/datadog-image-0.png)

また、Datadog 上で各 micro service に関するトレースが表示されていることも確認できた

![alt text](/images/otel-datadog-demo/datadog-image-1.png)

# おわりに
Datadog には OpenTelemetry の Exporter が用意されているので、テレメトリーデータの送信を Datadog で実現するのはメッチャカンタン🚀

# 参考
- [OpenTelemetry の Community Demo のトレースとメトリクスを Datadog に送信してみた](https://qiita.com/stanabe/items/0940f0268b4a0afe4381)
