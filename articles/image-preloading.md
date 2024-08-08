---
title: "Image Preloading による GKE の起動時間の短縮"
emoji: "🕐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ['GKE', 'GoogleCloud', 'Container' ]
published: false
---


```bash
$ export PROJECT_ID="$( gcloud config get-value project )"
Your active configuration is: [test-yokoo]
```

```
$ gcloud storage buckets create "gs://gke-disk-image-logs-${PROJECT_ID}" --enable-autoclass \
--location "asia-northeast1" --uniform-bucket-level-access --public-access-prevention
Creating gs://gke-disk-image-logs-test-yokoo/...
```


```bash
$ git clone https://github.com/GoogleCloudPlatform/ai-on-gke.git
Cloning into 'ai-on-gke'...
remote: Enumerating objects: 17929, done.
remote: Counting objects: 100% (2519/2519), done.
remote: Compressing objects: 100% (700/700), done.
remote: Total 17929 (delta 2132), reused 1823 (delta 1818), pack-reused 15410
Receiving objects: 100% (17929/17929), 278.37 MiB | 10.68 MiB/s, done.
Resolving deltas: 100% (9422/9422), done.
```


```bash
$ cd ai-on-gke/tools/gke-disk-image-builder/
```




```bash
$ export IMAGE_NAME="baseimage-$( date +"%Y%m%d" )"
go run ./cli --project-name "${PROJECT_ID}" --zone "asia-northeast1-a" \
--gcs-path "gs://gke-disk-image-logs-${PROJECT_ID}" \
--image-name "${IMAGE_NAME}" --disk-size-gb "100" \
--container-image "nvcr.io/nvidia/pytorch:24.06-py3" \
--container-image "nvcr.io/nvidia/tensorflow:24.06-tf2-py3"
go: downloading github.com/GoogleCloudPlatform/compute-daisy v0.0.0-20230929171844-6a3c47dc7a4f
go: downloading google.golang.org/api v0.148.0
go: downloading cloud.google.com/go/storage v1.31.0
go: downloading cloud.google.com/go/logging v1.8.1
go: downloading cloud.google.com/go v0.110.8
go: downloading golang.org/x/oauth2 v0.13.0
go: downloading google.golang.org/grpc v1.58.3
go: downloading google.golang.org/genproto v0.0.0-20231002182017-d307bd883b97
go: downloading google.golang.org/genproto/googleapis/api v0.0.0-20231002182017-d307bd883b97
go: downloading cloud.google.com/go/iam v1.1.2
go: downloading github.com/google/uuid v1.3.1
go: downloading cloud.google.com/go/longrunning v0.5.1
go: downloading google.golang.org/genproto/googleapis/rpc v0.0.0-20231012201019-e917dd12ba7a
go: downloading golang.org/x/sync v0.4.0
go: downloading golang.org/x/xerrors v0.0.0-20220907171357-04be3eba64a2
go: downloading golang.org/x/net v0.17.0
go: downloading github.com/googleapis/enterprise-certificate-proxy v0.3.1
go: downloading golang.org/x/crypto v0.14.0
[secondary-disk-image]: 2024-08-08T18:12:04+09:00 Validating workflow
[secondary-disk-image]: 2024-08-08T18:12:04+09:00 Validating step "create-disk"
[secondary-disk-image]: 2024-08-08T18:12:04+09:00 Validating step "create-instance"
[secondary-disk-image]: 2024-08-08T18:12:06+09:00 Validating step "wait-on-image-creation"
[secondary-disk-image]: 2024-08-08T18:12:06+09:00 Validating step "detach-disk"
[secondary-disk-image]: 2024-08-08T18:12:06+09:00 Validating step "create-image"
[secondary-disk-image]: 2024-08-08T18:12:06+09:00 Validation Complete
[secondary-disk-image]: 2024-08-08T18:12:06+09:00 Workflow Project: test-yokoo
[secondary-disk-image]: 2024-08-08T18:12:06+09:00 Workflow Zone: asia-northeast1-a
[secondary-disk-image]: 2024-08-08T18:12:06+09:00 Workflow GCSPath: gs://gke-disk-image-logs-test-yokoo
[secondary-disk-image]: 2024-08-08T18:12:06+09:00 Daisy scratch path: https://console.cloud.google.com/storage/browser/gke-disk-image-logs-test-yokoo/daisy-secondary-disk-image-20240808-09:11:57-056rx
[secondary-disk-image]: 2024-08-08T18:12:06+09:00 Uploading sources
[secondary-disk-image]: 2024-08-08T18:12:07+09:00 Running workflow
[secondary-disk-image]: 2024-08-08T18:12:07+09:00 Running step "create-disk" (CreateDisks)
[secondary-disk-image.create-disk]: 2024-08-08T18:12:07+09:00 CreateDisks: Creating disk "secondary-disk-image-disk".
[secondary-disk-image]: 2024-08-08T18:12:08+09:00 Step "create-disk" (CreateDisks) successfully finished.
[secondary-disk-image]: 2024-08-08T18:12:08+09:00 Running step "create-instance" (CreateInstances)
[secondary-disk-image.create-instance]: 2024-08-08T18:12:08+09:00 CreateInstances: Creating instance "secondary-disk-image-instance".
[secondary-disk-image]: 2024-08-08T18:12:28+09:00 Step "create-instance" (CreateInstances) successfully finished.
[secondary-disk-image.create-instance]: 2024-08-08T18:12:28+09:00 CreateInstances: Streaming instance "secondary-disk-image-instance" serial port 1 output to https://storage.cloud.google.com/gke-disk-image-logs-test-yokoo/daisy-secondary-disk-image-20240808-09:11:57-056rx/logs/secondary-disk-image-instance-serial-port1.log
[secondary-disk-image]: 2024-08-08T18:12:28+09:00 Running step "wait-on-image-creation" (WaitForInstancesSignal)
[secondary-disk-image.wait-on-image-creation]: 2024-08-08T18:12:28+09:00 WaitForInstancesSignal: Instance "secondary-disk-image-instance": watching serial port 1, SuccessMatch: "Unpacking is completed", FailureMatch: ["startup-script-url exit status 1" "Failed to pull and unpack the image"] (this is not an error).
[secondary-disk-image.wait-on-image-creation]: 2024-08-08T18:24:29+09:00 WaitForInstancesSignal: Instance "secondary-disk-image-instance": SuccessMatch found "Unpacking is completed."
[secondary-disk-image]: 2024-08-08T18:24:29+09:00 Step "wait-on-image-creation" (WaitForInstancesSignal) successfully finished.
[secondary-disk-image]: 2024-08-08T18:24:29+09:00 Running step "detach-disk" (DetachDisks)
[secondary-disk-image.detach-disk]: 2024-08-08T18:24:29+09:00 DetachDisks: Detaching disk "secondary-disk-image-disk" from instance "secondary-disk-image-instance".
[secondary-disk-image]: 2024-08-08T18:24:33+09:00 Step "detach-disk" (DetachDisks) successfully finished.
[secondary-disk-image]: 2024-08-08T18:24:33+09:00 Running step "create-image" (CreateImages)
[secondary-disk-image.create-image]: 2024-08-08T18:24:33+09:00 CreateImages: Creating image "baseimage-20240808".
[secondary-disk-image]: 2024-08-08T18:28:14+09:00 Step "create-image" (CreateImages) successfully finished.
[secondary-disk-image]: 2024-08-08T18:28:14+09:00 Workflow "secondary-disk-image" cleaning up (this may take up to 2 minutes).
[secondary-disk-image]: 2024-08-08T18:29:03+09:00 Workflow "secondary-disk-image" finished cleanup.
Image has successfully been created at: projects/test-yokoo/global/images/baseimage-20240808
```